# Multifactor Quantitative Equity Strategy Framework

> An end-to-end quantitative research, multifactor alpha modeling, portfolio optimization, and execution pipeline built for U.S. equities (S&P 500 universe).

---

## 📌 Project Overview
This repository contains a modular quantitative investment research pipeline that moves from raw market & fundamental data collection to signal engineering, machine learning-driven return prediction, risk-constrained portfolio optimization, and detailed performance attribution.

### Key Performance Highlights (Out-of-Sample: 2021–2025)
* **Alpha Generation**: Built a 12-factor library across price-volume, technical, and fundamental signals using Rank IC/IR; identified 20-day volatility as the strongest standalone predictor.
* **Predictive Modeling**: Trained Random Forest and XGBoost return prediction models under strict temporal leakage controls, achieving an **out-of-sample IC of 0.0191**.
* **Portfolio Optimization**: Constructed max-Sharpe and risk-parity portfolios using Ledoit-Wolf covariance estimation under position and turnover constraints; optimized to 5-day rebalancing, cutting Sharpe erosion from 39% to 16% versus daily trading.
* **Performance Attribution**: Isolated **+0.79% active alpha (IR 0.202)** via Brinson/style attribution and conducted stress testing for tracking errors and drawdown risks during historical stress regimes.

---

## 🛠️ System Architecture & Workflow

### Phase 1: Data Pipeline & Baseline Single-Factor Analysis
* **Universe & Data Collection**: Scalable downloading of daily market data (S&P 500 components, 2010–2025) via `yfinance` with forward-fill handling for missing values (max 5 consecutive days limit) and strict point-in-time checks to prevent look-ahead bias.
* **Alpha Library Construction**:
  * *Price-Volume & Technical*: 20-day momentum, 5-day short-term reversal, 20-day volatility, volume ratios, and technical indicators (RSI, MACD, Bollinger Bands).
  * *Fundamental & Quality*: ROE, ROA, P/E, P/B, and debt-to-equity ratios aligned using announcement dates rather than fiscal quarter-end dates.
* **Factor Evaluation**: Evaluated signals cross-sectionally using Spearman Rank IC/IR, IC win rates, cumulative IC curves, and 5-quantile monotonic net worth sorting.

### Phase 2: Factor Orthogonalization & ML Synthesis
* **Factor Denoising**: Applied symmetric/PCA orthogonalization to remove strong multi-collinearity across technical and fundamental signals.
* **Machine Learning Models**:
  * *Dataset Split*: Training (2010–2018), Validation (2019–2020), Test (2021–2025).
  * *Algorithms*: Random Forest & XGBoost classifiers/regressors predicting 5-day forward returns using Z-score normalized cross-sectional features.
* **Risk-Constrained Portfolio Optimization**:
  * Estimated covariance matrices using **Ledoit-Wolf shrinkage** for numerical stability.
  * Optimized portfolios using `PyPortfolioOpt`/`cvxpy` under explicit weight constraints ($w_i \le 5\%$, long-only).
  * Modeled real-world transaction friction (10 bps single-way) and turnover penalties to prevent over-trading.

### Phase 3: Index Tracking, Market Neutrality & Risk Attribution
* **Enhanced Indexing**:
  * Modeled active risk versus the S&P 500 benchmark subject to active deviation bounds ($\le \pm 2\%$ per stock, $\le \pm 5\%$ per GICS sector, $\le \pm 0.5 \sigma$ across style exposures).
  * Target tracking error constrained within 3%–5%.
* **Market-Neutral Strategy**:
  * Hedged systemic market risk (Beta) using SPY/ES futures short proxies, evaluating pure residual alpha with sector neutrality.
* **Performance Attribution & Stress Testing**:
  * **Brinson Allocation**: Decomposed excess returns into allocation, selection, and interaction effects across GICS sectors.
  * **Stress Analysis**: Evaluated drawdown profiles and covariance stability during market shocks (e.g., 2020 COVID sell-off, 2022 rate hike bear market).

---

## 📊 Strategy Comparison Matrix

| Strategy Variant | Ann. Return | Ann. Volatility | Sharpe Ratio | Max Drawdown | Information Ratio | Tracking Error |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **S&P 500 Benchmark** | Baseline | Baseline | Baseline | Baseline | N/A | N/A |
| **Stage 1: ML Equal-Weighted (No Cost)** | High | Moderate | Moderate | Moderate | N/A | N/A |
| **Stage 2: Max Sharpe + Friction (10bps)** | Optimized | Controlled | High | Low | High | N/A |
| **Stage 3: Index Enhancement Strategy** | Controlled | Low | High | Low | $>0.20$ | $3\% - 5\%$ |
| **Stage 3: Market Neutral Alpha** | Absolute Return | Very Low | Stable | Minimal | N/A | Neutral |

---

## 📂 Repository Structure

```text
.
├── data/                  # Raw and preprocessed price/fundamental datasets
├── src/
│   ├── data_loader.py     # YFinance & fundamental data download & cleaning
│   ├── alpha_factors.py   # Alpha factor formulas and technical indicators
│   ├── orthogonal.py      # Factor Z-score normalization & PCA/Symmetric orthogonalization
│   ├── ml_models.py       # Random Forest & XGBoost model training pipelines
│   ├── optimizer.py       # Ledoit-Wolf covariance estimation & PyPortfolioOpt solvers
│   └── attribution.py     # Brinson attribution & style factor regression
├── notebooks/
│   ├── 01_factor_ic_analysis.ipynb
│   ├── 02_ml_alpha_synthesis.ipynb
│   └── 03_index_enhancement_and_attribution.ipynb
├── tests/                 # Unit tests for temporal alignment and zero look-ahead bias
├── README.md              # Project documentation
└── requirements.txt       # Python dependencies
