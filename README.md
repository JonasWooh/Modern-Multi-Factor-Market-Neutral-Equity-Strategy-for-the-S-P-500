# Modern Multi-Factor Market Neutral Equity Strategy for S&P 500

A comprehensive quantitative finance research project that implements a multi-factor market-neutral equity strategy on the S&P 500 universe. The repository covers the full research pipeline: data acquisition, factor engineering, risk modeling, portfolio optimization, and monthly proxy backtesting with performance attribution.

> Authorship notice: all code in this repository was independently written and assembled by Jonas Wu. Please do not copy, reuse, or redistribute this work without explicit permission and proper attribution.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Key Features](#key-features)
- [Important Implementation Notes](#important-implementation-notes)
- [Project Pipeline](#project-pipeline)
- [Repository Structure](#repository-structure)
- [Data Pipeline & Outputs](#data-pipeline--outputs)
- [Methodology](#methodology)
  - [Phase 1: Data Acquisition & Preprocessing](#phase-1-data-acquisition--preprocessing)
  - [Phase 2: Factor Engineering](#phase-2-factor-engineering)
  - [Phase 3: Risk Modeling & Portfolio Optimization](#phase-3-risk-modeling--portfolio-optimization)
  - [Phase 4: Backtesting & Performance Attribution](#phase-4-backtesting--performance-attribution)
- [Results](#results)
- [Requirements](#requirements)
- [Usage](#usage)
- [License](#license)

---

## Project Overview

This project constructs a dollar-neutral, sector-balanced long/short equity portfolio using three classic alpha factors: Value, Size, and Momentum. The implementation combines:

- Fama-MacBeth-style monthly cross-sectional regressions to extract realized pure factor returns
- Factor Risk Parity (FRP) for dynamic multi-factor signal weighting
- A PCA-based statistical risk model for covariance estimation
- CVXPY optimization with turnover penalty and realistic portfolio constraints
- Monthly Fama-French 5-factor attribution on realized portfolio returns

Key windows in the implemented pipeline:

- Universe history: daily panel from 2008-01 to 2024-12
- Signal sample: 2010-01 to 2024-12
- Portfolio backtest sample: 2011-12 to 2024-12
- Rebalancing frequency: monthly

Data sources:

- CRSP and Compustat via WRDS
- Fama-French 5-factor daily data for monthly aggregation and attribution

---

## Key Features

| Feature | Description |
|---|---|
| Survivorship-Bias-Free Universe | Uses a dynamic S&P 500 membership history and retains delisted securities in the price panel |
| Point-in-Time Discipline | Fundamentals are aligned with `pit_date = max(rdq + 1 day, datadate + 45 days)` to avoid look-ahead bias |
| Intra-Sector Standardization | Z-scores are computed within `date x gsector` groups using MAD winsorization |
| Three-Factor Signal Stack | Value, Size, and Momentum are standardized and merged into a monthly 3-factor panel |
| Pure Factor Return Extraction | Monthly cross-sectional regressions produce realized pure factor returns for Value, Size, and Momentum |
| Dynamic FRP Weighting | Direct risk parity on rolling pure-factor covariance generates time-varying factor weights |
| PCA Risk Model | Stratified core selection, PCA on the correlation matrix, universe regressions, and post-PCA diagonal shrinkage |
| Convex Optimization | Dollar-neutral portfolio construction with leverage, single-name, sector, volatility, and turnover controls |
| Enhanced Constraint Set | Adds market beta neutrality, tighter single-name limits, and higher gross capacity for the enhanced branch |
| Monthly Proxy Backtest | Flat bps trading cost proxy plus borrow cost, monthly NAV, FF5 attribution, drawdown, and long/short leg analysis |


## Project Pipeline

1. Phase 1: Data Acquisition & Preprocessing
   - Notebook: `HFS_Project_Data_Fetcher_0304.ipynb`
   - Outputs:
     - `data/sp500_price_panel_with_delist.csv.gz`
     - `data/sp500_monthly_returns.csv.gz`
     - `data/sp500_monthly_signals_rebuilt_nodvmt.csv.gz`

2. Phase 2.1: Momentum Factor Construction
   - Notebook: `HFS_Phase2_Momentum.ipynb`
   - Output:
     - `data/sp500_monthly_signals_3factor.csv.gz`

3. Phase 2.2: Cross-Sectional Regression
   - Notebook: `HFS_Phase2_Task2_2_CrossSectionalRegression.ipynb`
   - Output:
     - `data/sp500_pure_factor_returns.csv.gz`

4. Phase 2.3 / 2.4: FRP and Expected Returns
   - Notebook: `HFS_Phase2_Task2_3_FRP_and_ExpectedReturns.ipynb`
   - Outputs:
     - `data/sp500_frp_weights.csv.gz`
     - `data/sp500_expected_returns.csv.gz`

5. Phase 3.1: PCA Risk Model
   - Notebook: `HFS_Phase3_Task3_1_PCA_RiskModel.ipynb`
   - Outputs:
     - `data/sp500_risk_model.pkl`
     - `data/sp500_risk_model_diagnostics.csv`

6. Phase 3.2: Baseline Optimizer
   - Notebook: `HFS_Phase3_Task3_2_Optimizer.ipynb`
   - Outputs:
     - `data/sp500_portfolio_weights.csv.gz`
     - `data/sp500_backtest_results.csv.gz`

7. Phase 3.3: Enhanced Optimizer
   - Notebook: `HFS_Phase3_Task3_3_Enhanced_Optimizer_Phase4.ipynb`
   - Outputs:
     - `data/sp500_portfolio_weights_enhanced.csv.gz`
     - `data/sp500_backtest_results_enhanced.csv.gz`
     - `figures_enhanced/`

8. Phase 4: Baseline Backtest Attribution
   - Notebook: `HFS_Phase4_Task3_2_Backtest_Attribution.ipynb`
   - Outputs:
     - `data/sp500_phase4_full_report.csv.gz`
     - `figures/`

---

## Repository Structure

```text
README.md
Project_Proposal_SP500 Market Neutral.pdf
HFS_Project_Data_Fetcher_0304.ipynb
HFS_Phase2_Momentum.ipynb
HFS_Phase2_Task2_2_CrossSectionalRegression.ipynb
HFS_Phase2_Task2_3_FRP_and_ExpectedReturns.ipynb
HFS_Phase3_Task3_1_PCA_RiskModel.ipynb
HFS_Phase3_Task3_2_Optimizer.ipynb
HFS_Phase3_Task3_3_Enhanced_Optimizer_Phase4.ipynb
HFS_Phase4_Task3_2_Backtest_Attribution.ipynb

/data
/figures
/figures_enhanced
/archived
```

---

## Data Pipeline & Outputs

Main saved artifacts used by downstream notebooks:

- `data/sp500_price_panel_with_delist.csv.gz`: survivorship-bias-free daily price panel with delisting adjustment inputs
- `data/sp500_monthly_returns.csv.gz`: monthly compounded stock returns
- `data/sp500_monthly_signals_3factor.csv.gz`: monthly Value, Size, Momentum exposures
- `data/sp500_pure_factor_returns.csv.gz`: realized monthly pure factor returns
- `data/sp500_frp_weights.csv.gz`: FRP weights and risk-contribution diagnostics
- `data/sp500_expected_returns.csv.gz`: expected-return panel with `z_frp_compound`, `ret_vol`, and `mu_compound`
- `data/sp500_risk_model.pkl`: rolling PCA risk model objects (`beta`, `omega_f`, `lambda_eps`, `permnos`)
- `data/sp500_risk_model_diagnostics.csv`: monthly risk-model diagnostics
- `data/sp500_portfolio_weights.csv.gz`: baseline portfolio weights
- `data/sp500_portfolio_weights_enhanced.csv.gz`: enhanced portfolio weights
- `data/sp500_backtest_results.csv.gz`: baseline gross backtest outputs
- `data/sp500_backtest_results_enhanced.csv.gz`: enhanced gross and net backtest outputs
- `data/sp500_phase4_full_report.csv.gz`: baseline gross and net backtest outputs after Phase 4 cost modeling

---

## Methodology

### Phase 1: Data Acquisition & Preprocessing

**Notebook**: `HFS_Project_Data_Fetcher_0304.ipynb`

- Fetches S&P 500 constituent history, CRSP prices and returns, delisting events, and Compustat quarterly fundamentals from WRDS.
- Constructs a survivorship-bias-free daily price panel.
- Applies delisting adjustments using `(1 + ret) * (1 + dlret) - 1` on delisting dates.
- Builds point-in-time fundamentals with `pit_date = max(rdq + 1 day, datadate + 45 days)`.
- Computes:
  - Value: `0.67 * Z(Earnings Yield) + 0.33 * Z(Price-to-Book)`
  - Size: `-Z(log market cap)`
- Standardizes all raw components within `date x gsector` groups using MAD winsorization.

### Phase 2: Factor Engineering

#### Task 2.1 - Momentum Factor (`HFS_Phase2_Momentum.ipynb`)

- Implements the Jegadeesh-Titman style momentum signal over months `[t-12, t-2]`.
- Skips the most recent month to reduce short-term reversal contamination.
- Requires at least 8 valid months out of the 11-month formation window.
- Applies the same intra-sector MAD winsorized z-score procedure used in Phase 1.

#### Task 2.2 - Cross-Sectional Regression (`HFS_Phase2_Task2_2_CrossSectionalRegression.ipynb`)

- Merges same-month stock excess returns with same-month factor exposures.
- Runs monthly cross-sectional OLS of excess returns on Value, Size, and Momentum exposures plus an intercept.
- Saves realized pure factor returns `f_value`, `f_size`, and `f_mom`.
- Reports coefficient t-statistics and cross-sectional `R^2` diagnostics.

#### Task 2.3 / 2.4 - FRP and Expected Returns (`HFS_Phase2_Task2_3_FRP_and_ExpectedReturns.ipynb`)

- Estimates a rolling covariance matrix of pure factor returns using EWMA.
- Solves direct risk parity in factor space; PCA-mapped weights are also saved for diagnostics.
- Forms the composite factor score:
  - `z_frp_compound = w_value * value_factor + w_size * size_factor + w_mom * momentum_factor`
- Estimates trailing 12-month stock volatility.
- Builds expected returns as:
  - `mu_compound = ret_vol * z_frp_compound`
- Winsorizes `mu_compound` cross-sectionally each month.
- Validates signal quality with next-month Information Coefficient diagnostics.

### Phase 3: Risk Modeling & Portfolio Optimization

#### Task 3.1 - PCA Risk Model (`HFS_Phase3_Task3_1_PCA_RiskModel.ipynb`)

- Selects a stratified core universe of about 55 stocks across 11 GICS sectors.
- Computes a rolling core covariance matrix from daily returns.
- Runs PCA on the **correlation matrix**, not the raw covariance matrix.
- Retains between 3 and 30 principal components to reach a 90% cumulative variance target.
- Regresses the wider universe on core PC returns to estimate loadings and idiosyncratic variance.
- Reconstructs the covariance model as `Sigma = beta * Omega_f * beta' + Lambda_eps`.
- Applies a 5% post-PCA diagonal shrink toward the full covariance diagonal.

#### Task 3.2 - Baseline Optimizer (`HFS_Phase3_Task3_2_Optimizer.ipynb`)

- Solves a CVXPY objective of expected return minus risk minus turnover penalty.
- Uses the risk model from Phase 3.1 and expected returns from Phase 2.4.
- Baseline hard constraints:
  - dollar neutrality: `sum(w) = 0`
  - gross leverage: `||w||_1 <= 2.0`
  - single-name bounds: `|w_i| <= 5%`
  - annualized volatility target: `10%`
  - sector exposure tolerance: `+-2%`
- Uses monthly rebalancing and a relaxation cascade if the strict problem is infeasible.

#### Task 3.3 - Enhanced Optimizer (`HFS_Phase3_Task3_3_Enhanced_Optimizer_Phase4.ipynb`)

| Constraint / Setting | Baseline | Enhanced |
|---|---|---|
| Market beta neutrality | Not enforced | `|beta_mkt . w| <= 0.05` |
| Beta estimator | N/A | 36-month monthly beta, minimum 24 observations |
| Single-name bound | `5%` | `2%` |
| Gross leverage | `2.0` | `2.5` |
| Sector tolerance | `+-2%` | `+-2%` |
| Hard SMB/HML exposure constraints | None | None |

### Phase 4: Backtesting & Performance Attribution

**Notebook**: `HFS_Phase4_Task3_2_Backtest_Attribution.ipynb`

- Implements a simplified monthly proxy cost model:
  - commission: `5 bp` one-way
  - market-impact proxy: `5 bp` one-way
  - borrow cost: `50 bp` annualized on the short leg
- Converts the baseline gross backtest into net returns and net NAV.
- Runs monthly Fama-French 5-factor regressions on realized portfolio excess returns.
- Produces:
  - rolling 36-month alpha
  - rolling 12-month FF5 factor betas
  - drawdown and recovery analysis
  - long/short leg attribution
  - strategy vs SPY comparison charts

---

## Results

Snapshot from the saved output files:

- Baseline gross backtest (`data/sp500_backtest_results.csv.gz`)
  - annualized return: about `11.73%`
  - annualized volatility: about `18.21%`
  - Sharpe: about `0.644`
- Baseline net backtest (`data/sp500_phase4_full_report.csv.gz`)
  - annualized return: about `9.77%`
  - annualized volatility: about `18.22%`
  - Sharpe: about `0.536`
- Enhanced net backtest (`data/sp500_backtest_results_enhanced.csv.gz`)
  - annualized return: about `7.48%`
  - annualized volatility: about `13.99%`
  - Sharpe: about `0.535`
- Expected-return signal quality
  - next-month IC mean: about `0.025`
  - positive IC months: about `59%`

These performance figures should be interpreted in the context of the implemented monthly proxy backtest and cost model described above.

### Baseline Strategy Performance

![Strategy vs SPY](figures/strategy_vs_spy.png)
![NAV & Drawdown](figures/nav_drawdown.png)

### Enhanced Strategy Performance

![Enhanced Strategy vs SPY](figures_enhanced/strategy_vs_spy_enhanced.png)
![Enhanced NAV & Drawdown](figures_enhanced/nav_drawdown_enhanced.png)

### Factor Attribution

![Rolling Alpha](figures/rolling_alpha.png)
![Factor Exposures](figures/factor_exposures.png)

---

## Requirements

### Python Environment

```text
python >= 3.9
numpy
pandas
scipy
statsmodels
scikit-learn
cvxpy
matplotlib
seaborn
```

### Data Access

- A WRDS account is required to run `HFS_Project_Data_Fetcher_0304.ipynb`.
- Pre-processed data files are already included in `data/`, so downstream notebooks can run without WRDS access.

### Install Dependencies

```bash
pip install numpy pandas scipy statsmodels scikit-learn cvxpy matplotlib seaborn
```

or with conda:

```bash
conda install numpy pandas scipy statsmodels scikit-learn matplotlib seaborn
conda install -c conda-forge cvxpy
```

---

## Usage

Run the notebooks sequentially.

```bash
# 1. Data fetching (requires WRDS credentials)
jupyter notebook HFS_Project_Data_Fetcher_0304.ipynb

# 2. Factor engineering
jupyter notebook HFS_Phase2_Momentum.ipynb
jupyter notebook HFS_Phase2_Task2_2_CrossSectionalRegression.ipynb
jupyter notebook HFS_Phase2_Task2_3_FRP_and_ExpectedReturns.ipynb

# 3. Risk model and baseline optimization
jupyter notebook HFS_Phase3_Task3_1_PCA_RiskModel.ipynb
jupyter notebook HFS_Phase3_Task3_2_Optimizer.ipynb

# 4. Enhanced branch (optional)
jupyter notebook HFS_Phase3_Task3_3_Enhanced_Optimizer_Phase4.ipynb

# 5. Baseline Phase 4 attribution
jupyter notebook HFS_Phase4_Task3_2_Backtest_Attribution.ipynb
```

Notes:

- `HFS_Phase4_Task3_2_Backtest_Attribution.ipynb` analyzes the **baseline** branch and writes `data/sp500_phase4_full_report.csv.gz` plus `figures/`.
- `HFS_Phase3_Task3_3_Enhanced_Optimizer_Phase4.ipynb` generates the **enhanced** branch outputs and writes `figures_enhanced/`.
- If the data files already exist in `data/`, you can start from any downstream phase.

---

## License

This project is for academic and educational purposes. Data sourced from WRDS (CRSP and Compustat) and the Fama-French Data Library remains subject to their respective terms of use.
