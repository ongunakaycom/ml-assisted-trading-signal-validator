# Performance

Live performance of the deployed system.

All numbers are from real, live signals. No backtests, no 
simulations.

---

## Aggregate Metrics

| Metric | Value |
|--------|-------|
| Total signals | 991 |
| Successful signals | 872 |
| Failed signals | 119 |
| Success rate | 88.0% |
| Average profit per signal | +1.79% |
| Total profit | +1778.51% |
| Profit factor | 14.75x |

**Profit factor** is total profit divided by total loss. A value 
above 2.0 is generally considered strong. A value above 10.0 is 
exceptional.

---

## Failure Detection (Live)

The ML layer flags signals likely to fail. Live performance on 
the held-out test set:

| Metric | Value |
|--------|-------|
| Failure recall (XGBoost) | 94.4% (17/18 caught) |
| Failure precision | 48.6% |
| Success specificity | 86.3% |
| ROC-AUC | 0.946 |

The system is tuned for **high recall** on failures. Missing a 
failed signal is worse than blocking a good one.

---

## Top-Performing Instruments

Instruments with 100% success rate over their sample:

| Instrument | Signals | Average Profit |
|------------|---------|----------------|
| AAVE | 9 | +1.65% |
| ADA | 8 | +3.01% |
| AERO | 4 | +3.11% |
| AIGENSYN | 3 | +1.50% |
| ALLO | 13 | +1.84% |

Small sample sizes are directional, not statistically significant. 
Consistency across instruments matters more than any single result.

---

## LLM Layer

The LLM validator operates as a **selective risk manager**. On the 
most recent batch of candidate signals, the model assigned medium 
failure probabilities to all of them, and declined the entire 
batch.

This is expected behavior. The validator is conservative by 
design — it would rather decline ten acceptable signals than 
accept one bad one.

---

## Why These Numbers Are Trustworthy

- **Live only.** No backtests, no simulations, no curve fitting.
- **Same pipeline.** The numbers come from the same code path 
  that runs in production.
- **Published methodology.** Success/failure labels, holding 
  window, and target definition are locked and documented.
- **No cherry-picking.** All signals are counted, including 
  declined and blocked ones.

---

## Update Cadence

This file is updated periodically. The last update date is 
recorded at the bottom.

**Last updated:** 2026-10-05