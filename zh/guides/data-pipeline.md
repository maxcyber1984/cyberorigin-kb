---
title: "CyberOrigin Data Pipeline"
description: "End-to-end data pipeline architecture from edge capture to platform services, covering ingestion, processing, storage, and observability"
icon: "arrows-split-up-and-left"
---

## Edge / Device Layer

Every sensor produces a different stream with its own rate, encoding, and precision. Video comes off a rolling shutter camera at 30-60 fps; IMU pushes at 200 Hz; eye gaze at 120 Hz; audio at 16-48 kHz; GPS at 1-10 Hz; and tactile at anywhere from 100 Hz to 1 kHz depending on the sensor array.

The [edge preprocessor](/zh/references/edge-preprocessor) handles per-modality compression (H.264/H.265 for video, delta encoding for IMU, u-law for audio), applies a hardware timestamp using a monotonic clock synchronized via PTP/NTP, and strips corrupted frames.

The [local buffer](/zh/references/local-buffering) (a ring buffer backed by a write-ahead log on flash storage) absorbs bursts during connectivity loss so no data is dropped during handoffs.

## Cloud Ingestion Layer

On reconnect, the device pushes batches through a secure TLS channel. The API gateway handles device authentication (mTLS + JWT), per-device rate limiting, and fan-out routing.

A message broker (Apache Kafka or Pulsar) gives you durable, partitioned queues per modality so each stream can be consumed independently at different rates.

A stream processor (Apache Flink or Spark Structured Streaming) sits downstream for lightweight real-time transforms — frame rate normalization, format conversion, and schema validation before data hits storage.

## Processing Layer

This is where multi-modal data gets harmonized.

- **QC & validation** flags dropped packets, sensor saturation, and statistical outliers per stream.
- **Time alignment** is the trickiest part: you need to warp all streams onto a shared timeline using the hardware timestamps from the edge, compensating for clock drift and variable network jitter.
- **Feature extraction** runs lightweight models — optical flow from video, pose/activity from IMU, gaze fixations from the eye tracker, and scene embeddings — producing metadata that makes downstream retrieval far cheaper.

## Storage Layer

Four specialized stores, each matched to its data type:

| Store | Data Type | Technology |
|---|---|---|
| Object storage | Raw video | S3, GCS, or Azure Blob with EgoVLP/EPIC-style folder structure |
| Time-series DB | Sensor streams (IMU, gaze, GPS, tactile) | InfluxDB or TimescaleDB |
| Relational/document store | Annotations and labels | Postgres + JSONB |
| Data lake | Bulk ML training (ground truth archive) | Delta Lake or Apache Iceberg as Parquet |

## Platform Services

- **Data catalog** (Apache Atlas, DataHub, or Amundsen) tracks lineage from raw sensor to processed feature to training split.
- **Observability** covers pipeline health (Prometheus + Grafana), data drift detection, and per-session completeness checks.
- **Access control** enforces row-level RBAC on the data lake and audit logs every query — critical for IRB compliance if you're collecting human-subject data.

## Key Design Decisions

- **Synchronization strategy**: hardware PTP vs. software NTP + drift correction in post
- **Encoding for tactile**: raw pressure maps vs. contact-event encoding
- **Chunking policy for the data lake**: session-level vs. fixed-duration windows
- **Stream processing location**: on-device (TFLite / ONNX on the SoC) or fully cloud-side

## 延伸阅读

<CardGroup cols={2}>
  <Card title="系统架构总览" icon="layer-group" href="/zh/guides/system-architecture">
    七层架构中数据管道所处的层级
  </Card>
  <Card title="边缘预处理器" icon="microchip" href="/zh/references/edge-preprocessor">
    每个模态的压缩与时间戳细节
  </Card>
  <Card title="本地缓冲" icon="hard-drive" href="/zh/references/local-buffering">
    环形缓冲 + WAL 保证零数据丢失
  </Card>
  <Card title="输出总线交接" icon="right-left" href="/zh/references/output-bus-handoff">
    预处理器到本地缓冲的五步交接
  </Card>
</CardGroup>
