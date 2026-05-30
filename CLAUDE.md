# skills-for-architects

A Claude Code plugin marketplace for architecture, design, and AEC (Architecture, Engineering, Construction) professionals. Provides 7 orchestration agents, 37 skills across 9 plugins, 7 always-on rules, and 3 opt-in hooks.

Built by [ALPA](https://alpa.llc) — MIT licensed.

## Repository Structure

```
skills-for-architects/
├── README.md                    # Full catalog: agents, plugins, skills, rules, hooks
├── PATTERNS.md                  # Canonical ALPA plugin conventions (10 principles)
├── CHANGELOG.md                 # Semantic versioning log
├── LICENSE                      # MIT, Alpaca Design Lab LLC, 2026
├── .gitignore
├── .claude-plugin/
│   └── marketplace.json         # Marketplace metadata + plugin list
├── agents/                      # 7 orchestration agents (reference docs)
├── plugins/                     # 9 installable skill bundles (37 total skills)
│   ├── 00-due-diligence/        # NYC property research (7 skills)
│   ├── 01-site-planning/        # Environmental, mobility, demographics (4 skills)
│   ├── 02-zoning-analysis/      # NYC zoning + envelope analysis (2 skills)
│   ├── 03-programming/          # Space programming + occupancy (2 skills)
│   ├── 04-specifications/       # Spec writing (1 skill)
│   ├── 05-sustainability/       # EPD parsing + comparison (4 skills)
│   ├── 06-materials-research/   # Product research + schedule (12 skills)
│   ├── 07-presentations/        # Slides, palettes, images (3 skills)
│   └── 08-dispatcher/           # Router + help menu (2 skills)
├── rules/                       # 7 always-on conventions (never invoked)
├── hooks/                       # 3 opt-in event-driven automations
└── scripts/
    └── lint.sh                  # Structural validation (run locally + CI)
```

## Plugin Architecture

### Dispatcher Pattern

Each plugin has a single entry-point skill that routes user intent to specialized sub-skills. The `08-dispatcher` plugin provides:
- `/studio` — top-level router across all installed plugins
- `/skills-menu` — help menu listing available skills

Sub-skills are single-purpose (one verb each) and document their expected inputs, outputs, and downstream handoffs explicitly.

### Typical SKILL.md Structure

```yaml
---
name: skill-name
description: One-sentence description of what this skill does.
allowed-tools:
  - WebFetch
  - Write
  - Read
  - Bash
user-invocable: true
---
# /skill-name — Short Title

[Domain knowledge, step-by-step instructions, data sources, error handling, handoff notes]
```

`allowed-tools` follows the principle of least privilege — only list tools the skill actually needs.

### Plugin Metadata

Each plugin has `.claude-plugin/plugin.json`:
```json
{
  "name": "00-due-diligence",
  "version": "1.0.0",
  "description": "...",
  "author": { "name": "ALPA", "url": "https://alpa.llc" },
  "keywords": [...],
  "license": "MIT"
}
```

The marketplace root has `.claude-plugin/marketplace.json` with the full plugin list and marketplace-level metadata.

## Agents

Agents live in `/agents/` as reference markdown files. They define orchestration behavior — how Claude should combine multiple skills, in what order, and with what context. Agents are **not invokable** directly; they are loaded as context when users describe multi-step tasks.

| Agent | Purpose |
|-------|---------|
| `site-planner.md` | Environmental, mobility, demographics, history analyses |
| `nyc-zoning-expert.md` | Full NYC property + zoning analysis (9 skills) |
| `workplace-strategist.md` | Space programming from headcount |
| `product-and-materials-researcher.md` | Find, extract, tag products |
| `ffe-designer.md` | FF&E schedules, QA, export |
| `sustainability-specialist.md` | EPD parsing, GWP comparison, LEED |
| `brand-manager.md` | Decks, palettes, QA |

## Rules (Always-On)

Rules in `/rules/` are loaded for every session — they are **never explicitly invoked**. They shape all skill output automatically.

| Rule | Governs |
|------|---------|
| `units-and-measurements.md` | Imperial/metric defaults, area types (GSF/USF/RSF) |
| `code-citations.md` | Building code edition years, section symbols |
| `professional-disclaimer.md` | Regulatory output disclaimer (marker-driven) |
| `csi-formatting.md` | MasterFormat 2018 sections (e.g., "09 29 00 — Gypsum Board") |
| `terminology.md` | AEC standard terms and abbreviations |
| `output-formatting.md` | Tables, headings, attribution, file naming |
| `transparency.md` | Show your work, link sources, expose inputs |

## Hooks (Opt-In)

Hooks in `/hooks/` are event-driven scripts configured via Claude Code's `settings.json`. Copy `hooks/settings-snippet.json` into your settings to enable.

| Hook | Trigger | Behavior |
|------|---------|----------|
| `post-write-disclaimer-check.sh` | After file write | Scans `.md` for regulatory keywords + `<!-- architecture-studio:requires-disclaimer -->` marker |
| `post-output-metadata.sh` | After output | Stamps markdown with YAML front matter |
| `pre-commit-spec-lint.sh` | Pre-commit | Validates CSI section number format |

All hooks warn but do not block (exit 0). Change to `exit 2` to make them enforcement hooks.

**Marker-driven design:** The disclaimer hook checks for a marker string, not keywords — this eliminates false positives on words like "code" or "fire rating" that appear in non-regulatory contexts.

## Versioning Discipline

All three versioning artifacts must move together on every shipped change:

1. `plugin.json` `version` — bump when skills or behavior change within a plugin
2. `marketplace.json` `metadata.version` — bump on any shipped change to the marketplace
3. Git tag: `git tag -a vX.Y.Z <sha> -m "vX.Y.Z — description"`

Use semantic versioning: MAJOR for breaking changes, MINOR for new skills/plugins, PATCH for fixes.

## Naming Conventions

- **Marketplace**: `skills-for-architects` (kebab-case)
- **Plugins**: `00-due-diligence`, `01-site-planning` (kebab-case, numbered for ordering)
- **Dispatcher**: matches plugin name (e.g., `studio` for `08-dispatcher`)
- **Sub-skills**: kebab-case, namespaced by plugin folder
- **User invocation**: `/studio`, `/nyc-landmarks`, `/spec-writer`

## Linting & CI

Run `./scripts/lint.sh` locally before pushing. CI runs it on every push/PR to `main`.

The lint script checks:
1. No `.DS_Store` files tracked in git
2. All `.json` files are valid JSON
3. All `SKILL.md` files have YAML frontmatter with `name` and `description`
4. Skill/plugin counts in `README.md` match actual file counts
5. `marketplace.json` plugin list matches actual plugin directories
6. Internal markdown links resolve (no broken links)
7. Hook scripts pass `shellcheck` (if installed)

**Prerequisites:** `jq`, `python3` with `pyyaml` (`pip install pyyaml`), optionally `shellcheck`

## Hard Rules from Production (PATTERNS.md)

These rules exist because real bugs occurred:

- **Audit always re-parses** — never trust cached values; re-read source data on every audit run
- **Read before write** — map actual column headers before generating output; never invent column names
- **Update in place** — use match-and-patch when modifying schedules, not append; prevents duplicates
- **No fabricated capabilities** — only claim tool actions that actually happened; never hallucinate API results

## Installation

```bash
claude plugin marketplace add mdelmaschio-cpu/skills-for-architects
```

Or via Claude Desktop settings. See `README.md` for platform-specific paths.

## Contributing

1. Follow the dispatcher pattern — new domain-specific features go in sub-skills, not the dispatcher.
2. Run `./scripts/lint.sh` before opening a PR.
3. Update `README.md` skill/plugin counts if adding skills.
4. Bump `plugin.json` and `marketplace.json` versions.
5. Create a git tag and GitHub release when shipping.
6. Add a CHANGELOG entry.

See `PATTERNS.md` for the complete checklist when starting a new plugin.
