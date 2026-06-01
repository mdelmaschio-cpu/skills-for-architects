# CLAUDE.md — Architecture Studio

## Overview

**Architecture Studio** (`AlpacaLabsLLC/skills-for-architects`) is a Claude plugin marketplace for architects, designers, and AEC (Architecture, Engineering, Construction) professionals. It teaches Claude domain-specific workflows across the full architectural project lifecycle — from site research and zoning analysis through specifications, materials selection, sustainability assessment, and presentations.

Built by [ALPA](https://alpa.llc). Current version: `1.1.1`. MIT licensed.

**Scale:** 7 agents · 37 skills · 7 rules · 3 hooks · 9 plugins

**Install:**
```bash
# Claude Desktop: Customize → Browse plugins → + → Add marketplace from GitHub
# Claude Code:
claude plugin marketplace add AlpacaLabsLLC/skills-for-architects
claude plugin install 01-site-planning@skills-for-architects
```

**Entry point:** `/studio [describe what you need]`

---

## Directory Structure

```
skills-for-architects/
├── .claude-plugin/
│   └── marketplace.json          ← Marketplace manifest (name, version, plugin list)
├── .github/
│   └── workflows/lint.yml        ← CI: runs scripts/lint.sh on push/PR to main
├── agents/                       ← Orchestration personas (7 agents)
│   ├── README.md                 ← Agent index + how agents differ from skills
│   ├── brand-manager.md
│   ├── ffe-designer.md
│   ├── nyc-zoning-expert.md
│   ├── product-and-materials-researcher.md
│   ├── site-planner.md
│   ├── sustainability-specialist.md
│   └── workplace-strategist.md
├── hooks/                        ← Opt-in event-driven automations (3 hooks)
│   ├── README.md                 ← Installation instructions
│   ├── post-output-metadata.sh
│   ├── post-write-disclaimer-check.sh
│   ├── pre-commit-spec-lint.sh
│   └── settings-snippet.json    ← Drop-in Claude Code settings fragment
├── plugins/                      ← 9 plugins organized by project lifecycle
│   ├── 00-due-diligence/         7 skills — NYC property data (landmarks, permits, etc.)
│   ├── 01-site-planning/         4 skills — climate, transit, demographics, history
│   ├── 02-zoning-analysis/       2 skills — NYC buildable envelope + 3D viewer
│   ├── 03-programming/           2 skills — space programs + occupancy loads
│   ├── 04-specifications/        1 skill  — CSI MasterFormat outline specs
│   ├── 05-sustainability/        4 skills — EPD parsing, research, comparison, spec
│   ├── 06-materials-research/   12 skills — FF&E product research through export
│   ├── 07-presentations/         3 skills — slide decks, color palettes, image resizing
│   └── 08-dispatcher/            2 skills — /studio router + /skills menu
├── rules/                        ← Always-on conventions (7 rules)
│   ├── README.md
│   ├── code-citations.md
│   ├── csi-formatting.md
│   ├── output-formatting.md
│   ├── professional-disclaimer.md
│   ├── terminology.md
│   ├── transparency.md
│   └── units-and-measurements.md
├── scripts/
│   └── lint.sh                   ← Structural lint (6 checks)
├── CHANGELOG.md
├── PATTERNS.md                   ← Canonical ALPA plugin conventions (10 principles)
└── README.md
```

### Plugin Internal Structure

Each plugin follows this layout:

```
plugins/<NN>-<name>/
├── .claude-plugin/
│   └── plugin.json               ← Plugin manifest (name, version, description)
├── README.md
└── skills/
    └── <skill-name>/
        ├── SKILL.md              ← Skill instructions (YAML frontmatter + body)
        ├── README.md
        └── [data/schema files]   ← Supporting JSON, markdown references
```

---

## Key Files

| File | Purpose |
|------|---------|
| `.claude-plugin/marketplace.json` | Marketplace manifest — lists all 9 plugins with `source` paths |
| `plugins/<N>/**.claude-plugin/plugin.json` | Per-plugin manifest — `name`, `version`, `description` |
| `plugins/<N>/skills/<name>/SKILL.md` | Skill instructions — YAML frontmatter + workflow body |
| `agents/*.md` | Agent orchestration logic — which skills to call, in what order |
| `rules/*.md` | Cross-cutting output conventions loaded automatically |
| `hooks/*.sh` | Bash scripts for lifecycle events (PostToolUse, PreToolUse) |
| `hooks/settings-snippet.json` | Claude Code settings fragment for enabling hooks |
| `PATTERNS.md` | Canonical ALPA plugin-building conventions (the authoritative design doc) |
| `scripts/lint.sh` | Structural lint — run before committing |

### SKILL.md Frontmatter

Every `SKILL.md` opens with YAML frontmatter:

```yaml
---
name: skill-name           # kebab-case, matches directory name
description: ...           # trigger-phrase-rich; drives dispatcher invocation
allowed-tools:             # scoped to what THIS skill needs
  - WebFetch
  - Write
user-invocable: true       # present when slash-invocable
disable-model-invocation: true  # optional: slash-only, no auto-routing
---
```

---

## Architecture Patterns

> Full detail lives in `PATTERNS.md`. This is the authoritative reference — read it when adding skills.

### Layer Model

| Layer | Behavior | Invocation |
|-------|----------|-----------|
| **Rules** | Always-on output conventions | Loaded automatically |
| **Skills** | Single-purpose tools; one verb each | `/skill-name [args]` |
| **Agents** | Orchestrate skills; exercise judgment | Describe task in natural language |
| **Hooks** | Fire on lifecycle events | Opt-in via settings.json |

### Dispatcher Pattern

The `/studio` skill is the entry point. It:
1. Reads user intent after `/studio`
2. Classifies against a routing table
3. Hands off to the right agent (reads `agents/<name>.md`) or skill
4. Never does the work itself — the agent files contain all orchestration logic

Agents load their own skill files at runtime via `Read agents/<name>.md`. The dispatcher does not contain orchestration logic.

### Naming Conventions

| Layer | Convention | Example |
|-------|-----------|---------|
| Marketplace | kebab-case product family | `skills-for-architects` |
| Plugin | `NN-kebab-case` | `00-due-diligence` |
| Skill (multi-plugin) | `<verb>` or `<domain>-<verb>` | `nyc-landmarks`, `spec-writer` |
| Dispatcher skill | matches plugin name | `studio` |

### Versioning (Three Artifacts)

On every shipped change, all three must move together:
1. `version` field in `plugin.json` and/or `marketplace.json` `metadata.version`
2. Git tag: `git tag -a vX.Y.Z <sha> -m "vX.Y.Z — description"`
3. GitHub release: `gh release create vX.Y.Z --title "vX.Y.Z — …"`

Version bump rules: patch = fixes/docs, minor = new skills/behavior, major = breaking layout.

### Marker-Driven Hooks

Hooks check for structured markers, not keywords. Regulatory skill outputs must end with:

```markdown
> **Disclaimer:** This is an AI-generated analysis for preliminary planning purposes. All findings must be verified by a licensed professional before use in design, permitting, or regulatory submissions.

<!-- architecture-studio:requires-disclaimer -->
```

The `post-write-disclaimer-check` hook looks for `<!-- architecture-studio:requires-disclaimer -->` and verifies the canonical disclaimer text is also present. This avoids false positives on non-regulatory docs (READMEs, changelogs) that happen to mention regulated terms.

---

## Rules (Always-On Conventions)

These apply to all skill outputs automatically — they are reference documents, not invocable commands.

| Rule | Key Behaviors |
|------|--------------|
| `units-and-measurements` | Imperial primary; GSF/USF/RSF distinctions; dimension formatting |
| `code-citations` | Building code edition years; section symbols; jurisdiction awareness |
| `professional-disclaimer` | Regulatory outputs require disclaimer + marker; never say "complies with" |
| `csi-formatting` | MasterFormat 2018 section format: `09 29 00 — Gypsum Board` (space-separated, em-dash) |
| `terminology` | AEC standard terms; abbreviation conventions; material names |
| `output-formatting` | Tables for comparative data; source attribution per section; YAML front matter in reports |
| `transparency` | Link sources; expose inputs; make outputs verifiable |

---

## Hooks Setup

Hooks are opt-in. Install:

```bash
chmod +x ~/.claude/plugins/skills-for-architects/hooks/*.sh
```

Then merge `hooks/settings-snippet.json` into `~/.claude/settings.json` (global) or `.claude/settings.json` (project). The snippet wires:
- `PostToolUse[Write]` → `post-write-disclaimer-check.sh` + `post-output-metadata.sh`
- `PreToolUse[Bash(git commit *)]` → `pre-commit-spec-lint.sh`

All hooks warn but do not block by default. To enforce, change `exit 0` to `exit 2` in the script.

---

## Development Workflows

### Adding a Skill

1. Create `plugins/<NN>-<plugin>/skills/<skill-name>/SKILL.md` with required frontmatter (`name`, `description`)
2. Add `plugins/<NN>-<plugin>/skills/<skill-name>/README.md`
3. Scope `allowed-tools` to only what the skill needs
4. Update `plugins/<NN>-<plugin>/.claude-plugin/plugin.json` — bump `version`
5. Update `marketplace.json` `metadata.version`
6. Update `README.md` counts (headline, details summary, plugin table, catalog row, dispatcher `/skills` menu in `plugins/08-dispatcher/skills/skills-menu/SKILL.md`)
7. Run `./scripts/lint.sh` — must pass all 6 checks
8. Add `CHANGELOG.md` entry
9. Commit, tag, cut GitHub release

### Lint

```bash
./scripts/lint.sh
```

Six checks (all must pass):
1. No tracked `.DS_Store` files
2. JSON validity (`marketplace.json`, `plugin.json`, etc.)
3. `SKILL.md` frontmatter has `name` and `description`
4. Count consistency — README claims, plugin tables, `marketplace.json`, and actual file count must all match
5. Internal markdown links resolve
6. `shellcheck` on `hooks/*.sh`

CI runs lint on push to `main` and on every PR (`.github/workflows/lint.yml`). Requires `pyyaml` (`pip install pyyaml`).

### Updating Counts

When adding or removing skills, update ALL of these in sync (lint will catch drift):
- `README.md` headline: `**37 skills**`
- `README.md` details block: `<summary><strong>All 37 skills</strong></summary>`
- Plugin table row count for the affected plugin
- Catalog rows inside the details block
- `plugins/08-dispatcher/skills/skills-menu/SKILL.md` skill count
- `marketplace.json` plugin list (if adding/removing plugins)

---

## Agents Reference

Agents are markdown files in `agents/` that define orchestration personas. They are read at runtime by the dispatcher (`/studio`) and followed as workflow instructions.

| Agent | Domain | Key Skills |
|-------|--------|-----------|
| `site-planner` | Site research | Runs 4 analysis skills in parallel; synthesizes a site brief |
| `nyc-zoning-expert` | Due diligence + zoning | 6 property data skills + zoning envelope + 3D viewer |
| `workplace-strategist` | Programming | Occupancy calculator + workplace programmer |
| `product-and-materials-researcher` | Materials research | Product search, spec extraction, PDF parsing, tagging |
| `ffe-designer` | FF&E schedules | Schedule cleanup, room packages, SIF export, QA |
| `sustainability-specialist` | Sustainability | EPD research, comparison, GWP thresholds, LEED spec |
| `brand-manager` | Presentations | Slide decks, color palettes, image resizing |

Agent files define: when to use, how to work (step-by-step), synthesis rules, handoff points, and what the agent explicitly does NOT do.

---

## Domain Knowledge

### NYC Property Data

Skills in `00-due-diligence` query **NYC Open Data (Socrata)** — no API key required for basic queries. PLUTO is the primary resolution layer (address → BBL/BIN).

Key datasets:
- PLUTO: `https://data.cityofnewyork.us/resource/64uk-42ks.json` — lot data, zoning, owner
- DOB Permits legacy: `resource/ipu4-2q9a.json` (uses `bin__` double underscore)
- DOB NOW permits: `resource/rbx6-tga4.json` (uses `bin` single)
- LPC Landmarks: `resource/7mgd-s57w.json`
- ACRIS: three-table join (Legals → Master → Parties) keyed on BBL

Rate limiting: HTTP 429 → wait 5s, retry once. Set `NYC_SOCRATA_TOKEN` for higher limits.

### FF&E Product Schema

Skills in `06-materials-research` share a 33-column product schema (`plugins/06-materials-research/schema/product-schema.md`). One row per product. Key conventions:
- 22 canonical category terms (e.g., Chair, Table, Light, Storage)
- Item number prefixes by category (S-01, T-01, L-01, etc.)
- Status values: `saved` → `specified` → `quoted` / `archived`
- Tags column (AD): append-only, never overwrite
- `Source` column (AG): tracks which skill created the row

CSV header is fixed — use exact column order from schema.

### CSI Specification Formatting

Section numbers use `MasterFormat 2018` format: `09 29 00 — Gypsum Board`
- Space-separated division/section/subsection
- Em-dash (—) before title
- Pre-commit hook (`pre-commit-spec-lint.sh`) flags malformed formats: `092900`, `09-29-00`, `09.29.00`, or missing titles

---

## Notes for AI Assistants

- **Read `PATTERNS.md` first** when making structural changes — it contains the canonical conventions for skill design, naming, versioning, and the reasoning behind each rule.
- **Skills are thin and single-verb.** If a `SKILL.md` starts handling multiple tasks, decompose it.
- **Skill contracts are explicit.** Document what input is expected, what is produced, and what to run next — in the SKILL.md body, in plain English.
- **Count consistency is enforced by CI.** Adding a skill without updating all README counts and the skills-menu will fail lint.
- **Versioning requires three artifacts.** JSON version field + git tag + GitHub release must all move on every shipped change. Bumping only one causes Cowork/Code update failures.
- **Marker-driven hooks, not keyword-sniffing.** Regulatory outputs need `<!-- architecture-studio:requires-disclaimer -->` at end of file for the hook to validate them. Do not add the marker to non-regulatory outputs.
- **Hard rules in skill bodies carry WHY.** When encoding a fix, document the incident that caused it (see PATTERNS.md §10 for examples). Future maintainers need the reasoning to judge edge cases.
- **Agent files are runtime documents.** The `/studio` dispatcher reads `agents/<name>.md` via the `Read` tool and follows its instructions — do not duplicate orchestration logic in the dispatcher.
- **`post-output-metadata.sh` skips** README.md, SKILL.md, CLAUDE.md, and files inside `rules/`, `hooks/`, or `.claude-plugin/` directories. It only stamps new project-output markdown reports.
- **`allowed-tools` must be minimal.** Scope each skill to exactly the tools it needs — not the union of what the plugin can use.
