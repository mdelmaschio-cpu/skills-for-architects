# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working in this repository.

## Project Purpose

**skills-for-architects** is a Claude plugin marketplace for architects, designers, and AEC (Architecture, Engineering, Construction) professionals. It provides specialized workflows for site analysis, NYC zoning/due diligence, space programming, CSI specifications, FF&E materials research, sustainability (EPDs/GWP), and presentations — installable as plugins in Claude Desktop or Claude Code.

There is no application code, no build system, and no package manager at the repo root. All content is plain Markdown, JSON, and Bash. The "product" is a collection of structured Markdown skill files that Claude reads at runtime, plus JSON manifests that Claude Desktop and Claude Code use for plugin discovery.

## Repository Structure

```
skills-for-architects/
├── .claude-plugin/
│   └── marketplace.json          # Multi-plugin marketplace registry (9 plugins); version must stay in sync
├── .github/
│   └── workflows/
│       └── lint.yml              # CI: runs scripts/lint.sh on push/PR to main
├── agents/                       # Orchestration personas (Markdown files Claude reads via Read tool)
│   ├── README.md
│   ├── site-planner.md
│   ├── nyc-zoning-expert.md
│   ├── workplace-strategist.md
│   ├── product-and-materials-researcher.md
│   ├── ffe-designer.md
│   ├── sustainability-specialist.md
│   └── brand-manager.md
├── hooks/                        # Event-driven Bash automations (opt-in, Claude Code only)
│   ├── README.md
│   ├── post-write-disclaimer-check.sh   # After Write: validates disclaimer marker presence
│   ├── post-output-metadata.sh          # After Write: stamps YAML front matter
│   ├── pre-commit-spec-lint.sh          # Before git commit: checks CSI section numbers
│   └── settings-snippet.json            # Merge into ~/.claude/settings.json to install hooks
├── plugins/                      # 9 installable plugin bundles
│   ├── 00-due-diligence/         # 7 skills: NYC landmarks, DOB, ACRIS, HPD, BSA, property report
│   │   ├── .claude-plugin/plugin.json
│   │   ├── README.md
│   │   └── skills/
│   │       ├── nyc-landmarks/SKILL.md
│   │       ├── nyc-dob-permits/SKILL.md
│   │       ├── nyc-dob-violations/SKILL.md
│   │       ├── nyc-acris/SKILL.md
│   │       ├── nyc-hpd/SKILL.md
│   │       ├── nyc-bsa/SKILL.md
│   │       └── nyc-property-report/{SKILL.md,socrata-reference.md}
│   ├── 01-site-planning/         # 4 skills: environmental, site-history, mobility, demographics
│   ├── 02-zoning-analysis/       # 2 skills: zoning-analysis-nyc, zoning-envelope
│   │   └── skills/zoning-analysis-nyc/
│   │       └── zoning-rules/     # 10 bundled NYC Zoning Resolution reference docs
│   ├── 03-programming/           # 2 skills: workplace-programmer, occupancy-calculator
│   │   └── skills/*/data/        # JSON data files: archetypes, space-types, occupancy-load-factors
│   ├── 04-specifications/        # 1 skill: spec-writer (CSI MasterFormat)
│   ├── 05-sustainability/        # 4 skills: epd-parser, epd-research, epd-compare, epd-to-spec
│   ├── 06-materials-research/    # 12 skills: product-research, spec extraction, SIF crosswalk, etc.
│   │   └── schema/               # product-schema.md, sheet-conventions.md, sif-crosswalk.md
│   ├── 07-presentations/         # 3 skills: slide-deck-generator, color-palette-generator, resize-images
│   └── 08-dispatcher/            # 2 skills: studio (entry-point router), skills-menu
├── rules/                        # 7 always-on convention docs (loaded automatically, never invoked)
│   ├── README.md
│   ├── units-and-measurements.md
│   ├── code-citations.md
│   ├── professional-disclaimer.md  # Canonical disclaimer block + required marker definition
│   ├── csi-formatting.md
│   ├── terminology.md
│   ├── output-formatting.md
│   └── transparency.md
├── scripts/
│   └── lint.sh                   # 6-check structural lint script; required before every commit
├── CHANGELOG.md                  # Keep-a-Changelog format, semver
├── PATTERNS.md                   # Canonical 10-rule plugin development conventions — READ FIRST
├── LICENSE                       # MIT
└── README.md                     # Full marketplace README with architecture diagram and skill tables
```

