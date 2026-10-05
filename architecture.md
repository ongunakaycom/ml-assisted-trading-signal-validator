# Architecture

High-level system design for the ML-Assisted Trading Signal Validator.

---

## Overview

The system is a real-time signal pipeline that ingests live market data 
from Coinbase Advanced Spot, runs it through a deterministic multi-gate 
decision engine, enriches it with ML and LLM layers, and emits validated 
signals to a notification channel.

**Core design principle:** Every component is replaceable. Gate logic is 
deterministic. ML and LLM layers enrich and veto — they never own state.

---

## Pipeline Stages

    ┌─────────────────────────────────────────────────────────┐
    │  Coinbase Advanced Spot API                             │
    │  ticker · order book · trades · OHLCV candles           │
    └──────────────────────┬──────────────────────────────────┘
                           │
                           ▼
    ┌─────────────────────────────────────────────────────────┐
    │  Stage 1 — Candle-Close Guard                           │
    │  Aligns every scan to a 5-minute candle boundary        │
    │  Ensures indicators read published, settled data        │
    └──────────────────────┬──────────────────────────────────┘
                           │
                           ▼
    ┌─────────────────────────────────────────────────────────┐
    │  Stage 2 — Universe Discovery                           │
    │  Enumerates active spot pairs, filters by volume,       │
    │  spread, and blacklist (read-only external config)      │
    └──────────────────────┬──────────────────────────────────┘
                           │
                           ▼
    ┌─────────────────────────────────────────────────────────┐
    │  Stage 3 — Macro Gatekeeper (1D)                        │
    │  Classifies each instrument into a market stage:        │
    │    accumulation · markup-ready · watch · neutral · weak │
    │  Only qualified stages proceed to Stage 4               │
    └──────────────────────┬──────────────────────────────────┘
                           │
                           ▼
    ┌─────────────────────────────────────────────────────────┐
    │  Stage 4 — 5-Gate Decision Engine                       │
    │  Each gate applies hard, deterministic requirements:    │
    │    Gate 1  Wind Hunter          (trend follow)          │
    │    Gate 2  Capitulation Sniper  (mean reversion)        │
    │    Gate 3A Whale Hunter         (trend accumulation)    │
    │    Gate 3B Whale Bottom Hunter  (bottom accumulation)   │
    │    Gate 4  Reject               (no signal)             │
    └──────────────────────┬──────────────────────────────────┘
                           │
                           ▼
    ┌─────────────────────────────────────────────────────────┐
    │  Stage 5 — Whale Detection Engine                       │
    │  13 independent methods classified by direction:        │
    │    bullish · bearish · neutral                          │
    │  Signals with bearish dominance are rejected before     │
    │  scoring (direction-aware filter)                       │
    └──────────────────────┬──────────────────────────────────┘
                           │
                           ▼
    ┌─────────────────────────────────────────────────────────┐
    │  Stage 6 — ML Failure Prediction                        │
    │  XGBoost / RandomForest / MLP ensemble predict the      │
    │  probability that the signal will fail                  │
    └──────────────────────┬──────────────────────────────────┘
                           │
                           ▼
    ┌─────────────────────────────────────────────────────────┐
    │  Stage 7 — LLM Risk Validator (Gemini)                  │
    │  7-step mandatory checklist. Default = DECLINE.         │
    │  Produces coin-specific commentary for the signal       │
    └──────────────────────┬──────────────────────────────────┘
                           │
                           ▼
    ┌─────────────────────────────────────────────────────────┐
    │  Stage 8 — Notification Delivery                        │
    │  Telegram message with entry, target, and AI analysis   │
    └─────────────────────────────────────────────────────────┘

---

## State Model

The system is **stateless per cycle** with two exceptions:

| Store | Scope | Purpose |
|-------|-------|---------|
| Macro cache | Persistent on disk | 1D stage classification, refreshed periodically |
| Session memory | RAM only | One-signal-per-instrument policy, cleared on restart |

Every other decision is recomputed from live data on each cycle. 
No long-term user model, no cross-session memory, no hidden state.

---

## Concurrency Model

- Fully async (`asyncio`) with rate-limited HTTP calls
- Token-bucket rate limiter aligned to exchange limits
- Background cache refresher stopped during scan, restarted after
- Signal building and delivery are synchronous per signal

---

## Failure Modes

| Failure | Behavior |
|---------|----------|
| Exchange API error | Retry with backoff, then skip instrument |
| Rate limit hit | Wait, then resume |
| ML model unavailable | Skip ML gate (log warning) |
| LLM unavailable | **DECLINE** the signal (fail-safe) |
| Notification failure | Log, continue; state still updates |
| Candle boundary drift | Re-align to next close |

The **fail-safe default** is critical: any ambiguity results in no 
signal being sent. Missing a trade is acceptable; sending a bad 
signal is not.

---

## Config Isolation

All tunable parameters live in one place per concern:

- Gate thresholds — a single module-level constants block
- Whale method registry — a single source of truth class
- LLM system prompt — a single method
- Blacklist — read-only external JSON, manually curated

No magic numbers scattered across the codebase.

---

## Observability

Every cycle emits a structured log with:

- Candle-close alignment status
- Universe size before / after filtering
- Macro stage distribution
- Gate-by-gate pass/reject counts
- Whale method counts per candidate
- ML prediction for each signal
- LLM decision + reasoning steps
- Session-blocked instruments

This makes production debugging a matter of reading one log file.