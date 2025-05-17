Portfolio Insurance: Exploring the Safe-Haven Nature of Commodities During Stock Market Turmoil.

To run the project on your own machine, please import the dataset from the following Excel file: 📂 Download the dataset: [Data.xlsx](./Data.xlsx)


📁 Portfolio Insurance in Systemic Crises – A Commodity Market Perspective
Stata | Quantitative Finance | Time-Series Econometrics

This project critically examines whether alternative investments, specifically precious metals (gold, silver, platinum) and diversified commodity indices (TRCC ex Energy, BCOM)—provide robust portfolio insurance during equity market stress. Motivated by the fragility of traditional hedging instruments and recent structural shifts in global markets, the analysis evaluates both naïve and mean-variance optimal portfolios across multiple crises (GFC, COVID-19, Tech Bubble) using advanced econometric techniques.

📊 Methodological Framework
The analysis leverages a rich monthly dataset (1985–2025), employing:

CAPM regressions with crisis interaction terms to capture state-dependent risk exposure

Rolling 12M, 60M, and 120M betas and correlations to uncover long-horizon integration dynamics

Quantile regressions to identify lower-tail sensitivity and decoupling behaviour

Risk-adjusted performance metrics: Sharpe, Sortino, Jensen’s Alpha, CVaR, Max Drawdown

Full integration of Stata automation and visualisation, including dynamic overlays and loop-structured diagnostics

📌 Key Contributions
Challenges the assumption that broader commodity diversification (e.g., TRCC, BCOM) offers stronger protection than concentrated metal baskets

Shows that while metals offer episodic hedging, no commodity-based strategy delivers consistent downside insulation

Demonstrates that beta compression ≠ insurance without drawdown resilience or tail-risk absorption

Calls for multi-asset diversification beyond commodities for institutional-grade portfolio insurance

⚙️ Repository Structure
Assessment 2 Do.File.do: Clean, loop-structured Stata file containing full replication pipeline

Figures/: High-resolution rolling window plots, CAPM diagnostics, quantile regression panels

