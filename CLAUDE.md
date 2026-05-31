# CLAUDE.md — skills-for-architects

This file provides guidance to AI assistants working in this repository.

## Project Overview

**Architecture Studio** is a multi-plugin Claude Code marketplace for architects, designers, and AEC (Architecture, Engineering, Construction) professionals. It ships 9 installable plugins, 7 orchestrating agents, 37 single-verb skills, 7 always-on rules, and 3 event-driven hooks.

Built and maintained by [ALPA](https://alpa.llc). Conventions are codified in `PATTERNS.md` — read it before making structural changes.

- **No application code** — all content is Markdown, JSON, and shell scripts
- **License**: MIT

## Repository Layout

```
skills-for-architects/
├── agents/                   # 7 orchestrating agent files
│   ├── site-planner.md
│   ├── nyc-zoning-expert.md
│   ├── workplace-strategist.md
│   ├── product-and-materials-researcher.md
│   ├── ffe-designer.md
│   ├── sustainability-specialist.md
│   └── brand-manager.md
│
├── plugins/                  # 9 installable plugin directories
│   ├── 00-due-diligence/     # 7 skills: NYC property records
│   ├── 01-site-planning/     # 4 skills: environmental, mobility, demographics, history
│   ├── 02-zoning-analysis/   # 2 skills: NYC zoning envelope + 3D viewer
│   ├── 03-programming/       # 2 skills: workplace programmer + occupancy calculator
│   ├── 04-specifications/    # 1 skill: CSI spec writer
│   ├── 05-sustainability/    # 4 skills: EPD parsing, research, compare, to-spec
│   ├── 06-materials-research/# 12 skills: product research, FF&E, SIF/CSV conversion
│   ├── 07-presentations/     # 3 skills: slide decks, color palettes, image resize
│   └── 08-dispatcher/        # 2 skills: /studio router + /skills help menu
│
├── rules/                    # 7 always-on rule files
│   ├── units-and-measurements.md
│   ├── code-citations.md
│   ├── professional-disclaimer.md
│   ├── csi-formatting.md
│   ├── terminology.md
│   ├── output-formatting.md
│   └── transparency.md
│
├── hooks/
│   ├── post-write-disclaimer-check.sh  # Warns if regulatory output lacks disclaimer
│   ├── post-output-metadata.sh         # Stamps reports with YAML front matter
│   ├── pre-commit-spec-lint.sh         # Flags malformed CSI section numbers
│   ├── settings-snippet.json           # Example Claude Code hook registration
│   └── README.md
│
├── scripts/
│   └── lint.sh               # CI validation: JSON validity, frontmatter, count checks
│
├── .github/workflows/
│   └── lint.yml              # Runs scripts/lint.sh on push to main and PRs
│
├── PATTERNS.md               # CANONICAL conventions reference — read before editing
├── CHANGELOG.md
└── README.md
```

## Conventions (from PATTERNS.md)

### Plugin Layout

Each `plugins/<N>-<name>/` directory follows this structure:

```
plugins/00-due-diligence/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest (name, version, description, author)
├── skills/
│   └── <skill-name>/
│       └── SKILL.md         # One skill per directory
├── README.md
└── .gitignore
```

### Skill Naming

In a multi-plugin marketplace, skill names are **not** prefixed with the plugin name (that's the single-plugin pattern). Names are verb-scoped and descriptive:
- `nyc-landmarks` not `due-diligence-nyc-landmarks`
- `spec-writer` not `specifications-spec-writer`
- `epd-parser` not `sustainability-epd-parser`

### Dispatcher Pattern

`plugins/08-dispatcher` contains two special skills:
- `/studio` — reads the user's intent and routes to the correct agent or skill
- `/skills` — shows all available skills and agents

The dispatcher's `SKILL.md` includes the full routing table and must be updated whenever a skill is added or removed.

### Agent Files

Agents are the orchestration layer — they invoke multiple skills in sequence, apply judgment, and hand off results. Each `agents/<name>.md` documents:
- Domain and responsibilities
- Which skills it can invoke and in what order
- Handoff logic

### Rules

Rules in `rules/` are always-on (loaded automatically, not slash-invoked). Each rule covers exactly one concern. The `professional-disclaimer.md` rule requires regulatory outputs to end with the canonical disclaimer block followed by `<!-- architecture-studio:requires-disclaimer -->` — the hook checks for this marker.

### Hooks

Hooks are opt-in via Claude Code settings. The `post-write-disclaimer-check.sh` hook is **marker-driven** (checks for `<!-- architecture-studio:requires-disclaimer -->`), not keyword-sniffed, to avoid false positives on non-regulatory documents.

## CI and Validation

`scripts/lint.sh` enforces six structural checks:
1. No tracked `.DS_Store` files
2. JSON validity for all `.json` files
3. SKILL.md frontmatter has required `name` + `description` fields
4. Count consistency: plugin list, per-plugin skill counts, and marketplace.json must all match the actual file count
5. Internal Markdown link resolution
6. `shellcheck` on all `hooks/*.sh` scripts

Run locally before opening a PR:

```bash
./scripts/lint.sh
```

CI runs on push to `main` and on every PR via `.github/workflows/lint.yml`.

## Development Workflow

### Adding a Skill

1. Choose the target plugin directory
2. Create `plugins/<N>-<name>/skills/<skill-name>/SKILL.md`
3. Write frontmatter (`name`, `description`, `allowed-tools`, optionally `user-invocable: true`)
4. Write the skill body — focused, single-verb, ≤500 lines
5. Update `plugins/08-dispatcher/skills/studio/SKILL.md` routing table
6. Update `plugins/08-dispatcher/skills/skills-menu/SKILL.md` menu
7. Bump the plugin's `plugin.json` version
8. Run `./scripts/lint.sh` — it fails if counts are out of sync
9. Update `README.md` skill count (headline, table, and details summary)

### Adding a Plugin

1. Create `plugins/<N>-<name>/` with the standard layout
2. Write `.claude-plugin/plugin.json`
3. Add skills following the skill conventions above
4. Update `README.md` plugin table
5. Add to the marketplace manifest if one exists

### Versioning

Three artifacts must move together on every shipped change:
1. `plugin.json` version field
2. `CHANGELOG.md` entry
3. Git tag: `git tag -a vX.Y.Z` + GitHub release

### Editing Hooks

- Hooks are marker-driven, not keyword-sniffed — modify the marker, not the keyword list
- Run `shellcheck` on any modified hook script before committing
- Test the hook fires correctly with `settings-snippet.json` as the Claude Code hook config

## Skill Authoring Quick Reference

```yaml
---
name: skill-name             # kebab-case, matches directory name exactly
description: >               # trigger-phrase-rich; drives model invocation
  When to use this skill. Include exact symptoms, user phrases, domain terms.
  Specific > generic.
allowed-tools:               # scope narrowly
  - WebSearch
  - Read
user-invocable: true         # include if the skill has a slash command
---
```

Body: one verb, one domain, ≤500 lines. Include explicit handoff references (`"After this, run /epd-compare"`). Document expected input and output format.
