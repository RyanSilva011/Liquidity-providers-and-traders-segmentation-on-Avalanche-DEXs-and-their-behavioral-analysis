https://github.com/RyanSilva011/Liquidity-providers-and-traders-segmentation-on-Avalanche-DEXs-and-their-behavioral-analysis

# Avalanche DEX Liquidity Providers and Trader Segmentation — Behavioral Analysis

This repository hosts a comprehensive dashboard and the accompanying analysis that maps how traders and liquidity providers behave across Avalanche-based DEXs. It uses Flipside’s Intelligence Driven Growth (IDG) methodology to reveal user journeys, engagement patterns, platform usage, and liquidity contributions from May 2024 through April 2025. The work focuses on major Avalanche DEXs like Pangolin, Pharaoh, Trader Joe, and others that matter for the ecosystem. It presents actionable insights for researchers, product managers, liquidity providers, and traders who want to understand the dynamics of the Avalanche DeFi scene.

- Project domain: avalanche, dex, liquidity-providers, pangolin, pharaoh, segmentation, traderjoe, traders, uniswap, user-behavior-analytics, whales

- Primary release: this project ships a bundle of data artifacts, dashboards, and notebooks. See the releases page to download the packaged artifact and run it locally or in your environment.

- Releases link: https://github.com/RyanSilva011/Liquidity-providers-and-traders-segmentation-on-Avalanche-DEXs-and-their-behavioral-analysis/releases

