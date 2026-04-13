---
title: "Onboarding Guide"
description: "Your first week at CyberOrigin: system orientation, tooling access, and collaboration norms"
icon: "user-plus"
---

## Welcome

CyberOrigin is a company focused on **embodied-intelligence data infrastructure**, providing high-quality EgoCentric (first-person view) training data for robots and embodied AI systems. This guide helps you get up to speed on how the team works in your first week.

---

## Day 1: Understand the Company

Read these three documents first to build baseline context:

1. [Company Overview](/en/company/overview) — who we are, what we do, core technology bets
2. [Product Roadmap](/en/company/roadmap) — current-stage goals and quarterly milestones
3. [System Architecture](/en/guides/system-architecture) — seven-layer architecture and data flow

**Key concept glossary:**

- **EgoCentric data**: first-person-view human action data, the core raw material for training robot policy models
- **GV (Glove Video)**: synchronized glove + video capture, used for fine-grained hand-motion modeling
- **Short-segment annotation**: fine-grained temporal segmentation and classification of GV videos
- **Grace DB**: our in-house core database, managing all capture and annotation data
- **Cloud Tracking System**: full-pipeline tracking system for video data

---

## Day 2: Tooling Access

### Required Tools

| Tool | Purpose | How to get it |
|---|---|---|
| Linear | Task management, Sprint tracking | Ping Max for an invite |
| Obsidian | Knowledge base, meeting notes, docs | Install + join the shared Vault |
| Bluebird Cloud | QC annotation platform | Ping Shiqi for an account |
| GitHub | Code and docs version control | Ping the engineering lead |

### Obsidian Vault Access

The team knowledge base is shared via Obsidian Vault + Git. Setup steps:

```bash
# Clone the Vault (ping Max for the repo URL)
git clone <vault-repo-url>

# Open the folder in Obsidian
# File → Open Vault → select the cloned folder
```

Vault directory structure: see [Knowledge Management](/en/guides/knowledge-management).

---

## Day 3: Data Capture Flow

CyberOrigin's core business is **EgoCentric capture → annotation → delivery**. Get familiar with:

**Capture phase**
- Collectors wear CyberCap (head-mounted camera) and CyberGlove (glove sensors)
- Execute operational tasks in standardized settings (16 household tasks / industrial scenes)
- Data is uploaded to the cloud after edge preprocessing

**Annotation phase**
- GV annotation (video segmentation) and short-segment fine-grained annotation done on Bluebird Cloud
- Annotation specs: [EgoCentric Annotation](/en/references/egocentric-annotation)
- Bluebird Cloud operations: [Bluebird Cloud Platform](/en/references/laniao-cloud)

**Delivery phase**
- Annotated data is auto-written into Grace DB
- Delivery state is managed via the Cloud Tracking System
- Export per customer requirements (HDF5 / JSON etc.)

---

## Day 4: Engineering Norms

### Looking Up Technical Decisions

Major technical decisions are recorded under `02-Engineering/Decisions/` (ADR format). When you hit a "why was this done this way?" question, start with INDEX.md.

### Common Questions

**Why TOS instead of Aliyun OSS?**
Q2 2026 completed the migration from Aliyun OSS to TOS — decision made after weighing cost and performance.

**What kind of database is Grace DB?**
Grace is our in-house data management system, built on a relational DB, supporting video metadata queries and annotation state tracking. API docs under `02-Engineering/API/`.

**What are the annotation QA standards?**
See the QC standards and 17 defect codes in [EgoCentric Annotation](/en/references/egocentric-annotation).

---

## First-Week Checklist

```
□ Finished reading company overview, roadmap, system architecture
□ Linear account activated, joined the relevant team
□ Obsidian Vault cloned locally
□ Bluebird Cloud account provisioned (if your role includes annotation)
□ Understood the current Sprint goals (Linear → Cycles)
□ First 1:1 with your direct manager
□ Wrote your first note into the Vault's _Inbox/
```

---

## Need Help?

- **Product / strategy**: Max
- **Annotation / delivery**: Shiqi
- **Engineering issues**: chengpei
- **Hardware / devices**: lizi
- **MoCap**: Yu Liang

## Related Reading

<CardGroup cols={2}>
  <Card title="Knowledge Management" icon="brain" href="/en/guides/knowledge-management">
    Vault structure and writing conventions
  </Card>
  <Card title="Company Overview" icon="building" href="/en/company/overview">
    Day-1 company context
  </Card>
  <Card title="System Architecture" icon="layer-group" href="/en/guides/system-architecture">
    Seven-layer architecture and data flow
  </Card>
  <Card title="Product Roadmap" icon="map" href="/en/company/roadmap">
    Current-stage goals and milestones
  </Card>
</CardGroup>
