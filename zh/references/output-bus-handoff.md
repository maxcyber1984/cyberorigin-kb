---
title: "Output Bus to Local Buffer Handoff"
description: "The five-step crash-safe handoff sequence from the edge preprocessor output bus to the ring buffer via write-ahead log"
icon: "right-left"
---

交接过程是一个五步序列，其中预写日志（WAL）始终最先写入——在任何数据触碰环形缓冲之前。这种顺序是保证缓冲区崩溃安全的关键。

## 第 1 步 — WAL 永远先写

在接触环形缓冲之前，先向 Flash 上的 WAL 追加一条记录：

| 字段 | 值 |
|---|---|
| Sequence number | 单调递增 |
| Hardware timestamp | 纳秒 |
| Modality tag | 流标识 |
| Payload size | 字节 |
| CRC32 | Payload 完整性校验 |
| Status | `PENDING` |

这是顺序追加——Flash 能实现的最快写入。关键属性：如果设备在第 1 步之后、第 3 步之前断电，这条 WAL 记录依然存在。重启时，系统会扫描找到没有对应环形槽位数据的 PENDING 条目，并从传感器硬件侧重放。

## 第 2 步 — 原子地占用槽位

写指针是一个原子整数，通过 compare-and-swap 自增。如果 `write_ptr % N` 位置的槽位已被未上传的数据占据，就会被驱逐——对应的旧包 WAL 条目标记为 `DROPPED`，drop 计数器递增，槽位释放给新写入使用。

这是数据丢失事件：仅发生在上传线程落后超过 N 帧时。槽位占用是单次原子操作，在预处理线程和上传线程并发执行时也安全。

## 第 3 步 — 写入 Payload

在带 DMA 控制器的 SoC 上（Snapdragon、Jetson），编码后的 payload 字节从编码器输出缓冲直接搬进环形槽位而不经过 CPU——DMA 引擎负责内存到内存的拷贝，完成后触发中断。在较简单的 MCU 上就是一次普通 `memcpy`。

环形槽位中存储三样东西：
- Modality tag
- Sequence number（用于反查 WAL）
- 原始 payload 字节

## 第 4 步 — 将 WAL 与槽位关联

payload 安全进入 RAM 后，更新 WAL 条目，写入对应的环形缓冲槽位索引。这种两阶段写入让崩溃恢复是确定性的：

重启时扫描 WAL，找到 PENDING 条目，查看其槽位索引，并检查环形槽位中是否真的有有效数据：
- **槽位索引未设置**（设备在第 1 步与第 4 步之间崩溃）→ 数据已丢失，条目标记为 `DROPPED`
- **槽位有效** → 上传线程从中断处继续

## 第 5 步 — 唤醒上传线程

在 POSIX 信号量上调用 `sem_post()`（或 RTOS 等价调用，例如 FreeRTOS 的 `xSemaphoreGive`）来唤醒上传线程。上传线程在其余时间处于睡眠状态以节省 CPU 和电量。

它醒来后，从 `upload_ptr` 指向的环形槽位读出数据，打开一个 TLS 连接（或复用已有连接），将数据包推送到 Kafka：
- **收到 ACK**：将 WAL 条目标记为 `SENT`，推进 `upload_ptr`
- **失败**：按指数退避重试，不推进 `upload_ptr`——槽位保持占用，数据保持安全

## 性能

整条交接路径——第 1 步到第 5 步——在中端 SoC 上对小 payload（IMU、GPS、触觉）耗时不到 **100 微秒**。视频帧耗时更长纯粹来自 memcpy/DMA 时间，但这个时间受内存带宽约束，而不是协议本身的开销。

## 延伸阅读

<CardGroup cols={2}>
  <Card title="本地缓冲" icon="hard-drive" href="/zh/references/local-buffering">
    环形缓冲与 WAL 设计细节
  </Card>
  <Card title="边缘预处理器" icon="microchip" href="/zh/references/edge-preprocessor">
    交接的上游：压缩与时间戳
  </Card>
  <Card title="数据管道" icon="arrows-split-up-and-left" href="/zh/guides/data-pipeline">
    端到端数据管道
  </Card>
</CardGroup>