Images to set the theme:
![Avalanche Theme](https://upload.wikimedia.org/wikipedia/commons/thumb/9/96/Avalanche_logo.png/320px-Avalanche_logo.png)
![Data Visualization Example](https://upload.wikimedia.org/wikipedia/commons/thumb/3/37/Line_chart_example.svg/640px-Line_chart_example.svg.png)

Table of contents
- About this project
- What you will learn
- Data sources and methodology
- IDG methodology at a glance
- DEX landscape on Avalanche
- Segmentation framework
- Dashboard components
- How to reproduce the analysis
- Repository structure
- Data model and definitions
- Visualization and storytelling
- How to contribute
- Data privacy and ethics
- Licensing and credits
- Frequently asked questions
- Release workflow and downloads

About this project
This project aims to illuminate how liquidity providers and traders interact with Avalanche DEXs. It combines transaction data, liquidity metrics, and user behavior signals to paint a complete picture of who participates, how often, and with what impact on liquidity and price formation. The analysis helps teams identify high-value segments, optimize pool composition, and design better onboarding and retention strategies for ecosystem participants.

What you will learn
- How traders and liquidity providers differ in activity, churn, and liquidity contribution across Pangolin, Pharaoh, Trader Joe, and Uniswap deployments on Avalanche.
- How user journeys flow across DEX interfaces, from onboarding to completed swaps or liquidity allocations.
- Which liquidity pools attract the most stable capital and how impermanent loss risk is perceived by different segments.
- How whales and mid-tier participants influence liquidity dynamics and price discovery.
- How the IDG framework guides the collection, modeling, and interpretation of behavioral signals.

Data sources and methodology
- Time window: May 2024 to April 2025.
- Core data: on-chain transactions, liquidity events (adds/removes), pool volumes, pool liquidity, and token prices.
- Supplementary data: offer and trade signals from Flipside’s Intelligence Driven Growth (IDG) framework, user session traces (where available), and liquidity provider issuance signals.
- Data quality: standardize timestamps, align tokens by canonical symbol, normalize volumes to USD using end-of-day price feeds, and reconcile cross-DEX liquidity positions.
- Privacy: the analysis focuses on aggregated and anonymized behavior signals. Individual wallets are treated with privacy-preserving aggregation.

IDG methodology at a glance
- Identify segments: classify users by activity levels, liquidity contributions, and engagement depth.
- Define journeys: map typical paths from entry to liquidity provision or trade, including gateway actions such as token swaps, liquidity adds, pool migrations, and yields exploration.
- Govern growth signals: track engagement amplification, retention gates, and value creation across the lifecycle of an Avalanche DEX user.
- Generate dashboards: produce interpretable visuals that reveal segment characteristics, journey bottlenecks, and opportunities for optimization.

DEX landscape on Avalanche
- Pangolin: one of the oldest and most active DEXs on Avalanche, known for its broad whitelist of pools and gas-efficient swaps.
- Trader Joe: a highly popular DEX with deep liquidity across major pools and a vibrant farming ecosystem.
- Pharaoh: a newer but rapidly growing venue with unique incentive structures and specialized pools.
- Uniswap (Avalanche deployment): ecosystem-agnostic liquidity and trading ideas spanning across Avalanche deployments.
- Other notable venues: community-led forks and niche pools that contribute to liquidity depth and price discovery.

Segmentation framework
- Trader segments: casual traders, regular traders, and high-frequency traders. Each segment shows distinct patterns in swap frequency, token diversity, and routing preferences.
- Liquidity providers: small, mid, and large LPs; seasonality in liquidity contribution; pool preferences; and sensitivity to impermanent loss.
- Whale watchers: high-value accounts whose activity significantly shifts liquidity distribution and pool risk exposure.
- Time-based cohorts: new users vs. long-term participants; onboarding friction points; and retention curves across 1 week, 1 month, and 3 months horizons.
- Behavior signals: swap counts, liquidity add/remove cadence, pool migration patterns, token concentration, and fee capture opportunities.

Dashboard components
- Overview panel: high-level metrics across all DEXs, such as total liquidity, daily volumes, number of unique traders, and LP counts.
- Segmentation explorer: interactive panels to filter by trader type, LP tier, time window, and DEX.
- Journey mapper: visualizes typical user paths from onboarding to liquidity provision or exit.
- Liquidity dynamics: pool-level metrics, liquidity concentration, and impermanent loss indicators.
- User engagement: engagement depth, session lengths, and revisit rates.
- Token flow and price impact: routing paths, token pairs, and price slippage metrics.
- Whale impact: concentration of liquidity and trades among the top wallets.
- Temporal analytics: day-by-day trends, week-over-week changes, and month-over-month seasonality.
- Compare dashboards: side-by-side comparisons of Pangolin, Pharaoh, Trader Joe, and Uniswap deployments.

How to reproduce the analysis
Note: the release asset contains the necessary notebooks, dashboards, and lightweight data layers. To reproduce the work locally, download the release artifact and run the provided scripts. See the Releases page for the exact asset and instructions.

- Step 1: Download the release artifact
  - From the repository releases page, download the packaged artifact that contains the dashboards, notebooks, and data schemas.
  - The asset is designed to be deployed in a local environment or in a lightweight data workspace.

- Step 2: Install prerequisites
  - Ensure you have Python 3.11+ or Node.js 18+ depending on the artifact type.
  - Install required libraries and packages listed in the environment file (requirements.txt or package.json).
  - If you are using a notebook-based flow, install Jupyter or JupyterLab and the necessary kernels.

- Step 3: Configure data access
  - Supply any required API keys or on-chain data access tokens if the artifact pulls optional data from external services.
  - Confirm the data path mappings to on-disk or cloud storage.

- Step 4: Run the dashboards
  - Start the server or notebook session as described in the artifact's instructions.
  - Open the local URL shown by the tool and navigate to the segmentation and behavior dashboards.

- Step 5: Validate results
  - Cross-check the date range (May 2024–April 2025) and the DEXs covered.
  - Validate key metrics such as liquidity, volumes, and user counts against the source data or the artifact’s validation notebooks.

- Step 6: Extend and customize
  - Use the modular components to add new segments, pools, or DEXs.
  - Integrate additional data sources to enrich the analysis, such as on-chain risk metrics, governance signals, or cross-chain flows.

Repository structure
- dashboards/: Interactive dashboards and visualization components (Plotly, Dash, or similar).
- notebooks/: Reproducible analyses and experiments (Jupyter notebooks).
- data/: Raw and processed data schemas, sample datasets, and data dictionaries.
- src/: Core code for data ingestion, modeling, and utility functions.
- docs/: User guides, methodology notes, and technical references.
- scripts/: Helper scripts for setup, data validation, and release packaging.
- tests/: Unit and integration tests for data processing and dashboard components.
- LICENSE: License terms for use and distribution.
- README.md: This file, providing an overview and how to use the project.

Data model and definitions
- User: A wallet address or account that interacts with any DEX. Anonymized for display in aggregates.
- Trader: A user who executes swaps, flash loans, or routing operations across pools.
- Liquidity provider (LP): A user who adds liquidity to pools and earns fees and rewards.
- Pool: A liquidity pool on a Dex that contains a pair of tokens and associated liquidity metrics.
- Token: A token symbol and its on-chain price data used in the pools.
- Volume: The number of swap operations and the total value traded within a window.
- Liquidity: The total value of assets locked in a pool.
- Impermanent loss (IL): The divergence in value the LP faces when prices change.

Visualization and storytelling
- We emphasize story-driven dashboards. Each dashboard tells a clear story about a segment’s behavior, the liquidity dynamics, and how traders route across DEXs.
- Visuals use consistent color schemes to differentiate traders, LPs, whales, and pools.
- Each visualization includes a short interpretation note to guide the reader through the takeaway.

How to contribute
- Prerequisites: a GitHub account, a local development environment, and a willingness to follow the project’s contribution guidelines.
- How to propose changes:
  - Fork the repository.
  - Create a feature branch.
  - Implement changes in notebooks, dashboards, or data schemas.
  - Run tests and validations.
  - Open a pull request with a clear description of the change and its impact.
- Coding standards: write clear, well-documented code; add unit tests where applicable; maintain consistent naming and formatting.

Data privacy and ethics
- The project aggregates behavior signals from on-chain data. We avoid exposing any personally identifiable information (PII).
- We emphasize responsible data handling, minimizing data exposure, and maintaining trust with the community.

Licensing and credits
- The project uses open data and open tooling. All code is provided under an appropriate open-source license as specified in the LICENSE file.
- Credits go to the Flipside IDG framework contributors and the Avalanche DeFi community for data sources and context.

Frequently asked questions
- What is the scope of the analysis?
  - It covers liquidity providers and traders across major Avalanche DEXs from May 2024 to April 2025.
- Which DEXs are included?
  - Pangolin, Pharaoh, Trader Joe, Uniswap deployments on Avalanche, and other notable venues in the ecosystem.
- How are segments defined?
  - Segments are defined by activity, liquidity contribution, and engagement depth. Whale signals are considered separately.
- Can I reuse the dashboards for my own data?
  - Yes. The artifact is designed for reuse, customization, and extension. See the release instructions for details.
- How do I know the results are accurate?
  - The dashboards include validation steps and cross-checks against source data. Ground-truth checks are described in the notebooks.

Release workflow and downloads
- The released artifact contains the fully built dashboards, the necessary data schemas, and the analysis notebooks. To reproduce, download the artifact from the Releases page, set up the environment, and run the provided scripts or notebooks.
- Release download link: https://github.com/RyanSilva011/Liquidity-providers-and-traders-segmentation-on-Avalanche-DEXs-and-their-behavioral-analysis/releases

Notes on releases
- The Releases section contains packaged files you can download and execute. If you need to reproduce the study locally, follow the steps in the artifact’s documentation.
- If you want to review or re-run the analyses, please visit the Releases page to fetch the latest artifact and any accompanying notes.

Community and support
- You can engage with the project maintainers through issues and pull requests.
- For questions about data sources or methodology, please reference the methodology notes in the docs folder or open an issue.

Acknowledgments
- Thanks to the Flipside IDG methodology team for the framework that underpins the analysis.
- Gratitude to the Avalanche DeFi community for real-world data signals and the vibrant ecosystem that makes this work possible.

Technical appendix
- Data schemas: a thorough glossary of entities, relationships, and fields used in the dashboards.
- Token pricing: how prices are sourced and normalized to USD.
- Time alignment: approach to aligning on-chain events with dashboard time windows.

Roadmap (high level)
- Expand coverage to additional pools and cross-chain flows.
- Add more segmentation dimensions, including risk appetite and liquidity sourcing behavior.
- Improve the interpretability of journey maps with narrative annotations and scenario testing.
- Integrate forecasting models to project liquidity changes and trader activity.

Appendix: FAQ on downloads
- Q: How do I know which file to download from the Releases page?
  - A: The release artifact contains a self-contained dashboard bundle with a readme tailored to reproduce the study. Use the most recent release to get the updated data, visuals, and notes.
- Q: The link to releases is not working. What should I do?
  - A: Check the Releases section of the repository for the latest artifact. If the link is temporarily down, you can still browse the repository’s Releases tab to locate the latest asset and its instructions.
- Q: I want to extend this work to new DEXs. How should I proceed?
  - A: Start by adding new pools to the data model, update the ingestion scripts, and incorporate the new pools into the segmentation framework. Update the dashboards to reflect the expanded scope.

Final note
- The repository aims to be a living resource. It is designed for clarity, reproducibility, and practical insights for people who work with Avalanche DEXs. The dashboards are built to be navigable even for readers new to DeFi analytics, while offering depth for advanced researchers and practitioners.

Release link embedded reference
- See the releases: https://github.com/RyanSilva011/Liquidity-providers-and-traders-segmentation-on-Avalanche-DEXs-and-their-behavioral-analysis/releases

End of document