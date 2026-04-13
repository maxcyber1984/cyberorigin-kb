---
title: "EgoCentric Annotation"
description: "First-person video annotation methodology: GV annotation, short-segment annotation, task coverage, and output format spec"
icon: "tag"
---

## Definition

A methodology for segmenting and labeling first-person (EgoCentric) captured video along a predefined step workflow. The goal is to accurately identify each independent task segment in a full video, and within each segment, label the interaction actions in step order.

<Warning>
Annotation must strictly follow the defined step order. Segments with missing steps, wrong ordering, or inconsistency with the standard workflow are considered **invalid** and do not count toward output.
</Warning>

## Annotation Layers

Annotation is done in two layers, in order:

**GV Annotation** → **Short-Segment Annotation**

### GV Annotation

<Steps>
  <Step title="Calibration segment">
    Every video opens with a calibration sequence (slow walk, stationary pose, hands out of frame). Annotators mark the calibration end frame. The end frame may slightly lag the actual transition, but everything after it must be task footage.
  </Step>
  <Step title="Action boundary points">
    Mark natural action boundaries — any frame between one completed action and the start of the next. Adjacent boundaries must be no more than 1 minute apart. No language labels needed.
  </Step>
</Steps>

### Short-Segment Annotation

Every interaction with an object needs an annotation clip, typically **2-5 seconds** long. Gaps between clips are allowed.

Each clip has two fields:

| Field | Description | Example |
|---|---|---|
| **Skill** | Verb capturing the core action | `pick`, `place`, `hold`, `retrieve` |
| **Description** | verb + object + manner + spatial relation | `Place the held pear into the plastic bag in the shopping bag.` |

<Tip>
Verbs and description patterns reference the [HD-EPIC Dataset](https://hd-epic.github.io/).
</Tip>

## Task Coverage

Currently defined: **16 household task types**:

| # | Task | # | Task |
|---|---|---|---|
| 1 | Prep vegetables | 9 | Organize fridge |
| 2 | Kitchen item sorting | 10 | Organize toys |
| 3 | Clear dining table | 11 | Organize tools |
| 4 | Wash dishes | 12 | Organize toolbox |
| 5 | Make tea / coffee | 13 | Make salad |
| 6 | Arrange pillows / bedding | 14 | Make snack bags |
| 7 | Organize clothes / socks | 15 | Fold clothes |
| 8 | Organize bookshelf / desk | 16 | Sweep and mop |

After the self-built factory launch, industrial scenes have been added: PC disassembly, tent setup/teardown, furniture assembly/disassembly, bicycle assembly/disassembly.

## Output Format

Annotation results export as `.json`, with three sections:

```json
{
  "video_info": {
    "name": "GX010755.MP4",
    "total_frames": 9419,
    "fps": 29.97,
    "annotator_id": "cyber001"
  },
  "annotations": [
    {
      "start_frame": 95,
      "end_frame": 256,
      "valid": true,
      "content": "place bags on the table"
    }
  ],
  "segmentations": [
    {
      "start_frame": 8,
      "end_frame": 5499,
      "valid": true,
      "env": "kitchen",
      "task": "unpack_groceries"
    }
  ]
}
```

- `video_info`: video metadata, auto-populated
- `annotations`: action annotations within each valid segment, order must match step workflow
- `segmentations`: start/end frames of each complete data segment

## Related Reading

<CardGroup cols={2}>
  <Card title="Data Collection Ops" icon="video" href="/en/guides/data-collection-ops">
    Calibration protocol and session structure
  </Card>
  <Card title="Bluebird Cloud" icon="cloud" href="/en/references/laniao-cloud">
    Platform for QC and annotation
  </Card>
  <Card title="Annotation Team KPI" icon="chart-bar" href="/en/references/annotation-team-kpi">
    Output baselines and quality bar
  </Card>
  <Card title="Annotation Pipeline" icon="tags" href="/en/references/annotation-pipeline">
    Two-layer flow, defect codes, and JSON schema
  </Card>
</CardGroup>
