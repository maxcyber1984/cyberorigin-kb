---
title: "CyberOrigin System Architecture"
description: "Seven-layer system architecture of CyberOrigin, covering the full data flow from hardware manufacturing to customer delivery"
icon: "layer-group"
---

## CyberOrigin Full-Stack Architecture (Seven Layers)

<img src="/assets/cyberorigin_arch_1.svg" alt="CyberOrigin seven-layer system architecture diagram" />

## Core Design Questions per Layer

**L1 Hardware production** — Sensor selection (RGB-D vs pure RGB + computed depth), SoC choice, firmware OTA update strategy, ODM/OEM supply-chain risk management.

**L2 Operations channels** — The three channels differ sharply in data density and quality. Consumer-side has high volume but high noise; B2B is highly structured but narrow in scope; research-side is high-quality but limited in scale. Channel strategy directly determines data distribution.

**L3 Capture layer** — Multi-modal temporal synchronization is the core challenge (high-frequency IMU vs video frame rate vs low-frequency GPS). Hardware triggers vs software alignment on the device each have trade-offs.

**L4 Transport** — Edge compression (lossy vs lossless) has large downstream QC impact; resume-from-break state management across 5G/WiFi mixed scenarios; encryption at rest and compliance audit trails.

**L5 Cloud QC** — This is the platform's quality bottleneck, involving: false-rejection control in auto-QC, cost and scalability of HITL, annotation-schema version consistency, and PII detection/redaction.

**L6 Modeling & validation** — Evaluation metric design for world models (especially since EgoCentric scenarios have no mature benchmarks); closing the loop between algorithmic validation and real-data iteration.

**L7 Customer delivery** — Dataset format standards, latency/throughput SLAs for model APIs, and explainability at customer-side acceptance (how to make customers trust model quality).

## Related Reading

<CardGroup cols={2}>
  <Card title="Data Pipeline" icon="arrows-split-up-and-left" href="/en/guides/data-pipeline">
    End-to-end pipeline from edge capture to platform services
  </Card>
  <Card title="Edge Preprocessor" icon="microchip" href="/en/references/edge-preprocessor">
    L2 multi-modal compression and timestamping
  </Card>
  <Card title="Company Overview" icon="building" href="/en/company/overview">
    How architecture fits the company strategy
  </Card>
  <Card title="Data Collection Ops" icon="video" href="/en/guides/data-collection-ops">
    L3 capture-layer operations
  </Card>
</CardGroup>
