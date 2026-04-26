---
title: "Onboarding Feedback Loop"
description: "Keep onboarding docs alive by forcing each new hire's experience to feed back into the templates"
icon: "rotate"
---

## Why Onboarding Docs Always Go Stale

Most teams' onboarding docs share one problem: **written carefully once, never updated.**

Root cause — missing feedback loop:

- The person writing the doc is not the person joining
- The person joining has no place to report pitfalls
- No feedback channel → the next hire hits the same pitfalls

The goal of the onboarding feedback loop: **every new hire's onboarding experience becomes input for the template.**

---

## Collection Structure

Add a `_feedback/` subfolder under onboarding:

```
onboarding/
  ├── <role>-guide.md          # Role-specific onboarding guide (main doc)
  └── _feedback/
      ├── README.md            # Rules and template
      └── YYYY-MM-DD-name-role.md
```

Every new hire submits one feedback entry within 30 days of joining. Filename format is standardized.

---

## Feedback Template

```markdown
---
name: {{name}} Onboarding Feedback
onboarding_start: YYYY-MM-DD
role: {{role}}
team: {{team}}
status: {{in-progress / ramped-up / left}}
---

## Onboarding Timeline
- Day 1–3: ______
- Week 1: ______
- Week 2–4: ______

## Friction Points (where was it slow / confusing)
1. ______

## Missing Docs / Resources
1. ______

## Suggestions for the Onboarding Template
1. ______

## Did This Trigger a Template Update
- [ ] Updated the role's main doc
- [ ] Related PR / commit: ______
```

**The two critical fields are "friction points" and "missing resources"** — these convert most directly into concrete doc changes.

---

## Iteration Cadence

- **End of each month**: aggregate all feedback, perform one template revision
- **Same friction appears ≥ 2 times**: must enter the main doc (not "should" — "must")
- **Recurring systemic gaps**: promote to a standalone guide page, linked from onboarding

---

## What Breaks the Loop

**❌ Too many feedback fields**: new hires won't complete it. Keep the template under 5 fields
**❌ No one aggregates**: feedback sits unread. Assign an owner (usually the team's knowledge lead)
**❌ Feedback doesn't trigger revision**: collected but the main doc never changes. Use the "2 occurrences = must update" rule
**❌ Only collect problems, not wins**: the doc becomes a warning list. Also collect "what was especially helpful"

---

## Minimum Viable Implementation

If a team is just starting, don't over-engineer:

1. **Step 1**: create `_feedback/` + a 5-field template
2. **Step 2**: the next hire must fill it once
3. **Step 3**: end-of-month aggregate + one revision
4. **Step 4**: after 3 consecutive monthly revisions, then refine the process

Get the loop spinning first. Optimize later.
