---
title: "Low-Friction Inbox"
description: "How to make a team keep feeding information in — ultra-light input, AI-assisted async integration"
icon: "inbox"
---

## The Core Tension: A System Must Be Low-Friction to Stay Alive

The most common failure mode for team collaboration systems (weekly reports, Linear, knowledge bases):

- The more you demand from users, the lower the compliance
- Require users to classify, link, and fill fields on the spot → they bypass the system

Conclusion: **If you rely on humans to produce structured input, the system will die.**

The only viable path: **ultra-light input, AI-assisted asynchronous integration.**

---

## Division of Labor

| Step | Owner | Burden |
|------|-------|--------|
| Raw information capture | Team members | Minimal — drop it when it occurs |
| Classification / structuring / linking | AI (e.g. Claude) | Async batch |
| Final review / revision | Knowledge owner | Low-frequency, batched |

Users only do one thing: **drop raw information into the inbox**. No classification, no linking, no frontmatter required.

---

## Inbox Directory Convention

```
raw/curr/                          # Unified inbox
  ├── YYYY-MM-DD-<short-desc>.md   # Meeting notes, raw notes
  ├── lark-YYYY-MM-DD-*.md         # Exports from IM, prefix by source
  ├── wechat-YYYY-MM-DD-*.md
  └── voice-YYYY-MM-DD-*.md        # Voice-to-text
```

**Only one hard rule: the file lands under `raw/curr/`. Nothing else is enforced.**

Filenames can be arbitrary. Content format can be arbitrary. Sacrifice normalization for zero cognitive load on users.

---

## Input Methods (from Lowest to Highest Friction)

1. **IM bot (lowest friction)**: a Lark/Slack bot receives voice or text messages and auto-writes to `raw/curr/`
2. **Batch import**: periodic exports from existing doc platforms (Lark Docs, Notion)
3. **Manual paste**: drop a new md file with arbitrary name

The bot approach matters because **users don't change their existing habits**. Already discussing in IM? Keep doing that — the bot captures it.

---

## Rules for AI Async Integration

When invoked (or on schedule), AI processes the inbox:

1. **Classify**: determine which knowledge layer/directory the content belongs to
2. **Generate structured entries**:
   - Meetings → analysis doc + decision record if needed
   - Issue reports → issue-tracking directory
   - Device anomalies → device profile
3. **Archive originals**: move processed files to `raw/archive/YYYY/`
4. **Reverse-reference**: in the generated entry's `related` field, link back to the original

Benefit: **raw input is always traceable**. Even if AI integration is imperfect, humans can trace back to the source.

---

## Common Pitfalls

**❌ Pitfall 1: Require users to add tags/categories**
→ This pushes AI's job onto humans — back to the high-friction path

**❌ Pitfall 2: Split the inbox into subject-based subfolders**
→ Users now must judge "which folder does this belong to" — cognitive load returns. Keep a single entry point

**❌ Pitfall 3: Delete originals after integration**
→ AI classification won't always be right. Keep originals for traceability

---

## Summary

- Only one input rule: land in `raw/curr/`
- Classification, linking, structuring — all async by AI
- Originals retained, reverse-referenced, always traceable

Letting users keep their existing habits is the precondition for the system to stay alive.
