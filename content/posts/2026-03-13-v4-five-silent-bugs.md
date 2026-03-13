---
title: "My Quant Model Had 5 Silent Data Bugs. The Backtest Looked Great."
date: 2026-03-13T08:00:00+08:00
description: "I audited my V4 trading system line by line. Found 5 data bugs that never caused errors, never appeared in backtests, and silently corrupted every live inference for weeks."
tags: [quant, crypto, algotrading, machinelearning, debugging, live trading]
---

My V4 trading system used 11 data sources: price data, funding rates, open interest, institutional long/short ratio, liquidation data, fear & greed index, CVD, and more.

The backtest results looked solid. Win rates above 80%. Drawdown under 15%.

Then I audited the code line by line. What I found made me rebuild the entire system from scratch.

---

## Bug 1: The Fear & Greed index was always 50

The Alternative.me API returns data in this format:

```json
{"values": [72, 65, 48...], "dates": [1709...]}
```

My code was doing `data.get('data')` — which always returns `None`. So every single value was replaced with the default: 50.

Fear & Greed ranges from 0 to 100. 50 is dead center — neutral. The model trained on 365 days of data thinking the market was perpetually neutral.

It never saw a real fear reading. It never saw a real greed reading. It learned from a constant.

---

## Bug 2: Open interest velocity meant two different things

During training:
```python
oi_velocity = oi_chg_pct.rolling(4).mean()  # 4-bar rolling average
```

During live trading:
```python
oi_velocity = single_bar_change_rate  # just one value
```

Same feature name. Completely different calculations. The model learned patterns from the rolling average version, then during live trading received a completely different input dressed in the same label.

---

## Bug 3: The rolling window was 24x too short

Institutional long/short ratio zscore.

Training: `rolling(30 days)`
Live: `rolling(30 hours)`

A 30-day zscore describes where the current ratio sits relative to the past month. A 30-hour zscore describes the past day and a quarter.

They describe completely different market states. The model learned one signal. Live trading fed it another.

---

## Bug 4: The liquidation zscore was always 0

Computing a 30-day zscore on liquidation data requires at minimum 720 rows of 1-hour data.

My live API call fetched 200 rows.

`rolling(720)` on 200 rows produces NaN for every single row. NaN gets filled with 0. So the model saw a liquidation zscore of exactly 0 on every single bar, regardless of what was actually happening in the market.

---

## Bug 5: BTC funding rate was reading from the wrong path

The funding rate file for BTC was stored in the root data directory instead of the `funding/` subdirectory. The code looked in the subdirectory. It found nothing. BTC's funding rate feature defaulted to zero for the entire live trading period.

---

## Why did none of this show up in the backtest?

Because the backtest used historical files — and the historical files were correct.

The fear & greed index had real values in the historical data. The institutional ratio had complete daily history going back years. The liquidation data had thousands of rows, more than enough for rolling(720).

So during backtesting, all five features worked exactly as intended. The model learned real patterns from real data.

Then in live trading, those features silently became constants, zeros, or wrong calculations. The model kept making decisions. It just wasn't using the signals it thought it was using.

I call this the **shadow feature problem**: your model believes it's using 11 data sources. In reality, it's running on 6.

---

## The fix: one function, shared everywhere

V5 has a single file called `feature_core.py`. Every part of the pipeline — training data generation, backtesting, and all five live trading scripts — imports from this one file.

```
feature_core.py
    ↕ import
prepare_dataset_v5.py  (training)
    ↕ import
backtest_engine.py     (backtest)
    ↕ import
live_sol.py, live_eth.py, live_btc.py...  (live)
```

Physical impossibility of divergence. If the feature calculation is wrong in live trading, it's wrong in the backtest too — which means the backtest catches it before you go live.

---

## But unified code isn't enough

Even with a single shared function, I still found 13 NaN features in every live inference run after V5 launched.

The function was identical. The bug was the data flowing into it.

Historical OI data: 5-minute granularity. Live API: 1-hour granularity. Same resample-to-15min operation. Completely different results — because one was dense and one was sparse.

Code consistency is necessary. It's not sufficient. The data feeding into the code has to be consistent too.

That's the lesson V4 taught me the hard way.

---

*This content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Trade responsibly.*
