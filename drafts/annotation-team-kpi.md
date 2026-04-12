# Annotation Team KPI Framework

How CyberOrigin manages annotation and QC team performance: dynamic quotas, quality gates, tiered bonuses, and zero-tolerance policies.

## Design Philosophy

Quality gates before quantity rewards. If your output doesn't pass the quality bar, your volume is irrelevant — no bonus, no matter how much you produce. This prevents the classic failure mode where annotators optimize for speed at the expense of accuracy.

## Daily Quotas

Quotas are measured in **effective media duration** — the length of video/audio content processed, not hours spent at the desk.

| Role | Daily Quota |
|---|---|
| QA (Quality Assurance) | 19 hours of media |
| Annotator | 2 hours of media |

The ~10x gap between QA and Annotator quotas reflects the difference in work density: QA is rapid pass/fail review, while annotation is frame-level labeling at 2-5 second granularity.

## Dynamic Adjustment

Quotas are not static. The annotation software iterates frequently and material complexity varies across projects, so KPI targets follow a dynamic adjustment mechanism:

- **Authority**: Project leads can adjust targets based on measured test data
- **Cadence**: Standards update every Monday, with possible mid-week revisions on Wednesday
- **Enforcement**: New standards take effect immediately upon written notice

## Tiered Bonus System

Bonuses are calculated daily, based on how much output exceeds the daily quota:

| Excess | Daily Bonus |
|---|---|
| 10% to < 20% | 5 RMB |
| 20% to < 30% | 20 RMB |
| 30% to < 50% | 50 RMB |
| >= 50% | 80 RMB |

Rules: highest applicable tier only (no stacking). Quality must pass the threshold — if your accuracy falls below the passing line, the entire day's bonus is forfeited.

## Disciplinary Framework

### Attendance and Output

| Strike | Consequence |
|---|---|
| 1st | Verbal or written warning |
| 2nd | Public notice + 20 RMB penalty |
| 3rd | Termination |

### Zero-Tolerance Red Lines

The following issues trigger termination after 3 formal warnings with no improvement:

- **Quality**: Accuracy below passing threshold, no improvement after coaching
- **Attitude**: Negative behavior, poor work ethic, refusal to follow reasonable instructions
- **Workspace**: Dirty workstation, littering, failure to meet 5S standards
- **Waste disposal**: Leaving food waste or takeout containers in the office area overnight

### Asset Liability

Equipment damage due to personal negligence or intentional misuse requires full replacement cost reimbursement at purchase price.

## Improvement Incentive

A "Suggestions of Excellence" program encourages bottom-up process improvement:

- **Scope**: Workflow optimization, annotation/QC tool improvements, management efficiency
- **Review chain**: Employee submits → supervisor screens → project manager approves → deployed
- **Reward**: 10 RMB per adopted suggestion, with the possibility of additional bonuses for high-impact proposals
