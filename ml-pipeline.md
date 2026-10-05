# ML Pipeline

Machine learning is used for **failure prediction**, not signal 
generation. The ML layer answers one question: "Given this signal, 
what is the probability it will fail?"

Models are trained offline and deployed as a gate on the live 
signal path.

---

## Models

Three models are trained and evaluated in parallel. The best 
performer is selected for production.

| Model | Role |
|-------|------|
| XGBoost | Primary production model |
| RandomForest | Ensemble alternative |
| MLP | Neural baseline |

All three consume the same feature set and target the same label.

---

## Target Label

The label is **`signal_failed`** — a binary flag computed from 
historical signal outcomes.

A signal is labelled as failed if it did not reach its target 
within the expected holding window. The definition of failure is 
locked at training time and does not change between runs.

---

## Live Performance (held-out test set)

| Model | Accuracy | ROC-AUC | PR-AUC | Brier | Failure Recall | Failure Precision | Success Specificity |
|-------|----------|---------|--------|-------|----------------|-------------------|---------------------|
| XGBoost | 0.872 | 0.946 | 0.672 | 0.079 | 0.944 | 0.486 | 0.863 |
| RandomForest | 0.852 | 0.934 | 0.689 | 0.062 | 0.944 | 0.447 | 0.840 |
| MLP | 0.785 | 0.889 | 0.658 | 0.072 | 0.833 | 0.341 | 0.779 |

**XGBoost is deployed.** It catches 17 of 18 failures on the 
held-out set while keeping success specificity above 0.85.

---

## Confusion Matrix (XGBoost)

    TN = 113   FP =  18
    FN =   1   TP =  17

- **TP (17)** — correctly flagged as failed
- **FN (1)** — failed signals that were not caught
- **FP (18)** — good signals that were unnecessarily blocked
- **TN (113)** — good signals correctly allowed

---

## Deployment

The deployed model runs **before** the LLM validator. It is a 
hard gate: if the model predicts failure above a fixed threshold, 
the signal is dropped.

The threshold is tuned for **high failure recall** — the cost of 
sending a bad signal is higher than the cost of missing a good one.

---

## Retraining Cadence

Models are retrained on a fixed cadence using the accumulated 
live signal outcomes. The training pipeline is offline and 
separate from the live bot.

Retraining is a manual, operator-triggered event. Model artifacts 
are versioned and never overwritten in place.

---

## Why ML Is Not Used for Signal Generation

Signal generation is deterministic. It must be:

- Reproducible
- Auditable
- Testable without retraining

Signal *filtering* is where ML excels, because the problem is 
statistical (predicting failure from a rich feature set), not 
structural.

Combining the two — deterministic generation + ML filtering — 
gives the best of both worlds.

---

## Feature Categories (Public)

The exact feature list is private. High-level categories include:

- Signal metadata (gate, timeframe alignment, confidence)
- Order-book state at signal time
- Trade-flow state at signal time
- Volume regime classification
- Multi-timeframe trend alignment flags
- Macro stage classification
- Time-of-day and session features

---

## What Is Not Public

- The full feature vector
- Feature engineering code
- Hyperparameters
- Training data composition
- Exact failure threshold
- Model artifacts