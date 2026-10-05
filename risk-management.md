# Risk Management

Risk in this system is managed by **preventing bad signals from 
being emitted**, not by adding stop-losses after the fact.

The default posture is defensive. Every layer is designed to fail 
closed — if the system cannot be certain a signal is good, it 
does not send one.

---

## Principle 1 — Fail-Safe Defaults

| Component | Default when uncertain |
|-----------|------------------------|
| Candle guard | Skip the scan, wait for next close |
| Gate evaluation | Reject (Gate 4) |
| Whale direction | Reject if ambiguous |
| ML failure model | Block if failure probability high |
| LLM validator | **DECLINE** if any error occurs |
| Notification layer | Do nothing on error |

The LLM layer is the most important fail-safe: if the validator 
cannot be reached, errors, or returns unparseable output, the 
signal is **declined** — never sent unvalidated.

---

## Principle 2 — Multi-Timeframe Alignment ("All or Nothing")

A signal must satisfy the following simultaneously:

- Price above EMA200 on the **1D** timeframe
- Price above EMA200 on the **1H** timeframe
- Price above EMA200 on the **15m** timeframe

Exception: bottom-accumulation gates (Gate 2 and Gate 3B) are 
allowed to be below 1D EMA200 by design, but still require 1H 
and 15m alignment.

If any required timeframe disagrees, the signal is rejected before 
the LLM layer. This prevents both "falling knife" entries and 
"late to the party" entries on the same instrument.

---

## Principle 3 — Direction-Aware Whale Filter

A high whale method count is not sufficient. The methods must also 
be directionally bullish.

The engine computes a direction breakdown and rejects the signal 
if bearish methods dominate. This catches the specific failure 
mode where a high method count is produced entirely by sell-side 
signals (aggressive selling, seller dominance, strong sell imbalance).

---

## Principle 4 — Hard Gate Requirements

Each gate has minimum requirements that must be met independently 
of the aggregate score. Examples of the requirement categories:

- Minimum order-book pressure (at least one large bid)
- Minimum market activity (volume spike floor to avoid dead markets)
- Minimum momentum (24h change or method count)
- Minimum liquidity (aggregate 24h volume)

A gate never passes on aggregate score alone. The requirements 
are hard filters that run before scoring.

---

## Principle 5 — Session-Level Signal Memory

Once an instrument has been signalled in a session, it is blocked 
for the remainder of the session.

- **RAM only** — no disk write
- **No expiry** — a blocked instrument stays blocked until restart
- **Independent** of the read-only blacklist file

This eliminates the "same instrument signalled repeatedly in a 
short window" failure mode that plagues naive scanners.

---

## Principle 6 — Candle-Close Alignment

Every scan is aligned to a 5-minute candle boundary with a small 
offset to account for exchange publication latency.

- Scans never fire mid-candle
- Indicators read only settled, published data
- A post-sleep guard re-verifies alignment after the sleep

This prevents the "read a stale candle" class of bugs.

---

## Principle 7 — Read-Only External Config

The blacklist of instruments is stored in a JSON file that the 
bot reads but never writes.

- Operator controls the file manually
- No automated writes
- No risk of the bot corrupting its own config
- No race conditions between the bot and external tooling

---

## Principle 8 — No Stop-Loss (Documented Design Choice)

Signals are emitted **without a stop-loss**.

This is intentional. The strategy targets mean-reversion and 
accumulation breakouts where the instrument is expected to move 
upward quickly. A stop-loss placed inside the expected noise 
band would be hit on normal volatility before the target move.

Risk is instead managed **before entry** by rejecting weak signals. 
This is a documented design trade-off, not an oversight.

---

## Summary

| Layer | Defends Against |
|-------|-----------------|
| Candle guard | Stale data |
| Macro gatekeeper | Wrong regime |
| Hard gate requirements | Dead markets, fake setups |
| Direction-aware filter | Bearish wolf in bullish clothing |
| Multi-timeframe rule | Falling knives, late entries |
| ML failure model | Statistically bad setups |
| LLM validator | Anything the above missed |
| Session memory | Repeated signals on same instrument |
| Read-only config | Self-corruption |