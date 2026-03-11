---
title: "Day 1: Three Crypto Quant Strategies Live. Here's Everything."
date: 2026-03-11T14:00:00+08:00
description: "BTC, ETH, SOL. 10x leverage. Pyramid sizing. Today is day 1. Not a simulation — real money, real trades. Here's the full deployment story."
tags: [quant, crypto, live trading, BTC, ETH, SOL, deployment]
---

Today I pushed three live trading scripts to production.

BTC, ETH, SOL. All running simultaneously. 15-minute signals. 10x leverage. Pyramid position sizing up to 5 layers.

This is the deployment story.

## Background

I've been building this system for months. The short version:

- Trained LightGBM models on 70 features per symbol
- Backtested with realistic fees (0.035% with 30% rebate)
- Ran 70-point audit checks on the live script before touching real money
- Results: SOL 64% monthly, ETH 56% monthly, BTC 34% monthly (no compounding, $143/symbol)

The longer version is in [yesterday's post](/posts/2026-03-11-transformer-failed-lgbm-won/).

## System Architecture

Three independent scripts, one per symbol:

```
~/trading/symbols/BTC/live_v3.py
~/trading/symbols/ETH/live_v3.py
~/trading/symbols/SOL/live_v3.py
```

Each script runs on its own loop. They don't communicate with each other. If BTC is in a long position, that has zero effect on ETH's decisions.

**Signal logic:**
1. Fetch latest 15-minute OHLCV data from Bybit
2. Calculate 70 features (momentum, volatility, liquidation ratios, time encoding)
3. Run LightGBM prediction → output: long / short / hold
4. Execute via Bybit API with automatic SL/TP

**Position sizing (pyramid):**
- Entry 1: base size
- Entry 2-5: additional layers if signal strengthens
- Max 5 layers per direction
- Reverse signal → full close, flip direction

## The Audit

Before going live, I ran a 70-point audit script. All passed. A few checks that matter:

| Check | Result |
|-------|--------|
| SL/TP uses high/low (not close) | ✅ |
| No duplicate signal execution | ✅ |
| Fee model matches Bybit actual | ✅ |
| Capital per position fixed (not compounding) | ✅ |
| Reverse signal closes all layers | ✅ |
| API rate limit handling | ✅ |

The audit was the result of five broken backtest versions. Each version revealed a new bug. v5 was the first one I trusted enough to trade real money.

## What I'm Tracking

Three metrics matter in the first month:

**1. Live win rate vs backtest win rate**

Backtest: SOL 78%, ETH 86%, BTC 72%

If live win rate drops below 60% sustained, the model isn't generalizing. I'll stop and investigate.

**2. Slippage**

Backtest assumed entry at signal close price. Live trades will have slippage. If slippage consistently eats more than 0.3% per trade, the fee model needs updating.

**3. Drawdown**

Max acceptable: 15%. If total account drawdown hits 15%, I pause everything and review.

## Why Three Symbols

Diversification. But not naive diversification.

BTC, ETH, and SOL have different volatility profiles and different liquidity structures. Their liquidation cascades often happen at different times. Running all three means fewer dead periods — when BTC is ranging, SOL might be trending.

The signals are independent. The risk is not perfectly correlated.

## What I'm Not Doing

A few things I deliberately chose not to do:

**No dynamic position sizing based on confidence.** The model outputs a probability, but I'm not scaling position size by it yet. That's an optimization for month 2.

**No cross-symbol signals.** No "if BTC dumps 3% in 5 minutes, close all SOL longs." That complexity will come later, after the base system proves itself.

**No compounding.** Fixed capital per trade. The backtest numbers are based on this. Adding compounding before validating the base win rate is a good way to turn a working strategy into a margin call.

## The Honest Part

I've lost a lot of money trading crypto. Years of it. Manual trading, pure intuition, maximum leverage — the kind of trading where you're glued to the screen at 3am and still lose.

This system is the opposite of that.

It doesn't care about my feelings. It doesn't revenge-trade. It doesn't let winners run because "this time is different." It just executes the model's signal and walks away.

Whether it works is still unknown. The backtest says yes. The live market will give the real answer.

I'm giving it 3 months. If it produces consistent results — not necessarily huge returns, just consistent and explainable ones — I'll scale up. If it doesn't, I'll write that post too.

---

Day 1 update will follow.

*Follow [@dayou_tech](https://x.com/dayou_tech) for live updates.*
