---
title: "Local Buffering on Edge Devices"
description: "Ring buffer and write-ahead log design for zero data loss on wearable devices during connectivity loss"
icon: "hard-drive"
---

本地缓冲是设备的安全网——即便上行链路中断也要保证零数据丢失。它由两个核心结构组成：**环形缓冲**（固定大小，满时覆盖最早的数据）和**预写日志（WAL）**（每次写入都有持久记录）。

## 环形缓冲

内存中的固定大小循环数组，由 Flash 提供持久化。在可穿戴设备上，容量一般按所有模态 30-60 秒的数据量来设置——足够覆盖一次典型的 Wi-Fi 盲区或网络切换，同时又不会过度消耗 Flash 寿命。

写指针随每个传感器 tick（每秒多次）推进；上传指针在连接正常时跟进。如果写指针追上了上传指针——也就是说设备离线时间超过了缓冲可承受的上限——最早未上传的槽位会被覆盖，这就是数据丢失事件。

## 预写日志（WAL）

持久化层。在写入环形缓冲任何一个槽位之前，先往 Flash 上的顺序日志追加一条记录，字段如下：

| 字段 | 说明 |
|---|---|
| Sequence number | 单调递增的标识 |
| Modality | 对应的传感器流 |
| Timestamp | 纳秒级硬件时间戳 |
| Byte size | 负载字节数 |
| Status | `pending` / `sent` / `acked` |

这条日志能够挺过崩溃或断电。开机时设备从最后一条 `acked` 往后扫 WAL，重放所有 `pending` 条目——因此即便上传过程中崩溃，也不会悄无声息地丢数据。

WAL 还使**幂等上传**成为可能：每条记录都有单调递增的 sequence number，云端 Kafka producer 侧启用 `enable.idempotence=true`，对重传片段自动去重。

## Flash 与 RAM 分工

| 数据 | 位置 | 原因 |
|---|---|---|
| 原始传感器数据 | RAM 环形缓冲 | 快，且不磨损 Flash |
| WAL 索引（仅元数据） | Flash | 慢但持久 |
| Payload 字节 | 仅在 RAM 吃紧或设备休眠时换页到 Flash | 保证高频流写路径快速 |

这样既能让视频、IMU 这类高频流的写入路径保持快速，又能在断电后恢复。

## 上传协议

严格按 sequence number 顺序出队，而不是按模态优先级。这让云端的时间对齐任务变得简单——它总可以假定：只要某 Session 的 #N 已到达，所有 N 以下的 sequence 也已到达。

一条来自 Kafka 的 ACK 会推动对应 WAL 条目走过状态机：

```
pending → sent → acked
```

到达 acked 后，环形缓冲对应的槽位即可复用。失败时按指数退避重试，不推进 `upload_ptr`——槽位保持占用，数据保持安全。

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
