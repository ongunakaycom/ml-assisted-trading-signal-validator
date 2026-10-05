# ML-Assisted Trading Signal Validator

A production-grade real-time signal engine for liquid markets. Detects 
accumulation phases before price expansion using deterministic multi-gate 
filtering, ML-based failure prediction, and LLM-based risk validation.

**Source code is private.** This repository documents the architecture, 
risk model, and engineering decisions behind the live system.

- **Exchange:** Coinbase Advanced Spot
- **Status:** ✅ Live in production
- **Performance:** 88% signal success · 14.75x profit factor · 991 signals

---

## 🎯 What it does

A multi-timeframe scanner that identifies accumulation phases in liquid 
markets before breakout, combining:

- Order-book microstructure analysis (13 detection methods)
- Multi-timeframe trend alignment (1D + 1H + 15m EMA200)
- Volume regime classification (dry-up / spike / accumulation)
- ML-based failure prediction (XGBoost, RandomForest, MLP)
- LLM risk validation (Gemini, 7-step checklist, fail-safe decline)

**Design principle:** Deterministic gate logic owns the decision. ML and 
LLM layers provide enrichment and veto. No model owns state.

---

## 📊 Live Performance

| Metric | Value |
|--------|-------|
| Total signals | 991 |
| Success rate | 88.0% |
| Profit factor | 14.75x |
| Avg profit per signal | +1.79% |
| Total profit | +1778.51% |
| Failure recall (XGBoost) | 94.4% (17/18) |
| Success specificity | 86.3% |
| ROC-AUC | 0.946 |

---

## 🧩 Tech Stack

| Layer | Technology |
|-------|-----------|
| Exchange | Coinbase Advanced Spot API |
| Language | Python 3.11 (asyncio) |
| ML | XGBoost, RandomForest, MLP (scikit-learn) |
| LLM | Google Gemini (approval + coin-specific commentary) |
| Notifications | Telegram Bot API |
| Storage | JSON + CSV (local, no DB) |

---

## 📚 Documentation

- `architecture.md` — Pipeline stages, gate logic, data flow
- `whale-engine.md` — 13 detection methods, direction taxonomy
- `risk-management.md` — Multi-timeframe rule, fail-safe, session memory
- `signal-quality.md` — Signal quality overhaul (public summary)
- `ml-pipeline.md` — Model metrics, failure detection, retraining cadence
- `performance.md` — Live stats, top instruments, weekly snapshots
- `engineering-notes.md` — Trade-offs, lessons learned

---

## 📜 License

MIT — see LICENSE.

**Author:** Ongun Akay
**Status:** ✅ Live in production