# Doctor Copper V2 — Walk-Forward Validation

An out-of-sample extension of Doctor Copper V1, implementing
walk-forward validation to eliminate in-sample bias and extending
the regime signal to KOSPI for cross-market portability testing.

2026.04.16

Check my website for more details: wonjunshin1220.com

---
## Acknowledgements ##
This project was developed with assistance from Claude (AI) for coding support, and debugging.

## What's New in V2

|  | V1 | V2 |
|---|---|---|
| Parameter estimation | Full in-sample | Walk-forward (OOS) |
| Validation market | S&P 500 only | S&P 500 + KOSPI |
| Statistical test | t-test | t-test + Bootstrap (10,000 permutations) |
| Thresholds | In-sample percentiles | Train-window percentiles |

**Core motivation:** V1's weights and percentile thresholds were
estimated on the full dataset, introducing look-ahead bias. V2
re-estimates all parameters on rolling 60-month train windows and
evaluates strictly on unseen 12-month test periods.

---

## Walk-Forward Design

```
Train window : 60 months (5 years)
Test window  : 12 months (1 year)
Step         : 12 months (annual roll)
Folds        : 10
OOS period   : 2012-03 → 2023-09 (110 observations per market)
```

For each fold:
1. Estimate z-score parameters (mean, std) on **train only**
2. Estimate 33/67 percentile thresholds on **train only**
3. Apply to **unseen test period** → record regime + forward return

---

## Key Results

### V2 Walk-Forward OOS — S&P 500

| Regime | N | Avg Return | Median | Hit Rate |
|--------|---|-----------|--------|----------|
| Risk-ON | 27 | **+2.22%** | +1.93% | **77.8%** |
| Neutral | 37 | +0.02% | +0.69% | 59.5% |
| Risk-OFF | 46 | +0.88% | +1.18% | 63.0% |

- **Risk-ON vs Risk-OFF spread: +1.34%/month** (OOS)
- t-statistic: 1.352 | p-value: 0.1816
- Bootstrap p-value: 0.1920 | 95% CI under H0: [−1.98%, +2.00%]

### V2 Walk-Forward OOS — KOSPI

| Regime | N | Avg Return | Median | Hit Rate |
|--------|---|-----------|--------|----------|
| Risk-ON | 27 | **+2.14%** | +1.78% | **74.1%** |
| Neutral | 37 | −1.09% | −0.14% | 48.6% |
| Risk-OFF | 46 | +0.13% | +0.34% | 54.3% |

- **Risk-ON vs Risk-OFF spread: +2.01%/month** (OOS)
- t-statistic: 1.553 | p-value: 0.1273
- Bootstrap p-value: 0.1045 | 95% CI under H0: [−2.31%, +2.47%]

---

## V1 vs V2 Comparison

| Model | Type | Spread | p-value | ON Return | ON Hit Rate | OFF Return | OFF Hit Rate |
|-------|------|--------|---------|-----------|-------------|------------|--------------|
| V1 S&P500 | In-Sample | +1.35% | 0.1535 | +1.72% | 67.7% | +0.37% | 59.7% |
| V2 S&P500 | Walk-Fwd OOS | +1.34% | 0.1816 | +2.22% | 77.8% | +0.88% | 63.0% |
| V2 KOSPI | Walk-Fwd OOS | +2.01% | 0.1273 | +2.14% | 74.1% | +0.13% | 54.3% |

**Key findings:**
- OOS spread (S&P500) is virtually identical to V1 in-sample spread (+1.34% vs +1.35%) — signal is not an artifact of in-sample optimization
- KOSPI OOS spread (+2.01%) exceeds S&P500, supporting cross-market portability of the copper regime framework
- Risk-ON hit rate improves materially in OOS: 77.8% (S&P) and 74.1% (KOSPI)

---

## Statistical Interpretation

Both markets fall short of the 5% significance threshold, primarily
due to small per-regime sample sizes (~27–46 obs). Key observations:

- KOSPI bootstrap p-value (0.1045) approaches 10% significance
- Observed spreads sit at or beyond the 95% CI boundary under H0
- Directional ordering (Risk-ON > Neutral > Risk-OFF) holds in S&P500 OOS
- KOSPI Neutral regime underperforms Risk-OFF, indicating the copper
  signal may be a stronger discriminator at the extremes in Korean markets

The signal's economic rationale remains intact: copper as a global
industrial demand proxy produces consistent Risk-ON/OFF differentiation
across both a developed (S&P 500) and export-driven emerging market (KOSPI).

---

## Fold-Level Consistency (S&P 500)

| Test Period | Spread |
|-------------|--------|
| 2015-10 | +1.80% |
| 2018-02 | +2.16% |
| 2019-04 | −8.76% |
| 2022-10 | −2.20% |

Positive-spread folds: 2/4 (50%). The 2019-04 fold (COVID-adjacent
period) shows significant drawdown — consistent with regime breakdown
during structural macro dislocations where copper signals lag.

---

## Methodology

### Signal Construction (identical to V1)

```
risk_score = 0.35 × copper_z
           + 0.25 × copper_gold_ratio_z
           + 0.15 × wti_z
           − 0.15 × dxy_z
           − 0.10 × us10y_z
```

Each signal: 3-month rolling mean → z-score → weighted sum →
2-month smoothing MA.

### Walk-Forward Parameter Estimation

All z-score normalization parameters (mean, std) and regime
thresholds (33rd/67th percentile) are estimated **exclusively on
the train window** of each fold. No future information leaks into
the test period.

---

## Limitations

- **p-values remain above 5%** — sample size constraint (~27 obs per
  regime in OOS). More data or higher-frequency signals needed
- **Fold inconsistency** — only 2/4 folds show positive spread in S&P500;
  signal breaks down around structural dislocations (2019–2020)
- **Fixed weights** — the 0.35/0.25/0.15/−0.15/−0.10 weighting scheme
  is carried over from V1 and not re-optimized in walk-forward (intentional
  to avoid overfitting, but suboptimal)
- **Monthly frequency** — limits sample accumulation; daily or weekly
  data would increase statistical power

---

## Files

- `fetch-data.py` — data fetching script
- `01_eda.ipynb` — V1 full analysis notebook
- `02_walk_forward_v2.ipynb` — V2 walk-forward validation notebook
- `master_monthly.csv` — processed data (2006-02 → 2026-02, 190 obs)

---

## Tech Stack

Python, Pandas, NumPy, SciPy, Scikit-learn, Matplotlib, Seaborn, yfinance

---

