---
title: "Company Overview"
description: "Who CyberOrigin is, what we do, our core technology bets, and our current stage"
icon: "building"
---

## Who We Are

CyberOrigin is a company focused on **embodied-intelligence data infrastructure**. We provide high-quality EgoCentric (first-person view) training data for robots and embodied AI systems, covering the full pipeline from on-site capture and annotation QC to structured delivery.

The team is 20-40 people, currently in mid-stage, with Shenzhen as our primary operating base.

---

## Core Technology Bets

Our core bet is built on three judgments:

**1. Data is the bottleneck for embodied intelligence**
The fundamental obstacle in robot policy training and world-model research today is not algorithms, but the scarcity of high-quality, diverse real-world interaction data.

**2. EgoCentric data has unique value**
First-person view data is the most natural form for training robots to "understand and execute human actions." Compared to third-person data, EgoCentric data has systematic advantages in hand-motion precision, spatial-relation perception, and task-step modeling.

**3. End-to-end pipeline engineering is the moat**
Capture capability alone isn't a barrier. Full-pipeline capability — from in-house capture hardware, standardized task systems, cloud data infrastructure, all the way to the annotation QC platform — is the asset that is genuinely hard to replicate.

---

## What We Do

<CardGroup cols={3}>
  <Card title="Data Capture" icon="video">
    Using our in-house CyberCap / CyberGlove wearables to perform standardized EgoCentric data collection in household and industrial settings
  </Card>
  <Card title="Annotation & QC" icon="tag">
    GV annotation and short-segment fine-grained annotation through the Bluebird Cloud platform, ensuring every record is traceable and meets training-grade specs
  </Card>
  <Card title="Structured Delivery" icon="database">
    Data lifecycle managed via Grace DB and the Cloud Tracking System, supporting large-scale delivery to multiple customers
  </Card>
</CardGroup>

---

## System Architecture Layers

CyberOrigin's technology stack is organized into seven layers:

| Layer | Name | Responsibility |
|---|---|---|
| L1 | Sensors & capture hardware | CyberCap, CyberGlove devices |
| L2 | Edge preprocessing | Local buffering, real-time compression, data validation |
| L3 | EgoCentric multi-modal capture | Video, IMU, hand-tracking synchronization |
| L4 | Cloud data pipeline | Upload, transcoding, storage, dedup |
| L5 | Cloud QC & annotation | Bluebird Cloud QC, short-segment labeling, ingestion |
| L6 | Data infrastructure | Grace DB, Cloud Tracking, query services |
| L7 | Model training consumption | Policy Training, world-model training |

Full architecture: [System Architecture Overview](/en/guides/system-architecture).

---

## Current Stage

**2026 Q2 — Infrastructure wrap-up + commercialization kickoff**

We're finishing the engineering of the capture → annotation → delivery pipeline, validating Policy Training feasibility, and establishing an external delivery cadence.

Product roadmap: [Roadmap](/en/company/roadmap).

## Related Reading

<CardGroup cols={2}>
  <Card title="Product Roadmap" icon="map" href="/en/company/roadmap">
    Current-stage goals, milestones, and long-term plans
  </Card>
  <Card title="System Architecture" icon="layer-group" href="/en/guides/system-architecture">
    Seven-layer technology architecture and data flow
  </Card>
  <Card title="Q2 2026 Gantt" icon="chart-gantt" href="/en/company/gantt">
    This quarter's project timelines and milestones
  </Card>
  <Card title="Onboarding Guide" icon="user-plus" href="/en/guides/onboarding">
    First-week ramp-up and tooling access
  </Card>
</CardGroup>
