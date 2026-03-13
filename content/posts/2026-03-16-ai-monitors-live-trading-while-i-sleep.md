---
title: "My AI Monitors Live Trades While I Sleep"
date: 2026-03-16T09:00:00+08:00
description: "My quant system fires every 15 minutes on a remote server. I don't watch it. My AI does — heartbeats, SSH checks, WeChat alerts."
tags: [AI, quant, crypto, automation, buildinpublic]
---

My quant system runs every 15 minutes.

On a remote server in the cloud.

Trading five coins simultaneously.

I'm not watching it.

---

## The Problem With Running Unattended

A live trading system running without supervision is a liability.

Scripts crash silently. Cron jobs overlap. API calls fail without alerts. Stop-loss orders get rejected and nobody notices until the position is deeply underwater.

The traditional solution: hire someone to watch it. Or watch it yourself.

I'm a solo operator. I have a day job. I sleep.

So I built a different solution.

---

## What the Agent Does While I'm Asleep

My AI agent — 豆豆 (Doudou) — has direct access to the server.

Not to trade. Not to change parameters. Just to observe and alert.

Here's what it monitors:

**Heartbeat checks.**
The live trading scripts write a timestamp to a log file after every successful execution. The agent checks that timestamp. If the last run was more than 20 minutes ago, something went wrong. Immediate WeChat alert.

**SSH status checks.**
The agent can SSH into the server to check process status, disk usage, and memory. If the server is unreachable, that's also an alert.

**Log parsing.**
After each execution cycle, the agent can read the tail of the log file and flag anything that looks like an error — failed orders, NaN warnings, unexpected exit reasons.

**SL placement failures.**
If an exchange stop-loss order fails to set, the position is running without protection. The agent escalates this immediately, regardless of time of day.

---

## How Alerts Work

Everything comes through WeChat — specifically through a work group I set up for the system.

The agent uses a Python push script:

```python
# scripts/send_wecom.py
# reads from stdin, posts to WeChat Work webhook
```

When the trading system detects something worth flagging, it calls the script. When the agent detects something during a monitoring check, it also calls the script.

Different sources. Same channel. Same phone notification.

I wake up to a summary. Not a panic.

---

## What I Actually Wake Up To

Most mornings: nothing. The system ran clean overnight.

Some mornings: one or two messages.

- "XRP trailing stop hit at 0.5821, position closed."
- "ETH SL placement failed on last bar, no active stop — check manually."
- "Server heartbeat missed at 03:15. Recovered at 03:30. Likely API timeout."

The second one requires action. The first and third are informational.

The agent doesn't make the distinction for me — I decide what to act on. But it gives me everything I need to make that call in under two minutes.

---

## The Design Principle

I'm not trying to automate judgment. I'm trying to automate observation.

Humans are bad at sustained attention. We miss things. We get tired. We rationalize.

Automated systems are good at sustained attention. They check the same thing every 15 minutes without drifting.

The agent handles observation. I handle judgment.

That division is what makes this actually work.

---

## What I Didn't Automate

The agent cannot:

- Change position sizes
- Modify live trading parameters
- Add or remove coins from the active list
- Execute trades directly

These require explicit approval from me. The agent can propose. I execute.

The boundary matters. An autonomous system that can modify itself or its parameters is a liability. An autonomous system that only observes and alerts is a tool.

I want a tool.

---

*This content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Trade responsibly.*
