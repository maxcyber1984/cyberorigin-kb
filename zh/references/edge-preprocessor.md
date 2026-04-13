---
title: "Edge Preprocessor"
description: "How the edge preprocessor handles per-modality compression, timestamping, and packet framing for egocentric multi-modal data"
icon: "microchip"
---

边缘预处理器是整条管道中对延迟最敏感的组件——它跑在可穿戴设备的 SoC 上（根据形态不同，可能是 Qualcomm Snapdragon、NVIDIA Jetson Orin Nano 或 Apple M 系列），必须在不耗尽电池、不过热的前提下实时处理全部六路数据流。

## 按模态编码 / 压缩

### 视频

使用硬件编码器运行 H.265（绝不要走软件编码——会把 CPU 和电池烤干）。CRF 23 是不错的起点：对绝大多数下游 CV 任务而言感知无损，相对于原始数据体量降低约 60-70%。如果 SoC 支持 AV1（Snapdragon 8 Gen 2+ 支持），可切换过去——在同等画质下比 H.265 小 30%。

### IMU

先过一遍卡尔曼滤波（平滑陀螺噪声，对延迟影响可忽略），然后对结果做差分编码（每个样本只存与前一个的差值），再用 lz4 批压缩。这样相对于原始 float 可以压缩 5-10 倍。

### 眼动

深度图使用 RVL（Run-Length Variable-length）编码——这是深度帧的标准做法，对深度数据的空间相干性利用远胜于通用编码器。2D 注视点（x, y, confidence）量化为定点整数；超过 0.1 度的亚像素精度本来就是噪声。

### 音频

Opus 16 kbps、20 ms 帧长，是语音和环境声的甜点位。VAD（Voice Activity Detection）内联运行，给静音帧打标签，免除存储大量静音。

### GPS

坐标量化为定点，精度约 10 cm（1 Hz 下纬度 7 位小数已远超所需），随后做 geohash 以便后续空间索引。带宽太低，压缩几乎不重要，但编码一致性很重要。

### 触觉

最有意思的模态。1 kHz 的完整压力阵列数据量极大。接触事件编码更聪明：不再每帧存储完整的 NxM 压力网格，而是只存储超过阈值变化的那些格子（稀疏表示），再加上接触区域的 Run-Length 编码。对典型的操作任务，可压缩 10-20 倍且没有有意义的信息损失。

## 共享时间戳引擎

这是整个预处理器最重要的组件。每条流都被打上一个单调的硬件时钟值，时钟来自 PTP（Precision Time Protocol）硬件时钟，在可用时同步到 GPS 时间源。

关键属性：**所有六个模态的时间戳来自同一个时钟域**——不是六个各自漂移的独立时钟。没有这一点，云端的时间对齐就从一次简单的 join 变成反卷积问题。

在多数可穿戴 SoC 上的实际做法是：通过 `CLOCK_TAI` 暴露 PTP 时钟（不要用 `CLOCK_REALTIME`，它会随 NTP 调整而跳变），并在每个编码数据包到达本地缓冲之前将时间戳写入包头。用 64 位纳秒整数存储——不要用 float，精度会丢。

## 输出总线数据包封装

每条流以带帧、带标签、封装一致的数据包输出：

```
{
  session_id,
  device_id,
  modality,
  sequence_number,
  hardware_timestamp_ns,
  payload_bytes
}
```

这套统一封装，让下游 Kafka producer 和时间对齐任务能以相同方式处理所有六路流，无论原始编码差别多大。

## 延伸阅读

<CardGroup cols={2}>
  <Card title="本地缓冲" icon="hard-drive" href="/zh/references/local-buffering">
    预处理器下游的环形缓冲 + WAL
  </Card>
  <Card title="输出总线交接" icon="right-left" href="/zh/references/output-bus-handoff">
    从预处理器到本地缓冲的五步交接
  </Card>
  <Card title="数据管道" icon="arrows-split-up-and-left" href="/zh/guides/data-pipeline">
    端到端数据管道概览
  </Card>
  <Card title="系统架构总览" icon="layer-group" href="/zh/guides/system-architecture">
    L2 在七层架构中的位置
  </Card>
</CardGroup>
