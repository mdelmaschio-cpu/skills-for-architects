# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

**Architecture Studio** is a multi-plugin Claude Code marketplace for architects, designers, and AEC (Architecture, Engineering & Construction) professionals. It ships **7 agents**, **37 skills**, **7 rules**, and **3 hooks** across **9 plugins**.

Built by [ALPA](https://alpa.llc). The canonical plugin-authoring reference for this repo (and all ALPA plugins) is [`PATTERNS.md`](./PATTERNS.md) — read it before adding anything.

## Repository Structure

```
skills-for-architects/
├── PATTERNS.md                         # Canonical plugin-authoring reference (READ FIRST)
├── CHANGELOG.md                        # Version history
├── README.md
├── LICENSE                             # MIT
├── .gitignore
├── .claude-plugin/
│   └── marketplace.json                # Marketplace descriptor (lists all 9 plugins)
├── .github/
│   └── workflows/
│       └── lint.yml                    # CI: runs scripts/lint.sh on every push
├── agents/                             # 7 orchestration agents
│   ├── README.md
│   ├── site-planner.md
│   ├── nyc-zoning-expert.md
│   ├── workplace-strategist.md
│   ├── product-and-materials-researcher.md
│   ├── ffe-designer.md
│   ├── sustainability-specialist.md
│   └── brand-manager.md
├── rules/                              # 7 always-on output conventions
│   ├── README.md
│   ├── units-and-measurements.md
│   ├── code-citations.md
│   ├── professional-disclaimer.md
│   ├── csi-formatting.md
│   ├── terminology.md
│   ├── output-formatting.md
│   └── transparency.md
├── hooks/                              # 3 event-driven automations
│   ├── README.md
│   ├── settings-snippet.json           # Paste into Claude Code settings to activate hooks
│   ├── post-write-disclaimer-check.sh  # Warns on missing professional disclaimer
│   ├── post-output-metadata.sh         # Stamps markdown reports with YAML front matter
│   └── pre-commit-spec-lint.sh         # Flags malformed CSI section numbers
├── scripts/
│   └── lint.sh                         # Structural lint (JSON, frontmatter, count consistency)
└── plugins/                            # 9 installable skill bundles
    ├── 00-due-diligence/               # 7 skills: NYC property records
    ├── 01-site-planning/               # 4 skills: environmental, mobility, demographics, history
    ├── 02-zoning-analysis/             # 2 skills: NYC buildable envelope + 3D viewer
    ├── 03-programming/                 # 2 skills: space programs, occupancy loads
    ├── 04-specifications/              # 1 skill: CSI outline specs
    ├── 05-sustainability/              # 4 skills: EPD parsing, research, comparison, thresholds
    ├── 06-materials-research/          # 12 skills: FF&E product research through SIF export
    ├── 07-presentations/               # 3 skills: slide decks, colour palettes, image resizing
    └── 08-dispatcher/                  # 2 skills: /studio router + /skills help menu
```

Each plugin directory contains:
```
plugins/<N>-<name>/
├── .claude-plugin/
│   └── plugin.json                     # Plugin descriptor (version, skills list)
└── skills/
    └── <skill-name>/
        ├── SKILL.md                    # Required: instructions + frontmatter
        └── README.md                   # Optional: user-facing description
```

## Plugin and Skill Conventions

**Read `PATTERNS.md` before making any changes.** It documents why each rule exists, including which bugs it prevents. Below is a summary.

### Naming

| Layer | Format | Example |
|-------|--------|---------|
| Marketplace | kebab-case product family | `skills-for-architects` |
| Plugin | `NN-kebab-name` | `00-due-diligence` |
| Dispatcher skill | matches plugin intent | `studio`, `skills-menu` |
| Sub-skill | `<verb>` (namespaced by plugin folder) | `nyc-landmarks`, `spec-writer` |

### SKILL.md Frontmatter

```yaml
---
name: skill-name               # kebab-case, matches directory
description: Short description  # trigger-phrase-rich; model invocation depends on this
allowed-tools: [Read, WebFetch] # scoped to what THIS skill actually needs
user-invocable: true            # present if slash-invocable
---
```

### Dispatcher (`plugins/08-dispatcher/skills/studio/SKILL.md`)

The single entry point. It:
1. Reads the user's intent (text after `/studio`)
2. Classifies against the routing table
3. Hands off to the right agent or sub-skill
4. Falls back to working mode for ambiguous freeform input

### Rules (Always-on)

Rules in `rules/` apply to every output — they are not invoked, they are loaded automatically. Never suppress or bypass them.

| Rule | Governs |
|------|---------|
| `units-and-measurements` | Imperial/metric, GSF/USF/RSF, dimension formatting |
| `code-citations` | Building code references, edition years, jurisdiction |
| `professional-disclaimer` | What AI outputs can and cannot claim |
| `csi-formatting` | MasterFormat 2018 section numbers, three-part structure |
| `terminology` | AEC standard terms, abbreviations, material names |
| `output-formatting` | Tables, source attribution, file naming, lists |
| `transparency` | Link sources, expose inputs, make outputs verifiable |

### Hooks (Opt-in Automations)

Copy `hooks/settings-snippet.json` content into your Claude Code settings to activate:

| Hook | Trigger | Action |
|------|---------|--------|
| `post-write-disclaimer-check.sh` | After Write | Warns if disclaimer marker is absent from regulatory output |
| `post-output-metadata.sh` | After Write | Adds YAML front matter to markdown reports |
| `pre-commit-spec-lint.sh` | Before git commit | Flags malformed CSI section numbers |

Hooks use **marker-driven enforcement** — skills emit an HTML comment like `<!-- architecture-studio:requires-disclaimer -->` when the rule applies. The hook checks for the marker, not for keywords. This prevents false positives on docs that mention regulated terms in passing.

## Agents

Agents are orchestration personas that chain multiple skills in response to a high-level user request. They live in `agents/*.md` and are referenced by the dispatcher.

| Agent | Domain | Chains |
|-------|--------|--------|
| `site-planner` | Site Planning | Environmental, mobility, demographics, history skills in parallel |
| `nyc-zoning-expert` | Due Diligence + Zoning | All 7 NYC data skills + zoning analysis |
| `workplace-strategist` | Programming | Occupancy calculator + workplace-programmer |
| `product-and-materials-researcher` | Materials | Product-research → spec-bulk-fetch → data-cleanup |
| `ffe-designer` | FF&E | Master-schedule → enrich → match → pair → csv-to-sif |
| `sustainability-specialist` | Sustainability | EPD-research → epd-parser → epd-compare → epd-to-spec |
| `brand-manager` | Presentations | Slide-deck-generator + colour-palette-generator |

## Versioning

Two version fields, both must be bumped and tagged on every shipped change (see PATTERNS.md §6):

| Field | File | Bumps when |
|-------|------|----------|
| `version` | `plugins/<N>-<name>/.claude-plugin/plugin.json` | That plugin's behavior changes |
| `metadata.version` | `.claude-plugin/marketplace.json` | Anything in the repo ships |

**Commit checklist:** bump JSON version → add CHANGELOG entry → commit → `git tag -a vX.Y.Z` → `git push origin vX.Y.Z` → GitHub release.

## CI Lint

`scripts/lint.sh` runs on every push via `.github/workflows/lint.yml`. It fails on:

1. Tracked `.DS_Store` files
2. Invalid JSON (`marketplace.json`, `plugin.json`, `.mcp.json`)
3. `SKILL.md` missing `name` or `description` frontmatter
4. Count drift between README claims and actual `SKILL.md` file count
5. Broken internal markdown links
6. `shellcheck` failures on `hooks/*.sh`

Always run `./scripts/lint.sh` locally before pushing.

## Adding a New Skill

1. Identify the right plugin folder (`plugins/NN-name/`)
2. Create `plugins/NN-name/skills/<skill-name>/SKILL.md` with:
   - Frontmatter: `name`, `description` (trigger-phrase-rich), `allowed-tools` (minimal scope)
   - Body: what the skill does, input it expects, output it produces, and which skill to hand off to next
3. Add a `README.md` in the skill directory (user-facing)
4. Add the skill to `plugins/NN-name/.claude-plugin/plugin.json` skills list
5. Update `README.md` skill count and table
6. Update `CHANGELOG.md`
7. Bump `plugin.json` version AND `marketplace.json` `metadata.version`
8. Run `./scripts/lint.sh` and fix any failures
9. Commit, tag, push, cut GitHub release

## Adding a New Plugin

Follow PATTERNS.md §7 (layout selection) and the Quick Checklist in §Quick checklist. In summary:

1. Create `plugins/NN-name/` with a `skills/` subdirectory
2. Add `.claude-plugin/plugin.json`
3. Write the dispatcher skill first
4. Add the plugin to `.claude-plugin/marketplace.json`
5. Create agents if needed in `agents/`
6. Update top-level README

## Hard Rules from Real Bugs (PATTERNS.md §10)

- **Audit always re-parses** — never report cached sheet values as "verified"; always re-fetch
- **Read before write** — always read target files before writing to map exact headers
- **Update in place** — when a record already exists (by SKU/ID), update it; never append a duplicate
- **No fabricated capabilities** — only claim what the tools actually did; never invent async or background operations

These rules belong in the dispatcher SKILL.md AND in each sub-skill that touches the affected behavior.

## Installation (for users)

**Claude Desktop:**
> Customize → Browse plugins → + → Add marketplace from GitHub → `AlpacaLabsLLC/skills-for-architects`

**Claude Code:**
```bash
claude plugin marketplace add AlpacaLabsLLC/skills-for-architects
claude plugin install 01-site-planning@skills-for-architects
```

Then type `/studio <description of your task>`.
