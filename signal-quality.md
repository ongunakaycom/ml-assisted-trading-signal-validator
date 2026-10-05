# Signal Quality Overhaul

A comprehensive pass to eliminate false-positive signals. The 
overhaul touched ten areas of the pipeline and rewrote roughly 
9,000 lines of code.

This document is a public summary. The implementation details are 
kept in the private repository.

---

## Motivation

The scanner was already producing profitable signals, but the 
success rate fluctuated more than desired. Root-cause analysis 
identified ten distinct sources of false positives:

1. Signals passing gates on aggregate score alone, without 
   hard requirements
2. Signals with high whale method counts but bearish direction
3. Signals detected on lower timeframes against higher-timeframe 
   trend
4. History and statistics loaded from two different file paths, 
   causing silent data drift
5. Whale method count inflated by duplicate method aliases
6. Messaging that claimed "breakout confirmed" when only 
   pre-breakout conditions were true
7. Cross-symbol contamination in rapid-fire signal bursts
8. LLM output not being persisted to the notification layer
9. Repeated signals on the same instrument within a session
10. Unbounded growth of per-symbol stores and history lists

---

## Fixes Applied (Public Summary)

### Fix 1 — Hard Gate Requirements

Each gate now enforces minimum hard requirements **before** any 
scoring. A gate never passes on score alone.

### Fix 2 — Direction-Aware Rejection

Signals where bearish whale methods dominate are rejected before 
scoring. This catches the specific failure mode where method count 
is high but the underlying pressure is sell-side.

### Fix 3 — Multi-Timeframe Alignment

Price must agree with EMA200 across three timeframes (1D, 1H, 15m) 
simultaneously, with documented exceptions for bottom-accumulation 
gates.

### Fix 4 — Unified Signal Path

A single loader is now the sole source of truth for performance 
history. The legacy duplicate path was removed.

### Fix 5 — Accurate Whale Method Count

Method aliases are deduplicated. The method count is computed 
against the canonical registry, not against the raw detection 
output.

### Fix 6 — Honest Messaging

The notification layer now reports the exact gate verdict. If the 
gate did not confirm a breakout, the message does not claim one.

### Fix 7 — Per-Symbol State Isolation

Signal state is now keyed by symbol. Rapid-fire signals no longer 
contaminate each other's data.

### Fix 8 — LLM Commentary Persistence

The LLM's coin-specific analysis is now carried through every 
downstream layer and appended to the notification message.

### Fix 9 — Session-Level Signal Memory

Each instrument can be signalled at most once per session. The 
block is RAM-only and has no expiry until restart.

### Fix 10 — Bounded State Growth

Per-symbol stores and history lists are trimmed on a schedule. 
Long-running sessions no longer accumulate unbounded memory.

---

## Results

| Metric | Before | After |
|--------|--------|-------|
| Success rate | Fluctuating | 88.0% (stabilised) |
| Profit factor | Lower | 14.75x |
| False-positive categories | 10 identified | All addressed |
| Signal count | Comparable | 991 live signals |

The overhaul did not reduce signal volume. It removed the noise 
from the signal set.

---

## Engineering Lessons

1. **Aggregate scores hide direction.** A high score with wrong 
   direction is worse than a low score with right direction.
2. **Two file paths for the same data is a bug, not a feature.** 
   Unify early.
3. **Fail-safe defaults beat fail-open defaults.** When uncertain, 
   decline.
4. **Cross-symbol state is a footgun.** Key everything by symbol.
5. **Bounded growth is a design requirement, not an optimisation.**