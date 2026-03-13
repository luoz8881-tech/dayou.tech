---
title: "Why I Rebuilt My Quant System From Scratch"
date: 2026-03-15T08:00:00+08:00
description: "V4 had 5 silent data bugs. The backtest looked great. The model was trading on shadow features the whole time."
tags: [quant, crypto, machinelearning, systemdesign, algotrading]
---

V5 is my fifth full version of this system.

Not five iterations. Five rebuilds.

Each one came after I discovered the previous version had a fundamental flaw — one that didn't show up in backtests, didn't cause errors, and only became visible when I looked hard enough.

Here's how the system evolved, and what forced each rebuild.

---

## V3: The First Real System

V3 was the first version I considered serious.

- XGBoost model trained on 1-hour bars
- 6 features: price action, volume, RSI, MACD, funding rate, open interest
- Simple backtest: fixed SL, fixed TP, in-sample validation only
- One coin: ETH

It worked in backtesting. Win rate around 62%. Sharpe above 1.

I went live. Results were inconsistent. Some weeks good. Most weeks flat or slightly negative.

The problem: the model was overfit to a single market regime. The backtest didn't test it across different conditions. Walk-forward validation didn't exist.

I needed more data, more features, and a better validation framework.

---

## V4: More Data, New Problems

V4 expanded everything:

- 11 data sources (price, funding, OI, liquidations, CVD, fear & greed, institutional ratios, and more)
- 5 coins: ETH, BTC, DOGE, SOL, XRP
- Walk-forward optimization added
- Multi-timeframe features: 15m, 1h, 4h

The backtest results were the best I'd ever seen.

Win rates above 80%. Drawdown under 15%. Consistent performance across WFO windows.

I went live with confidence.

Six weeks later, I audited the code.

Five silent data bugs. Every one of them invisible in backtesting because the historical data files were correct. Every one of them active in live trading because the live API calls were different.

The fear & greed index was always 50. OI velocity meant different things in training vs. live. The liquidation zscore was always 0. BTC funding rate was reading from the wrong file path.

The model wasn't trading on 11 features. It was trading on 6 real ones and 5 constants.

I call it the shadow feature problem.

---

## Why I Didn't Just Fix V4

The fixes seemed straightforward at first. Patch the API calls. Align the feature calculations. Re-train.

But as I dug in, I realized the architecture itself was the problem.

Each live trading script had its own feature calculation logic. Copied from the training script at some point, then modified independently. Five scripts, five slightly different implementations of the same functions.

Any time I changed a feature calculation, I had to update it in five places. Any time a data source changed format, I had to find every place it was parsed.

The bugs weren't accidents. They were structurally inevitable.

The only real fix was a new architecture: one shared feature module, imported everywhere.

---

## V5: One Source of Truth

The core design principle of V5: one function, called everywhere.

```
feature_core.py
    ↕
prepare_dataset.py  (training)
backtest_engine.py  (backtesting)
live_eth.py, live_btc.py, live_sol.py, live_xrp.py, live_bnb.py  (live)
```

If a feature calculation is wrong, it's wrong everywhere simultaneously. The backtest catches it before live trading. Physical impossibility of silent divergence.

V5 also added:

- Hard assertions before every inference (no silent NaN fills)
- Concurrency protection on all live scripts
- Exchange-independent trailing stop management
- Momentum fade exit path
- Alert escalation on SL placement failures

---

## What V5 Still Got Wrong on Day One

Even with the new architecture, V5 launched with 13 NaN features per inference.

Code was unified. Data wasn't.

Historical OI: 5-minute bars. Live OI API: 1-hour bars. Same resampling function. Completely different output density. Velocity features — which depend on granular rate-of-change — produced NaN on every live bar.

Code consistency is necessary. It's not sufficient.

The data flowing into the code has to match what the model was trained on.

That's the lesson V5 is still teaching me.

---

## The Pattern

Every version looked solid in backtest. Every version had a flaw that only appeared in live conditions.

V3: regime overfitting.
V4: data divergence between training and live.
V5: data granularity mismatch.

I don't expect V5 to be the last rebuild.

But each version has made the flaw harder to hide — shorter time from launch to discovery, more systematic audit process, fewer places for bugs to live undetected.

That's progress.

---

*This content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Trade responsibly.*