## Development Environment Setup

There is no build system or package manager. Setup is minimal:

**For lint (required):**
```bash
pip install pyyaml   # only Python dependency; used by inline scripts inside lint.sh
```

**Optional — for full local lint parity with CI:**
```bash
brew install shellcheck        # macOS
sudo apt-get install shellcheck  # Ubuntu/Debian
```

**CI environment:** ubuntu-latest, Python 3.12, pyyaml installed via pip, shellcheck available system-wide.

## Key Commands

### Lint (run before every commit)

```bash
./scripts/lint.sh
```

This is the only required command. It runs 6 checks:
1. No tracked `.DS_Store` files
2. JSON validity (`jq`) — all `plugin.json` and `marketplace.json` files
3. SKILL.md YAML frontmatter — `name` and `description` fields must be present (parsed via PyYAML)
4. Count consistency — README headline counts, details block count, per-plugin counts, `skills-menu` SKILL.md claim, and `marketplace.json` plugin list must all match actual file counts
5. Internal markdown link resolution
6. `shellcheck` on all `hooks/*.sh` scripts

CI runs this on every push to main and every PR. Do not skip it.

### Versioning (required on every shipped change — three artifacts)

```bash
# 1. Bump version in the relevant plugin.json AND marketplace.json metadata.version
# 2. Add CHANGELOG.md entry under ## [X.Y.Z] - YYYY-MM-DD
# 3. Commit everything together (version bump + CHANGELOG + actual change)
git tag -a vX.Y.Z <sha> -m "vX.Y.Z — short description"
git push origin vX.Y.Z
gh release create vX.Y.Z --title "vX.Y.Z — ..." --notes-file <changelog-section-tempfile>
```

All three artifacts (JSON version field, git tag, GitHub release) must move together.

## Architecture Overview

The repo follows a **multi-plugin marketplace layout**: one repo, multiple independently-installable plugins, one marketplace registry.

```
User invokes /studio
      |
      v
plugins/08-dispatcher/skills/studio/SKILL.md   (entry-point router)
      |
      v
reads agents/<agent>.md                         (orchestration persona)
      |
      v
calls specific skill(s) in plugins/<NN-*/skills/<skill>/SKILL.md
```

**Layers:**

| Layer | Location | Role |
|-------|----------|------|
| Marketplace registry | `.claude-plugin/marketplace.json` | Plugin discovery for Claude Desktop/Code |
| Plugin manifests | `plugins/<NN-name>/.claude-plugin/plugin.json` | Per-plugin metadata and versioning |
| Skills | `plugins/<NN-name>/skills/<skill>/SKILL.md` | Single-purpose workflow instructions |
| Agents | `agents/<agent>.md` | Orchestration personas; define which skills to call and in what order |
| Rules | `rules/*.md` | Always-on convention docs; loaded automatically, never invoked directly |
| Hooks | `hooks/*.sh` | Event-driven Bash automations; opt-in via settings-snippet.json |
| Reference data | `plugins/*/skills/*/data/`, `plugins/*/schema/`, `zoning-rules/` | Supporting data Claude reads on demand |

**Dispatcher pattern:** The `studio` skill in `08-dispatcher` is the single entry point. Hard rules from production bugs belong in the dispatcher (global inheritance) AND in each affected sub-skill (enforced even when dispatcher is bypassed).

## Key Files

