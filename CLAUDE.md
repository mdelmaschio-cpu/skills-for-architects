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
# Install shellcheck (varies by OS)
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

### Install (user-facing, not development)

**Claude Desktop:**
```
Customize → Browse plugins → + → Add marketplace from GitHub → AlpacaLabsLLC/skills-for-architects
```

**Claude Code:**
```bash
claude plugin marketplace add AlpacaLabsLLC/skills-for-architects
claude plugin install 01-site-planning@skills-for-architects   # install a specific plugin
```

**Enable hooks (after plugin install):**
```bash
chmod +x ~/.claude/plugins/skills-for-architects/hooks/*.sh
# Then merge hooks/settings-snippet.json into ~/.claude/settings.json
```

### Versioning (required on every shipped change — three artifacts)

```bash
# 1. Bump version in the relevant plugin.json AND marketplace.json metadata.version
# 2. Add CHANGELOG.md entry under ## [X.Y.Z] - YYYY-MM-DD
# 3. Commit everything together (version bump + CHANGELOG + actual change)
git tag -a vX.Y.Z <sha> -m "vX.Y.Z — short description"
git push origin vX.Y.Z
gh release create vX.Y.Z --title "vX.Y.Z — ..." --notes-file <changelog-section-tempfile>
```

All three artifacts (JSON version field, git tag, GitHub release) must move together. Skipping any one leaves discoverability holes: plugin runtimes won't see updates without the JSON bump; `git checkout` won't work without the tag; no shareable URL without the GitHub release.

There are no build, test, or compile commands. No `package.json`, no `Makefile`, no `pyproject.toml`.

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

**Dispatcher pattern:** The `studio` skill in `08-dispatcher` is the single entry point. It reads agent files from `agents/` and routes to the right agent or plugin skill. Hard rules captured from production bugs belong in the dispatcher (for global inheritance) AND repeated in each affected sub-skill (so they are enforced even when the dispatcher is bypassed).

## Key Files

| File | Purpose |
|------|--------|
| `PATTERNS.md` | Canonical 10-rule reference for plugin/marketplace conventions. Read this before any structural change — it documents the WHY behind every convention and covers rules distilled from real production bugs. |
| `.claude-plugin/marketplace.json` | Marketplace registry listing all 9 plugins. Version must stay in sync with actual plugin directories and README counts. |
| `scripts/lint.sh` | The only required command. 6 structural checks. Run before every commit. |
| `plugins/08-dispatcher/skills/studio/SKILL.md` | Entry-point router for the entire marketplace. Routes requests to agents and skills. |
| `rules/professional-disclaimer.md` | Defines the canonical disclaimer block and the `<!-- architecture-studio:requires-disclaimer -->` marker required on all regulatory outputs. |
| `hooks/settings-snippet.json` | JSON to merge into `~/.claude/settings.json` to enable the three hooks locally. |
| `plugins/06-materials-research/schema/product-schema.md` | 33-column product sheet schema shared by all 12 materials-research skills. |
| `plugins/02-zoning-analysis/skills/zoning-analysis-nyc/zoning-rules/` | 10 bundled NYC Zoning Resolution reference docs consumed by the zoning skill. |

## Code Conventions and Patterns

### Skill File Conventions

Every skill lives at: `plugins/<NN-plugin-name>/skills/<skill-name>/SKILL.md`

SKILL.md must begin with YAML frontmatter (`---` delimited) containing at minimum:
- `name`: kebab-case, must match the directory name
- `description`: trigger-phrase-rich; this is what drives model invocation — optimize for discoverability

Optional frontmatter fields:
- `allowed-tools`: scoped only to what this specific skill actually uses (not the union of all plugin tools — overly broad grants are a lint/audit concern)
- `user-invocable: true`: for skills that can be invoked via slash command
- `disable-model-invocation: true`: for slash-only skills that should not be auto-invoked

Skills are **single-purpose** (one verb). If a SKILL.md exceeds ~500 lines or handles multiple distinct verbs, split it into separate skills.

Each skill directory also has a `README.md`.

Supporting data and reference files go in subdirectories: `data/`, `zoning-rules/`, `schema/`, etc.

### Plugin Manifest Conventions

Each plugin has `.claude-plugin/plugin.json` with: `name`, `version` (semver), `description`, `author`, `keywords`, `homepage`, `license`.

The marketplace `.claude-plugin/marketplace.json` lists all plugins with `name`, `source` (e.g., `./plugins/<name>`), `description`.

### Naming Conventions

- Plugin directories: kebab-case with numbered prefix (`00-`, `01-`, etc.)
- Skill names: bare verb in kebab-case (`nyc-landmarks`, `spec-writer`) — already namespaced by plugin directory
- Dispatcher skill name matches plugin name (`studio` for `08-dispatcher`)
- Agent files: `<role>.md` in kebab-case under `agents/`

