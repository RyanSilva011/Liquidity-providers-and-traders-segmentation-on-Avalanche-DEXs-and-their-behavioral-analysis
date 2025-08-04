# Avalanche DEX Ecosystem Analysis: User Segmentation, Retention & Growth (Traders & Liquidity Providers)
[Full Dashboard](https://joshuatochinwachi.github.io/Liquidity-providers-and-traders-segmentation-on-Avalanche-DEXs-and-their-behavioral-analysis/) | [Data story telling on 𝕏](https://x.com/defi__josh/status/1927992581989781746)

This dashboard and entire analysis was built and developed using [Flipside Crypto](https://flipsidecrypto.xyz) by me. Unfortunately, Flipside’s querying and dashboarding tool has gone dark because of their collaboration with [Snowflake](https://www.snowflake.com/en/).

Flipside also used Snowflake’s SQL dialect as their SQL flavor when they were active, so the SQL code you’ll see in this project—for both querying and dashboarding—is very similar to Snowflake’s syntax.

Using [Python scripts](https://github.com/joshuatochinwachi/Flipside_dashboard_porter), I scraped my Flipside dashboard and visualizations from the Flipside website.


<img width="1824" height="754" alt="image" src="https://github.com/user-attachments/assets/d8ddf922-229b-4979-aa04-05a5380f2924" />


## Queries/Metrics used

### 1. Avalanche DEX User Segmentation and Retention Analysis by Trading Volume Tiers

```
/*
Title: Avalanche DEX User Segmentation and Retention Analysis by Trading Volume Tiers

Description: Comprehensive analysis of DEX user behavior, segmented by wallet tiers,
including metrics for user retention, churn, activity, and volume across different trader categories

Aim: To provide comprehensive insights into user trading behavior on Avalanche DEXes through wallet segmentation and behavioral analysis, enabling data-driven protocol optimization and user engagement strategies.

-----Key Metrics Coverage
User Base: Active users, new users, churned users
Activity: Total swaps, trading volume (USD)
Performance: Average metrics per user
Retention: Cohort retention, reactivation rates

-----Analysis Boundaries
Exclusive focus on swap transactions
Wallet-based user identification
USD value derived from transaction timestamps
Monthly aggregation level


This analysis serves as a foundation for protocol development, marketing strategies, and user experience enhancement, following Flipside's IDG framework for comprehensive growth analytics.
*/

WITH dex_users AS (
  SELECT 
    origin_from_address as trader,
    DATE_TRUNC('month', block_timestamp) as swap_month,
    COUNT(DISTINCT tx_hash) as swap_count,
    SUM(COALESCE(amount_in_usd, amount_out_usd)) as total_volume_usd
  FROM avalanche.defi.ez_dex_swaps
  WHERE block_timestamp >= '2024-05-01'
    AND block_timestamp < '2025-05-01'
  GROUP BY 1,2
  HAVING swap_count >= 1
),

trader_volume AS (
  SELECT 
    trader,
    SUM(total_volume_usd) as total_volume_usd
  FROM dex_users
  GROUP BY 1
),

wallet_tiers AS (
  SELECT 
    d.trader,
    d.swap_month,
    d.total_volume_usd as monthly_volume,
    CASE 
      WHEN PERCENT_RANK() OVER (PARTITION BY d.swap_month ORDER BY d.total_volume_usd DESC) <= 0.01 THEN '🐳 Whale'
      WHEN PERCENT_RANK() OVER (PARTITION BY d.swap_month ORDER BY d.total_volume_usd DESC) <= 0.05 THEN '🦈 Shark'
      WHEN PERCENT_RANK() OVER (PARTITION BY d.swap_month ORDER BY d.total_volume_usd DESC) <= 0.10 THEN '🐬 Dolphin'
      WHEN PERCENT_RANK() OVER (PARTITION BY d.swap_month ORDER BY d.total_volume_usd DESC) <= 0.20 THEN '🐟 Fish'
      WHEN PERCENT_RANK() OVER (PARTITION BY d.swap_month ORDER BY d.total_volume_usd DESC) <= 0.40 THEN '🐙 Octopus'
      WHEN PERCENT_RANK() OVER (PARTITION BY d.swap_month ORDER BY d.total_volume_usd DESC) <= 0.60 THEN '🦀 Crab'
      ELSE '🦐 Shrimp'
    END as wallet_tier
  FROM dex_users d
),

monthly_activity AS (
  SELECT 
    w.wallet_tier,
    d.swap_month,
    COUNT(DISTINCT d.trader) as active_users,
    SUM(d.swap_count) as total_swaps,
    SUM(COALESCE(d.total_volume_usd, 0)) as monthly_volume_usd
  FROM dex_users d
  JOIN wallet_tiers w ON d.trader = w.trader
  GROUP BY 1,2
),

cohort_tracking AS (
    SELECT 
        wallet_tier,
        swap_month as cohort_month,
        active_users as original_cohort_size,
        LEAD(active_users) OVER (PARTITION BY wallet_tier ORDER BY swap_month) as next_month_active
    FROM monthly_activity
),

retention AS (
  -- First month: All users are retained (100% retention rate)
  -- First month: Churn rate is 0% (inverse of retention)
  -- First month: Activation rate is 0% (no previous month to compare)
  -- First two months: Reactivation rate is 0% (needs 3 months of history)
  SELECT 
    ma.wallet_tier,
    ma.swap_month,
    ma.active_users,
    ma.total_swaps,
    ma.monthly_volume_usd,
    LAG(ma.active_users) OVER (PARTITION BY ma.wallet_tier ORDER BY ma.swap_month) as previous_month_users,
    COUNT(DISTINCT CASE 
      WHEN previous_month.trader IS NOT NULL THEN current_month.trader
      WHEN ma.swap_month = (SELECT MIN(swap_month) FROM monthly_activity) THEN current_month.trader -- First month condition
    END) as retained_users,
    COUNT(DISTINCT CASE 
      WHEN previous_inactive.trader IS NOT NULL 
      AND two_months_ago.trader IS NULL 
      THEN current_month.trader
    END) as reactivated_users,
    ct.original_cohort_size
  FROM monthly_activity ma
  LEFT JOIN dex_users current_month
    ON current_month.swap_month = ma.swap_month
  LEFT JOIN dex_users previous_month  
    ON previous_month.trader = current_month.trader
    AND previous_month.swap_month = DATEADD('month', -1, current_month.swap_month)  
  LEFT JOIN dex_users previous_inactive
    ON previous_inactive.trader = current_month.trader
    AND previous_inactive.swap_month < current_month.swap_month
  LEFT JOIN dex_users two_months_ago
    ON two_months_ago.trader = current_month.trader
    AND two_months_ago.swap_month = DATEADD('month', -1, current_month.swap_month)
  JOIN wallet_tiers w 
    ON current_month.trader = w.trader 
    AND w.wallet_tier = ma.wallet_tier
  LEFT JOIN cohort_tracking ct
    ON ct.wallet_tier = ma.wallet_tier
    AND ct.cohort_month = ma.swap_month
  GROUP BY 1,2,3,4,5,9
)

SELECT 
  wallet_tier as "Wallet Tier",
  swap_month as "Month",
  active_users as "Active Users",
  total_swaps as "Total Swaps",
  ROUND(monthly_volume_usd, 2) as "Volume (USD)",
  retained_users as "Retained Users",
  ROUND(100.0 * retained_users / NULLIF(active_users, 0), 2) as "Retention Rate %",
  (active_users - retained_users) as "Churned Users",
  (100 - ROUND(100.0 * retained_users / NULLIF(active_users, 0), 2)) as "Churn Rate %",
  (active_users - retained_users) as "New Users",
  (active_users - previous_month_users) as "Net User Growth",
  reactivated_users as "Reactivated Users",
  LEAST(100, ROUND(100.0 * (active_users - retained_users) / NULLIF(previous_month_users, 0), 2)) as "Activation Rate %",
  LEAST(100, ROUND(100.0 * reactivated_users / NULLIF((active_users - (active_users - retained_users)), 0), 2)) as "Reactivation Rate %",
  ROUND(total_swaps::float / NULLIF(active_users, 0), 2) as "Avg Swaps per User",
  ROUND(monthly_volume_usd / NULLIF(active_users, 0), 2) as "Avg Volume per User (USD)",
  ROUND(100.0 * retained_users / NULLIF(original_cohort_size, 0), 2) as "Cohort Retention %"
FROM retention
WHERE swap_month >= '2024-05-01'
  AND swap_month < '2025-05-01'
ORDER BY 
  CASE wallet_tier
    WHEN '🐳 Whale' THEN 1
    WHEN '🦈 Shark' THEN 2
    WHEN '🐬 Dolphin' THEN 3
    WHEN '🐟 Fish' THEN 4
    WHEN '🐙 Octopus' THEN 5
    WHEN '🦀 Crab' THEN 6
    WHEN '🦐 Shrimp' THEN 7
  END,
  swap_month DESC;
```

###### Query result:
<img width="1918" height="790" alt="image" src="https://github.com/user-attachments/assets/9b2d43bf-7c90-4e11-8973-f99009882c69" />

### 2. Avalanche DEX Liquidity Provider Analytics: User Segmentation & Behavioral Analysis

```
/*
TITLE: Avalanche DEX Liquidity Provider Analytics: User Segmentation & Behavioral Analysis

DESCRIPTION: Comprehensive analysis of LP user behavior, segmented by wallet tiers,
including metrics for user retention, churn, activity, volume, and platform-specific metrics

AIM: To provide a comprehensive understanding of liquidity provider behavior across Avalanche's DEX ecosystem through the lens of user acquisition, engagement, retention, and monetization. This analysis segments users into distinct tiers based on their liquidity provision volume, tracking their journey from entry to sustained participation while measuring cross-platform engagement patterns and value creation metrics.

DATA COVERAGE:
Time Frame: Rolling 12-Month Window
Blockchain: Avalanche
Protocol Type: Decentralized Exchanges
Data Sources: ez_token_transfers, dim_dex_liquidity_pools, ez_dex_swaps

ANALYTICAL BOUNDARIES:
- Focus on liquidity addition events
- Exclusion of withdrawal metrics
- Platform coverage: TraderJoe, Pangolin, Pharaoh, and other major DEXs
- Limited to direct DEX interactions


This analysis serves as a strategic tool for understanding LP behavior patterns, market dynamics, and platform performance within the Avalanche DEX ecosystem, providing actionable insights through the IDG framework.

*/

WITH lp_deposit_events AS (
  SELECT 
    origin_from_address as provider,
    block_timestamp,
    tx_hash,
    contract_address
  FROM avalanche.core.fact_event_logs fel
  WHERE topics[0] IN (
    '0x4c209b5fc8ad50758f13e2e1088ba56a560dff690a1c6fef26394f4c03821c4f', -- Mint
    '0x26f55a85081d24974e85c6c00045d0f0453991e95873f52bff0d21af4079a768', -- addLiquidity
    '0x423f6495a08fc652425cf4ed0d1f9e37e571d9b9529b1c1c23cce780b2e7df0d'  -- addLiquidityETH
  )
  AND block_timestamp >= '2024-05-01'
  AND block_timestamp < '2025-05-01'
),

lp_users AS (
  SELECT 
    provider,
    DATE_TRUNC('month', block_timestamp) as lp_month,
    COUNT(DISTINCT tx_hash) as lp_count,
    SUM(COALESCE(amount_usd, 0)) as total_volume_usd
  FROM (
    -- LP Deposits from transfers
    SELECT 
      from_address as provider,
      t.block_timestamp,
      t.tx_hash,
      t.amount_usd
    FROM avalanche.core.ez_token_transfers t
    JOIN avalanche.defi.dim_dex_liquidity_pools p
      ON t.to_address = p.pool_address
    WHERE t.block_timestamp >= '2024-05-01'
    AND t.block_timestamp < '2025-05-01'
    AND NOT EXISTS (
      SELECT 1 
      FROM avalanche.defi.ez_dex_swaps s
      WHERE s.tx_hash = t.tx_hash
    )
    
    UNION ALL
    
    SELECT 
      e.provider,
      e.block_timestamp,
      e.tx_hash,
      t.amount_usd
    FROM lp_deposit_events e
    LEFT JOIN avalanche.core.ez_token_transfers t
      ON e.tx_hash = t.tx_hash
  )
  GROUP BY 1,2
  HAVING lp_count >= 1
),

pool_metrics AS (
  SELECT 
    from_address as provider,
    DATE_TRUNC('month', t.block_timestamp) as lp_month,
    COUNT(DISTINCT p.pool_address) as unique_pools,
    COUNT(DISTINCT p.platform) as unique_platforms,
    COUNT(DISTINCT tokens:token0) as unique_tokens_provided,
    COUNT(DISTINCT CASE WHEN platform ilike 'trader%joe%' THEN p.pool_address END) as trader_joe_pools,
    COUNT(DISTINCT CASE WHEN platform = 'pangolin' THEN p.pool_address END) as pangolin_pools,
    COUNT(DISTINCT CASE WHEN platform ilike 'pharaoh%' THEN p.pool_address END) as pharaoh_pools,
    COUNT(DISTINCT CASE WHEN platform ilike '%curve%' THEN p.pool_address END) as curve_pools,
    COUNT(DISTINCT CASE WHEN platform ilike '%kyber&swap%' THEN p.pool_address END) as kyber_pools,
    COUNT(DISTINCT CASE WHEN platform ilike '%uniswap%' THEN p.pool_address END) as uniswap_pools
  FROM avalanche.core.ez_token_transfers t
  JOIN avalanche.defi.dim_dex_liquidity_pools p
    ON t.to_address = p.pool_address
  WHERE t.block_timestamp >= '2024-05-01'
  AND t.block_timestamp < '2025-05-01'
  GROUP BY 1, 2
),

swap_metrics AS (
  SELECT 
    l.provider,
    DATE_TRUNC('month', s.block_timestamp) as lp_month,
    COUNT(DISTINCT s.tx_hash) as swaps_in_pools,
    SUM(s.amount_in_usd) as total_swap_volume_usd
  FROM lp_users l
  JOIN avalanche.defi.ez_dex_swaps s
    ON l.provider = s.origin_from_address
  WHERE s.block_timestamp >= '2024-05-01'
  AND s.block_timestamp < '2025-05-01'
  GROUP BY 1, 2
),

provider_volume AS (
  SELECT 
    provider,
    SUM(total_volume_usd) as total_volume_usd
  FROM lp_users
  GROUP BY 1
),

wallet_tiers AS (
  SELECT 
    provider,
    lp_month,
    total_volume_usd,
    CASE 
      WHEN PERCENT_RANK() OVER (PARTITION BY lp_month ORDER BY total_volume_usd DESC) <= 0.01 THEN '🐳 Whale'
      WHEN PERCENT_RANK() OVER (PARTITION BY lp_month ORDER BY total_volume_usd DESC) <= 0.05 THEN '🦈 Shark'
      WHEN PERCENT_RANK() OVER (PARTITION BY lp_month ORDER BY total_volume_usd DESC) <= 0.10 THEN '🐬 Dolphin'
      WHEN PERCENT_RANK() OVER (PARTITION BY lp_month ORDER BY total_volume_usd DESC) <= 0.20 THEN '🐟 Fish'
      WHEN PERCENT_RANK() OVER (PARTITION BY lp_month ORDER BY total_volume_usd DESC) <= 0.40 THEN '🐙 Octopus'
      WHEN PERCENT_RANK() OVER (PARTITION BY lp_month ORDER BY total_volume_usd DESC) <= 0.60 THEN '🦀 Crab'
      ELSE '🦐 Shrimp'
    END as wallet_tier
  FROM lp_users
),

monthly_activity AS (
  SELECT 
    w.wallet_tier,
    d.lp_month,
    COUNT(DISTINCT d.provider) as active_users,
    SUM(d.lp_count) as total_lp_actions,
    SUM(COALESCE(d.total_volume_usd, 0)) as monthly_volume_usd
  FROM lp_users d
  JOIN wallet_tiers w ON d.provider = w.provider
  GROUP BY 1,2
),

cohort_tracking AS (
    SELECT 
        wallet_tier,
        lp_month as cohort_month,
        active_users as original_cohort_size,
        LEAD(active_users) OVER (PARTITION BY wallet_tier ORDER BY lp_month) as next_month_active
    FROM monthly_activity
),

inactive_pool AS (
  SELECT 
    w.wallet_tier,
    l1.lp_month,
    COUNT(DISTINCT l1.provider) as potential_reactivations
  FROM lp_users l1
  JOIN wallet_tiers w ON l1.provider = w.provider
  WHERE l1.lp_month >= DATEADD('month', 2, (SELECT MIN(lp_month) FROM lp_users))
  AND EXISTS (
    SELECT 1 
    FROM lp_users l2
    WHERE l2.provider = l1.provider
    AND l2.lp_month < l1.lp_month
    AND l2.lp_month >= DATEADD('month', -3, l1.lp_month)
  )
  AND NOT EXISTS (
    SELECT 1 
    FROM lp_users l3
    WHERE l3.provider = l1.provider
    AND l3.lp_month = DATEADD('month', -1, l1.lp_month)
  )
  GROUP BY 1, 2
),

retention AS (
  SELECT 
    ma.wallet_tier,
    ma.lp_month,
    ma.active_users,
    ma.total_lp_actions,
    ma.monthly_volume_usd,
    LAG(ma.active_users) OVER (PARTITION BY ma.wallet_tier ORDER BY ma.lp_month) as previous_month_users,
    CASE 
      WHEN ma.lp_month = (SELECT MIN(lp_month) FROM monthly_activity)
      THEN ma.active_users
      ELSE COUNT(DISTINCT CASE 
        WHEN previous_month.provider IS NOT NULL THEN current_month.provider 
      END)
    END as retained_users,
    COUNT(DISTINCT CASE 
  WHEN current_month.provider IS NOT NULL  -- Active now (Time 3)
  AND previous_month.provider IS NULL      -- Inactive last month (Time 2)
  AND EXISTS (                             -- Was active before (Time 1)
    SELECT 1 
    FROM lp_users earlier
    WHERE earlier.provider = current_month.provider
    AND earlier.lp_month < DATEADD('month', -1, current_month.lp_month)
  )
  THEN current_month.provider
END) as reactivated_users,
    ct.original_cohort_size,
    
    AVG(pm.unique_pools) as avg_pools,
    AVG(pm.unique_platforms) as avg_platforms,
    AVG(pm.unique_tokens_provided) as avg_tokens,
    AVG(pm.trader_joe_pools) as avg_tj_pools,
    AVG(pm.pangolin_pools) as avg_pang_pools,
    AVG(pm.pharaoh_pools) as avg_pharaoh_pools,
    AVG(pm.curve_pools) as avg_curve_pools,
    AVG(pm.kyber_pools) as avg_kyber_pools,
    AVG(pm.uniswap_pools) as avg_uniswap_pools,
    AVG(sm.swaps_in_pools) as avg_swaps,
    AVG(sm.total_swap_volume_usd) as avg_swap_volume
  FROM monthly_activity ma
  LEFT JOIN lp_users current_month
    ON current_month.lp_month = ma.lp_month
  
  LEFT JOIN lp_users previous_month
    ON previous_month.provider = current_month.provider
    AND previous_month.lp_month = DATEADD('month', -1, current_month.lp_month)
  LEFT JOIN lp_users previous_inactive
    ON previous_inactive.provider = current_month.provider
    AND previous_inactive.lp_month < current_month.lp_month
  LEFT JOIN lp_users two_months_ago
    ON two_months_ago.provider = current_month.provider
    AND two_months_ago.lp_month = DATEADD('month', -2, current_month.lp_month)
  JOIN wallet_tiers w 
    ON current_month.provider = w.provider 
    AND w.wallet_tier = ma.wallet_tier
  LEFT JOIN cohort_tracking ct
    ON ct.wallet_tier = ma.wallet_tier
    AND ct.cohort_month = ma.lp_month
  LEFT JOIN pool_metrics pm
    ON current_month.provider = pm.provider
    AND current_month.lp_month = pm.lp_month
  LEFT JOIN swap_metrics sm
    ON current_month.provider = sm.provider
    AND current_month.lp_month = sm.lp_month
  GROUP BY 1,2,3,4,5,9
)

SELECT 
  r.wallet_tier as "Liquidity Providers' Tier",
  r.lp_month as "Month",
  active_users as "Active LPs",
  total_lp_actions as "Total LP Actions",
  ROUND(monthly_volume_usd, 2) as "Volume (USD)",
  retained_users as "Retained LPs",
  ROUND(100.0 * retained_users / NULLIF(active_users, 0), 2) as "Retention Rate %",
  (active_users - retained_users) as "Churned LPs",
  (100 - ROUND(100.0 * retained_users / NULLIF(active_users, 0), 2)) as "Churn Rate %",
  (active_users - retained_users) as "New LPs",
  (active_users - previous_month_users) as "Net LP Growth",
  reactivated_users as "Reactivated LPs",
  LEAST(100, ROUND(100.0 * (active_users - retained_users) / NULLIF(previous_month_users, 0), 2)) as "Activation Rate %",
 -- LEAST(100, ROUND(100.0 * reactivated_users / NULLIF((active_users - (active_users - retained_users)), 0), 2)) as "Reactivation Rate %",

--ROUND(100.0 * reactivated_users / NULLIF(active_users, 0), 2) as "Reactivation Rate %" , 
--ROUND(100.0 * reactivated_users / NULLIF(active_users - retained_users, 0), 2) as "Reactivation Rate %",
--ROUND(100.0 * reactivated_users / NULLIF(ip.potential_reactivations, 0), 2) as "Reactivation Rate %",
CASE 
  WHEN r.lp_month <= DATEADD('month', 1, (SELECT MIN(lp_month) FROM lp_users)) THEN 0
  ELSE LEAST(100, ROUND(100.0 * reactivated_users / NULLIF(previous_month_users, 0), 2))
END as "Reactivation Rate %",

ROUND(total_lp_actions::float / NULLIF(active_users, 0), 2) as "Avg Actions per LP",
  ROUND(monthly_volume_usd / NULLIF(active_users, 0), 2) as "Avg Volume per LP (USD)",
  ROUND(100.0 * retained_users / NULLIF(original_cohort_size, 0), 2) as "Cohort Retention %",
 
  ROUND(avg_pools, 2) as "Avg Pools per LP",
  ROUND(avg_platforms, 2) as "Avg Platforms per LP",
  ROUND(avg_tokens, 2) as "Avg Unique Tokens per LP",
  ROUND(avg_tj_pools, 2) as "Avg TraderJoe Pools",
  ROUND(avg_pang_pools, 2) as "Avg Pangolin Pools",
  ROUND(avg_pharaoh_pools, 2) as "Avg Pharaoh Pools",
  -- ROUND(avg_curve_pools, 2) as "Avg Curve Pools",
  -- ROUND(avg_kyber_pools, 2) as "Avg Kyber Pools",
  ROUND(avg_uniswap_pools, 2) as "Avg Uniswap Pools",
  ROUND(avg_swaps, 2) as "Avg Swaps in LP Pools",
  ROUND(avg_swap_volume, 2) as "Avg Swap Volume in LP Pools (USD)"
FROM retention r
LEFT JOIN inactive_pool ip 
  ON ip.wallet_tier = r.wallet_tier 
  AND ip.lp_month = r.lp_month
WHERE r.lp_month <= '2025-04-01'  
ORDER BY 
  CASE r.wallet_tier  
    WHEN '🐳 Whale' THEN 1
    WHEN '🦈 Shark' THEN 2
    WHEN '🐬 Dolphin' THEN 3
    WHEN '🐟 Fish' THEN 4
    WHEN '🐙 Octopus' THEN 5
    WHEN '🦀 Crab' THEN 6
    WHEN '🦐 Shrimp' THEN 7
  END,
  r.lp_month DESC;  
```

###### Query result:

<img width="1920" height="795" alt="image" src="https://github.com/user-attachments/assets/67895a98-8175-4637-bfef-4875c113a893" />
<img width="1920" height="792" alt="image" src="https://github.com/user-attachments/assets/b1b0c5b2-269a-4d58-a959-9221c65f89f3" />
