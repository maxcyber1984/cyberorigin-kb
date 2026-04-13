---
title: "Bluebird Cloud QC Platform"
description: "CyberOrigin's video annotation and QC tool: task segmentation, defect classification, and keyboard shortcuts"
icon: "cloud"
---

## Overview

Bluebird Cloud (Laniao Cloud) is the video annotation and QC platform in CyberOrigin's data collection pipeline, used daily by both the QC and annotation teams. The platform supports two core operations: **task segmentation** and **defect tagging**.

## Shortcuts

| Key | Function |
|---|---|
| `J` | Task segmentation (start / end) |
| `L` | Defect segment tagging (start / end) |

## Task Segmentation (J)

<Steps>
  <Step title="Start segment">
    Press `J` once, select the **scene** and **task name** — the system records the current moment as the start frame.
    
    **Thumbs-up gesture** = task-start signal, marks the initial frame.
  </Step>
  <Step title="End segment">
    Press `J` a second time and confirm to finish the segmentation of the current task.
    
    **Two-finger gesture** = task-end signal, marks the end frame.
  </Step>
</Steps>

## Defect Tagging (L)

During QC, whenever a non-compliant segment is found, use `L` to mark start/end and pick the defect reason on close.

### Household Scenes (5 codes)

| Code | Meaning |
|---|---|
| `image_flicker` | Persistent brightness oscillation during playback |
| `operator_action_too_fast` | Unnaturally fast motion |
| `operator_hand_static` | Hands frozen for more than 5 seconds |
| `illegal_use_props` | Props used outside protocol |
| `other_unqualified` | Other defect |

### Supermarket Scenes (17 codes)

On top of the 5 household codes, supermarket scenes add:

| Code | Meaning |
|---|---|
| `target_out_of_frame` | Target object out of frame |
| `face_appeared` | Unredacted face appears |
| `multiple_hands_appeared` | More than two hands in frame |
| `part_of_hands_out_of_frame` | Partial hand out of frame |
| `hands_out_of_frame` | Both hands out of frame |
| `push_shopping_cart` | Hands leave frame while pushing cart |
| `task_definition_and_items_not_related` | Task definition doesn't match items |
| `image_blur` | Blurred image |
| `mirror_problem` | Mirror-related artifact |
| `phone_appeared` | Phone appears in frame |
| `chinese_appeared` | Chinese text visible in frame |
| `operator_unpleasant_behavior` | Operator inappropriate behavior |

## Related Reading

<CardGroup cols={2}>
  <Card title="EgoCentric Annotation" icon="tag" href="/en/references/egocentric-annotation">
    Annotation methodology and output format
  </Card>
  <Card title="Annotation Team KPI" icon="chart-bar" href="/en/references/annotation-team-kpi">
    QC output baseline (19h/day)
  </Card>
  <Card title="Data Collection Ops" icon="video" href="/en/guides/data-collection-ops">
    Capture-side specs mirrored by QC standards
  </Card>
  <Card title="Annotation Pipeline" icon="tags" href="/en/references/annotation-pipeline">
    Two-layer annotation with the QC defect codebook
  </Card>
</CardGroup>