### Count Consistency (enforced by lint)

These counts must all match actual file counts at all times:
- README headline: `**N plugins**` and `**N skills**`
- README details block: `All N skills`
- Per-plugin skill counts in README table
- `skills-menu` SKILL.md claim
- `marketplace.json` plugin list count vs. actual plugin directories

When adding any skill or plugin, update ALL of these in the same commit. The lint script will catch any drift.

### Disclaimer and Regulatory Output Rules

- **Never say** "complies with" — say "appears consistent with [code section]"
- **Never say** "no violations" — say "no violations were identified in the data reviewed"
- All zoning, occupancy, structural, environmental, and code outputs require the canonical disclaimer block (defined in `rules/professional-disclaimer.md`) followed by the marker `<!-- architecture-studio:requires-disclaimer -->` as the last line of the file
- Hooks are **marker-driven**, not keyword-sniffed. The `post-write-disclaimer-check.sh` hook validates that the marker is present; it does not scan for keywords
- Hooks warn but do not block by default (exit 0 at warning; change to exit 2 in the hook script to enforce blocking)

### Rules vs. Agents vs. Skills

| Type | Location | How it works |
|------|----------|-------------|
| Rules | `rules/*.md` | Passive reference docs; loaded automatically; never invoked directly |
| Agents | `agents/*.md` | Orchestration personas; Claude reads them via Read tool when the dispatcher routes a request; they define which skills to call and in what order; they do not contain tool calls directly |
| Skills | `plugins/*/skills/*/SKILL.md` | Single-purpose workflow instructions; invoked by the dispatcher or directly by the user |

### Cross-References Between Skills

Cross-plugin references use the full slash invocation path as documented in `PATTERNS.md` section 3. There is no implicit shared state between skills. Contracts (expected input/output format) are documented in each SKILL.md body.

### MCP Tools

Some skills reference MCP tools (e.g., `mcp__google-sheets__*` in the product-research skill). These require users to have the relevant MCP servers configured separately in their Claude Desktop or Claude Code settings. Document any MCP dependencies in the skill's frontmatter `allowed-tools` field and in its README.

## How AI Assistants Should Work in This Repo

**Before any structural change:** Read `PATTERNS.md`. It documents the WHY behind every convention and covers 10 rules distilled from real production bugs.

**Before every commit:** Run `./scripts/lint.sh`. Fix all failures before pushing. CI will catch them anyway, but fixing locally is faster.

**When adding a new skill:**
1. Create `plugins/<NN-plugin-name>/skills/<new-skill>/SKILL.md` with correct YAML frontmatter
2. Create `plugins/<NN-plugin-name>/skills/<new-skill>/README.md`
3. Update the README.md headline counts (`**N skills**`)
4. Update the README.md details block (`All N skills`)
5. Update the per-plugin row count in the README table
6. Update the `skills-menu` SKILL.md count claim
7. If adding a new plugin: update `marketplace.json` and the README plugin count
8. Run `./scripts/lint.sh` — it will catch any count drift you missed

**When adding a new plugin:**
1. Create `plugins/<NN-new-plugin>/` with `.claude-plugin/plugin.json`, `README.md`, and `skills/` subdirectory
2. Add entry to `.claude-plugin/marketplace.json`
3. Update all README counts
4. Update `skills-menu` SKILL.md
5. Run `./scripts/lint.sh`

**When bumping versions:**
All three artifacts must move together in a single logical change:
- The `version` field in the relevant `plugin.json` (for behavior changes in that plugin)
- The `metadata.version` in `marketplace.json` (for any repo-level change)
- A git tag AND a GitHub release (both required — neither alone is sufficient)

**When writing or editing skills that produce regulatory outputs:**
Ensure the skill's output instructions include the canonical disclaimer block followed by `<!-- architecture-studio:requires-disclaimer -->` as the final line. Do not rely on keyword matching.

**`allowed-tools` scoping:** When writing or editing SKILL.md frontmatter, scope `allowed-tools` only to what that specific skill actually uses — not the union of all tools in the plugin or marketplace.

**No new build dependencies:** There is no build system, no package manager, no compile step. Keep it that way. The only external dependency is `pyyaml` for lint. Do not introduce `npm`, `cargo`, `go build`, or any other build toolchain unless there is an extraordinary reason and it is explicitly approved.

**Hard rules belong in two places:** Any rule captured from a production bug belongs in both the dispatcher (`08-dispatcher/skills/studio/SKILL.md`) for global inheritance AND in each affected sub-skill body, so it is enforced even when the dispatcher is bypassed.

**Agent files are read-only reference documents:** Files in `agents/` are Markdown documents that Claude reads via the Read tool when routing a request. They do not contain tool calls. They define orchestration logic (which skills to call, in what order, what judgment to apply). Do not add executable content to agent files.
