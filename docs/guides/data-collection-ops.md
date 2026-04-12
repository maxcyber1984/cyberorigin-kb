---
title: "Data Collection Operations"
description: "Running egocentric data collection at scale: calibration protocol, session structure, task requirements, and metadata tracking"
icon: "video"
---

## Calibration Protocol

Every recording starts with a mandatory calibration sequence. This is non-negotiable — it enables spatial reconstruction and camera intrinsic estimation downstream.

### Step 1: QR Code Registration

Place a QR code flat on the floor (paper must be smooth, flush with the surface). The collector looks straight ahead (eye-level, parallel to ground) for 2 seconds, then tilts to capture the full QR code in frame for 2 seconds. After that, the QR code is removed and must not appear again for the rest of the session.

### Step 2: Environment Scan

For the first 5-15 seconds after QR removal:
- Collector faces forward, ground visible in > 30% of the frame
- Hands stay out of frame
- Walk a slow loop around the capture area (the path must cover all locations where tasks will be performed)
- Maintain forward gaze throughout

### Step 3: Calibration End Signal

After the environment scan, hands enter the frame and hold still for 1 second. This marks the end of calibration. From this point, there are no constraints on hand visibility, ground visibility, gaze direction, or movement speed.

## Session Structure

Standard collection day runs 5 sessions totaling 9 hours of active capture:

| Session | Time | Duration |
|---|---|---|
| 1 | 08:30 - 10:30 | 2h |
| 2 | 11:00 - 12:00 | 1h |
| 3 | 13:00 - 15:00 | 2h |
| 4 | 15:30 - 17:30 | 2h |
| 5 | 18:00 - 20:00 | 2h |

Some collection days run reduced schedules (Sessions 4-5 only, or Sessions 1-2 only) depending on setup and logistics.

## Task Requirements

### Household Scenes

Each task has specific constraints beyond the general workflow:

| Task | Requirements |
|---|---|
| Prep vegetables | Washing and cutting only — no cooking |
| Clear dining table | Full sequence: dirty dishes to kitchen sink, then wipe table clean. Hands must be visible during movement |
| Wash dishes | Minimum 4 items (plates, cups, or bowls) |
| Make bed | Pillowcase and pillow arrangement only — no blanket folding |
| Fold clothes | Start: messy pile (any types). End: neatly folded stack. Min 5 items |
| Organize bookshelf | Unit action: approach shelf, pick one book, place on desk. 3-5 units per task, must be continuous |
| Stock fridge | Put a bag of purchased vegetables into the fridge |
| Organize toys | Move scattered toys from desk into a box on the desk. Min 3-5 items |
| Organize tools | Move scattered tools from desk into a box on the desk. Min 3-5 items |

### Factory Scenes (Self-Built Facility)

Newer task categories running on dedicated floors:

**5th Floor**: Snack area (2 people), toy area (2 people), produce area (1 person)

**4th Floor**: PC disassembly/assembly, tent setup/teardown, furniture assembly (children's bed, chair), bicycle assembly/disassembly

## Metadata Tracking

Each person-session record captures:

| Field | Description |
|---|---|
| Task | Which task was performed |
| `device_id` | Camera/device identifier |
| Scene | Environment type (household, supermarket, factory) |
| Address | Physical collection location |
| Perspective | First-person or third-person |
| Collector height | For camera height calibration |
| Collection method | `human` or `teleoperation` |

## Documentation System

Four core documents maintain operational state:

1. **Attendance log** — Hourly worker and intern attendance by month
2. **Task definition doc** — Canonical task list with diversity requirements and rotation schedule
3. **Metadata doc** — Per-session records with all fields above, plus session-level incident logs
4. **Procurement doc** — Props, materials, and supplies tracking

Data uploads flow through two channels on Feishu (Lark): a demo submission form and a video upload folder. NAS storage tracks cumulative volume.

## Collection Timeline

The operation has been running continuously since at least June 2025, rotating through multiple locations in Shenzhen:

- **Phase 1** (Jun-Jul 2025): Rented apartments — Longgang, Bao'an, Guangming districts
- **Phase 2** (Aug-Sep 2025): Continued apartment rotation with expanded team
- **Phase 3** (Nov 2025+): Self-built factory facility with dedicated floors for industrial tasks

The shift from rented apartments to a self-built facility signals scaling from proof-of-concept collection to production-grade data operations.
