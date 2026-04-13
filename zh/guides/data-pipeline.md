---
title: "CyberOrigin Data Pipeline"
description: "End-to-end data pipeline architecture from edge capture to platform services, covering ingestion, processing, storage, and observability"
icon: "arrows-split-up-and-left"
---

## 边缘 / 设备层

每类传感器输出的数据流各不相同，采样率、编码方式和精度都不一样。视频来自卷帘快门相机，30-60 fps；IMU 输出 200 Hz；眼动 120 Hz；音频 16-48 kHz；GPS 1-10 Hz；触觉传感器根据阵列规模不同，从 100 Hz 到 1 kHz 不等。

[边缘预处理器](/zh/references/edge-preprocessor) 负责按模态执行压缩（视频 H.264/H.265、IMU 差分编码、音频 u-law），使用通过 PTP/NTP 同步的单调时钟打上硬件时间戳，并剔除损坏帧。

[本地缓冲](/zh/references/local-buffering)（基于 Flash 上的 WAL 预写日志实现的环形缓冲）负责在连接中断时吸收突发流量，确保切换过程中零丢包。

## 云端接入层

重新建立连接后，设备通过安全的 TLS 通道批量推送数据。API 网关负责设备认证（mTLS + JWT）、按设备限速以及扇出路由。

消息总线（Apache Kafka 或 Pulsar）为每种模态提供持久化、分区化的队列，让每条流都能按各自速率被独立消费。

流式处理引擎（Apache Flink 或 Spark Structured Streaming）位于其下游，负责轻量级实时变换——帧率归一、格式转换、以及数据入库前的 Schema 校验。

## 处理层

多模态数据在这里被对齐整合。

- **QC 与校验**：按流标记丢包、传感器饱和和统计异常值。
- **时间对齐**：最棘手的一步，需要依据边缘端打上的硬件时间戳将所有流对齐到同一时间轴，并补偿时钟漂移和网络抖动。
- **特征抽取**：运行轻量模型——视频光流、IMU 姿态/活动识别、眼动注视点、场景 embedding——产出的元数据显著降低下游检索成本。

## 存储层

四类专用存储，各自匹配对应数据类型：

| 存储类型 | 数据类型 | 技术方案 |
|---|---|---|
| 对象存储 | 原始视频 | S3 / GCS / Azure Blob，采用 EgoVLP / EPIC 风格的目录组织 |
| 时序数据库 | 传感器流（IMU、眼动、GPS、触觉） | InfluxDB 或 TimescaleDB |
| 关系 / 文档库 | 标注与标签 | Postgres + JSONB |
| 数据湖 | 大规模 ML 训练（GT 归档） | Delta Lake 或 Apache Iceberg + Parquet |

## 平台服务

- **数据目录**（Apache Atlas、DataHub 或 Amundsen）跟踪从原始传感器 → 处理特征 → 训练切分的血缘关系。
- **可观测性**覆盖管道健康（Prometheus + Grafana）、数据漂移检测，以及每 Session 的完整性检查。
- **访问控制**在数据湖层面执行行级 RBAC，所有查询都有审计日志——对涉及人体对象采集数据的 IRB 合规至关重要。

## 关键设计权衡

- **同步策略**：硬件 PTP 对比软件 NTP + 后期漂移校正
- **触觉编码**：原始压力图对比接触事件编码
- **数据湖切片策略**：按 Session 切对比固定时长窗口
- **流处理部署位置**：设备端（SoC 上的 TFLite / ONNX）对比完全云端

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
