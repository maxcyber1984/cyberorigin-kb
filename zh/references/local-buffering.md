---
title: "Local Buffering on Edge Devices"
description: "Ring buffer and write-ahead log design for zero data loss on wearable devices during connectivity loss"
icon: "hard-drive"
---

The local buffer is the device's safety net — it ensures zero data loss even when the uplink drops. The two core structures are a **ring buffer** (fixed-size, overwrites oldest when full) and a **write-ahead log** (durable record of every write).

## The Ring Buffer

A fixed-size circular array in RAM, backed by flash for durability. On a wearable, you'd size it to hold roughly 30-60 seconds of all-modality data — large enough to survive a typical Wi-Fi dead zone or network handoff, but small enough not to drain flash endurance.

The write head advances every sensor tick (multiple times per second); the upload head follows when the connection is live. If the write head laps the upload head — i.e. the device has been offline longer than the buffer can hold — the oldest unuploaded slot gets overwritten, which is the data loss event.

## The Write-Ahead Log

The durability layer. Before any slot is written to the ring buffer, a record is appended to a sequential log on flash with:

| Field | Description |
|---|---|
| Sequence number | Monotonically increasing identifier |
| Modality | Which sensor stream |
| Timestamp | Hardware timestamp in nanoseconds |
| Byte size | Payload size |
| Status | `pending` / `sent` / `acked` |

This survives a crash or power cut. On boot, the device scans the WAL from the last `acked` entry forward and replays any `pending` entries — so you never silently lose data due to a crash mid-upload.

The WAL also enables **idempotent uploads**: each entry has a monotonically increasing sequence number, and the cloud uses `enable.idempotence=true` on the Kafka producer side to deduplicate retransmitted segments.

## Flash vs RAM Strategy

| Data | Location | Rationale |
|---|---|---|
| Raw sensor data | Ring buffer in RAM | Fast, no flash wear |
| WAL index (metadata only) | Flash | Slow but durable |
| Payload bytes | Paged to flash only under RAM pressure or device sleep | Keeps write path fast for high-rate streams |

This keeps the write path fast for high-rate streams like video and IMU while still guaranteeing recovery after power loss.

## Upload Protocol

Drains in strict sequence-number order, not by modality priority. This keeps the time-alignment job on the cloud side simple — it can always assume that if sequence #N has arrived for a given session, all sequences below N have also arrived.

An acknowledgement from Kafka triggers the WAL entry to move through the state machine:

```
pending → sent → acked
```

At which point that slot in the ring buffer is free to be reused. On failure, the upload retries with exponential backoff without touching `upload_ptr` — the slot stays occupied and the data stays safe.

## 延伸阅读

<CardGroup cols={2}>
  <Card title="输出总线交接" icon="right-left" href="/zh/references/output-bus-handoff">
    预处理器到缓冲的五步交接
  </Card>
  <Card title="边缘预处理器" icon="microchip" href="/zh/references/edge-preprocessor">
    上游的压缩与时间戳
  </Card>
  <Card title="数据管道" icon="arrows-split-up-and-left" href="/zh/guides/data-pipeline">
    端到端管道架构
  </Card>
</CardGroup>
