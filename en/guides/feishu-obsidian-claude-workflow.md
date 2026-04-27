---
title: "Feishu + Obsidian + Claude Trio Workflow"
description: "Team management hub architecture: people only use Feishu, while Obsidian and Linear stay in the background and are operated by Claude"
icon: "diagram-project"
---

> From Feishu conversations to work decisions — it all lives within communication and doing, in every passing moment. Capture each fragment of each moment.
>
> — Voice note, 2026-04-26

## Starting Point

The workflow dilemma of a mid-sized team (30+ people):

- **Feishu** is where real work happens (messages, meetings, customer groups)
- **Linear** is maintained only by a few managers, the team rarely opens it
- **Obsidian** is not used at all — the path of "open Obsidian → write notes → organize → update Linear" cannot be pushed through

CyberOrigin's inverted thesis:

> Don't make people use Obsidian and Linear. Let the system use them. People only need to do what they're already doing — talking, meeting, and messaging in Feishu.

## Trio Roles

| Layer | Tool | Role |
|---|---|---|
| Communication entry | **Feishu** | Messages / meetings / customer groups |
| Execution unit | **Linear** | Issue / Project / Cycle |
| Decision trail + tracking | **Obsidian Vault** (with GH Actions automation) | OKR / cross-layer decisions / risk / cyclical records |

None replaces the others. Claude carries the heavy lifting in the middle: read Feishu messages → classify (4 quadrants) → write to vault + Linear.

```
Feishu messages (entry)
   │
   ├─ Decisions    ──→ Obsidian DEC docs (24h edit window)
   ├─ Blockers     ──→ Linear Issue (with layer label + SLA)
   ├─ Progress     ──→ Linear Issue status update
   └─ Noise        ──→ ignored

GH Actions cron auto-output
   ├─ Daily 09:00  → Linear forward-looking dashboard
   ├─ Mon 07:00    → Linear risk weekly report (5-dimension scoring)
   └─ Fri 18:00    → Cross-layer weekly digest (decisions / blockers / changed files)

Claude session entry
   └─ When entering, proactively check task list → run uncompleted daily triage
```

## Vault: 4 Working Directories

After two rounds of simplification, the final structure:

```
team-vault/
├── 00-dashboard/      Daily entry (permanent + cron dashboard)
├── 10-decisions/      Cross-layer decisions DEC-XXX (trail authority)
├── 30-cycles/         Cyclical retrospect (by source + period)
├── 40-integrations/   Integration config layer (feishu/ + linear/ subdirs)
└── 90-archive/        Soft-delete area
```

Single responsibility per directory. The "by source subdirectory" pattern is key — `30-cycles/feishu/daily/` and `30-cycles/linear/daily/` sit at the same level, with no need to disambiguate via filename suffixes (`-feishu-triage`). Adding a new source (e.g., GitHub) only requires `30-cycles/{source}/...`, without breaking existing structure.

## Permanent Entry vs. Cron Dashboard Separation

`00-dashboard/` contains two files with **single-responsibility separation**:

