# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

**skills-for-architects** (Architecture Studio) is a Claude plugin marketplace for architects, designers, and AEC professionals. It ships 7 agents, 37 skills, 7 rules, and 3 hooks across 9 installable plugins, covering site analysis, zoning, space programming, specifications, materials research, sustainability, and presentations.

Built by [ALPA](https://alpa.llc). Entry point: `/studio`.

## Repository Layout

```
skills-for-architects/
├── .claude-plugin/marketplace.json  # Marketplace manifest
├── agents/
│   ├── site-planner.md              # Site research + synthesis (4 skills)
│   ├── nyc-zoning-expert.md         # Property records + zoning + 3D (9 skills)
│   ├── workplace-strategist.md      # Occupancy + programming (2 skills)
│   ├── product-and-materials-researcher.md  # Find, extract, tag (5 skills)
│   ├── ffe-designer.md              # FF&E schedule, QA, export (7 skills)
│   ├── sustainability-specialist.md # EPDs, GWP, LEED (4 skills)
│   └── brand-manager.md             # Decks + palettes (2 skills)
├── plugins/
│   ├── 00-due-diligence/            # 7 skills: site investigation
│   ├── 01-site-planning/            # 4 skills: site synthesis
│   ├── 02-zoning-analysis/          # 2 skills: zoning + 3D envelope
│   ├── 03-programming/              # 2 skills: space programming
│   ├── 04-specifications/           # 1 skill: CSI spec writing
│   ├── 05-sustainability/           # 4 skills: EPDs, GWP, LEED
│   ├── 06-materials-research/       # 12 skills: product research
│   ├── 07-presentations/            # 3 skills: decks + palettes
│   └── 08-dispatcher/              # 2 skills: /studio routing
├── rules/
│   ├── units-and-measurements.md    # Imperial/metric conventions
│   ├── code-citations.md            # IBC, NFPA, ADA citation format
│   ├── professional-disclaimer.md   # Required on every output
│   ├── csi-formatting.md            # CSI MasterFormat conventions
│   ├── terminology.md               # AEC industry vocabulary
│   ├── output-formatting.md         # Structured output format
│   └── transparency.md              # Source attribution
├── hooks/
│   ├── post-write-disclaimer-check.sh  # Enforces disclaimer presence
│   ├── post-output-metadata.sh         # Appends output metadata
│   ├── pre-commit-spec-lint.sh         # Lints CSI spec format before write
│   └── settings-snippet.json           # Hook configuration
├── scripts/                         # Utility scripts
├── PATTERNS.md                      # Canonical plugin-building patterns
├── CHANGELOG.md
└── README.md
```

## Installation

**Claude Code:**
```bash
claude plugin marketplace add AlpacaLabsLLC/skills-for-architects
claude plugin install 01-site-planning@skills-for-architects
```

**Claude Desktop:** Customize → Browse plugins → + → Add marketplace from GitHub → `AlpacaLabsLLC/skills-for-architects`

## Development Patterns (from PATTERNS.md)

### Small Skills, One Verb Each

Each skill does exactly one thing. If `SKILL.md` exceeds ~500 lines or starts handling multiple verbs in sequence, decompose it.

- `name` matches the directory, kebab-case
- `description` is trigger-phrase-rich (model invocation depends on it)
- `allowed-tools` is scoped to what THIS skill needs only
- `user-invocable: true` when slash-invokable; `disable-model-invocation: true` when slash-only

### Clear Contracts Between Skills

Skills hand off via explicit cross-references in their bodies — never via implicit shared state:
- Document what input a skill expects and what it produces
- Cross-references use the actual slash invocation: "After this completes, run `/studio:next-skill`"
- Hard rules that span multiple skills are repeated in every skill that touches them

### Naming Conventions

| Layer | Format | Example |
|-------|--------|---------|
| Marketplace name | kebab-case | `skills-for-architects` |
| Plugin name | `NN-kebab-case` | `01-site-planning` |
| Dispatcher skill | matches plugin | `studio` |
| Sub-skills | verb-noun kebab-case | `extract-product-data` |

### Rules Apply to Every Output

All 7 rules in `rules/` govern every skill output. The `post-write-disclaimer-check` hook enforces that the professional disclaimer appears in every response. Do not disable this hook.

### CI / Linting

```bash
# GitHub Actions runs on push
# Manually lint:
bash hooks/pre-commit-spec-lint.sh <file>
```

## Key Conventions

- **AEC terminology**: use industry vocabulary from `rules/terminology.md`; never use generic "building" when "structure", "envelope", or "assembly" is correct
- **CSI formatting**: specifications use MasterFormat section numbering per `rules/csi-formatting.md`
- **Units**: always state units explicitly; use imperial unless project context specifies metric (`rules/units-and-measurements.md`)
- **Code citations**: cite IBC section, NFPA code, and ADA paragraph when referencing regulatory requirements
- **Professional disclaimer is mandatory**: every output that touches design decisions, code compliance, or structural adequacy must include the disclaimer from `rules/professional-disclaimer.md`
- **Hooks are Bash scripts**: keep them fast and exit-code-safe; a failing hook should not block the user's session
- **Marketplace manifest stays in sync**: when adding a plugin, update `.claude-plugin/marketplace.json`
