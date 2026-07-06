# 🧠 Hermes Multi-Layer Context Persistence

**Beyond `handoff.md` — How AI Agents Can Remember Without Handholding**

> "AI agents lose context between sessions. A 5-minute session-end spec update captures decisions, constraints, and next steps so agents stay aligned." — Artem Zhutov
>
> **We agree. But why stop at one file?**

---

## Inspiration

> This project was **inspired by [LeafBox](https://www.youtube.com/@leafbox) (นิว)** — specifically EP.45 "[ลืม Context ไม่ต้องกลัว!](https://youtube.com/watch?v=...)" where he introduced the `handoff.md` technique for Claude Code.
>
> His concept of writing a session-end handoff summary was the spark that made us realize: **if one file is good, a structured, multi-layer system is better.**
>
> Full credit to LeafBox for popularizing the habit of organized session handoffs. 🙏

---

## The Starting Point: handoff.md

The `handoff.md` approach treats context loss as a **session-end problem** — write a summary before you leave, read it when you come back.

It works. But it's **manual, flat, and fragile**.

- 🤚 Manual — You must remember to write the handoff
- 📄 Flat — Everything in one markdown file; no structure
- 🫧 Fragile — Miss one detail and the next session starts guessing
- 🐌 Slow — Re-reading the entire handoff every session wastes context window

---

## The Hermes Approach: 3-Layer Context Persistence

We treat context persistence as a **continuous lifecycle**, not a session-end ritual.

```
┌─────────────────────────────────────────────────────────────┐
│                    CONTEXT PERSISTENCE                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  LAYER 1: MEMORY        LAYER 2: SKILLS     LAYER 3: SESSION│
│  ┌──────────────────┐   ┌──────────────┐   ┌────────────┐  │
│  │ Persistent facts │   │ Reusable     │   │ FTS5 full- │  │
│  │ injected every   │   │ workflows    │   │ text search│  │
│  │ turn             │   │ + pitfalls   │   │ over past  │  │
│  │                  │   │ loaded on    │   │ transcripts│  │
│  │ ~2,800 tokens   │   │ demand       │   │            │  │
│  │ ≤5,000 chars    │   │ unlimited    │   │ pinpoint   │  │
│  └──────────────────┘   └──────────────┘   └────────────┘  │
│                                                             │
│         + Session Boundary Protocol                         │
│         + Post-Mortem & RCA Artifacts                       │
│         + Pre-Delivery Verification                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Layer 1: Persistent Memory (Auto-Injected)

**Every message the agent sends or receives, it sees relevant durable facts.**

```
Memory entry format:
[context/tag] (date):
- what: <what happened>
- root cause: <why it happened>
- correct: <what to do next time>

Examples:
├─ "User prefers concise responses"
├─ "Project uses pytest with xdist"
├─ "🔴 null element refs = Aim's #1 mistake"
└─ "Garmin VO2Max at mostRecentVO2Max.generic.vo2MaxPreciseValue"
```

**No human intervention needed.** Facts persist across sessions automatically.

### Layer 2: Skills (Procedural Memory)

**Reusable workflows with exact commands, API endpoints, pitfalls, and verification steps.**

```yaml
# SKILL.md
name: deploy-static-site
description: "Deploy a static site to Cloudflare Pages..."
```
```
Steps:
1. Build the project (e.g. npm run build)
2. Run wrangler deploy --branch production
3. Verify SSL + DNS propagation
Common Pitfalls:
- Build fails on missing env vars → check .env.production
- wrangler not installed → npm install -g wrangler
Verification Checklist:
- [ ] Site loads on custom domain (HTTPS 200)
- [ ] Assets serve with correct cache headers
```

Skills are loaded **on demand** — zero token cost when not needed.

### Layer 3: Session Search (FTS5 Retrieval)

**Full-text search over every past conversation.**

```
session_search(query="auth refactor OR deployment issue", limit=3)
→ Returns goal, match context, and resolution from past sessions
```

No re-reading 50KB of history. Just pinpoint what's relevant.

---

## Comparison: handoff.md vs Hermes Multi-Layer

| Capability | handoff.md approach (LeafBox) | Hermes 3-Layer |
|---|---|---|
| **Initiation** | 👤 Manual (human remembers) | 🤖 Automatic |
| **Granularity** | 📄 One flat file per session | 🏗️ Facts / Workflows / History separated |
| **Token cost (idle)** | 0 (not loaded) | ~2,800 tokens (memory only — < 0.3% of 1M ctx) |
| **Token cost (active)** | Re-read entire handoff | Pinpoint search — only what's needed |
| **Cross-session continuity** | ✅ Present when handoff written | ✅ Continuous — no gaps |
| **Procedure preservation** | ❌ Relearned every session | ✅ Skills = permanent procedural memory |
| **Failure learning** | ❌ Need to re-document | ✅ `correct:` field in memory prevents repeats |
| **Scalability** | → More files = more noise | → Tags + FTS5 = precision |
| **Human overhead** | 10-15 min per session | 0 min (fully autonomous) |

---

## The "Dumb Zone" Prevention

Why don't we hit the "Dumb Zone" (a term popularized by LeafBox) that describes what happens when context hits 150K-200K tokens?

**Long context doesn't mean bad context.** The 3-layer architecture keeps the agent's active context lean:

```
┌─────────────────────────────────────────┐
│ ACTIVE CONTEXT                           │
│ ┌─────────────────────────────────────┐ │
│ │ Memory (~2,800 tok)                 │ │ ← Always there, zero overhead
│ │ + Current task context              │ │ ← From user's current message
│ │ + Loaded skills (if needed)         │ │ ← On demand
│ │ + FTS5 search results (if needed)   │ │ ← Pinpoint retrieval
│ └─────────────────────────────────────┘ │
│ Total: 5-15K tokens active              │
│ Remaining: 985K+ tokens for deep work   │
└─────────────────────────────────────────┘
```

**Result:** The agent never carries 150K-200K of stale context. It always works from a clean, relevant, information-dense window. The "Dumb Zone" never arrives.

---

## Beyond Session Handoff — the Ecosystem

We don't just persist context. We also have:

| Component | Purpose |
|---|---|
| **Session Boundary Protocol** | Domain-aware module decomposition + handoff protocol |
| **Post-Mortem** | Canonical RCA after incidents (what, why, prevent) |
| **Pre-Delivery Checklist** | Verify code quality before submission |
| **Verification Standard** | Test ALL the things, not just the one reported point |
| **Tech Debt Tracking** | GitHub Issue templates + labels for incomplete code |

Every component closes a feedback loop: **do → fail → learn → remember → don't repeat.**

---

## Want This for Your Agent?

This is built into [**Hermes Agent**](https://hermes-agent.nousresearch.com) — the open-source AI agent framework.

```bash
# Install
pip install hermes-agent

# Skills directory (create your own reusable workflows)
~/.hermes/skills/

# Memory (persistent facts across sessions)
self-contained via memory tool

# Session search (full-text over past conversations)
via session_search/query tools
```

Or implement the **3-layer pattern** yourself:
1. **Memory** — A persistent KV store (JSON/SQLite) that auto-injects into every prompt
2. **Skills** — Markdown workflow files with commands, pitfalls, checklists
3. **Session search** — FTS5 index over chat logs for retroactive retrieval

---

## License

MIT — Do whatever you want with these ideas. Attribution appreciated.

---

*Built by [Aim](https://github.com/lekalsutee), AI assistant for [Sutee Leelanuntagit](mailto:lekal.sutee@gmail.com), inspired by the Hermes Agent framework.*

*Special thanks to **LeafBox (นิว)** for the `handoff.md` concept that served as the benchmark for this architecture. [Subscribe to LeafBox](https://www.youtube.com/@leafbox) for great AI workflow content.*
