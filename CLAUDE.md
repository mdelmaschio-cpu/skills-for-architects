# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

Architecture Studio is a multi-plugin Claude Code marketplace for AEC (Architecture, Engineering, Construction) professionals. It ships 7 agents, 37 skills, 7 rules, and 3 hooks across 9 plugins organized by project lifecycle — from due diligence through delivery. Built by [ALPA](https://alpa.llc).

## Installation & Usage

```bash
# Install the full marketplace
claude plugin marketplace add AlpacaLabsLLC/skills-for-architects

# Install a single plugin
claude plugin install 01-site-planning@skills-for-architects
```

Entry point: `/studio <task>` — the dispatcher reads your request and routes to the right agent or skill.
Full menu: `/skills`

## CI Lint

```bash
./scripts/lint.sh
```

Fails on: tracked `.DS_Store`, invalid JSON manifests, SKILL.md missing `name`/`description` frontmatter, skill-count drift between README/plugin tables/actual files, broken internal Markdown links, shellcheck errors in hooks.

## Repository Architecture

This is a **multi-plugin nested** layout (see `PATTERNS.md` §7 for layout selection rationale):

```
skills-for-architects/
├── agents/              # 7 orchestration agents (site-planner, nyc-zoning-expert, etc.)
├── plugins/             # 9 installable plugin bundles
│   ├── 00-due-diligence/       # NYC property data: 7 skills
│   ├── 01-site-planning/       # Site research: 4 skills
│   ├── 02-zoning-analysis/     # Zoning envelope: 2 skills
│   ├── 03-programming/         # Workplace strategy: 2 skills
│   ├── 04-specifications/      # CSI specs: 1 skill
│   ├── 05-sustainability/      # EPDs & GWP: 4 skills
│   ├── 06-materials-research/  # FF&E research: 12 skills
│   ├── 07-presentations/       # Decks & palettes: 3 skills
│   └── 08-dispatcher/          # Router: /studio, /skills — 2 skills
├── rules/               # 7 always-on rules (units, codes, disclaimers, CSI formatting, etc.)
├── hooks/               # 3 event-driven automations
│   ├── post-write-disclaimer-check.sh
│   ├── post-output-metadata.sh
│   └── pre-commit-spec-lint.sh
├── .claude-plugin/
│   └── marketplace.json        # Marketplace registry
└── PATTERNS.md          # Authoritative conventions doc — read before contributing
```

Each plugin contains `<plugin>/.claude-plugin/plugin.json` (semver-pinned) and `<plugin>/skills/<verb-name>/SKILL.md`.

## Conventions (from PATTERNS.md)

These rules emerged from real production bugs — rationale is documented inline in `PATTERNS.md`:

- **One verb per skill** — split any SKILL.md that exceeds ~500 lines or handles multiple sequential actions
- **Dispatcher first** — `plugins/08-dispatcher/skills/studio/SKILL.md` contains the routing table; add routing entries there before creating sub-skills
- **Allowed-tools scoped** — set `allowed-tools` in each skill's frontmatter to only what that skill actually calls; never inherit the union of the whole plugin
- **Explicit cross-references** — skills hand off via named slash commands in the body ("After this, run `/spec-writer`"), never via implicit shared state
- **Marker-driven hooks** — skills emit HTML comment markers (e.g., `<!-- architecture-studio:requires-disclaimer -->`); hooks match on markers, not keywords
- **Hard rules from bugs** — encode production bug fixes as explicit rules in the affected skill body, with WHY documented (see `PATTERNS.md` §10 for examples from canoa V1)

## Versioning (required on every change)

Three artifacts must move together on every shipped change (`PATTERNS.md` §6):

1. Bump `version` in the affected plugin's `plugin.json`
2. Bump `metadata.version` in `.claude-plugin/marketplace.json`
3. Add entry under `## [X.Y.Z] - YYYY-MM-DD` in `CHANGELOG.md`
4. Commit all three, tag: `git tag -a vX.Y.Z <sha> -m "vX.Y.Z — description"` and push
5. Cut GitHub release with the CHANGELOG section as release notes

Omitting the `plugin.json` version bump causes `plugin marketplace update` to report "already up to date" even after new commits are pushed.

## Adding a New Skill

1. Choose the right plugin folder; for a new plugin, add it under `plugins/` with its own `plugin.json`
2. Create `plugins/<number>-<name>/skills/<verb>/SKILL.md` with `name` and `description` frontmatter
3. Scope `allowed-tools` to only what the skill needs
4. Add a routing entry to `plugins/08-dispatcher/skills/studio/SKILL.md`
5. If the skill produces regulatory output, emit `<!-- architecture-studio:requires-disclaimer -->`
6. Run `./scripts/lint.sh` — fix any failures before committing
7. Bump both `plugin.json` and `marketplace.json` versions + update `CHANGELOG.md`
