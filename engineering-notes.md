# Engineering Notes

Trade-offs, lessons learned, and design decisions behind the 
ML-Assisted Trading Signal Validator.

---

## Stack Summary

| Layer | Choice | Why |
|-------|--------|-----|
| Language | Python 3.11 | Async ergonomics, rich ML ecosystem |
| Async model | asyncio | Sufficient for I/O-bound rate-limited scanning |
| Exchange | Coinbase Advanced Spot | Deep liquidity, clean API, USDC pairs |
| ML | XGBoost / RF / MLP | Established, interpretable, fast |
| LLM | Google Gemini | Strong structured output, low latency |
| Storage | JSON + CSV | Sufficient scale, operator-inspectable |
| Notifications | Telegram | Native, no infra to run |

---

## Key Trade-Offs

### Determinism vs Adaptivity

Signal generation is deterministic. Signal filtering includes ML 
and LLM layers that are adaptive. This split is intentional: the 
generation logic must be reproducible and auditable; the filtering 
logic benefits from statistical and contextual nuance.

### Fail-Safe vs Fail-Open

Every layer defaults to **no signal** on uncertainty. This 
sacrifices some good signals but guarantees that no unvalidated 
signal is delivered.

### LLM Cost vs Signal Quality

The LLM layer adds latency and API cost per signal. It is worth it 
because it catches failure modes the deterministic gates miss — 
specifically, context-dependent concerns like macro regime and 
multi-timeframe alignment that are hard to encode as static rules.

### No Stop-Loss

The system does not emit stop-losses. This is documented and 
intentional. Risk is managed before entry, not after.

### RAM-Only Session Memory

Session memory is not persisted. On restart, previously-signalled 
instruments can be signalled again. This is acceptable because 
restarts are rare, and it avoids an entire class of state-file 
corruption bugs.

### Read-Only External Config

The bot never writes to its own configuration. The operator 
controls blacklists manually. This is slower than automation but 
eliminates self-corruption risk.

---

## Lessons Learned

### 1. Direction matters more than count

An early version counted whale methods without tracking direction. 
Signals with 7 detected methods could still be bearish. Adding a 
direction filter removed a whole category of false positives.

### 2. Two sources of truth is a bug

History was initially loaded from two different file paths. They 
drifted silently. Unifying to a single loader was a one-line fix 
with outsized impact.

### 3. Fail-safe is not optional

An early LLM integration defaulted to **approve** on error. That 
inverted the risk profile — an unavailable validator meant *more* 
signals, not fewer. Reversing the default to **decline** fixed it.

### 4. Cross-symbol state is a footgun

A single "last signal" dict was shared across symbols. Rapid-fire 
signals contaminated each other. Keying state by symbol removed 
the bug entirely.

### 5. Bounded growth is a design requirement

Per-symbol stores, history lists, and caches all need explicit 
size limits. Unbounded growth is invisible until it isn't.

### 6. Honest messaging builds trust

The notification layer used to claim "breakout confirmed" whenever 
a pre-breakout signal fired. This eroded trust in the output. 
Reporting the exact gate verdict fixed it.

---

## Open Problems

- **Cross-session user model** — currently session-scoped only
- **Adaptive thresholds** — gate thresholds are static; a regime-aware 
  layer would be a natural next step
- **Feature drift** — ML models are retrained manually; automated 
  drift detection is future work
- **Multi-exchange** — currently single-exchange; a venue-agnostic 
  layer would broaden the universe

---

## Stack Summary (One-Liner)

> Python · asyncio · XGBoost · scikit-learn · Google Gemini · 
> Coinbase Advanced Spot API · Telegram Bot API · JSON/CSV 
> persistence · token-bucket rate limiting

---

## Final Note

This project is not a trading bot in the "signal group" sense. It 
is a **systematic signal engine** with layered filtering, ML-based 
failure prediction, and LLM-based risk validation. The engineering 
challenge was not the trading — it was building a real-time 
pipeline that fails safely under uncertainty.