# ML-Assisted Trading Signal Validator

[![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)](https://python.org)
[![XGBoost](https://img.shields.io/badge/XGBoost-ML-EB5B3E?logo=xgboost&logoColor=white)](https://xgboost.readthedocs.io)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-F7931E?logo=scikit-learn&logoColor=white)](https://scikit-learn.org)
[![Gemini](https://img.shields.io/badge/Gemini-LLM-4285F4?logo=google&logoColor=white)](https://deepmind.google/technologies/gemini)
[![Coinbase](https://img.shields.io/badge/Coinbase-Advanced-0052FF?logo=coinbase&logoColor=white)](https://coinbase.com)
[![Telegram](https://img.shields.io/badge/Telegram-Bot-26A5E4?logo=telegram&logoColor=white)](https://core.telegram.org/bots)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

> A production-grade real-time signal engine for liquid markets. Detects accumulation phases before price expansion using deterministic multi-gate filtering, ML-based failure prediction, and LLM-based risk validation.

- **Exchange:** Coinbase Advanced Spot
- **Status:** ✅ Live in production
- **Performance:** 88% signal success · 14.75x profit factor · 991 signals
- **Source code:** 🔒 Private (architecture documented here)

---

## 🔗 Live

| Environment | URL |
|-------------|-----|
| **Source Code** | 🔒 Private Repo |
| **Documentation** | This repository |

---

## 🎯 What it does

A multi-timeframe scanner that identifies accumulation phases in liquid markets before breakout, combining:

- **Order-book microstructure analysis** (13 detection methods)
- **Multi-timeframe trend alignment** (1D + 1H + 15m EMA200)
- **Volume regime classification** (dry-up / spike / accumulation)
- **ML-based failure prediction** (XGBoost, RandomForest, MLP)
- **LLM risk validation** (Gemini, 7-step checklist, fail-safe decline)

**Design principle:** Deterministic gate logic owns the decision. ML and LLM layers provide enrichment and veto. No model owns state.

---

## 🏛️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  Coinbase Advanced Spot API                     │
│                  (real-time market data feed)                   │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│              Multi-Timeframe Scanner (Python 3.11)              │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Whale Engine (13 detection methods)                     │   │
│  │  ├── Order-book imbalance                                │   │
│  │  ├── Iceberg detection                                   │   │
│  │  ├── Spoofing detection                                  │   │
│  │  ├── Absorption patterns                                 │   │
│  │  └── ... (9 more)                                        │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Trend Alignment Gate                                    │   │
│  │  └── 1D + 1H + 15m EMA200 confluence                     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Volume Regime Classifier                                │   │
│  │  └── dry-up / spike / accumulation                       │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Deterministic Gate Logic  ← OWNS THE DECISION           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ML Failure Predictor (XGBoost / RF / MLP)               │   │
│  │  └── Enrichment + veto                                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  LLM Risk Validator (Gemini)                             │   │
│  │  └── 7-step checklist + fail-safe decline                │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│              Telegram Bot API (notifications)                   │
└─────────────────────────────────────────────────────────────────┘
```

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

| Layer | Technology | Engineering Decision |
|-------|-----------|---------------------|
| Exchange | Coinbase Advanced Spot API | Deep liquidity, reliable REST + WebSocket |
| Language | Python 3.11 (asyncio) | Async-native for concurrent market streams |
| ML | XGBoost, RandomForest, MLP (scikit-learn) | Ensemble for failure prediction |
| LLM | Google Gemini | Approval gate + coin-specific commentary |
| Notifications | Telegram Bot API | Real-time alerts, low latency |
| Storage | JSON + CSV (local, no DB) | Zero-ops, file-based persistence |

---

## 📚 Documentation

| Document | Contents |
|----------|----------|
| [architecture.md](architecture.md) | Pipeline stages, gate logic, data flow, state model, failure modes |
| [whale-engine.md](whale-engine.md) | 13 detection methods, direction taxonomy, aggregation logic |
| [risk-management.md](risk-management.md) | Multi-timeframe rule, fail-safe defaults, session memory, read-only config |
| [signal-quality.md](signal-quality.md) | The 10-fix overhaul that stabilised signal quality |
| [ml-pipeline.md](ml-pipeline.md) | Model metrics, failure detection, retraining cadence |
| [performance.md](performance.md) | Live stats, top instruments, weekly snapshots |
| [engineering-notes.md](engineering-notes.md) | Trade-offs, lessons learned, stack summary |

---

## 🚀 Key Features

### 1. Deterministic Multi-Gate Filtering

Every signal must pass through a cascade of deterministic gates before any ML or LLM layer sees it. Gates enforce:

- **Trend alignment** — 1D, 1H, and 15m EMA200 must agree on direction
- **Volume regime** — dry-up (accumulation) or spike (confirmation)
- **Microstructure** — order-book imbalance and absorption patterns
- **Liquidity** — minimum depth and spread requirements

### 2. Whale Engine (13 Detection Methods)

Order-book microstructure analysis that detects accumulation footprints:

- Iceberg orders (hidden size behind small visible size)
- Spoofing (large orders that vanish before execution)
- Absorption (price holds despite heavy selling)
- Imbalance streaks (persistent bid/ask skew)
- ...and 9 more methods, aggregated into a direction taxonomy

### 3. ML Failure Predictor

Three models trained on historical signal outcomes:

| Model | Role |
|-------|------|
| XGBoost | Primary failure classifier (94.4% recall) |
| RandomForest | Ensemble diversity |
| MLP | Non-linear pattern capture |

Models **never own state** — they only enrich or veto a signal that already passed deterministic gates.

### 4. LLM Risk Validation (Gemini)

A 7-step checklist prompt validates each signal before execution:

- Macro context (BTC dominance, funding rates, news)
- Coin-specific risk factors
- Fail-safe default: **decline** if uncertain

### 5. Real-Time Telegram Alerts

Every approved signal is pushed to Telegram with:
- Instrument, direction, entry
- Stop-loss and take-profit levels
- ML confidence score
- LLM commentary

---

## 🧩 Signal ↔ Execution Contract

| Stage | Owner | Can Veto? |
|-------|-------|-----------|
| Trend alignment | Deterministic gates | ✅ Yes |
| Volume regime | Deterministic gates | ✅ Yes |
| Whale engine | Deterministic gates | ✅ Yes |
| ML failure prediction | ML ensemble | ✅ Yes |
| LLM risk validation | Gemini | ✅ Yes (fail-safe decline) |
| Final signal | Gate logic | ✅ Owns state |

---

## 🗂️ Repository Structure

```
.
├── README.md                          # This file
├── architecture.md                    # Pipeline + gate logic
├── whale-engine.md                    # 13 detection methods
├── risk-management.md                 # Multi-timeframe + fail-safe
├── signal-quality.md                  # The 10-fix overhaul
├── ml-pipeline.md                     # Model metrics + retraining
├── performance.md                     # Live stats + snapshots
├── engineering-notes.md               # Trade-offs + lessons
└── LICENSE
```

> **Note:** Implementation, scoring weights, ML model artifacts, and LLM system prompt are kept in a private repo to protect the trading edge.

---

## 📈 Engineering Metrics

### Signal Quality

| Metric | Before 10-fix | After 10-fix |
|--------|---------------|--------------|
| Success rate | ~60% | **88.0%** |
| Profit factor | ~2x | **14.75x** |
| False positives | High | **Low** |
| Failure recall | — | **94.4%** |
| ROC-AUC | — | **0.946** |

### Design Trade-offs

| Decision | Trade-off |
|----------|-----------|
| Deterministic gates own state | Slower iteration, but no model drift risk |
| ML as veto only | Misses some ML-only wins, but prevents catastrophic failures |
| LLM fail-safe decline | Fewer signals, but higher quality |
| JSON + CSV storage | No DB ops, but no concurrent writes |
| Telegram notifications | Fast delivery, but no audit log |

---

## 🔐 Security Highlights

- **No API keys in repo** — all secrets via environment variables
- **Read-only config** — trading parameters are immutable at runtime
- **Fail-safe defaults** — LLM declines on uncertainty
- **Session memory** — prevents duplicate signals on same instrument
- **Private source** — architecture public, implementation private

---

## 🛣️ Roadmap

- [x] Coinbase Advanced Spot integration
- [x] 13-method whale engine
- [x] Multi-timeframe trend alignment
- [x] XGBoost failure predictor
- [x] Gemini LLM risk validation
- [x] Telegram alerts
- [x] 88% success rate · 14.75x profit factor
- [ ] Multi-exchange support (Binance, Kraken)
- [ ] Backtesting framework
- [ ] Web dashboard for signal history
- [ ] On-chain data integration

---

## 👤 Author

**Ongun Akay** — Senior Cloud & AI Engineer

🌐 [ongunakay.com](https://ongunakay.com) · 💼 [LinkedIn](https://linkedin.com/in/ongunakay) · 🧑‍💻 [GitHub](https://github.com/ongunakaycom) · 📧 [info@ongunakay.com](mailto:info@ongunakay.com)

Specialized in GCP/OCI infrastructure, Terraform IaC, Kubernetes, serverless, container security, and AI/LLM application orchestration.

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

---

⚠️ **Source code is private.** This repository documents architecture and design decisions only. The implementation, scoring weights, ML model artifacts, and LLM system prompt are kept in a private repo to protect the trading edge.