| File | Purpose |
|------|---------|
| `PATTERNS.md` | Canonical 10-rule reference for plugin/marketplace conventions. Read this before any structural change. |
| `.claude-plugin/marketplace.json` | Marketplace registry listing all 9 plugins. Version must stay in sync with actual plugin directories and README counts. |
| `scripts/lint.sh` | The only required command. 6 structural checks. Run before every commit. |
| `plugins/08-dispatcher/skills/studio/SKILL.md` | Entry-point router for the entire marketplace. |
| `rules/professional-disclaimer.md` | Defines the canonical disclaimer block and the `<!-- architecture-studio:requires-disclaimer -->` marker. |
| `hooks/settings-snippet.json` | JSON to merge into `~/.claude/settings.json` to enable the three hooks locally. |
| `plugins/06-materials-research/schema/product-schema.md` | 33-column product sheet schema shared by all 12 materials-research skills. |
| `plugins/02-zoning-analysis/skills/zoning-analysis-nyc/zoning-rules/` | 10 bundled NYC Zoning Resolution reference docs. |

## Code Conventions and Patterns

### Skill File Conventions

Every skill lives at: `plugins/<NN-plugin-name>/skills/<skill-name>/SKILL.md`

SKILL.md must begin with YAML frontmatter (`---` delimited) containing at minimum:
- `name`: kebab-case, must match the directory name
- `description`: trigger-phrase-rich; this is what drives model invocation — optimize for discoverability

Optional frontmatter fields:
- `allowed-tools`: scoped only to what this specific skill actually uses
- `user-invocable: true`: for skills that can be invoked via slash command
- `disable-model-invocation: true`: for slash-only skills

Skills are **single-purpose** (one verb). If a SKILL.md exceeds ~500 lines or handles multiple distinct verbs, split it.

### Plugin Manifest Conventions

Each plugin has `.claude-plugin/plugin.json` with: `name`, `version` (semver), `description`, `author`, `keywords`, `homepage`, `license`.

The marketplace `.claude-plugin/marketplace.json` lists all plugins with `name`, `source`, `description`.

### Naming Conventions

- Plugin directories: kebab-case with numbered prefix (`00-`, `01-`, etc.)
- Skill names: bare verb in kebab-case (`nyc-landmarks`, `spec-writer`)
- Agent files: `<role>.md` in kebab-case under `agents/`

### Count Consistency (enforced by lint)

These counts must all match actual file counts at all times:
- README headline: `**N plugins**` and `**N skills**`
- README details block: `All N skills`
- Per-plugin skill counts in README table
- `skills-menu` SKILL.md claim
- `marketplace.json` plugin list count vs. actual plugin directories

When adding any skill or plugin, update ALL of these in the same commit.

### Disclaimer and Regulatory Output Rules

- **Never say** "complies with" — say "appears consistent with [code section]"
- **Never say** "no violations" — say "no violations were identified in the data reviewed"
- All zoning, occupancy, structural, environmental, and code outputs require the canonical disclaimer block followed by `<!-- architecture-studio:requires-disclaimer -->` as the last line
- Hooks are **marker-driven**, not keyword-sniffed

## How AI Assistants Should Work in This Repo

**Before any structural change:** Read `PATTERNS.md`.

**Before every commit:** Run `./scripts/lint.sh`. Fix all failures before pushing.

**When adding a new skill:**
1. Create `plugins/<NN-plugin-name>/skills/<new-skill>/SKILL.md` with correct YAML frontmatter
2. Create `plugins/<NN-plugin-name>/skills/<new-skill>/README.md`
3. Update ALL README counts and `skills-menu` SKILL.md count claim
4. Run `./scripts/lint.sh`

**When adding a new plugin:**
1. Create `plugins/<NN-new-plugin>/` with `.claude-plugin/plugin.json`, `README.md`, and `skills/` subdirectory
2. Add entry to `.claude-plugin/marketplace.json`
3. Update all README counts and `skills-menu`
4. Run `./scripts/lint.sh`

**When bumping versions:** All three artifacts must move together — `plugin.json` version field, `marketplace.json` metadata.version, git tag AND GitHub release.

**`allowed-tools` scoping:** Scope only to what that specific skill actually uses.

**No new build dependencies:** The only external dependency is `pyyaml` for lint.

**Hard rules belong in two places:** Both the dispatcher (`08-dispatcher`) AND each affected sub-skill body.

**Agent files are read-only reference documents:** Files in `agents/` are Markdown documents Claude reads via the Read tool. They do not contain tool calls.
