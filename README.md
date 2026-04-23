AlphaEngine MVP: 3-Year Forward Equity Signal Regressor
Overview

AlphaEngine is a machine learning pipeline designed to predict long-horizon (3-year) cross-sectional equity returns.

Rather than forecasting absolute price levels or macroeconomic cycles, the model focuses on relative performance—estimating each asset’s expected excess return versus a benchmark (S&P 500). This enables direct ranking of investment opportunities and systematic portfolio construction based on conviction.

This repository represents a Minimum Viable Product (MVP) for a signal-driven portfolio allocation system, combining momentum, risk, and fundamental features into a unified framework.

Objective

The model answers the following question:

Given information available today, which stocks are most likely to outperform the market over the next ~3 years, and by how much?

This reframes stock selection as a continuous regression problem rather than a binary classification task, aligning more closely with real-world portfolio construction.

Data Pipeline
Inputs
Historical equity price data (via yfinance)
Fundamental metrics (P/E, earnings yield)
Benchmark: SPY (S&P 500 ETF)
Target Construction

Forward returns are computed over approximately 3 trading years (~756 days):

Stock forward return
Benchmark forward return
Excess return:
excess_fwd_3y_return = stock_return - benchmark_return
Methodology
1. Target Transformation

Equity returns exhibit heavy tails, particularly on the upside. To stabilize training and reduce the influence of extreme outliers, the target is transformed using a symmetric log function:

y = sign(x) * log(1 + |x|)

This compresses extreme values while preserving directional information.

2. Time-Aware Preprocessing

To mitigate leakage from overlapping forward returns, the dataset is downsampled to monthly observations (last trading day of each month).

This reduces autocorrelation and forces the model to learn generalized cross-sectional relationships rather than memorizing overlapping daily noise.

3. Feature Engineering

Features are constructed across three layers:

Raw Features
Momentum: 6-month and 12-month returns
Risk: 12-month volatility, drawdown
Valuation: P/E ratio, earnings yield
Cross-Sectional Features
Within-date percentile ranks for all major features
Ensures comparability across time and market regimes
Composite Factor

A key engineered feature combines value and quality:

quality_value_combo = earnings_yield_rank + low_vol_rank

Where:

earnings_yield_rank captures relative cheapness
low_vol_rank captures relative stability

This avoids scale distortions and eliminates instability from negative earnings.

4. Model
Algorithm: LightGBM Regressor
Objective: regression (predict transformed excess returns)
Time-based split:
Train: pre-2018
Validation: 2018–2020
Test: 2021 onward
5. Calibration

Predictions are calibrated using Isotonic Regression:

Fitted on validation data only (to prevent leakage)
Applied to test predictions
Enforces a monotonic relationship between predicted and realized returns

This improves interpretability and aligns predicted magnitudes with observed outcomes.

6. Portfolio Construction

Portfolios are constructed monthly and evaluated over a 3-year forward horizon.

Strategy Variants
Low Risk: broader exposure (top ~30% of stocks)
Medium Risk: balanced exposure (top ~20%)
High Risk: concentrated exposure (top 5 stocks)
Weighting Scheme

Rank-based exponential weighting:

weight ∝ rank^power

This avoids excessive concentration from magnitude-based weighting while still emphasizing high-conviction signals.

Results (Out-of-Sample: 2021–2022)

The model demonstrates strong cross-sectional ranking performance:

Monotonic performance across prediction buckets
Approximately 80% spread between top and bottom quintiles
Higher-conviction portfolios exhibit:
Higher mean excess returns
Higher hit rates versus benchmark
Increased variance consistent with concentration

These results suggest the model is capturing meaningful signal rather than purely reflecting broad market movements.

Limitations
Overlapping forward returns introduce dependence between samples
Transaction costs, slippage, and turnover are not modeled
Out-of-sample period is relatively short
Feature set is intentionally limited (MVP scope)
Future Work
Rolling backtest with non-overlapping return windows
Monthly/short-horizon model variant
Sector-neutral portfolio constraints
Expanded fundamentals (profitability, growth, balance sheet metrics)
Macro regime features (interest rates, volatility indices, inflation)

Disclaimer
This project is a research prototype and not an investment strategy. Results are for exploratory and educational purposes only and should not be interpreted as financial advice.

Summary
This project demonstrates:

End-to-end machine learning pipeline design
Time-aware modeling on financial data
Robust feature engineering using cross-sectional normalization
Translation of predictions into actionable portfolio strategies
