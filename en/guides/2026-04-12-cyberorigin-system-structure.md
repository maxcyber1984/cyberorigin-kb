---
title: "Egocentric Capture System Architecture"
description: "Full-stack architecture for egocentric multi-modal data capture: from edge preprocessing through cloud ingestion, real-time processing, and storage to platform services."
icon: "layer-group"
---

## Overview

CyberOrigin's egocentric capture pipeline is a full-spectrum hybrid architecture designed to handle video, IMU, eye gaze, audio, GPS, and tactile streams at varying frequencies and encodings. The pipeline spans five layers: edge/device, cloud ingestion, processing, storage, and platform services.

## Edge / Device Layer

Every sensor produces a different stream with its own rate, encoding, and precision. Video comes off a rolling shutter camera at 30–60 fps; IMU pushes at 200 Hz; eye gaze at 120 Hz; audio at 16–48 kHz; GPS at 1–10 Hz; and tactile at anywhere from 100 Hz to 1 kHz depending on the glove or sensor array.

The edge preprocessor handles per-modality compression (H.264/H.265 for video, delta encoding for IMU, µ-law for audio), applies a hardware timestamp using a monotonic clock synchronized via PTP/NTP, and strips corrupted frames. The local buffer (a ring buffer backed by a write-ahead log on flash storage) absorbs bursts during connectivity loss so no data is dropped during handoffs.

## Cloud Ingestion Layer

On reconnect, the device pushes batches through a secure TLS channel. The API gateway handles device authentication (mTLS + JWT), per-device rate limiting, and fan-out routing. A message broker (Apache Kafka or Pulsar) provides durable, partitioned queues per modality so each stream can be consumed independently at different rates. A stream processor (Apache Flink or Spark Structured Streaming) sits downstream for lightweight real-time transforms — frame rate normalization, format conversion, and schema validation before data hits storage.

## Processing Layer

This is where multi-modal data gets harmonized. QC & validation flags dropped packets, sensor saturation, and statistical outliers per stream. Time alignment is the trickiest part: all streams must be warped onto a shared timeline using the hardware timestamps from the edge, compensating for clock drift and variable network jitter.

Feature extraction runs lightweight models here — optical flow from video, pose/activity from IMU, gaze fixations from the eye tracker, and scene embeddings — producing metadata that makes downstream retrieval far cheaper.

## Storage Layer

Four specialized stores, each matched to its data type:

| Store | Technology | Data |
|---|---|---|
| Object storage | S3 / GCS / Azure Blob | Raw video, EgoVLP/EPIC-style folder structure keyed by session + timestamp |
| Time-series DB | InfluxDB / TimescaleDB | Sensor streams (IMU, gaze, GPS, tactile) — high-frequency writes, fast range queries |
| Relational / document | Postgres + JSONB | Annotations and labels |
| Data lakehouse | Delta Lake / Apache Iceberg (Parquet) | Ground truth archive for bulk ML training |

## Platform Services

The data catalog (Apache Atlas, DataHub, or Amundsen) tracks lineage from raw sensor → processed feature → training split. Observability covers pipeline health (Prometheus + Grafana), data drift detection, and per-session completeness checks. Access control enforces row-level RBAC on the data lake and audit-logs every query — critical for IRB compliance when collecting human-subject data.

## Key Design Decisions

- **Synchronization strategy** — hardware PTP vs. software NTP + drift correction in post
- **Tactile encoding** — raw pressure maps vs. contact-event encoding
- **Data lake chunking** — session-level vs. fixed-duration windows
- **Stream processing location** — on-device (TFLite / ONNX on the SoC) vs. fully cloud-side

## Related Reading

<CardGroup cols={2}>
  <Card title="System Architecture Overview" icon="layer-group" href="/en/guides/system-architecture">
    Seven-layer CyberOrigin system model from hardware to customer delivery
  </Card>
  <Card title="Edge Preprocessor" icon="microchip" href="/en/references/edge-preprocessor">
    Edge-layer multi-modal compression and timestamping details
  </Card>
  <Card title="Local Buffering" icon="hard-drive" href="/en/references/local-buffering">
    Ring buffer and write-ahead log design for connectivity loss
  </Card>
  <Card title="Data Collection Ops" icon="video" href="/en/guides/data-collection-ops">
    Operational guide for L3 capture-layer sessions
  </Card>
</CardGroup>
