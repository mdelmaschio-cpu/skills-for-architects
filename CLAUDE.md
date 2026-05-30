# CLAUDE.md — skills-for-architects

This file provides guidance to Claude Code when working in this repository.

## Project Purpose

**Architecture Studio** is a multi-plugin Claude marketplace built by [ALPA](https://alpa.llc) that brings architecture-specific AI workflows to Claude Desktop and Claude Code. It teaches Claude the vocabulary, standards, and workflows of architects, designers, and AEC (Architecture, Engineering, Construction) professionals — covering site analysis, NYC zoning, space programming, materials research, FF&E schedules, sustainability (EPD/LEED), and presentation deliverables.

The repository contains 7 agents, 37 skills, 7 rules, 3 hooks, and 9 installable plugins. It follows the ALPA plugin authoring patterns documented in `PATTERNS.md` — a canonical reference that should be consulted before making any structural changes to skills, plugins, or version metadata.

## Repository Structure

```
skills-for-architects/
├── .claude-plugin/            # Marketplace manifest (marketplace.json)
├── .github/                   # CI lint workflow (.github/workflows/lint.yml)
├── PATTERNS.md                # Canonical build patterns — READ BEFORE EDITING
├── README.md                  # Full skill table, quick start, install instructions
├── CHANGELOG.md               # Version history (semver, bump on every change)
│
├── agents/                    # Orchestration-layer agent personas (7 agents)
│   ├── site-planner.md
│   ├── nyc-zoning-expert.md
│   ├── workplace-strategist.md
│   ├── product-and-materials-researcher.md
│   ├── ffe-designer.md
│   ├── sustainability-specialist.md
│   └── brand-manager.md
│
├── plugins/                   # Installable plugin bundles (9 plugins)
│   ├── 00-due-diligence/      # NYC property data (7 skills)
│   ├── 01-site-planning/      # Environmental, mobility, demographics (4 skills)
│   ├── 02-zoning-analysis/    # NYC FAR/zoning envelope (2 skills)
│   ├── 03-programming/        # Space programs, occupancy loads (2 skills)
│   ├── 04-specifications/     # CSI outline specifications (1 skill)
│   ├── 05-sustainability/     # EPD parsing, comparison, LEED (4 skills)
│   ├── 06-materials-research/ # FF&E research, spec extraction, schedules (12 skills)
│   ├── 07-presentations/      # Slide decks, color palettes, image resize (3 skills)
│   └── 08-dispatcher/         # /studio router + /skills menu (2 skills)
│
├── rules/                     # Always-on cross-cutting conventions (7 rules)
│   ├── units-and-measurements.md
│   ├── code-citations.md
│   ├── professional-disclaimer.md
│   ├── csi-formatting.md
│   ├── terminology.md
│   ├── output-formatting.md
│   └── transparency.md
│
├── hooks/                     # Event-driven automations (3 hooks)
│   ├── post-write-disclaimer-check.sh
│   ├── post-output-metadata.sh
│   └── pre-commit-spec-lint.sh
│
└── scripts/
    └── lint.sh                # Structural lint — runs in CI
```

Each plugin directory contains:
```
plugins/<name>/
├── .claude-plugin/
│   └── plugin.json            # Plugin manifest (name, version, skills path)
├── skills/
│   └── <skill-name>/
│       ├── SKILL.md           # Required — skill instructions + frontmatter
│       └── README.md          # Optional — human-readable description
└── README.md                  # Plugin overview
```

## Skill File Format

Every `SKILL.md` requires YAML frontmatter with at least `name` and `description`:

```yaml
---
name: skill-name-in-kebab-case
description: Trigger-phrase-rich description. Include the exact situations,
  symptoms, and phrases that should invoke this skill.
allowed-tools: [WebSearch, WebFetch]   # scoped to what THIS skill needs
user-invocable: true                   # present when slash-invocable
---

# Skill Title

[Skill body with instructions, routing tables, examples]
```

Key frontmatter rules:
- `name` must match the directory name exactly (kebab-case)
- `description` is the primary mechanism for model skill-selection — write it with rich trigger phrases
- `allowed-tools` must be scoped to only the tools this specific skill actually uses
- `user-invocable: true` when the skill should respond to a slash command
- `disable-model-invocation: true` when a skill is slash-only and should not be auto-invoked

## Architecture Conventions

### The Three-Layer Pattern

| Layer | Files | Role |
|-------|-------|------|
| **Agents** | `agents/*.md` | Orchestrate — assess intent, choose skills, exercise judgment, run parallel research |
| **Skills** | `plugins/*/skills/*/SKILL.md` | Execute — single-purpose tools invoked by slash command or agent |
| **Rules** | `rules/*.md` | Govern — always-on cross-cutting conventions applied to every output |

### Dispatcher Pattern

Every plugin has exactly one entry-point dispatcher skill. For this repo the top-level dispatcher is `/studio` in `plugins/08-dispatcher/`. The dispatcher:
1. Reads the user's intent (everything after `/studio`)
2. Routes to the right agent or skill using a routing table
3. Falls back to working mode for ambiguous freeform messages

Dispatcher skill name must match the plugin name.

### Rules Are Cross-Cutting

Rules in `rules/` apply to all plugins and all output. They are not invoked — they govern passively. The `professional-disclaimer` rule is enforced by the `post-write-disclaimer-check.sh` hook, which looks for a marker comment (`<!-- architecture-studio:requires-disclaimer -->`) that skills emit when they produce regulatory or code-related output. Do not use keyword-sniffing — only marker-based enforcement.

### Versioning (CRITICAL)

Three artifacts must move together on every shipped change:

1. **`plugin.json` version field** — bumped when plugin behavior changes
2. **`marketplace.json` metadata.version** — bumped for any repo-level change
3. **Git tag + GitHub release** — tag every version with `git tag -a vX.Y.Z` and create a matching GitHub release

Bump rules: patch for fixes/docs, minor for new skills or non-breaking enhancements, major for breaking layout changes. Without a version bump, `claude plugin marketplace update` reports "already up to date" even when new commits exist.

### Naming Conventions

| Layer | Format | Example |
|-------|--------|---------|
| Plugin (multi-plugin) | kebab-case with sequence prefix | `00-due-diligence` |
| Skill (multi-plugin) | `<verb>` (namespaced by plugin) | `nyc-landmarks`, `spec-writer` |
| Dispatcher | matches plugin family | `studio` |
| Slash invocation | `/<skill-name>` | `/nyc-landmarks`, `/spec-writer` |

## Domain Knowledge Conventions

When writing or editing skills in this repo, apply these AEC-specific conventions:

- **Units**: Imperial primary (feet, inches, square feet), metric in parentheses where relevant. Area types: GSF (gross square feet), USF (usable SF), RSF (rentable SF) — never conflate them.
- **Code citations**: Include edition year and jurisdiction. "IBC 2021 Section 1004.1" not "IBC Section 1004.1". Flag when local amendments may differ.
- **CSI formatting**: Use MasterFormat 2018 section numbers (e.g., `09 30 00 — Tiling`). Three-part section structure: Part 1 General / Part 2 Products / Part 3 Execution.
- **Professional disclaimer**: Any output that provides regulatory analysis, code interpretations, or structural recommendations must include: "This output is generated by an AI assistant and does not constitute professional architectural or engineering advice. Verify all code interpretations with a licensed professional in the relevant jurisdiction."
- **Transparency**: Always link sources, expose inputs, and make outputs verifiable. Never cite data without attribution.

## Development Workflow

### Adding a New Skill

1. Identify which plugin it belongs to (or propose a new plugin)
2. Create `plugins/<plugin>/skills/<skill-name>/SKILL.md` with frontmatter
3. Set `allowed-tools` to only what the skill actually needs
4. Update the dispatcher's routing table in `plugins/08-dispatcher/skills/studio/SKILL.md`
5. Update README.md skill count and table
6. Bump `plugin.json` version (patch or minor), bump `marketplace.json` version
7. Add CHANGELOG.md entry
8. Run `scripts/lint.sh` — CI will also run it on push

### Editing an Existing Skill

- Keep the `name` frontmatter unchanged unless intentionally renaming
- If you change behavior (not just wording), bump `plugin.json` version
- Hard rules go in the skill body with WHY they exist — not just what
- Cross-references use the actual slash invocation: "After this completes, run `/epd-compare`"

### Running Lint

```bash
bash scripts/lint.sh
```

Lint checks: tracked `.DS_Store` files, invalid JSON, SKILL.md missing frontmatter, count drift between README and actual files, broken internal links, shellcheck on hooks.

### Installing Locally (Claude Code)

```bash
claude plugin marketplace add mdelmaschio-cpu/skills-for-architects
claude plugin install 01-site-planning@skills-for-architects
```

## Important Files

| File | Role |
|------|------|
| `PATTERNS.md` | Canonical authoring patterns — consult before structural changes |
| `README.md` | Full skill/agent table, install instructions, contributing guide |
| `CHANGELOG.md` | Version history — update on every shipped change |
| `.claude-plugin/marketplace.json` | Marketplace manifest — controls what plugins are published |
| `plugins/08-dispatcher/skills/studio/SKILL.md` | Top-level router — update routing table when adding skills |
| `rules/professional-disclaimer.md` | Disclaimer language — apply to all regulatory/code output |
| `hooks/post-write-disclaimer-check.sh` | Enforces disclaimer presence via marker, not keywords |

## How AI Assistants Should Behave Here

- **Read `PATTERNS.md` before any structural edit** — it documents hard-earned lessons from real bugs
- **Never break the three-artifact version contract** (plugin.json + marketplace.json + git tag)
- **Scope `allowed-tools` tightly** — each skill should only have access to tools it actually uses
- **Write dispatch tables in plain English** — routing logic lives in SKILL.md, not in code
- **Apply all 7 rules to generated output** — especially units, code citations, and professional disclaimer
- **Do not fabricate capabilities** — only describe what the tools in `allowed-tools` actually do
- **Hard rules belong in skill bodies** — if a real bug produced a rule, document WHY it exists
- **One skill, one verb** — if a skill handles multiple distinct scenarios, decompose it
- **Test with realistic pressure scenarios** — a skill that fails under time pressure or sunk cost bias is not ready
