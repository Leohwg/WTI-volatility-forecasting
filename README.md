# Forecasting WTI Crude Oil Futures Returns and Volatility: An ARMA-GARCH Approach with Multivariate Analysis

Econometric research project modeling and forecasting daily returns and conditional variance of **NYMEX West Texas Intermediate (WTI)** crude oil futures across the 2022–2026 geopolitical shock regime.

Developed for the **Econometrics for Credit & Market Risk** course within the M.Sc. in Computational Finance at the **University ofall'Università degli Studi di Padova** (Supervisor: Prof. Angelica Gianfreda).

📄 **[Read the Full Paper (PDF)](./WTI_Volatility_Forecasting_Paper.pdf)**

---

## Overview & Dataset

Energy markets exhibit severe non-normality, volatility clustering, and asymmetric responses to geopolitical and macroeconomic shocks. This study develops a three-stage econometric framework to generate one-step-ahead rolling forecasts of WTI futures log-returns and conditional variance.

* **Sample Period:** July 2022 – April 2026 ($T = 1,002$ daily trading observations, 80/20 train-test split).
* **Synchronization Protocol:** Multi-exchange alignment (NYMEX, ICE, COMEX, NYSE Arca, CBOE) using Last Observation Carried Forward (LOCF) to prevent look-ahead bias and preserve empirical lead-lag structures.
* **Multivariate Candidate Pool (11 Series):**
  * *Endogenous Target:* `WTI` (NYMEX WTI Crude Oil Futures)
  * *Energy & Refining:* `RBOB` (Gasoline Futures), `NYFM6` (Heating Oil Futures), `XLE` (Energy Select Sector SPDR ETF), `CFI2M6` (ICE EUA Carbon Emissions Futures)
  * *Macro, FX & Industrial:* `USD` (U.S. Dollar Index Futures), `BDI` (Baltic Dry Index), `HG` (COMEX Copper Futures), `GC` (COMEX Gold Futures), `VIX` (CBOE Volatility Index), `ESM26` (E-mini S&P 500 Futures)

### Empirical Stylized Facts (WTI Daily Percentage Log-Returns)
| Mean | Median | Std. Dev. | Min | Max | Skewness | Excess Kurtosis | ADF Stat ($p$-val) | ARCH-LM(15) ($p$-val) |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| -0.0136% | 0.0840% | 2.3636% | -17.93% | +11.52% | -0.8127 | 9.7078 | -8.610 ($0.010$) | 204.68 ($<0.001$) |

---

## Econometric Methodology

1. **Univariate Conditional Mean (`ARMA`):** Identification of a parsimonious, custom-restricted $\text{ARMA}(6,6)$ specification retaining only statistically significant lags ($\text{AR}(2), \text{AR}(6), \text{MA}(2), \text{MA}(6)$) with zero intermediate constraints to prevent over-parameterization.
2. **Multivariate Lead-Lag Screening (`CCF` & `VAR`):** Cross-Correlation Function analysis ($\text{lag } -10$ to $+10$), pairwise Granger causality, and a restricted $\text{VAR}(2)$ system isolate once-lagged Energy equities ($\text{XLE}_{t-1}$), once-lagged Copper ($\text{HG}_{t-1}$), and twice-lagged U.S. Dollar Index returns ($\text{USD}_{t-2}$) as statistically significant leading indicators, while discarding contemporaneous crack-spread co-movers (`RBOB`, `NYFM6`, `ESM26`).
3. **Conditional Volatility (`GARCH` Family):** Benchmarking 8 competing specifications crossing univariate `ARMA` (M1–M4) and multivariate `ARMAX` (M5–M8) mean equations with four variance structures (**sGARCH**, **eGARCH**, **GJR-GARCH**, **sGARCH-M**) under **Skewed Student-$t$ (`sstd`)** innovations.
4. **Out-of-Sample Validation:** Recursive rolling 1-step-ahead forecasts over the final 20% test window ($n_{\text{test}} = 200$) with periodic refitting, evaluated via $\text{RMSE}$, $\text{MAE}$, **Patton's QLIKE** loss function, and the **Diebold-Mariano (DM)** test.

