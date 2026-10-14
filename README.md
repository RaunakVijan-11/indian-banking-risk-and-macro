# What drives risk and relative value in Indian banks?

Volatility modelling, cointegration and a macro VAR, applied to one market (Indian banking, 2008 to Oct 2026) and tied together in a final synthesis. Built in Python (Google Colab) as an applied project for the Financial Econometrics module of the WQU MSc in Financial Engineering.

**Data:** Nifty Bank, Nifty 50, India VIX and five bank stocks (HDFC Bank, ICICI Bank, Axis Bank, Kotak Mahindra Bank, SBI), daily, from Yahoo Finance. Macro series (monthly): Brent, US 10-year yield and OECD India CPI from FRED, USD/INR from Yahoo Finance, and the RBI repo rate compiled from policy decisions.

![Nifty Bank EGARCH volatility vs India VIX](results/figures/01_garch_vol_vs_vix.png)

## Notebooks

| # | Notebook | Question | Open |
|---|---|---|---|
| 00 | Data and stylized facts | Is the data clean, and what does it look like statistically? | [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/RaunakVijan-11/indian-banking-risk-and-macro/blob/main/notebooks/00_data_and_stylized_facts.ipynb) |
| 01 | GARCH volatility | How does bank risk behave, and can it be forecast? | [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/RaunakVijan-11/indian-banking-risk-and-macro/blob/main/notebooks/01_garch_volatility.ipynb) |
| 02 | Cointegration and pairs | Is there stable relative value among the banks? | [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/RaunakVijan-11/indian-banking-risk-and-macro/blob/main/notebooks/02_cointegration_pairs.ipynb) |
| 03 | Macro VAR | Do macro shocks drive bank returns and volatility? | [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/RaunakVijan-11/indian-banking-risk-and-macro/blob/main/notebooks/03_macro_var_vecm.ipynb) |
| 04 | Synthesis | Does risk affect relative value? What is the overall picture? | [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/RaunakVijan-11/indian-banking-risk-and-macro/blob/main/notebooks/04_synthesis.ipynb) |

## Key findings

**00. Data and stylized facts.** Eight non-trading rows that Yahoo had padded were removed, and one stale day (18 Mar 2025) was set to missing for the stocks only. Nothing was forward-filled. Returns are fat-tailed (excess kurtosis 7 to 16), show strong volatility clustering, and have small but significant autocorrelation. Log prices are I(1).

**01. GARCH.** Nine models (GARCH, GJR, EGARCH, each with normal, Student-t and skewed-t errors) were fitted with an AR(1) mean. EGARCH(1,1) with Student-t errors is best by BIC (tail parameter about 6.3).
- Volatility is highly persistent (shock half-life about 71 trading days) and asymmetric: a negative shock raises next-day log-variance about three times as much as an equal positive one.
- Standardized residuals still show some structure in squared terms, and higher-order EGARCH does not fix it. FIGARCH has clean residuals but no leverage term and a much worse BIC.
- Out of sample (expanding window from 2015, one-day-ahead, QLIKE with Diebold-Mariano tests), GARCH, GJR and EGARCH are statistically indistinguishable. EWMA is not significantly worse, and a 250-day historical variance is clearly worse.
- Daily conditional volatility tracks India VIX in levels (correlation 0.87). Daily changes line up when the VIX is lagged one day (0.39), because the VIX contains same-day news that a GARCH forecast cannot.

![Pairs strategy backtest](results/figures/02_pairs_backtest.png)

**02. Cointegration and pairs trading.** Of the ten pairs, only HDFC Bank / Kotak is cointegrated (Engle-Granger p < 0.001 before the merger, survives Holm correction; hedge ratio about 0.95; half-life about 38 days). After the HDFC Bank-HDFC Ltd merger (July 2023) the relationship weakens (p = 0.86). A walk-forward backtest (re-selected every 63 days on a 756-day window; parameters frozen out of sample) gives a gross Sharpe of 0.19 and 0.11 at 10 bps per side, which is indistinguishable from zero.

