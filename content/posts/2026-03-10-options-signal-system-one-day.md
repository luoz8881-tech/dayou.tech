---
title: "How I Built a Deep OTM Options Signal System in One Day"
date: 2026-03-10T11:00:00+08:00
description: "Not a tutorial. Not an ad. Just what actually happened today — using 100% free data."
tags: [crypto, options, quant, system, free data]
categories: [Quant Trading]
draft: false
---

*Not a tutorial. Not an ad. Just what actually happened today.*

---

This morning I asked myself one question:

Can I build a complete options signal system using only free data?

By tonight, the answer is yes.

---

## The Data — All Free

**Deribit public API** (no account needed):
- Historical implied volatility
- Full options chain with Greeks
- Max Pain calculation

**Bybit public API:**
- Historical price data
- Funding rates

**Alternative.me:**
- Fear & Greed Index — history back to 2018

**CoinGlass** (paid, but data already downloaded):
- Liquidation history — 400 days

---

## The Signal Logic

Four conditions. The more that align, the stronger the signal.

```
IV Rank < 30%        → Options historically cheap
Fear & Greed < 20    → Extreme market fear
Large liquidation    → Sentiment at extremes
MaxPain deviation > 5% → Clear price gravity direction
```

Score 0-3: don't trade.
Score 4-5: small position.
Score 6+: larger position.

---

## The Backtest (BTC, 400 days)

Multi-signal strategy triggered 28 times.

Win rate: 3.6%.

Sounds terrible. But when it won, average return: **+4,991%.**

Average ROI per trade: **+81.8%.**

Positive expected value. That's all that matters.

---

## What I Actually Built Today

- IV Rank module (Deribit free API)
- GEX calculator (real-time gamma exposure)
- Max Pain calculator (replaces paid CoinGlass with free Deribit data)
- Signal scorer (combines all signals, outputs 0-10 score)
- XGBoost classifier (trained on 14 features, still needs more data)
- Backtest engine (validates historical signal effectiveness)

Total time: one day.
Total cost of data: $0.

---

The system is running. The signals are live.

Whether it works at scale — that's what the next few months will answer.

*→ dayou.tech*

---

*All content is for informational purposes only. Cryptocurrency trading involves significant risk. Trade responsibly.*