---

## Key Empirical Results

### 1. In-Sample Diagnostics & Variance Misspecification Test
A critical finding of the study is that **all four purely univariate GARCH models (M1–M4) fail the residual variance diagnostics** (Weighted Ljung-Box on squared standardized residuals and ARCH-LM tests at lags 5 and 7), whereas **all four multivariate ARMAX-GARCH models (M5–M8) pass every variance and stability diagnostic cleanly**:

| Model | Specification | AIC | BIC | LogLik | $\text{LB}^2[9]$ $p$-val | $\text{ARCH}[5]$ $p$-val | $\text{ARCH}[7]$ $p$-val | Sign Bias $p$-val | Nyblom Joint |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **M1** | $\text{ARMA}-\text{sGARCH}(1,1)$ | 4.2367 | 4.2895 | -1681.46 | 0.020 ❌ | 0.004 ❌ | 0.003 ❌ | 0.315 | 1.259 |
| **M2** | $\text{ARMA}-\text{eGARCH}(1,1)$ | 4.2422 | 4.3008 | -1682.62 | 0.010 ❌ | 0.006 ❌ | 0.004 ❌ | 0.548 | 1.673 |
| **M3** | $\text{ARMA}-\text{GJR-GARCH}(1,1)$ | 4.2389 | 4.2976 | -1681.32 | 0.022 ❌ | 0.004 ❌ | 0.004 ❌ | 0.378 | 1.794 |
| **M4** | $\text{ARMA}-\text{sGARCH-M}(1,1)$ | 4.2390 | 4.2977 | -1681.35 | 0.018 ❌ | 0.003 ❌ | 0.003 ❌ | 0.297 | 1.295 |
| **M5** | $\text{ARMAX}-\text{sGARCH}(1,1)$ | 4.2193 | 4.2842 | -1665.02 | 0.754 ✅ | 0.387 ✅ | 0.496 ✅ | 0.347 | 1.846 |
| **M6** | $\textbf{ARMAX}-\textbf{eGARCH(1,1)}$ | **4.2136** | **4.2957** | **-1666.28** | **0.183** ✅ | **0.172** ✅ | **0.370** ✅ | **0.528** | **2.111** |
| **M7** | $\text{ARMAX}-\text{GJR-GARCH}(1,1)$ | 4.2160 | 4.2924 | -1664.97 | 0.733 ✅ | 0.399 ✅ | 0.760 ✅ | 0.340 | 2.256 |
| **M8** | $\text{ARMAX}-\text{sGARCH-M}(1,1)$ | 4.2160 | 4.2924 | -1664.95 | 0.726 ✅ | 0.353 ✅ | 0.063 ✅ | 0.460 | 1.881 |

### 2. Selected Specification: $\text{M6} = \text{ARMAX}(6,6)\text{-EGARCH}(1,1)\text{-sstd}$
Model **M6** is selected as the optimal specification, achieving the lowest AIC (`4.2136`), clean residual diagnostics, excellent Pearson goodness-of-fit ($p \in [0.72, 0.999]$),jointly significant exogenous regressors under robust standard errors ($\hat{\beta}_{\text{XLE}} = 0.1050, p = 0.0255$; $\hat{\beta}_{\text{USD}} = 0.3402, p = 0.0032$), high volatility persistence ($\hat{\beta}_1 = 0.9791, p < 0.0001$), and a strongly significant asymmetric leverage effect ($\hat{\gamma}_1 = 0.0962, p < 0.0001$).

---

## Authors

* **Marco Battiston**
* **Michele Beltrame**
* **Lorenzo Palmieri**
* **Claudia Rocas Corrons**
* **Leonardo Zamprotta**
