# CLAUDE.md — skills-for-architects (Architecture Studio)

This file provides guidance to AI assistants working in this repository.

## Repository Purpose

**Architecture Studio** is a Claude plugin system with **37 skills, 7 agents, 7 rules, and 3 hooks** organized into **9 plugins** for architects, designers, and AEC (Architecture, Engineering, Construction) professionals. Built by [ALPA](https://alpa.llc). Works with Claude Desktop and Claude Code.

## Repository Structure

```
skills-for-architects/
├── README.md                   # Full project overview and skill catalog
├── PATTERNS.md                 # AUTHORITATIVE conventions — read before modifying
├── CHANGELOG.md                # Version history
├── LICENSE                     # MIT
├── .gitignore
├── .github/                    # GitHub Actions
├── .claude-plugin/             # Claude Code marketplace metadata
│
├── agents/                     # 7 orchestration agents
│   ├── site-planner.md
│   ├── nyc-zoning-expert.md
│   ├── workplace-strategist.md
│   ├── product-and-materials-researcher.md
│   ├── ffe-designer.md
│   ├── sustainability-specialist.md
│   └── brand-manager.md
│
├── plugins/                    # 9 installable plugin bundles
│   ├── 00-due-diligence/       # 7 skills — NYC property data
│   ├── 01-site-planning/       # 4 skills — environmental, mobility, demographics, history
│   ├── 02-zoning-analysis/     # 2 skills — buildable envelope + 3D viewer
│   ├── 03-programming/         # 2 skills — space programs, IBC occupancy loads
│   ├── 04-specifications/      # 1 skill  — CSI MasterFormat outline specs
│   ├── 05-sustainability/      # 4 skills — EPD parsing, research, comparison, spec
│   ├── 06-materials-research/  # 12 skills — FF&E product research + schedules
│   ├── 07-presentations/       # 3 skills — slide decks, color palettes, image resize
│   └── 08-dispatcher/          # 2 skills — /studio router + /skills help menu
│
├── rules/                      # 7 always-on output conventions (loaded automatically)
│   ├── units-and-measurements.md
│   ├── code-citations.md
│   ├── professional-disclaimer.md
│   ├── csi-formatting.md
│   ├── terminology.md
│   ├── output-formatting.md
│   └── transparency.md
│
├── hooks/                      # 3 event-driven automations (opt-in)
│   ├── post-write-disclaimer-check.sh  # warns if disclaimer missing
│   ├── post-output-metadata.sh         # stamps YAML front matter
│   └── pre-commit-spec-lint.sh         # validates CSI section numbers
│
└── scripts/                    # Maintenance utilities
```

## Core Architecture Concepts

### Agents vs Skills vs Rules vs Hooks

| Type | How it works | Invocation |
|------|-------------|------------|
| **Agents** | Orchestrate — assess input, choose skills, apply judgment | `/studio <task>` |
| **Skills** | Single-purpose tools with domain knowledge | `/skill-name [args]` |
| **Rules** | Always-on output conventions | Loaded automatically |
| **Hooks** | Event-driven automations | Via Claude Code settings |

### The Dispatcher Pattern (`/studio`)

`/studio` is the main entry point — it reads your request and routes to the right agent or skill. Example:
```
/studio task chair, mesh back, under $800
/studio 123 Main St, Brooklyn NY
/studio I need a space program for 200 people
/studio parse this EPD
```
Type `/skills` for the full menu.

### Plugin Structure

Each plugin follows:
```
plugins/NN-name/
├── README.md           # Plugin overview
└── skills/
    └── skill-name/
        ├── SKILL.md    # Required: skill instructions
        └── README.md   # Required: skill documentation
```

## Development Conventions

### Read PATTERNS.md First

[PATTERNS.md](./PATTERNS.md) is the authoritative reference for:
- Skill naming conventions and file layout
- The dispatcher routing pattern
- Versioning rules
- Hard-won rules from real bugs (never skip these)

### Skill File Rules

- Slash command name = directory name exactly (e.g., `/epd-parser` → `plugins/05-sustainability/skills/epd-parser/`)
- Rules are loaded automatically — do NOT duplicate rule content inside skill files
- Each skill must have both `SKILL.md` (instructions) and `README.md` (user-facing docs)
- No hardcoded paths — use relative paths only

### CSI Formatting

Any specification output must use MasterFormat 2018 section numbers. The `pre-commit-spec-lint.sh` hook validates this before commits. Malformed section numbers will block the commit.

### Professional Disclaimer

The `professional-disclaimer` rule is mandatory for all regulatory, code-compliance, and engineering output. The `post-write-disclaimer-check.sh` hook warns if it's missing. Never remove or bypass this.

### NYC-Specific Skills

Skills in `00-due-diligence/` and `02-zoning-analysis/` query live NYC public APIs (DOB, ACRIS, PLUTO, etc.). Do not cache results — they must be fresh per request.

## Installation

**Claude Desktop (marketplace):**
Open Customize → Browse plugins → + → Add marketplace from GitHub → `AlpacaLabsLLC/skills-for-architects`

**Claude Code (CLI):**
```bash
claude plugin marketplace add AlpacaLabsLLC/skills-for-architects
claude plugin install 01-site-planning@skills-for-architects
```

## All 37 Skills Quick Reference

| Plugin | Slash Commands |
|--------|---------------|
| Due Diligence | `/nyc-landmarks`, `/nyc-dob-permits`, `/nyc-dob-violations`, `/nyc-acris`, `/nyc-hpd`, `/nyc-bsa`, `/nyc-property-report` |
| Site Planning | `/environmental-analysis`, `/mobility-analysis`, `/demographics-analysis`, `/history` |
| Zoning | `/zoning-analysis-nyc`, `/zoning-envelope` |
| Programming | `/workplace-programmer`, `/occupancy-calculator` |
| Specifications | `/spec-writer` |
| Sustainability | `/epd-parser`, `/epd-research`, `/epd-compare`, `/epd-to-spec` |
| Materials | `/product-research`, `/product-spec-bulk-fetch`, `/product-data-cleanup`, `/product-spec-pdf-parser`, `/product-image-processor`, `/product-data-import`, `/master-schedule`, `/product-enrich`, `/product-match`, `/product-pair`, `/csv-to-sif`, `/sif-to-csv` |
| Presentations | `/slide-deck-generator`, `/color-palette-generator`, `/resize-images` |
| Dispatcher | `/studio`, `/skills` |

## Important Notes for AI Assistants

- **PATTERNS.md is law** — read it before adding or restructuring any plugin
- All 37 skills are production-ready — do not add experimental or incomplete skills
- Agents are composable — individual skills must remain independently invocable
- Rules are global — any AEC output must respect units, terminology, CSI formatting, and disclaimer conventions
- NYC data skills query live government APIs — results must not be cached or fabricated
- The dispatcher (`/studio`) uses routing logic in `plugins/08-dispatcher/skills/studio/` — preserve that routing logic when adding new plugins
- When adding a new plugin, follow the `NN-name` numbering convention and update `plugins/08-dispatcher/` to include it
