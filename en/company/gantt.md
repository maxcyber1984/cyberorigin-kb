---
title: "Q2 2026 Project Gantt"
description: "CyberOrigin 2026 Q2 project timelines and milestones"
icon: "chart-gantt"
---

## Overview

**Cycle**: 2026-04-01 → 2026-06-30  |  **Core theme**: wire up the capture→annotation→delivery pipeline, commercialization kickoff

Three-phase rhythm: **April sprint execution** → **May system integration** → **June wrap-up delivery**

---

## Gantt

```mermaid
gantt
    title CyberOrigin Q2 2026 Project Timeline
    dateFormat YYYY-MM-DD
    axisFormat %m/%d

    section Data Infrastructure O1
    Data Infra Structure delivery      :active, dis,  2026-04-01, 2026-06-15
    CI/CD dedicated machine online     :active, cicd, 2026-04-01, 2026-04-30
    Grace video query perf <200ms      :active, grc,  2026-04-15, 2026-05-15
    Cloud Tracking System online       :active, cts,  2026-04-15, 2026-05-31
    Ray MLOps full takeover            :active, ray,  2026-05-01, 2026-06-15

    section External Delivery O2
    BIT 16K delivery complete          :active, bit,  2026-04-01, 2026-04-30
    Standardized delivery checklist    :        chk,  2026-04-15, 2026-05-31
    Q2 external delivery x2            :        del,  2026-05-01, 2026-06-30

    section Annotation & QC O3
    QC data auto-ingest                :active, qc,   2026-04-01, 2026-04-30
    Annotation platform industrial     :active, ann,  2026-04-15, 2026-05-15
    Annotation-to-DB latency <24h      :        e2e,  2026-05-01, 2026-06-30

    section Hardware & MoCap O4
    Glove Mujoco integration           :active, glv,  2026-04-01, 2026-05-31
    CyberCap2+ stability (fault <5%)   :        cap,  2026-04-15, 2026-05-31
    MoCap phase-2 plan finalized       :        mc2,  2026-05-01, 2026-06-30

    section Policy Training O5
    GV Policy Training task validation :        pt,   2026-05-01, 2026-06-30
    Reproducible pipeline docs         :        ptd,  2026-05-15, 2026-06-30

    section Team Building O6
    Engineering key hires              :        hire, 2026-04-01, 2026-05-31
    Onboarding standardization         :active, onb,  2026-04-01, 2026-05-31

    section External PR O7
    WebKB public launch                :active, kb,   2026-04-01, 2026-04-30
    Technical articles x2              :        art,  2026-04-15, 2026-06-30
    Industry events                    :        evt,  2026-05-01, 2026-06-30
```

---

## Key Milestones

| Date | Milestone | Owner | Priority |
|---|---|---|---|
| 2026-04-30 | BIT 16K data delivery complete | Shiqi | P0 |
| 2026-04-30 | CI/CD dedicated machine online | chengpei | P0 |
| 2026-04-30 | QC data auto-ingest | Shiqi | P1 |
| 2026-04-30 | WebKB public launch | Max | P1 |
| 2026-05-15 | Grace query response < 200ms | Shiqi | P1 |
| 2026-05-15 | Annotation platform industrial | Shiqi | P1 |
| 2026-05-31 | Cloud Tracking System online | Shiqi | P0 |
| 2026-05-31 | Glove Mujoco integration complete | lizi | P1 |
| 2026-05-31 | Engineering key hires | Max | P1 |
| **2026-06-15** | **Data Infra Structure delivery (hard deadline)** | **Max** | **P0** |
| 2026-06-15 | Ray MLOps full takeover | Max | P0 |
| 2026-06-30 | Policy Training initial validation | Shiqi | P2 |
| 2026-06-30 | MoCap phase-2 plan finalized | Yu Liang | P2 |
| 2026-06-30 | Q2 retro & Q3 planning kickoff | Max | — |

---

## Phase Highlights

### April (sprint execution)

Launch the tasks that ship results fastest and kill all manual workflows. BIT 16K and WebKB are the month's two externally visible delivery points.

### May (system integration)

All systems run end-to-end: capture → ingest → query fully automated. Cloud Tracking System is May's main thread.

### June (wrap-up delivery)

Data Infra Structure must ship by 6/15 — the only hard deadline this quarter. Late June shifts into Q3 planning.

---

## Full OKR Document

Detailed Key Results and status: [Product Roadmap](/en/company/roadmap).

## Related Reading

<CardGroup cols={2}>
  <Card title="Product Roadmap" icon="map" href="/en/company/roadmap">
    Current-stage goals and quarterly milestones
  </Card>
  <Card title="Company Overview" icon="building" href="/en/company/overview">
    Who we are, what we do, core technology bets
  </Card>
</CardGroup>
