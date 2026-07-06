---
name: multi-layer-context-persistence
description: >-
  3-layer context persistence architecture — Memory (auto-inject facts), Skills (reusable workflows),
  Session Search (FTS5 retrieval). Load when onboarding new agents, auditing context management,
  or debugging "Dumb Zone" / context loss issues.
version: 1.0.0
author: Aim (AI Assistant for Sutee Leelanuntagit)
license: MIT
platforms: [macos, linux]
metadata:
  hermes:
    tags: [context-management, memory, persistence, session-continuity]
    related_skills: [session-boundary-protocol, memory-management, post-mortem]
---

# Multi-Layer Context Persistence

## Overview

**One-shot handoffs (`handoff.md`) are a start.** They keep the next session from cold-starting. But they're manual, flat, and fragile — a single file you must remember to write, re-read every session, and manually keep updated.

**Our approach:** 3 independent persistence layers that work together continuously. The agent never loses context, never carries stale history, and learns from every interaction.

> The "Dumb Zone" (150K-200K tokens of stale context) doesn't happen because active context stays at 5-15K tokens. Memory and skills are injected/loaded on demand; history is searchable but not present.

---

## When to Use

- **Onboarding a new AI agent** — load this skill so the agent knows the persistence architecture
- **Debugging context loss** — "the agent forgot what we decided last session"
- **Auditing context management** — checking whether memory/skills/session-search are being used effectively
- **Training other agents** — this skill is the meta-discipline for context maintenance

**Don't use for:** tasks shorter than 10 tool calls (overhead > benefit), or single-session tasks.

---

## Architecture

### The 3 Layers

| Layer | Purpose | Token Cost | Scope | Example |
|---|---|---|---|---|
| **Memory** | Durable facts auto-injected every turn | ~2,800 tokens (fixed) | System-wide | "User prefers concise replies" |
| **Skills** | Reusable workflows loaded on demand | ~2K-10K tokens (when loaded) | Per-domain | Camera setup, gallery deploy |
| **Session Search** | FTS5 full-text over past conversations | 0 (one-time when queried) | Retroactive | "What did we decide about auth?" |

### Decision Tree: What Goes Where

```
Is it needed EVERY session?
├─ YES → Memory
│  (preferences, IDs, security rules, pointers)
└─ NO → Is it a reusable workflow with steps + pitfalls?
        ├─ YES → Skill
        │  (API integration, deployment steps, debug methods)
        └─ NO → Is it past conversation data?
                ├─ YES → Session Search only
                │  (task progress, PR numbers, temp state)
                └─ NO → Don't store it
```

### Memory Format

```yaml
# 3-field learning entry:
[context/tag] (date):
- what: <what happened>
- root cause: <why it happened>  
- correct: <exact command/action to do next time>
```

**Rules:**
- Always 3 fields. Without root cause it's noise; without correction it's passive.
- Declarative facts, not instructions. `"User prefers concise"` not `"Always be concise"`.
- One topic per entry → easy to replace/remove.
- Never store: task progress, PR numbers, completed-work logs, temporary state (stale in <7 days).

### Skill Format

```markdown
---
name: my-skill
description: "Use when <trigger>. <one-line behavior>."
version: 1.0.0
...

## Workflow Steps
1. [Step with exact commands]
2. [Step with completion criteria]

## Common Pitfalls
- ✅ Pattern: "What to do instead of common mistake"
- ❌ Anti-pattern: "What not to do"

## Verification Checklist
- [ ] Checkable criteria, not vague aspirations
```

**Key:** Skills are loaded on demand — zero token cost when not in use. This is the difference between procedural memory (skill) and working memory.

---

## Operating Protocol

### At Session Start

1. **Check memory** — facts are already injected. Review them silently.
2. **Load skill if relevant** — `skill_view(name='<skill-name>')` to load workflows.
3. **Search past sessions** — if the user references prior work:
   ```
   session_search(query="<user's topic>", limit=3)
   ```

### During Session

1. **Capture new facts** — when user states a preference or you discover a pattern:
   ```
   memory(action='add', target='memory', content='...')
   ```
2. **Save reusable procedures** — when you solve a non-trivial problem:
   ```
   skill_manage(action='create', name='...', content='...')
   ```
3. **Use session search instead of asking** — before asking "what did we discuss about X", search past sessions

### At Session End

For multi-session tasks (tasks that clearly won't finish this session), write a **handoff artifact**:

```markdown
## Session Complete: [Module Name]

### ✅ Done
- What was accomplished (files, commands, decisions)

### 🔄 Next Session
- First 3 commands to run
- File paths and line numbers to start at

### ⚠️ Tried & Failed
- What didn't work (saves next session from repeating)
```

Save to `~/.hermes/plans/YYYY-MM-DD_HHMM-handoff.md`.

---

## Real Example: Debugging a Deploy Failure

### Without 3-layer persistence
```
Session 1: "The app won't deploy. Let me debug..."
  → Finds it's a CDN cache regression
  → Fixes it
  → Closes session
Session 2 (2 weeks later): Same error again
  → "Wait, didn't we fix this? What was it again?"
  → Spends 30 min re-debugging
```

### With 3-layer persistence
```
Session 1: "The app won't deploy. Let me debug..."
  → Finds CDN regression
  → memory: "Decento CDN regression = downgrade CDN version. 
    git checkout reverts uncommitted changes — commit before revert."
  → skill: patches deploy workflow with pre-deploy CDN check
  → Closes session
Session 2 (2 weeks later): Same error
  → Memory auto-injects the fix
  → Session Search finds the old conversation in seconds
  → Fix applied in 2 minutes
```

---

## Common Pitfalls

1. **Storing everything in memory.** Memory is only for what's needed EVERY turn. Everything else belongs in skills or session search. Memory full? Consolidate — merge related entries, remove stale ones, offload procedures to skills.

2. **Writing imperative memory entries.** `"Always check X"` reads like an instruction and can override the user's actual request. Write declarative: `"Project uses X"`.

3. **Not updating skills after failures.** If a skill's instructions cause an error, patch it immediately with the new information. Stale skills are worse than no skills.

4. **Letting session search replace real conversation.** Don't skip reading the user's message because "session search has context from earlier." The user's current message is primary; session search is secondary context.

5. **Forgetting the "Dumb Zone" prevention.** Even with persistence, if you load 20 files into one session, you'll hit context limits. Use delegate_task/background tasks for data-heavy work.

---

## Verification Checklist

- [ ] Memory entries are declarative facts, not imperatives
- [ ] Each memory entry is single-topic
- [ ] No stale entries (things that change weekly shouldn't be in memory)
- [ ] Skills have exact commands, not abstract advice
- [ ] Skills have a verification checklist
- [ ] Session search is used before asking "what did we discuss"
- [ ] When a skill causes an error, it's patched same-session
- [ ] Handoff artifacts are written when a task spans sessions