**03. Macro VAR.** A VAR(2) on monthly stationary transforms (Feb 2008 to Mar 2025, 206 months) finds no reliable macro prediction of bank returns or volatility.
- Same-month co-movement with USD/INR and Brent depends on the Cholesky ordering. The 6-month USD/INR effect on bank returns is about -4% in one ordering and -0.7% in another.
- The repo-rate effect is null with an unstable sign.
- Falling bank prices raise later volatility in all specifications (the leverage effect again), with a weaker Granger result when volatility is measured independently of the model (p = 0.034).

**04. Synthesis.** The HDFC/Kotak spread reverts faster when EGARCH volatility is high (half-life about 29 days versus 77 to 86 otherwise; interaction test p = 0.003). The result survives starting in 2010 and excluding 2020. After the merger, reversion is much slower: the spread took 746 days to come back after crossing 2 standard deviations. Per-pair attribution shows two pairs earning more than the whole strategy, while eight lose money after passing the selection test.

![Synthesis overview](results/figures/04_synthesis_overview.png)

## What the evidence supports

| Question | Evidence | Strength |
|---|---|---|
| How does bank risk behave? | Clustered, persistent, fat-tailed, with leverage | Strong (in-sample) |
| Can GARCH forecast it better than simple rules? | GARCH variants indistinguishable; EWMA nearly as good | Strong null |
| Do macro shocks drive bank risk (monthly)? | No reliable prediction; contemporaneous links depend on ordering | Mostly null |
| Is there stable relative value among banks? | One pair, broken by the merger | Fragile |
| Does market risk affect relative value? | Spread reverts faster in high-volatility regimes | Moderate |

## Limitations

- **Pair choice:** HDFC/Kotak was highlighted using the full-sample screen in notebook 02, so the regime analysis in notebook 04 describes one pair selected with hindsight.
- **Multiple testing:** the backtest selects pairs at the 5% level without correction, which admits false positives. P&L is concentrated in a few pairs and years.
- **Regime thresholds** are full-sample terciles and are descriptive, not a tradable signal.
- **Shorting:** the backtest assumes the short leg is available. In India cash-market shorting is intraday only, so a real implementation would use futures.
- **Macro sample** ends in March 2025, where the OECD India CPI series stops (the Indian CPI was rebased to 2024=100 in Feb 2026). The OECD series may mix index definitions around 2011 (not verified).
- **Repo rate** is a hand-compiled step series of policy decisions. It does not capture surprises, and it misses the July 2013 emergency tightening, which ran through the MSF rate.
- **Monthly VAR:** 7 variables on 206 observations, orthogonalised impulse responses are ordering-dependent, and residuals are non-normal (so bootstrapped bands are used).
- **Merger comparison** is a pre/post split, not a formal structural-break test (a Gregory-Hansen test would be the next step).
- **Data:** index data from Yahoo Finance has gaps; those are handled explicitly in notebook 00.

## How to run

Open any notebook with the Colab badge above. Notebooks read the cleaned data in `data/processed/`, so they can be run in any order after notebook 00 has produced it (the repository already includes the outputs).

The first cell of each notebook was written for my own workflow and uses a GitHub token stored in Colab Secrets, so it will fail for you. To run read-only, replace it with:

```python
!pip install -q arch yfinance
!git clone -q https://github.com/RaunakVijan-11/indian-banking-risk-and-macro.git
%cd indian-banking-risk-and-macro
```

and skip the final `git add/commit/push` cells. To run locally: `pip install -r requirements.txt` and open the notebooks in Jupyter. The `data/raw/` folder is intentionally empty (raw downloads are gitignored), and notebook 00 re-creates it.

Tested on Google Colab. Package versions are in `requirements.txt`.

## Repository structure

```
notebooks/          00 to 04, run with outputs saved
data/processed/     cleaned prices, log returns, EGARCH volatility, monthly macro data
results/            summary tables (CSV)
results/figures/    charts used above
requirements.txt
```

## Author

Raunak Vijan. Questions or suggestions are welcome via GitHub issues.

## License

MIT
