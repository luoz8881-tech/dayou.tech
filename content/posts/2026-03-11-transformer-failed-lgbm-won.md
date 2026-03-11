---
title: "Transformer Failed. LightGBM Won. Here's What Actually Happened."
date: 2026-03-11T10:00:00+08:00
description: "I spent weeks training Transformer models for crypto prediction. val_acc stuck at 54%. Then I switched to LightGBM. One day later: 67.5%. Here's the full story."
tags: [quant, machine learning, LightGBM, Transformer, crypto, backtest]
---

I'll skip the theory. Here's what actually happened.

## The Problem

I was building a crypto price prediction model for SOL, ETH, and BTC perpetual futures. The plan was simple: train a Transformer on 70 features, get a good signal, trade it live.

Three weeks of training later:

- **SOL Transformer v3b**: train_acc = 69.3%, val_acc = 54.1%, gap = 15.2%
- **ETH Transformer v3b**: train_acc = 78.6%, val_acc = 55.9%, gap = 22.7%

The gap between training and validation accuracy was 15-22%. Classic overfitting. The model was memorizing patterns, not learning them.

I tried everything:
- Reduce d_model from 128 to 64
- Increase dropout from 0.1 to 0.25
- More regularization

Nothing worked. The gap stayed.

## The Switch

I asked myself a simple question: *Is this a model problem or a data problem?*

To answer it, I needed a baseline. Something that couldn't overfit as easily.

**LightGBM.**

The same 70 features. The same train/val split. No architecture tweaking. Just throw it in and see what happens.

Results in one training run:

| Model | val_acc | gap |
|-------|---------|-----|
| SOL Transformer | 54.1% | 15.2% |
| **SOL LightGBM** | **58.1%** | **11%** |
| ETH Transformer | 55.9% | 14.3% |
| **ETH LightGBM** | **60.5%** | **14.3%** |
| **BTC LightGBM** | **67.5%** | **2.4%** |

BTC was the surprise. 67.5% val_acc with only a 2.4% gap. Almost no overfitting.

## Why LightGBM Won

Three reasons:

**1. It handles tabular data better.**

Transformer was designed for sequences. Financial features at 15-minute intervals aren't really sequences in the way text is. LightGBM's tree structure handles the interaction between features like RSI, ATR, and liquidation data more naturally.

**2. Missing values aren't a problem.**

Some external data (Bybit retail sentiment) only started in October 2024. The Transformer struggled with zero-filled gaps. LightGBM handles NaN natively — it just routes samples differently at splits.

**3. Feature importance tells you what's actually working.**

After training, I got a ranked list. Top 3 features across all three symbols:

1. `return_1_4h` — short-term momentum on the 4-hour timeframe
2. `liq_long_ratio` — proportion of long liquidations in total liquidations
3. `hour_cos` — time of day (cosine-encoded)

That `hour_cos` in the Top 3 was unexpected. It means the model learned that certain hours produce more reliable signals. European/US session crossover hours. Makes sense.

## What This Means

The features have real signal. The Transformer just couldn't extract it without overfitting.

LightGBM could.

That's the answer to the "model or data" question: **model problem**.

## The Backtest (Fixed)

I ran five versions of the backtest engine before getting numbers I trusted. Each version fixed a new bug:

- v1: Sharpe = 50. Monthly return = 1363%. Not real.
- v2: Sharpe = 14. Still wrong.
- v3: SL/TP using close price only (not high/low). Win rate inflated to 87%.
- v4: Same 4h signal triggered 16 times per bar. Sharpe stayed high.
- **v5 (final)**: Fixed all of the above. Added real fee model (0.035% with 30% rebate).

Final results with fixed backtest, fixed capital per position, no compounding:

| Symbol | Monthly | Win Rate | P/L Ratio |
|--------|---------|----------|-----------|
| SOL | 64% | 78% | 2.13 |
| ETH | 56% | 86% | 1.82 |
| BTC | 34% | 72% | 3.25 |

These numbers are based on $143 per symbol at 10x leverage. Not fantasy compounding.

## What's Live Now

All three are running. 15-minute signals. Pyramid position sizing. Automatic SL/TP via Bybit. Reverse signal triggers full close.

Today is day 1.

I'll report actual results in a future post. If the live win rate matches the backtest — around 72-78% — the system works. If it doesn't, I'll say so.

---

*No predictions. Just data. Follow [@dayou_tech](https://x.com/dayou_tech) for updates.*
