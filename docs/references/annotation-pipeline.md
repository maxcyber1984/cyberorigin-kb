---
title: "Annotation Pipeline"
description: "How egocentric video is annotated: two-layer segmentation, action labeling, QC defect codebook, and JSON output format"
icon: "tags"
---

## Overview

A single capture video may contain multiple valid collection tasks. The annotation pipeline segments the full video into independent tasks, then labels each segment with step-by-step action annotations following a predefined workflow.

Segments with missing steps, wrong ordering, or inconsistent flow are marked invalid and excluded from downstream training.

## Two-Layer Annotation

### Layer 1: GV Annotation (Global Video)

**Calibration segment tagging** — Every video starts with a calibration sequence (slow walk, hands out of frame, stationary pose). Annotators mark the calibration end frame. Everything after that frame is task footage.

**Action segmentation** — Annotators mark natural action boundaries (the transition between one completed action and the next). Adjacent boundaries must be no more than 60 seconds apart. No language labels required at this layer — it's purely temporal segmentation.

### Layer 2: Short-Segment Annotation (Fine-Grained)

Each interaction with an object gets its own annotation clip, typically 2-5 seconds long. Gaps between clips are allowed.

Every clip has two fields:

- **Skill**: a single verb capturing the core action (`hold`, `retrieve`, `pick`, `place`, `push`, `fold`, etc.)
- **Description**: a structured sentence following the pattern `verb + object + manner + spatial relation`

Example: *"Place the held pear into the plastic bag in the shopping bag."*

Spatial relations should describe both the object-to-object relationship (from place A to place B) and the precise location context (e.g., "into the shopping cart next to the instant noodles").

Verb and object vocabularies reference the [HD-EPIC dataset](https://hd-epic.github.io/).

## Task Library

16 household task types are currently defined, each with an explicit step-by-step workflow and start/end frame criteria:

| ID | Task | Key Constraint |
|---|---|---|
| 1 | Kitchen item sorting | Full unpack flow from bags to shelves and fridge |
| 2 | Making tea | Includes cleanup: wash teaware in bathroom, return to table |
| 3 | Making coffee | Same cleanup pattern as tea |
| 4 | Organizing clothes | Min 5 items; start from messy pile, end with folded stack |
| 5 | Organizing socks | Collect, fold, box, store, close cabinet |
| 6 | Making bed | Replace pillowcase and sheets, fold neatly |
| 7 | Kitchen cleaning | Stove, sink, countertop, cabinets, towel cleanup |
| 8 | Fridge cleaning | Remove contents and shelves, wash, wipe interior, reassemble |
| 9 | Living room tidying | Floor items, sofa cushions, coffee table |
| 10 | Dishwashing | Collect, sink, wash, dry, store by category |
| 11 | Sweeping and mopping | Enter with tools, sweep, dustpan, mop, store tools |
| 12 | Toolbox organizing | Collect, assemble missing parts, box, store |
| 13 | Making salad | Transfer vegetables, stir, dress, clean up |
| 14 | Making snack bags | Open bag, scoop, pour, seal, store |
| 15 | Folding clothes | Spread, fold left sleeve/side, right sleeve/side, bottom up |
| 16 | Desk organizing | Pens, canisters, cups, boxes — each to designated spot |

Industrial tasks (factory floor) are being added: PC disassembly, tent setup, furniture assembly, bicycle assembly.

## Quality Control

QC runs on a platform called Bluebird Cloud, with two keyboard-driven workflows:

**Task segmentation (J key)**: First press selects scene + task name and marks start frame. Second press confirms end frame. Visual cues from the collector: thumbs-up = task start, two-finger gesture = task end.

**Defect tagging (L key)**: First press marks defect start. Second press selects the defect reason from a codebook.

### Defect Codebook

Household scenes (5 codes):

| Code | Trigger |
|---|---|
| `image_flicker` | Persistent brightness oscillation during playback |
| `operator_action_too_fast` | Unnaturally fast movements |
| `operator_hand_static` | Hands frozen for > 5 seconds |
| `illegal_use_props` | Props used outside protocol |
| `other_unqualified` | Catch-all |

Supermarket scenes add 12 more codes: `target_out_of_frame`, `face_appeared`, `multiple_hands_appeared`, `part_of_hands_out_of_frame`, `hands_out_of_frame`, `push_shopping_cart` (hands leaving frame while pushing), `task_definition_and_items_not_related`, `image_blur`, `mirror_problem`, `phone_appeared`, `operator_unpleasant_behavior`, `chinese_appeared`.

## Output Format

Annotations export as JSON with three top-level keys:

```json
{
  "video_info": {
    "name": "GX010755.MP4",
    "total_frames": 9419,
    "fps": 29.97,
    "annotator_id": "cyber001",
    "duration": 314.28,
    "codec": "hevc",
    "shape": [1080, 1920, 3]
  },
  "annotations": [
    {
      "start_frame": 95,
      "end_frame": 256,
      "start_timestamp": 4170.833,
      "end_timestamp": 8575.233,
      "valid": true,
      "content": "place bags on the table",
      "recheck": false
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

- `video_info` is auto-populated from the file metadata
- `annotations` are ordered by step sequence — out-of-order entries invalidate the segment
- `segmentations` define task boundaries with environment and task type labels
