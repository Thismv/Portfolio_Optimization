# Portfolio Optimization & Factor Model Analysis

Fund returns do not exist in a vacuum. Before allocating capital, a risk-aware 
investor needs to know two things: where a fund's returns actually come from, 
and how a portfolio of those funds behaves under realistic constraints. This 
project addresses both.

Using three distinct fund universes — U.S. mutual funds, smart beta ETFs, and 
hedge fund indices — the analysis first decomposes each fund's returns using the 
Fama-French five-factor model plus momentum, separating genuine alpha from 
compensated factor exposure. It then constructs mean-variance optimized portfolios 
under six constraint regimes and evaluates how well in-sample results hold up 
out-of-sample.

The findings carry a practical message: unconstrained optimization overfits. 
Portfolios that look best in-sample — maximum Sharpe, long-short, no guardrails — 
consistently underperform their constrained counterparts out-of-sample. Minimum 
variance and tracking-error-bounded strategies transfer better precisely because 
they make fewer assumptions about expected returns, which are notoriously difficult 
to forecast. This trade-off between optimization sophistication and generalizability 
is central to real-world portfolio construction.

This project covers factor model regression and alpha decomposition, constrained 
mean-variance optimization using scipy, in-sample vs. out-of-sample performance 
evaluation, and systematic data quality analysis across three asset classes spanning 
up to 34 years of monthly returns.

---

## Data

- **Fama-French factors + momentum (UMD):** sourced from [Kenneth R. French's Data Library](http://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html)
- **Mutual funds (10 U.S. funds):** monthly returns, January 2001 to December 2024
- **Smart beta ETFs (10 funds):** monthly returns, January 2010 to December 2024
- **Hedge fund indices (10 HFRI indices):** monthly returns, January 1991 to December 2024

Fund return data is included in `fund_returns_factors.xlsx` with permission.
Fama-French factor data is publicly available and redistributed here for reproducibility.

---

## Part I: Factor Model Performance Evaluation

Each fund's excess returns are regressed on six factors: market excess return (MKTRF), 
size (SMB), value (HML), momentum (UMD), profitability (RMW), and investment (CMA). 
Alpha and factor loadings are reported with t-statistics. Funds with statistically 
significant positive alpha (t > 2) are flagged.

### Key findings

**Mutual funds** (sample: 2001–2024): Four of ten funds generated significant positive 
alpha — FSELX (semiconductors), INPIX (leveraged internet), SLMCX (communications), 
and NASDX (Nasdaq-100). All four are concentrated technology and internet-sector funds 
that benefited from the prolonged tech-driven bull market. Their outperformance reflects 
sector concentration rather than broad stock-picking skill. The remaining six funds 
produced alphas statistically indistinguishable from zero, consistent with market 
efficiency in diversified active management.

**Smart beta ETFs** (sample: 2010–2024): Zero funds with significant positive alpha. 
R² values range from 0.83 to 0.97, confirming that the six-factor model fully explains 
smart beta returns. These funds are transparent factor portfolios by construction — 
no residual skill component is expected or found.

**Hedge fund indices** (sample: 1991–2024): Eight of ten indices generated significant 
positive alpha, with the strongest results in Distressed Securities (t = 6.17), 
Merger Arbitrage (t = 5.64), and Equity Hedge (t = 4.84). Low market betas 
(ranging from 0.09 to 0.57) confirm effective reduction of systematic market exposure. 
The persistent alpha is consistent with the literature on hedge fund return 
non-linearity — strategies such as merger arbitrage and short selling have 
option-like payoff profiles not captured by linear factor models.

---

## Part II: Portfolio Optimization

Six portfolio strategies are constructed and evaluated for each fund universe:

| Strategy | Description |
|---|---|
| Long-Only GMV | Global minimum variance, no short selling |
| Long-Only MVP | Maximum Sharpe ratio, no short selling |
| Long-Short GMV | Global minimum variance, short selling allowed |
| Long-Short MVP | Maximum Sharpe ratio, short selling allowed |
| TE-Constrained MVP | Maximum Sharpe ratio, annual tracking error ≤ 5% vs equal-weighted benchmark |
| Factor-Neutral MVP | Minimum variance portfolio with all six factor exposures constrained to zero, weight limit ±200% |

All optimization inputs (mean returns, covariance matrix, factor betas) are estimated 
from the in-sample period (January 2010 to December 2019). Portfolios are then evaluated 
on the out-of-sample period (January 2020 to December 2024) without reestimation. 
All reported statistics are annualized.

### Key findings

**In-sample**, unconstrained long-short portfolios achieved the highest Sharpe ratios 
across all three asset classes — mutual funds (1.27), smart beta ETFs (1.89), and hedge 
fund indices (1.85). Constrained portfolios produced lower but more realistic in-sample 
Sharpe ratios.

**Out-of-sample**, performance deteriorated across the board — a well-documented 
consequence of estimation error in sample means and covariance matrices. The most 
consistent pattern: GMV and TE-constrained portfolios generalized better out-of-sample 
than unconstrained maximum Sharpe portfolios, because they rely less on precise mean 
return estimates which are notoriously difficult to forecast.

**Hedge fund indices** showed the most stable transfer from in-sample to out-of-sample, 
reflecting the lower volatility and more diversified nature of these indices relative 
to equity-focused fund universes.

**The factor-neutral portfolio** for mutual funds hit the ±200% weight bounds on 
multiple assets (FSELX at +200%, INPIX at -200%), exposing a structural limitation: 
simultaneously zeroing out six correlated factor exposures across only ten funds is 
an over-constrained problem. The optimizer finds a solution that satisfies the 
constraints but concentrates heavily in a small number of positions. For hedge fund 
indices, the TE constraint was non-binding — the unconstrained MVP naturally produced 
a tracking error below 5%, so the constrained and unconstrained solutions are identical.

---

## Data Quality

The source data contained one error requiring correction: FSPHX (Fidelity Select Health 
Care) had a return of 827% recorded for August 2015, caused by a price level being 
stored instead of a monthly return. The error was identified through a systematic 
outlier scan and corrected using verified returns from Yahoo Finance. All data 
cleaning steps are documented in the notebook.

A second flagged observation — INPIX returning -50.9% in February 2000 — was retained 
after investigation. The broad market fell 10% that month, internet-sector peers fell 
20-30%, and INPIX carries approximately 2x leverage on the internet index. The return 
is economically consistent and was not modified.

---

## Stack

- Python 3.12
- pandas, numpy, statsmodels, scipy
- Jupyter Notebook

---

## Attribution

Factor data: Kenneth R. French Data Library  
Hedge fund index data: HFR (Hedge Fund Research) — HFRI Family Indices  
Fund return data used with permission for academic and portfolio demonstration purposes  
Project completed as part of the Master of Financial Risk Management (MFRM) program, 
Rotman School of Management, University of Toronto

---
