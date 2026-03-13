---
title: "One Person. One AI. What We Actually Did Today."
date: 2026-03-17T09:00:00+08:00
description: "A real log of one day building a solo quant system with an AI partner. No team. No exaggeration. Just what happened."
tags: [AI, quant, buildinpublic, productivity, crypto]
---

I keep seeing posts about "building with AI" that read like marketing copy.

Here's what it actually looks like.

One day. Logged honestly. No polish.

---

## Morning: System Audit

**08:30**

I asked the agent to cross-audit the V5 trading system. Scope: training, backtest, and live execution code. All 13 files. 28 audit dimensions.

I went to make coffee.

**09:15**

The agent came back with a structured report. It found three issues:

1. The trailing stop was updating state every bar but never triggering exits.
2. SL placement failures were being logged silently — no alert.
3. The live shell scripts had no concurrency guard.

I reviewed the report. All three were real. None were obvious from reading the code casually.

I approved the proposed fixes.

---

## Midday: Fixes and Testing

**10:30**

The agent wrote the trailing stop exit check:

```python
if trail_sl is not None:
    hit_trail = (side == 'long' and bar_low <= trail_sl) or \
                (side == 'short' and bar_high >= trail_sl)
    if hit_trail:
        do_close('TrailingStop', f'{trail_sl:.4f}')
```

Added the WeChat alert on SL failure. Added `flock` to the shell wrapper.

I reviewed each change. Approved. The agent pushed the files to the server.

**11:00**

I went to a government meeting. (I have a day job. This is a side project.)

The agent waited.

---

## Afternoon: Verification

**14:00**

Back. I asked: did the patched code run clean?

The agent checked the server log. The 13:30 and 13:45 cron cycles had both completed successfully. No errors. No missing exit checks.

I asked it to tail the last 50 lines of the XRP log.

One thing I noticed: the trailing stop had activated on XRP but hadn't triggered yet — price hadn't crossed it. That was correct behavior. For the first time.

**14:30**

The agent drafted three blog posts based on what happened this morning. I reviewed them. Minor edits. Approved.

---

## Evening: Live Execution

**17:30**

First live cycle after the patch.

XRP momentum fade triggered. Position closed.

Not a trailing stop exit — that'll come when price gives me one. But the exit system ran end-to-end for the first time without issues.

The agent logged the event:

```
[17:30] XRP: MomentumFade exit | score: 0.31 | price: 0.5814 | result: +0.4%
```

Small win. Correct behavior. That's what I needed to see.

---

## What the Agent Actually Did Today

- Audited 6,500 lines of code across 13 files
- Identified 3 real bugs, none of which I'd noticed
- Proposed and wrote the fixes
- Verified the patched code ran clean on the server
- Drafted 3 blog posts
- Monitored the 17:30 execution cycle
- Logged the day's events to the project devlog

**What I did:**

- Reviewed and approved the audit report
- Reviewed and approved each code change
- Reviewed and approved the blog drafts
- Went to a meeting
- Made decisions when decisions were needed

---

## The Honest Part

This didn't happen in one smooth session. There were gaps where I was unavailable. There were moments where I asked for something and got something slightly off and had to redirect.

The agent doesn't get things right 100% of the time. I don't either.

But the total work output of this day — code audit, three bug fixes, server verification, three blog posts, devlog — is more than I'd have managed alone in a week of evenings.

That's the real number.

One person. One AI. That's what we actually did.

---

*This content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Trade responsibly.*
