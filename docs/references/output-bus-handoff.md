---
title: "Output Bus to Local Buffer Handoff"
description: "The five-step crash-safe handoff sequence from the edge preprocessor output bus to the ring buffer via write-ahead log"
icon: "right-left"
---

The handoff is a five-step sequence where the write-ahead log is always written first — before anything touches the ring buffer. That ordering is what makes the buffer crash-safe.

## Step 1 — WAL First, Always

Before touching the ring buffer, a record is appended to the write-ahead log on flash:

| Field | Value |
|---|---|
| Sequence number | Monotonically increasing |
| Hardware timestamp | Nanoseconds |
| Modality tag | Stream identifier |
| Payload size | Bytes |
| CRC32 | Payload integrity check |
| Status | `PENDING` |

This is a sequential append — the fastest possible flash write. The critical property: if the device loses power after this step but before step 3, the WAL entry survives. On reboot, the system scans for PENDING entries with no corresponding ring slot data and replays them from the sensor hardware.

## Step 2 — Claim the Slot Atomically

The write pointer is an atomic integer incremented with a compare-and-swap. If the slot at `write_ptr % N` is already occupied by unuploaded data, it gets evicted — the WAL entry for that older packet is marked `DROPPED`, the drop counter increments, and the slot is freed for the new write.

This is the data-loss event: it only happens when the upload thread has fallen more than N frames behind. The slot claim is a single atomic operation so it's safe across the preprocessor thread and the upload thread running concurrently.

## Step 3 — Write the Payload

On SoCs with a DMA controller (Snapdragon, Jetson), the encoded payload bytes move from the codec output buffer directly into the ring slot without touching the CPU — the DMA engine handles the memory-to-memory copy and raises an interrupt on completion. On simpler MCUs it's a plain `memcpy`.

The ring slot stores three things:
- Modality tag
- Sequence number (for linking back to the WAL)
- Raw payload bytes

## Step 4 — Link WAL to Slot

Now that the payload is safely in RAM, the WAL entry is updated with the ring buffer slot index. This two-phase write is what makes crash recovery deterministic:

On reboot you can scan the WAL, find PENDING entries, look up their slot index, and check whether the ring slot actually contains valid data:
- **Slot index is unset** (device crashed between step 1 and step 4) → data is gone, entry marked `DROPPED`
- **Slot is valid** → upload thread resumes from where it left off

## Step 5 — Wake the Uploader

A `sem_post()` on a POSIX semaphore (or an RTOS equivalent like FreeRTOS `xSemaphoreGive`) wakes the upload thread. The upload thread is otherwise sleeping to conserve CPU and therefore battery.

It wakes, reads from the ring slot at `upload_ptr`, opens a TLS connection (or reuses an existing one), and streams the packet to Kafka:
- **On ACK**: marks the WAL entry `SENT` and advances `upload_ptr`
- **On failure**: retries with exponential backoff without touching `upload_ptr` — the slot stays occupied and the data stays safe

## Performance

The whole handoff path — steps 1 through 5 — runs in under **100 microseconds** on a mid-range SoC for small payloads (IMU, GPS, tactile). Video frames take longer purely because of the memcpy/DMA time, but that's bounded by memory bandwidth, not the protocol overhead.
