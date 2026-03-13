---
title: "13 NaN Features. Same Code. Different Data."
date: 2026-03-14T08:00:00+08:00
description: "Every inference run had 13 NaN features. I knew. I ignored it. Turns out those features carried 12% of model importance."
tags: [quant, crypto, machinelearning, dataengineering, debugging]
---

When V5 went live, every single inference run logged the same warning:

```
WARNING: 13 features contain NaN values. Filling with 0.
```

I saw it on day one. I told myself it was a data warmup issue — the live feed just needed more bars to stabilize.

That was wrong.

---

## Same Code, Different Data

V5 solved the training/live code divergence problem from V4. One shared `feature_core.py`. Every pipeline — training, backtest, all five live scripts — imports from the same file.

The feature calculation was identical. The bug was in the data.

Historical open interest data: 5-minute granularity.
Live API open interest data: 1-hour granularity.

Both got resampled to 15-minute bars — the system's operating frequency.

When you resample 5-minute data to 15 minutes, you have 3 source points per output bar. Clean interpolation. Stable velocity calculations.

When you resample 1-hour data to 15 minutes, you have gaps. 3 output bars share 1 source point. The velocity features — rate of change, acceleration — produce NaN because there's nothing to differentiate.

Same function. Different input density. Completely different outputs.

---

## Why I Ignored It

Thirteen NaN features, filled with zero.

I rationalized: the model saw NaN-filled features during training too, in early rows before the rolling windows warmed up. It learned to work around them.

That was wishful thinking.

The model trained on NaN values in the first ~30 rows of 365 days of data. A small fraction of total training examples, weighted heavily toward recent history.

In live trading, those 13 features were NaN on *every single bar*. Every inference. Every decision.

---

## What Those 13 Features Were Actually Worth

After I fixed the data feed and audited feature importance, I found that the affected features collectively accounted for roughly 12% of the model's total predictive weight.

Not dominant. But not negligible either.

The model was making live decisions with 12% of its input effectively zeroed out. It was running partially blind — and producing signals that *looked* plausible because the remaining features still had signal.

That's the dangerous kind of bug. Not the kind that crashes. The kind that quietly degrades.

---

## The Fix

Two parts:

**1. Align live data granularity with training data granularity.**

Switch the live OI feed to 5-minute resolution. More API calls. Worth it.

**2. Add a hard assertion before every inference.**

```python
nan_features = [f for f in feature_cols if np.isnan(row[f])]
if len(nan_features) > 0:
    raise ValueError(f"NaN detected in {len(nan_features)} features before inference: {nan_features}")
```

Don't fill. Don't warn. Abort.

A failed inference that generates no signal is better than a confident prediction built on corrupted inputs.

---

## The Lesson

Code consistency is necessary. It's not sufficient.

The data flowing into the code has to be consistent too — in shape, in granularity, in density.

Check your features at the point of inference, not just at the point of training. If something looks wrong, don't rationalize it. Find out what it is.

Thirteen NaN features was a clue. I just waited too long to follow it.

---

*This content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Trade responsibly.*
