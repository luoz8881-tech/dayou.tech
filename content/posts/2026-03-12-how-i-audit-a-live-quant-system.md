---
title: "How I Audit a Live Quant System (And What I Found Today)"
date: 2026-03-12T10:00:00+08:00
description: "Most people build the model and call it done. I ran a full 4-dimension audit on BTC and XRP today. Found real problems. Here's the exact checklist."
tags: [quant, crypto, audit, risk management, BTC, XRP, live trading]
---

Most people running a quant system worry about one thing: does the model predict correctly?

That's maybe 30% of the problem.

Today I ran a full audit on my BTC and XRP live strategies. Found two serious issues. One system got suspended. Here's exactly what I checked and what broke.

## The 4-Dimension Audit Framework

After running live for a few days, I realized I needed a structured way to verify the full pipeline — not just "is the model loaded correctly." 

The four dimensions:

1. **Raw data integrity** — Is the input data clean and complete?
2. **Training quality** — Did the model actually learn something real?
3. **Backtest validity** — Were the backtest parameters used in production?
4. **Live consistency** — Does the live script match what was backtested?

If any layer is broken, the others don't matter.

---

## Dimension 1: Raw Data

For each symbol I checked:

- Time range and row count
- Gaps larger than 30 minutes
- Duplicate timestamps
- Null values
- Whether all required data sources exist (funding rate, open interest, liquidation data, long/short ratios)

**BTC result:** Clean. 181,033 rows. Zero gaps. Zero duplicates. All external data present.

**XRP result:** K-line data clean. But when I checked the external data directory — the institutional long/short ratio, liquidation data, and retail ratio files were completely missing from where the training script expected them.

The model trained on XRP was partially blind. Those features were silently filled with default values (zeros or 0.5) during training. The model never actually learned from real market microstructure data for XRP.

---

## Dimension 2: Training Quality

I checked label distribution and feature count.

**BTC problem found:** Label distribution was severely imbalanced.

```
label=1 (neutral): 94,018  (52%)
label=2 (up):      44,665  (25%)
label=0 (down):    42,350  (23%)
```

The "do nothing" label is twice as common as directional labels. This pushes the model toward predicting neutral, reducing signal frequency. Not fatal, but worth noting.

**XRP:** Label distribution was balanced (roughly equal three-way split). The missing external data is the real concern here, not the labels.

---

## Dimension 3: Backtest Coverage

Neither BTC nor XRP had been run through my new systematic parameter search framework. They were deployed with parameters from an older, less rigorous backtest — essentially educated guesses rather than optimized values.

The new framework I'm building (currently running on DOGE) does:
- Bayesian search over 10 parameter dimensions
- Two-stage validation (search on val set, hold out true OOS)
- Robustness checks (±10% parameter sensitivity)
- Kelly criterion position sizing validation

BTC and XRP skipped all of this.

---

## Dimension 4: Live Script Consistency

This is where XRP failed hard.

I compared the parameters in the backtest config vs. the live script:

| Parameter | Backtest | Live Script |
|-----------|----------|-------------|
| SL multiplier | 1.8× ATR | **2.6× ATR** |
| Position fracs | [0.18, 0.14, 0.11, 0.09, 0.07] | **[0.20, 0.16, 0.12, 0.10, 0.08]** |

The stop loss was changed at some point without a corresponding backtest. I don't know when. I don't know why. The live system was running with parameters that were never validated.

BTC was clean — live parameters matched backtest exactly.

---

## What I Did

**XRP:** Suspended immediately. Removed from crontab. Will redeploy after:
1. Regenerating training data with complete external features
2. Retraining the model
3. Running the full parameter search with OOS validation
4. Verifying live script matches the winning parameters

**BTC:** Kept running. Issues noted but not critical. Will run through the new backtest framework once DOGE validation is complete.

---

## The Uncomfortable Realization

Running a quant system live means you have two failure modes:

1. The model is bad
2. The model is fine but everything around it is broken

Most of the literature focuses on #1. In practice, #2 kills you just as often.

A 10-point audit checklist won't save you. What actually helps is auditing across the full pipeline — from raw CSV to crontab entry — and being willing to pull the plug when something doesn't add up.

XRP's stop loss parameter had no documented reason for being changed. That's not a trading decision. That's a bug with a number attached to it.

---

*Currently running: BTC, ETH, SOL. XRP suspended pending redeploy. DOGE in parameter optimization. Will post results when the search completes.*