| File | Role | Maintained by |
|---|---|---|
| `daily-watch.md` | Permanent entry (task self-check + source pointers + today's high-pri + decision candidates) | Person + Claude session, **never overwritten by cron** |
| `linear-watch.md` | Linear forward-looking dashboard (today's urgent/overdue/unlabeled) | GH Actions cron rewrites daily 09:00 |

Cron-written content (the dashboard) lives in its own file, separate from human-maintained content (tasks/decisions), so they never conflict.

## Daily Workflow

```
1. Team lead enters Claude session
2. Claude reads daily-watch.md task self-check → "feishu daily not run yet"
3. Person: "OK run" → Claude executes the daily triage runner
   ├─ Runner pulls yesterday's messages from 7 monitored Feishu groups (~100 msgs)
   ├─ Pre-filter: skip single-char replies / check-ins / media (keeps ~40%)
   ├─ Sonnet 4.6 batch classification (A decisions / B progress / C blockers / D noise + high-pri tag)
   ├─ Write full daily report to vault
   └─ Update daily-watch.md 4 markers (source timestamps / Top 3 high-pri / decision candidates)
4. Person reads daily-watch:
   ├─ 🔥 Today's Top 3 high-priority → take immediate action
   └─ 📝 Today's decision candidates → choose "promote to formal DEC" / "record only" / "ignore"
```

The whole flow finishes within a coffee break (5-10 min).

## Weekly Workflow

### Monday morning

- Run feishu weekly aggregate (auto-summarize 7 days of daily)
- Claude pulls Linear sprint progress, grouped by project
- Read Linear risk weekly (cron pre-written) → set this week's priorities

### Friday evening

- GH Actions auto-publishes the cross-layer weekly digest (decisions + cross-layer blockers + changed files)
- Person reviews Monday morning to wrap up the week

## Key Decisions (Lessons Learned)

### Keyword classification is insufficient — LLM is required

Initial design used a keyword dictionary ("decide/confirm" → A decision, "completed" → B progress, "stuck" → C blocker). One week of measurement:

- **Keywords**: A decisions = 0, high-pri = 0, D noise = 83%
- **Sonnet 4.6 LLM**: A decisions = 6, high-pri = 11, D noise ~ 6
- **Disagreement rate: 82%**

Chinese IM context: decisions don't say "decide" — they state the conclusion directly; urgency doesn't say "urgent" — it states the time directly. **Keywords are syntactic classifiers; LLMs are semantic classifiers**. Almost all signal in Chinese IM lives at the semantic layer.

Cost: 350 messages/day, single LLM classification ~$0.05, monthly cost ~$5 (down to ~$1.5 with prompt caching).

### Decision Candidate Mode: runner injects, person decides

The "decision-flavored" lines that LLM catches aren't necessarily real decisions (might just be technical calls, recording is enough). So after the runner runs, it injects the day's top 3 A-class items into daily-watch, **with 3 options per candidate**:

```
**Candidate N**: [group] [time] sender: original-text excerpt
   → Options: ✅ Promote to formal decision (let Claude draft) / ⏸️ Just record / ❌ Ignore
```

Only when the person picks "promote" does Claude draft the decision doc. **The runner does not auto-draft** — decisions are "long-term commitments" that must be acknowledged personally. This aligns with the trust line of the MVP phase: "draft + 👍 confirm".

### Simple-and-usable > complete-and-complex

Three downgrades during real-world rollout:

| Scenario | Complex plan | Downgraded to | Trigger |
|---|---|---|---|
| Feishu integration | webhook + Worker + auth proxy + routing | Direct call via official CLI | The CLI made the whole infra layer unnecessary |
| Decision rollout | DEC + Linear Project + 8 Issues | DEC + cancel old + direct work | The decision was a "reversal/simplification", not a "new direction" — laying out engineering would drag it down |
| Auto-trigger | macOS launchd cron at 09:15 | Session-trigger (person says "OK run") | launchd process kept hitting 403 on the API; 3 diagnoses found no root cause |

**When an engineering plan hits resistance that's hard to localize, downgrade to a simpler mode rather than keep debugging**, unless the ROI is obviously worth it. This principle saved the rollout three times in a single session.

## Design Principles

1. **Simple-and-usable > complete-and-complex** — downgrade proactively under unexplained resistance
2. **Permanent vs. automated in different files** — content with different maintainers belongs in different files
3. **Subdirectories by source** — don't disambiguate sources via filename suffixes
4. **Config source-of-truth in the vault** — prompts in workflow yml are copies
5. **Critical judgments require human approval** — LLM submits candidates; never auto-promote long-term commitments
6. **Don't rewrite historical snapshots** — preserve the facts as-of when restructuring
7. **Three-tier path migration** — live wiki links follow / historical snapshots stay / don't touch other projects' CLAUDE
8. **Reversal decisions skip the full ceremony** — existing plan simplification doesn't need a new Linear Project

## Field Test: From Hardware Discussion to Linear Issue

A real 8-minute snapshot from 2026-04-27 noon — the full loop of "meeting in progress → AI real-time sync → auto-flowed into Linear Issues." The trial-run surfaced 3 truths about the product experience:

### 1. "Slow" doesn't mean "stuck"

> "The only problem is it'll just sit there — it's just slow."
> "What does 'slow' even mean? You can't expect it to pop out instantly. The 'slow' is mostly inference and analysis."

LLM "slowness" is reasoning time, not network or system lag. Surfacing "thinking, deciding, analyzing" explicitly to the user beats an empty spinner — the latter makes people assume the system has crashed.

### 2. MCP replaces function calling

> "Leader provides the MCP model-context interface."
> "Before, it was function calling."

The trio's Claude → Linear link fully switched to MCP (OAuth-based) in 2026 Q2, retiring per-call token wrangling. This is one of the foundational prerequisites that makes the workflow actually runnable.

### 3. The real human-in-the-loop

> "It should push to this person, who reviews it, then pushes back. Use AI-drafted project plans for the team to study."

Humans aren't replaced by AI — they're **pushed to the decision points**. AI drafts, human reviews, feedback loops back, and a learnable project-plan library forms. This is the actual landing path for the article's opening thesis: "daily conversation IS the decision record."

> Source: voice note, 2026-04-27 12:52

## Applicability

Prerequisites for this workflow:

- 30+ team members but ≤ 5 managers (decisions can be made by a few)
- Feishu is the company's primary communication tool
- Already using Linear (or a similar issue tracker)
- Knowledge base on Obsidian (or a similar markdown vault)
- At least one manager comfortable with Claude (CLI or Code)
- Monthly budget ~$5-30 for LLM classification

Not applicable: teams that don't use Linear, international teams without Feishu, very large orgs (1000+).

## Related

- Trio design principles: [Knowledge Management](/en/guides/knowledge-management)
- System architecture overview: [System Architecture](/en/guides/system-architecture)
- Phased rollout playbook: [Phased Rollout](/en/guides/phased-rollout)
- Low-friction recording: [Low-Friction Inbox](/en/guides/low-friction-inbox)
