---
title: "Backtest Design Is 60% of the Result. Model Training Is 40%."
date: 2026-03-13T09:00:00+08:00
description: "A great model gives you a signal. How much you make from that signal depends entirely on your backtest framework design."
tags: [quant, crypto, algotrading, wfo, machinelearning]
---

Most people obsess over model accuracy.

Win rate. Precision. Feature importance.

That's 40% of the problem.

The other 60% is how you build the backtest that validates it.

---

## What the Backtest Actually Controls

A model tells you: *this bar looks like a long.*

The backtest decides everything else:

- How big is the position?
- Where does the stop go?
- How does the stop trail?
- When do you decide the signal is gone?
- Do you re-enter after an exit?
- How do you handle overnight funding?

Every one of those decisions compounds over hundreds of trades.

A mediocre model with a well-designed backtest framework can outperform a great model with a sloppy one.

I've seen this firsthand.

---

## Walk-Forward Optimization

In-sample + out-of-sample split is not enough.

I use walk-forward optimization (WFO).

The idea: slide a window through history. Train on the first N months. Test on the next M months. Slide forward. Repeat.

```
[train][test]
      [train][test]
            [train][test]
```

Each test window is out-of-sample. You get multiple independent performance samples across different market regimes.

If your system only worked in bull markets, WFO exposes it.

If it worked consistently across regimes, you have something real.

---

## The Four Exit Paths

This is where most backtests are underbuilt.

Mine has four exit paths, each serving a different purpose:

**1. Fixed stop-loss**
Hard floor. Delegated to the exchange. Never negotiated.

**2. Fixed take-profit**
Lock in the target. Also exchange-delegated.

**3. Trailing stop**
Moves with the position as it profits. Managed by the script — not the exchange — because it needs to be dynamic.

**4. Momentum fade**
The signal that opened the trade has weakened below threshold. Exit before price confirms the reversal.

Most retail backtests have paths 1 and 2.
Path 3 is where you capture extended moves.
Path 4 is what separates a trading system from a stop order.

---

## Position Sizing Is Not an Afterthought

Fixed fraction sizing. Every trade risks the same percentage of equity.

Not the same dollar amount. Not the same number of contracts. The same percentage.

This means position size scales with account size. As you grow, exposure grows proportionally. As you draw down, exposure shrinks automatically.

It's the difference between a system that survives drawdowns and one that blows up in them.

---

## The Insight

The backtest is not a validation tool.

It's a design space.

Every parameter — stop distance, trail multiplier, momentum threshold, position size — is a degree of freedom. Getting those right matters more than whether your model is XGBoost or a neural net.

Train the best model you can.

Then spend at least as much time on the framework that turns its signals into trades.

---

*This content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Trade responsibly.*
