---
title: "My Trailing Stop Was Running. It Just Never Fired."
date: 2026-03-13T10:00:00+08:00
description: "6,500 lines of live code. 28 audit dimensions. The trailing stop updated state every bar. The exit check was simply missing."
tags: [quant, crypto, algotrading, livetrading, debugging]
---

Today I ran a full cross-audit of the V5 quant system.

Scope: training, backtest, and live execution — 13 files, 6,500 lines.

I was looking for drift. I found something worse.

---

## The Trailing Stop Was Updating. It Was Never Checking.

My backtest engine has four exit paths:

1. Fixed stop-loss
2. Fixed take-profit
3. Trailing stop
4. Momentum fade

SL and TP are delegated to the exchange. The trailing stop has to be managed by the script itself.

Here's what the code actually had:

```python
# This existed:
state = update_trail_sl(state, close, atr, trail_mult, bar_high=bar_high, bar_low=bar_low)

# This was missing — entirely:
# check if trail_sl was hit and close the position
```

The trailing stop value was being updated every bar. But the exit check — the part that actually closes the position when price crosses the stop — simply didn't exist.

Priority 2 (take-profit) connected directly to priority 4 (momentum fade). Priority 3 was a gap.

Four days of live trading. Zero trailing stop exits.

---

## The Fix

```python
if trail_sl is not None:
    hit_trail = (side == 'long' and bar_low <= trail_sl) or \
                (side == 'short' and bar_high >= trail_sl)
    if hit_trail:
        do_close('TrailingStop', f'{trail_sl:.4f}')
```

Six lines. That's all it took.

The logic was always there in the backtest. It just never made it into the live execution file.

---

## Two More Issues Found in the Same Audit

**Silent SL failure.**
When the exchange rejected a stop-loss order, the script logged it quietly and moved on. No alert. No retry. The position ran without protection.

Fixed: any SL placement failure now triggers an immediate WeChat push notification.

**No concurrency guard on the shell script.**
The cron job fires every 15 minutes. If a run takes longer than 15 minutes, two instances overlap. They both read the same state file. They both try to act on the same position.

Fixed: added `flock` to the shell wrapper.

```bash
flock -n /tmp/live_trade.lock python live_trade.py || echo "already running, skip"
```

---

## After the Fix

17:30 today — first execution after the patch.

XRP triggered a momentum fade exit. Position closed cleanly.

Not a trailing stop exit — but the exit logic ran. The system is working as designed for the first time since launch.

I'll see a real trailing stop exit when price gives me one.

---

*This content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Trade responsibly.*
