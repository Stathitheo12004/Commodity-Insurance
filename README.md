##Portfolio Insurance: Exploring the Safe-Haven Nature of Commodities During Stock Market Turmoil:##

To run the project on your own machine, please import the dataset from the following Excel file: 
📂 Download the dataset: [Data.xlsx](./Data.xlsx)


📁 Portfolio Insurance in Systemic Crises – A Commodity Market Perspective
Stata | Quantitative Finance | Time-Series Econometrics

This project critically examines whether alternative investments, specifically precious metals (gold, silver, platinum) and diversified commodity indices (TRCC ex Energy, BCOM)—provide robust portfolio insurance during equity market stress. Motivated by the fragility of traditional hedging instruments and recent structural shifts in global markets, the analysis evaluates both naïve and mean-variance optimal portfolios across multiple crises (GFC, COVID-19, Tech Bubble) using advanced econometric techniques.

🔍 Overview of Process
1. Research Framing and Hypotheses
This project investigates whether alternative investments — specifically precious metals and diversified commodity indices — offer meaningful portfolio insurance during episodes of equity market turmoil. We define insurance not just by correlation breakdowns, but by true downside protection: reduced systematic risk, lower drawdowns, and superior tail performance.

We test the following hypotheses:

H1: Alternative assets outperform equities during crisis periods.

H2: Betas and correlations between alternatives and the S&P500 decline under stress.

H3: Diversified indices (TRCC/BCOM) offer stronger insurance than concentrated metals.

H4: Mean-variance optimisation enhances downside protection relative to naïve portfolios.

2. Data Compilation and Portfolio Construction
Collected monthly data (1985–2025) on:

Gold, Silver, Platinum (futures-based)

TRCC ex-Energy and Bloomberg Commodity Index (BCOM)

S&P500 (equity benchmark)

Constructed two portfolios:

A naïve metals portfolio (equal-weighted)

A mean-variance optimal (MVO) portfolio spanning all assets

All assets were log-transformed and excess returns computed relative to the risk-free rate.

3. Descriptive Statistics and Initial Diagnostics
Assessed distributional properties of each asset: skewness, kurtosis, volatility.

Computed summary statistics for all portfolios and assets to benchmark average performance and tail behaviour.

Initial insights revealed heavy tails, non-normality, and episodic drawdown clustering.

4. Systematic Risk Analysis (CAPM & Crisis Regressions)
Estimated baseline CAPM regressions for each portfolio vs the S&P500.

Introduced interaction terms for:

Crisis dummies (Tech-Bubble, GFC, COVID-19)

High-volatility regimes (VIX > 30)

Captured crisis-specific beta shifts and crisis alpha.

Rolling regression plots (12M, 60M, 120M) revealed time-varying co-movements and structural beta instability.

5. Quantile Regression: Left-Tail Market Sensitivity
Estimated quantile regressions (τ = 1%, 2.5%, 5%) to capture tail dependence.

Compared beta responsiveness across Metals, TRCC, and BCOM under extreme equity losses.

Findings showed that metals compress beta significantly in the left-tail, unlike broader indices.

6. Performance Metrics and Downside Risk Assessment
Evaluated risk-return profiles using:

Sharpe, Sortino, Jensen’s Alpha

Max Drawdown, VaR (Parametric & Historical), CVaR

Benchmarked all portfolios and indices against the S&P500

Results highlighted limited standalone insurance from commodities, despite episodic safe-haven traits.

7. Visualisation and Strategic Interpretation
Created intuitive, stylised crisis-shaded plots:

Normalised price performance (2005–2025)

Drawdown comparisons

Rolling betas and correlations (short-, medium-, and long-term)

Interpretation focused on regime shifts, contagion, and the illusion of diversification in financialised commodities.

8. Conclusion
While metals offer episodic hedging benefits, true portfolio insurance requires broader diversification beyond commodities.

Correlation breakdown ≠ capital protection.

Institutional investors should consider multi-asset exposure (e.g., REITs, hedge funds, private equity) for more robust risk mitigation.

