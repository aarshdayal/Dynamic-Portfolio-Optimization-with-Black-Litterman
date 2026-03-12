# Black‑Litterman Portfolio Optimization with Sensitivity Analysis

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A complete implementation of the **Black‑Litterman asset allocation model**, combining market equilibrium with investor views to produce intuitive, well‑diversified portfolios. The project includes:

- Data fetching from **Yahoo Finance** (ETFs) and **FRED** (risk‑free rate).
- Computation of implied equilibrium returns from market capitalisation weights.
- Formulation of investor views (relative and absolute) with confidence levels.
- Application of the Black‑Litterman formula to obtain posterior expected returns.
- Mean‑variance optimisation (max Sharpe ratio) under long‑only and weight‑cap constraints.
- **Backtest** with monthly rebalancing to compare performance against benchmarks.
- **Sensitivity analysis** exploring the effect of the maximum weight cap and the prior confidence parameter `tau`.


## 📊 Results Overview

### Static Optimal Portfolios (Full‑Sample)
With a **20% per‑asset cap**, the optimizer produced an **equal‑weight** portfolio for both Black‑Litterman and equilibrium priors:

| Portfolio          | Weights                                      | Expected Return | Expected Volatility | Sharpe |
|--------------------|----------------------------------------------|-----------------|---------------------|--------|
| Black‑Litterman    | 20% each                                     | 4.54%           | 14.40%              | 0.05   |
| Equilibrium (Prior)| 20% each                                     | 4.69%           | 14.40%              | 0.06   |
| Market Cap         | SPY 40%, EFA 30%, AGG 20%, GSG 5%, VNQ 5%   | 4.50%           | 10.93%              | 0.06   |
| Equal Weight       | 20% each                                     | 10.70%          | 14.40%              | 0.48   |


*Note: The equal‑weight portfolio’s high expected return is based on historical averages, while BL and equilibrium returns are model‑implied.*

### Backtest Performance (Monthly Rebalancing, 2019–2023)

| Strategy             | Annual Return | Annual Volatility | Sharpe |
|----------------------|---------------|-------------------|--------|
| Black‑Litterman      | 7.47%         | 15.22%            | 0.36   |
| Market Cap           | 5.42%         | 11.56%            | 0.30   |
| Equal Weight         | 7.47%         | 15.22%            | 0.36   |
| Historical Mean‑Var  | 7.47%         | 15.22%            | 0.36   |


Under the 20% cap, the BL, equal‑weight, and historical mean‑variance portfolios produced **identical performance** because the constraints forced full diversification.

![Cumulative Returns Comparison](reports/cumulative_returns_comparison.png)

### Sensitivity Analysis: Effect of Max Weight and Prior Confidence `tau`

We varied the maximum allowed weight per asset from 20% to 100% (no cap) and tested three values of `tau` (0.01, 0.05, 0.1). The results show how the portfolio concentrates as constraints are relaxed, and how the prior influences the tilt.



| `tau` | Max Weight | Optimal Weights (SPY, EFA, AGG, GSG, VNQ) | Sharpe |
|-------|------------|--------------------------------------------|--------|
| 0.01  | 20%        | 20%, 20%, 20%, 20%, 20%                    | 0.057  |
| 0.01  | 30%        | 30%, 30%, 24%, 8%, 8%                      | 0.059  |
| 0.01  | 100%       | 43%, 27%, 20%, 5%, 5%                      | 0.059  |
| 0.05  | 20%        | 20%, 20%, 20%, 20%, 20%                    | 0.049  |
| 0.05  | 100%       | 55%, 12%, 19%, 9%, 4%                      | 0.053  |
| 0.1   | 20%        | 20%, 20%, 20%, 20%, 20%                    | 0.042  |
| 0.1   | 100%       | 66%, 0%, 18%, 12%, 4%                      | 0.049  |

*Interpretation*:
- As the cap increases, the optimizer tilts toward assets with higher expected Sharpe (SPY dominates).
- Smaller `tau` (more confidence in the prior) leads to more moderate tilts; larger `tau` gives the views more influence, resulting in extreme concentration (EFA drops to 0% for `tau=0.1`).
- Sharpe ratios improve marginally with fewer constraints, but at the cost of concentration risk.


Cumulative returns for static portfolios under different `tau` and max weights:

| `tau = 0.01` | `tau = 0.05` | `tau = 0.1` |
|--------------|--------------|-------------|
| ![tau0.01](reports/cumulative_returns_tau_0.01.png) | ![tau0.05](reports/cumulative_returns_tau_0.05.png) | ![tau0.1](reports/cumulative_returns_tau_0.1.png) |

## 🧠 Methodology

1. **Data**: Daily adjusted close prices for 5 ETFs (SPY, EFA, AGG, GSG, VNQ) from Yahoo Finance; 3‑month T‑bill rate from FRED (via `fredapi`).
2. **Covariance**: Annualised from daily returns.
3. **Equilibrium returns**: Using CAPM reverse‑engineering:  
   `μ_eq = λ Σ w_mkt` with λ = (market excess return) / (market variance).
4. **Views**:
   - View 1: SPY outperforms EFA by 3% p.a. (relative).
   - View 2: GSG excess return = 2% p.a. (absolute).
   - Confidence: variance of view portfolio scaled by 0.5.
  
5. **Black‑Litterman posterior**:
   - `μ_bl = [(τΣ)⁻¹ + PᵀΩ⁻¹P]⁻¹ [(τΣ)⁻¹ μ_eq + PᵀΩ⁻¹Q]`
6. **Optimisation**: Maximise Sharpe ratio (long‑only, fully invested) with a per‑asset cap.
7. **Backtest**: Monthly rebalancing using a 252‑day rolling window for covariance and equilibrium returns.
