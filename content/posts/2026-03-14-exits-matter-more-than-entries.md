---
title: "Exits Matter More Than Entries. Here's My Four-Path Exit System."
date: 2026-03-14T09:00:00+08:00
description: "I spent more time designing exits than entries. Where you put the stop, how you trail it, when you decide the signal is gone — that's where P&L is made."
tags: [quant, crypto, algotrading, riskmanagement, trading]
---

Everyone obsesses over entries.

Which indicator. Which model. Which signal threshold.

Entries get you into a trade. Exits determine what you keep.

I spent more engineering time on my exit system than my entry model. Here's why — and how it's structured.

---

## Why Exits Are Harder

An entry is a single decision: does this bar look like a long or a short?

An exit is an ongoing negotiation between multiple objectives:

- Protect capital if the trade goes wrong
- Capture profit if it goes right
- Trail the position if it keeps going
- Recognize when the original thesis no longer holds

Get any one of those wrong and a winning entry becomes a losing trade.

---

## Path 1: Fixed Stop-Loss

The hard floor.

Set at entry. Delegated to the exchange as a stop order. Never moved manually.

Distance is ATR-based:

```python
sl_price = entry_price - (atr * sl_mult)  # for longs
sl_price = entry_price + (atr * sl_mult)  # for shorts
```

`sl_mult` is optimized through walk-forward testing. It's not a gut feeling.

If this triggers, the trade was wrong. Accept it. Move on.

---

## Path 2: Fixed Take-Profit

The defined target.

Also exchange-delegated. Also ATR-based.

The R:R ratio — risk/reward — is built in here. If my stop is 1× ATR, my target needs to be at minimum 1.5× ATR for the system to have positive expectancy at any win rate below 100%.

```python
tp_price = entry_price + (atr * tp_mult)  # for longs
```

This exits when the trade hits its planned destination. Clean. No discretion required.

---

## Path 3: Trailing Stop

This is where extended moves get captured.

After the position reaches a profit threshold, the trailing stop activates. It follows price at a fixed distance, moving only in the direction of profit — never backward.

```python
state = update_trail_sl(state, close, atr, trail_mult, bar_high=bar_high, bar_low=bar_low)

# Then check if it's been hit:
if trail_sl is not None:
    hit_trail = (side == 'long' and bar_low <= trail_sl) or \
                (side == 'short' and bar_high >= trail_sl)
    if hit_trail:
        do_close('TrailingStop', f'{trail_sl:.4f}')
```

Note: this runs in the script, not on the exchange. The exchange SL gets canceled when the trail activates.

Path 3 is what converts a good trade into a great one.

(It's also the path that was completely missing from my live code for the first four days. That's a different post.)

---

## Path 4: Momentum Fade

The signal check.

At every bar, I re-score the current bar using the same model that opened the trade. If the score drops below a threshold — meaning the original signal has degraded — the position exits regardless of price action.

```python
current_score = model.predict_proba(current_features)[0][1]
if current_score < momentum_exit_threshold:
    do_close('MomentumFade', f'{current_score:.4f}')
```

This is qualitatively different from the other three exits. SL, TP, and trail are price-based. Momentum fade is signal-based.

It answers the question: *does the reason I entered this trade still exist?*

If not, exit. Even if price hasn't moved against me yet.

---

## Priority Order

When multiple conditions are met in the same bar, they execute in order:

1. Fixed stop-loss (always first — capital protection)
2. Fixed take-profit
3. Trailing stop
4. Momentum fade

Only one exit fires per bar.

---

## The Design Principle

Entries need to be good enough to be right more often than not.

Exits need to be precise enough to let winners run and cut losers short.

Most of my system's edge isn't in the model. It's in the exit architecture — specifically in path 3 and path 4 working together.

Path 3 keeps me in good trades longer than I'd stay instinctively.
Path 4 gets me out of turning trades before price confirms the move.

Together, they make the system behave less like a human and more like a policy.

That's exactly what I want.

---

*This content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Trade responsibly.*
