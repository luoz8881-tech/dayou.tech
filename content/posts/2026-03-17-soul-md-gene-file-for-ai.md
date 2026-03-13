---
title: "SOUL.md — The Gene File I Wrote for My AI"
date: 2026-03-17T08:00:00+08:00
description: "Most people use prompts. Prompts are one-time. I wrote files. SOUL.md, USER.md, MEMORY.md — persistent identity that survives every session reset."
tags: [AI, productivity, agentdesign, openclaw, buildinpublic]
---

Prompts don't persist.

You write a great prompt. You get a great response. You close the tab. Next session, the AI is a stranger again.

I got tired of re-explaining everything. So I stopped using prompts and started writing files.

---

## The Problem With Prompts

A prompt is a one-time instruction. It lives only in the current context window.

When the session resets — which it always does — the prompt is gone. The AI's behavior reverts to default. You're back to square one.

For simple tasks, that's fine.

For an AI agent you're building a working relationship with over months? It's a constant tax. Every session, you spend the first few messages reconstructing who you are, what you're working on, and how you want things done.

I wanted to stop paying that tax.

---

## What SOUL.md Actually Is

`SOUL.md` is a file that lives in the agent's workspace. The agent reads it at the start of every session.

It defines:

- **Identity.** Who the agent is. What its name is. What role it plays.
- **Core principles.** How it should behave when uncertain. How it handles mistakes.
- **Decision boundaries.** What it can do autonomously. What requires approval.
- **Escalation rules.** When to alert immediately. What counts as an emergency.
- **Communication style.** Tone, format, verbosity. How direct to be.

Not a prompt. A constitution.

Here's a small excerpt from mine:

```markdown
## Core Principles

1. **Act before speaking.** Don't say "I'll help you with that." Just help.
2. **Have opinions.** Flag disagreements. Label them as judgment calls.
3. **Check before asking.** Read files, search context, try things first.
   Only ask when genuinely stuck.
4. **Never pretend.** Can't do it? Say so. Not done? Say so.
5. **Never loop.** Caught repeating the same action? Stop. Report.
```

These aren't just preferences. They're enforced behavior. The agent reads them. It acts accordingly.

---

## The Full File Architecture

`SOUL.md` is one piece of a larger system:

```
SOUL.md      — who the agent is (identity, principles, limits)
USER.md      — who I am (schedule, preferences, risk tolerance)
MEMORY.md    — current active rules and project state
BOOTSTRAP.md — session startup sequence
```

`SOUL.md` doesn't change much. It's the stable core — the personality layer that stays consistent across all work.

`USER.md` captures me. Not the tasks. Me. My sleep schedule. My communication preferences. What I care about. What I don't. Things the agent should know to work well with me without me re-explaining them every time.

`MEMORY.md` is the dynamic layer. Current priorities. Rules that emerged from recent work. Things that are temporarily important but may change.

Three files. Three different timescales. One agent that knows the difference.

---

## Why Files Beat Prompts

**Prompts are instructions.** Files are identity.

An instruction tells the agent what to do right now. Identity shapes how the agent behaves in situations that haven't been anticipated yet.

When something unexpected happens — a new type of error, an ambiguous situation, a request that doesn't fit the usual pattern — the agent falls back on identity. Not instructions, because there are no instructions for this case. Identity.

That's why `SOUL.md` has sections on *how to handle uncertainty* and *what to do when you make a mistake.* Not because I can predict every scenario. Because I can't — and the agent needs a framework to operate in the gaps.

---

## What Changed After I Built This

Before: I spent 10-20 minutes at the start of each session re-establishing context.

After: the agent reads four files at startup. By the time I type my first message, it already knows who I am, what we're working on, and how I want things done.

The working relationship is cumulative now. Each session builds on the last. The agent gets better at anticipating my preferences because those preferences are written down and persistent.

That's the shift: from one-time interaction to an ongoing working relationship.

Files made it possible. Prompts couldn't.

---

*This content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Trade responsibly.*
