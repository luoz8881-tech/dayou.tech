---
title: "It Took Me 50 Days. It Should Take You 3."
date: 2026-03-09T21:00:00+08:00
description: "Every mistake I made building an AI quant system from scratch — so you don't have to."
tags: [crypto, quant trading, AI, tutorial, Bybit]
categories: [Quant Trading]
draft: false
---

*Every mistake I made building an AI quant system from scratch — so you don't have to.*

---

January 20, 2026. I started from zero.

Couldn't write a single line of code.

I just knew one thing: I was done letting humanity make my trading decisions.

---

**Step 1: The Server**

Asked AI where to start. Chose Google Cloud, Singapore region.

Bought an instance. Opened ports. Set up SSH.

The first time I connected from my phone, I felt like I'd come a long way.

I hadn't even started.

---

**Step 2: The AI Tools**

Started with basic chatbots. Good for questions, not deep enough for code.

Moved to DeepSeek. Much stronger for writing scripts.

Eventually found Claude. Complex problems, long codebases, system architecture — all of it.

At the same time, I deployed OpenClaw on the server — an AI agent running 24/7. It handles research, workflows, and helps run this blog. The quant system trades. OpenClaw handles everything else.

Can't code doesn't mean can't build. AI is your hands.

---

**Step 3: The Framework Trap**

I assumed ready-made quant frameworks would save time.

Deployment failed. Tried another. Failed again. Network issues, environment conflicts, compatibility problems.

Then I realized: those frameworks are built for institutions. Too heavy. Too complex. Way more than one person needs.

Threw them all out. Built from scratch. Lightweight, custom, controllable.

---

**Step 4: The Data**

Assumed more external data meant better results. Paid for several sources. All failed.

Ended up using only Bybit's own data — candlesticks, funding rates, open interest.

Clean. Reliable. Enough.

Lesson: don't pay for data you can't actually use.

---

**Step 5: The Models**

Changed models more times than I can count.

Random Forest — signals too weak.
LightGBM — better, still not enough.
Transformer — right direction.
TFT — too complex, not worth it.
Ensemble approaches — over-engineered, somehow worse.

Came back to lightweight Transformer.

The simplest one. The right one.

---

**Step 6: Backtest Lied to Me Three Times**

First time — Sharpe ratio off the charts. Overfitting.
Second time — stop-loss logic was broken. Results were fake.
Third time — no real high/low data. SL/TP never actually triggered.

Every time I thought it was working. Every time it wasn't.

Fourth version. Walk-forward validation. Real price data.

Finally real.

---

**Step 7: Going Live**

System errored. Then errored again. Then again.

Checkpoint loading failed. API key not found. Feature dimensions mismatched. Old processes still running.

Four bugs. One after another.

Fixed all of them. System went live.

---

It took me 50 days to walk through all of this.

With this post, you don't have to.

The next posts will cover every step in full — server setup, data pipeline, model training, backtesting, live deployment.

Just follow along.

3 days is enough.

---

---

*All content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Past performance is not indicative of future results. Trade responsibly and never risk more than you can afford to lose.*

*Questions: hi@dayou.tech*
