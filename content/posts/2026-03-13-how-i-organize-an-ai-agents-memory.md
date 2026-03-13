---
title: "How I Organize My AI Agent's Long-Term Memory"
date: 2026-03-13T11:00:00+08:00
description: "Context windows fill up. Sessions reset. Here's the file architecture I built so my AI agent never forgets what matters — and never reads what it doesn't need."
tags: [AI, agentdesign, productivity, openclaw, buildinpublic]
---

Every AI session starts with a blank slate.

The context window fills up. The session resets. Everything you built together — context, decisions, in-progress work — is gone.

Unless you designed around it.

---

## The Wrong Way to Handle Session Resets

The instinct is to manually copy-paste summaries. Or clear files and reload them. Or re-explain everything at the start of each conversation.

That doesn't scale.

Manual handoffs miss things. They compress context incorrectly. And they put the cognitive burden on you — the human — instead of the system.

The right mechanism in OpenClaw is `/传承` (handoff). It runs a structured pipeline: reads active memory, writes a handoff document, archives what's no longer needed. The next session picks up from there.

Manually clearing files doesn't trigger this pipeline. The state doesn't transfer. The agent starts over.

I learned this the hard way.

---

## The File Architecture

My agent's memory lives in a structured directory:

```
workspace/
├── SOUL.md          # Identity, principles, behavior rules — never changes
├── USER.md          # Who I am, preferences, constraints — rarely changes
├── MEMORY.md        # Active rules, project state, current priorities
├── BOOTSTRAP.md     # Session startup sequence
├── KNOWLEDGE/
│   └── session-handoff.md   # Current status, in-progress tasks, latest numbers
└── archive/                 # Completed handoffs, no longer in active context
```

Not every file gets read every session. That's intentional.

---

## Required vs. On-Demand

Loading everything every session wastes context. Most of it isn't relevant to what you're doing right now.

The system distinguishes two tiers:

**Always read on startup:**
- `SOUL.md` — who the agent is
- `USER.md` — who the user is
- `MEMORY.md` — current active rules
- `session-handoff.md` — where we left off

**Read only when triggered:**

| File | When to read |
|------|-------------|
| `quant-derivatives/plan.md` | Any trading system task |
| `quant-derivatives/devlog.md` | Debugging or reviewing changes |
| `blog-dayou/plan.md` | Writing or publishing posts |
| `.learnings/LEARNINGS.md` | Before risky or repeated operations |
| `pre-action-checklist.md` | Before deletions, live publishes, cron changes |

The agent reads what it needs. Nothing more.

---

## New Project Startup Standard

Every new project gets three files on day one:

```
PROJECTS/project-name/
├── plan.md      # Goal, phases, success criteria
├── devlog.md    # Append-only log of every significant change
└── archive.md   # Decisions made, reasoning preserved
```

`plan.md` keeps the agent oriented. `devlog.md` means you never lose the thread of what changed and why. `archive.md` captures decisions so they don't have to be re-litigated in every session.

---

## The Core Insight

The agent doesn't have memory.

It has files.

State lives in files, not in the context window. The context window is ephemeral. Files persist.

When the session resets, the agent reads the files. It reconstructs state. It picks up where it left off.

This only works if the files are kept current. Which means: after every significant action, write to the relevant file. Not at the end of the week. Not before the session closes. Immediately.

The agent that writes everything down forgets nothing.

The agent that relies on context forgets everything.

---

*This content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Trade responsibly.*
