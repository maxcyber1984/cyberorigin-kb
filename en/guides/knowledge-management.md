---
title: "Knowledge Management"
description: "How CyberOrigin builds its team knowledge system using Obsidian + Claude + Mintlify"
icon: "brain"
---

## Overall Architecture

CyberOrigin's knowledge system is built on three cooperating layers:

```
Obsidian Vault (shared)
    ↓  Claude reads, organizes, generates
Internal knowledge base (this site)
    ↓  Git sync
Mintlify (auto-publishing)
```

**Division of responsibilities:**
- **Obsidian**: source of all raw records (meetings, decisions, Sprint logs, research notes)
- **Claude**: bulk organization, cross-document analysis, structured document generation
- **WebKB (this site)**: queryable knowledge base published for team members

---

## Vault Directory Structure

```
cyberorigin/
├── 00-Company/          # Company identity layer (mission, strategy, team)
├── _Inbox/              # Scratch area (write here first, organize later)
├── 01-Product/          # Product roadmap, PRDs, competitive analysis
├── 02-Engineering/      # System architecture, technical decisions (ADRs), API docs
│   └── Decisions/       # Technical decision records, INDEX.md is the entry point
├── 03-Operations/       # Meeting notes, weekly reports, OKRs, Sprint logs
├── 04-Knowledge/        # Personal learning, industry research, onboarding material
├── 05-Releases/         # Release notes
└── 06-Templates/        # All note templates (prefixed _Template-)
```

---

## Daily Writing Norms

### Naming Conventions

| Type | Format | Example |
|---|---|---|
| Meeting notes | `YYYY-MM-DD-meeting.md` | `2026-04-13-eng-review.md` |
| Sprint log | `YYYY-WNN-Sprint.md` | `2026-W16-Sprint.md` |
| Technical decision | `ADR-NNN-title.md` | `ADR-012-Grace-DB-selection.md` |
| Weekly report | `YYYY-WNN-weekly.md` | `2026-W16-weekly.md` |

### Writing Flow

**Quick capture**: dump ideas into `_Inbox/` without worrying about format — just date the filename.

**Periodic organization**: weekly, sort `_Inbox/` contents into the right folders. Claude can help with categorization and formatting.

**Cross-doc references**: use Obsidian wiki-links `[[filename]]` to connect notes — don't use absolute paths.

---

## Claude's Role in Knowledge Management

### High-Value Task Examples

**Generating weekly reports**
```
Read this week's Sprint logs under 03-Operations/Sprint-Logs/,
plus completed and open tasks in Linear,
generate a weekly report and save it to 03-Operations/Weekly-Reports/.
```

**Organizing meeting notes**
```
Read today's meeting note 2026-04-13-XXX.md,
extract all action items, organize them as a table by owner,
and append to the end of the file.
```

**Generating onboarding docs**
```
Read 02-Engineering/Decisions/INDEX.md and all ADR files,
generate a technical-background guide for new hires in the "why we do it this way" voice,
organized around questions they'll hit in their first week.
```

**Risk summary**
```
Read 02-Engineering/risks.md and this month's meeting notes,
update the risk register, highlight new risks and status changes.
```

### Claude's Operational Boundaries

Claude can freely: add tags/links, create notes in `_Inbox/`, append content to notes, create weekly reports in the designated folder.

Needs confirmation: modifying the body of core docs like `Roadmap.md`, `Architecture.md`, `Vision.md`.

---

## Technical Decisions (ADRs)

### Why Record Decisions

Decision records solve the most common team knowledge gap: "why did we pick this back then?" With ADRs:
- New hires don't have to interview veterans — they read the doc
- Claude can read decision history to generate onboarding material
- Deprecated decisions keep their rationale, so we don't re-step on rakes

### ADR Format

Each decision file includes: context, decision, rationale, rejected options, consequences. See `06-Templates/_Template-ADR.md`.

**INDEX.md is the entry point**: `02-Engineering/Decisions/INDEX.md` is the index of all decisions — new hires and Claude start there without reading everything.

### Key Habits

- **Before a decision**: draft the ADR, then update status after discussion
- **After implementation**: add "actual results and lessons" — this is more valuable than the decision itself
- **On deprecation**: don't delete — mark with ❌ and document why

---

## WebKB Publishing Flow

From Obsidian content to public knowledge base:

1. Finish content in the appropriate Vault folder (Obsidian format)
2. Reformat to Mintlify doc format (frontmatter + MDX syntax)
3. Save under the right subfolder of `webkb/docs/`
4. Update the navigation config in `webkb/mint.json`
5. Git commit & push → Mintlify auto-deploys

A daily 7 AM automation scans `webkb/drafts/`, compiles pending drafts, and pushes them. To trigger manually:

```bash
cd /path/to/Obsidian Vault/WebKB
git add . && git commit -m "docs: update knowledge base" && git push origin main
```

---

## FAQ

**Should notes live in Vault or directly in WebKB?**
Raw records, meeting notes, and drafts go into Vault first. WebKB publishes the polished, team-queryable versions.

**How does multi-person Vault collaboration sync?**
Via Git (pull before editing, push after). Conflicts are rare because each person mostly owns their own folders.

**Can Claude read every file in Vault?**
Yes — via obsidian-mcp, Claude can read and write all Vault files, subject to the boundaries defined in CLAUDE.md.

## Related Reading

<CardGroup cols={2}>
  <Card title="Onboarding Guide" icon="user-plus" href="/en/guides/onboarding">
    First-week ramp-up and tooling access
  </Card>
  <Card title="Session-to-Mintlify Pipeline" icon="arrows-rotate" href="/en/guides/session-to-mintlify-pipeline">
    Auto-compile conversations into published docs
  </Card>
  <Card title="Company Overview" icon="building" href="/en/company/overview">
    Organizational context for the knowledge system
  </Card>
</CardGroup>
