# Whale Detection Engine

The whale engine is a modular layer that analyzes order-book 
microstructure and trade flow to detect institutional-scale 
positioning.

It produces a **direction-aware method count** that downstream 
gates consume. It never emits a trade on its own.

---

## Philosophy

Order books are noisy. A single large order can be spoofed, 
iceberged, or swept. Trade flow alone can be misleading. Volume 
spikes can be organic or mechanical.

The engine does **not** trust any single signal. It runs 13 
independent detectors and only reports aggregate evidence.

---

## Method Taxonomy

Methods are grouped by data source and tagged with a direction.

### Order-book methods

| Method | Direction | Reads |
|--------|-----------|-------|
| Iceberg bids | bullish | Repeated same-size bids at multiple levels |
| Sell walls | bearish | Large resting asks |
| Mega accumulation | bullish | Individual orders above an institutional size |
| Buy imbalance | bullish | Aggregated bid/ask depth ratio |
| Whale dominance | bullish | Count of large bids vs large asks |
| Spread compression | neutral | Tightening bid-ask spread |
| Volume anomaly | neutral | Depth concentration at a single level |

### Trade-flow methods

| Method | Direction | Reads |
|--------|-----------|-------|
| Multiple whale trades | neutral | Repeated large prints |
| Aggressive buying | bullish | Taker-side buy dominance |
| Aggressive selling | bearish | Taker-side sell dominance |
| Buyer dominance | bullish | Buy/sell count ratio |
| Seller dominance | bearish | Sell/buy count ratio |
| Strong buy imbalance | bullish | Extreme positive flow imbalance |
| Strong sell imbalance | bearish | Extreme negative flow imbalance |

### Structural methods

| Method | Direction | Reads |
|--------|-----------|-------|
| Iceberg orders | neutral | Hidden split orders across levels |
| Spoofing | bearish | Large orders that vanish |
| Stop-hunting | neutral | Support breaks with fast recovery |
| Dark pool | neutral | Large trades with no book impact |
| Liquidity sweep | neutral | Extreme bid/ask ratio events |
| Delta divergence | dynamic | Price vs cumulative delta mismatch |

---

## Direction-Aware Rejection

Each method carries a direction tag (`bullish`, `bearish`, `neutral`). 
Some tags are **dynamic** — for example, delta divergence is classified 
by the actual divergence type detected at runtime.

The engine computes a **direction breakdown**:

    bullish_count  = methods tagged bullish
    bearish_count  = methods tagged bearish
    neutral_count  = methods tagged neutral
    net_direction  = majority direction, with a margin

A signal is **rejected before scoring** if:

- bearish methods dominate by more than a fixed margin, OR
- net direction is explicitly bearish, OR
- order-book imbalance is below an extreme-seller floor

This filter exists because raw method count alone is misleading. 
A signal with 7 detected methods can still be a falling knife if 
5 of them are sell-side.

---

## Aggregation

The engine does not expose individual method confidences. It 
exposes an **aggregate method count** and a **direction breakdown**.

Downstream gates consume these two values and apply their own 
hard requirements. The whale engine does not decide trades.

---

## No Single Point of Trust

If any individual method fails or returns garbage, the engine 
still produces a valid aggregate from the remaining methods.

This is why the engine reports a count, not a boolean. A count 
degrades gracefully; a boolean does not.

---

## What Is Not Public

- Exact size thresholds for "large order"
- Exact iceberg detection criteria
- Direction margin values
- Imbalance floor
- Aggregation weights (if any)

These are kept in the private implementation to protect the edge.