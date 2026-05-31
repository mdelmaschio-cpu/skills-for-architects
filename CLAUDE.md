# CLAUDE.md — Architecture Studio (skills-for-architects)

This file documents the `skills-for-architects` repository for AI assistants. Read this before making any changes.

## Overview

`skills-for-architects` is a **multi-plugin Claude Code marketplace** called **Architecture Studio**, built by [ALPA](https://alpa.llc). It teaches Claude architecture-specific workflows for architects, designers, and AEC (Architecture, Engineering, Construction) professionals.

The marketplace ships:
- **7 agents** — orchestration personas that exercise judgment across skills
- **37 skills** — single-purpose slash-command tools
- **7 rules** — always-on output conventions
- **3 hooks** — event-driven automations
- **9 plugins** — installable bundles organized by project lifecycle

Entry point: `/studio` (the dispatcher skill). Users describe a task and the router hands off to the right agent or skill.

## Repository Structure

```
skills-for-architects/
├── .claude-plugin/
│   └── marketplace.json           # Marketplace manifest (name, version, plugin list)
├── .github/
│   └── workflows/lint.yml         # CI lint on push
├── .gitignore
├── CHANGELOG.md                   # Version history (semver)
├── LICENSE                        # MIT
├── PATTERNS.md                    # Canonical plugin authoring conventions (read this!)
├── README.md                      # User-facing docs with full skill table
├── agents/                        # Orchestration personas
│   ├── site-planner.md
│   ├── nyc-zoning-expert.md
│   ├── workplace-strategist.md
│   ├── product-and-materials-researcher.md
│   ├── ffe-designer.md
│   ├── sustainability-specialist.md
│   └── brand-manager.md
├── hooks/                         # Event-driven shell automations
│   ├── post-write-disclaimer-check.sh
│   ├── post-output-metadata.sh
│   └── pre-commit-spec-lint.sh
├── plugins/                       # One directory per plugin
│   ├── 00-due-diligence/
│   ├── 01-site-planning/
│   ├── 02-zoning-analysis/
│   ├── 03-programming/
│   ├── 04-specifications/
│   ├── 05-sustainability/
│   ├── 06-materials-research/
│   ├── 07-presentations/
│   └── 08-dispatcher/             # Contains /studio and /skills entry points
├── rules/                         # Always-on output conventions
│   ├── units-and-measurements.md
│   ├── code-citations.md
│   ├── professional-disclaimer.md
│   ├── csi-formatting.md
│   ├── terminology.md
│   ├── output-formatting.md
│   └── transparency.md
└── scripts/
    └── lint.sh                    # Structural lint script
```

### Plugin layout (multi-plugin nested)

Each plugin under `plugins/<name>/` follows this structure:
```
plugins/<name>/
├── .claude-plugin/
│   └── plugin.json                # Plugin manifest (name, version, skills list)
├── README.md
└── skills/
    └── <skill-name>/
        ├── SKILL.md               # Skill instructions + YAML frontmatter
        └── README.md
```

## Key Files

### `.claude-plugin/marketplace.json`
Marketplace-level manifest. Contains the marketplace name (`skills-for-architects`), owner (ALPA), metadata version (`1.1.1`), and the list of all 9 plugins with their source paths.

### `PATTERNS.md`
**Critical reference.** The canonical guide for how ALPA builds Claude plugins. Covers all conventions used in this repo — read before contributing. Key patterns:
1. Small skills, one verb each (max ~500 lines per SKILL.md)
2. Explicit contracts between skills via cross-references
3. Kebab-case naming at every layer
4. Single orchestrator (dispatcher) per plugin
5. Marker-driven hooks (not keyword-sniffing)
6. Semver versioning with git tags + GitHub releases
7. Multi-plugin nested layout (this repo)
8. MCP server bundling via `.mcp.json` + `${CLAUDE_PLUGIN_ROOT}`
9. Public repos by default
10. Hard rules encoded from real bugs

### `plugins/08-dispatcher/`
The entry-point plugin. Contains two skills:
- `/studio` — smart router; classifies user intent and hands off to the right agent or skill
- `/skills` — help menu listing all available skills and agents

### `agents/`
Agent markdown files define orchestration personas. Agents assess input, choose paths, and sequence skill invocations. They are not slash commands — they are invoked by the dispatcher based on intent.

### `rules/`
Always-on conventions loaded automatically (not slash-invoked). They apply to every output across all skills.

### `hooks/`
Shell scripts registered via Claude Code settings. Three hooks:
- `post-write-disclaimer-check.sh` — after Write: warns if regulatory output is missing the professional disclaimer (marker-driven, not keyword-sniffing)
- `post-output-metadata.sh` — after Write: stamps markdown reports with YAML front matter
- `pre-commit-spec-lint.sh` — before git commit: flags malformed CSI section numbers

## Available Plugins and Skills

| # | Plugin | Skills | Domain |
|---|--------|--------|--------|
| 0 | `00-due-diligence` | 7 | NYC property data: landmarks, DOB, violations, ACRIS, HPD, BSA |
| 1 | `01-site-planning` | 4 | Site research: environmental, mobility, demographics, history |
| 2 | `02-zoning-analysis` | 2 | NYC zoning envelope + 3D visualization |
| 3 | `03-programming` | 2 | Workplace strategy: space programs, IBC occupancy |
| 4 | `04-specifications` | 1 | CSI MasterFormat spec writing |
| 5 | `05-sustainability` | 4 | EPD parsing, research, comparison, GWP thresholds |
| 6 | `06-materials-research` | 12 | FF&E product research, spec extraction, cleanup, image processing |
| 7 | `07-presentations` | 3 | HTML slide decks, color palettes, image resizing |
| 8 | `08-dispatcher` | 2 | `/studio` router + `/skills` help menu |

### Agents

| Agent | Domain |
|-------|--------|
| `site-planner` | Parallel site research + synthesis |
| `nyc-zoning-expert` | Full NYC property + zoning + 3D |
| `workplace-strategist` | Headcount → space program |
| `product-and-materials-researcher` | Find, extract, tag, classify |
| `ffe-designer` | FF&E schedules, QA, dealer export |
| `sustainability-specialist` | EPDs, GWP, LEED |
| `brand-manager` | Decks + palettes |

## How to Use with Claude Code

### Install the full marketplace
```bash
claude plugin marketplace add AlpacaLabsLLC/skills-for-architects
```

### Install a specific plugin
```bash
claude plugin install 01-site-planning@skills-for-architects
```

### Claude Desktop
Open **Customize** → **Browse plugins** → **+** → **Add marketplace from GitHub** → enter `AlpacaLabsLLC/skills-for-architects`

### Invoke skills
```
/studio task chair, mesh back, under $800
/studio 123 Main St, Brooklyn NY
/studio I need a space program for 200 people
/studio parse this EPD

# Or call skills directly:
/environmental-analysis 123 Main St, Brooklyn NY
/nyc-landmarks 123 Main St, Brooklyn NY
/spec-writer flooring, wall tile, acoustic ceiling
```

### Enable hooks (Claude Code)
Register hooks in your Claude Code settings (`.claude/settings.json`):
```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Write", "hooks": [{ "type": "command", "command": "hooks/post-write-disclaimer-check.sh" }] },
      { "matcher": "Write", "hooks": [{ "type": "command", "command": "hooks/post-output-metadata.sh" }] }
    ],
    "PreToolUse": [
      { "matcher": "Bash", "hooks": [{ "type": "command", "command": "hooks/pre-commit-spec-lint.sh" }] }
    ]
  }
}
```
See each hook file's header for exact installation instructions.

## Development Conventions

All conventions are documented exhaustively in `PATTERNS.md`. Key points for contributors:

### Naming
- Marketplace: `skills-for-architects` (kebab-case)
- Plugins: `00-due-diligence`, `01-site-planning`, etc. (numbered + kebab-case)
- Skills in multi-plugin: verb-only, namespaced by plugin (e.g., `nyc-landmarks`, `spec-writer`)
- Dispatcher skill always named same as plugin (e.g., `studio`)

### SKILL.md frontmatter (required fields)
```yaml
---
name: <kebab-case-skill-name>
description: <trigger-phrase-rich description for model invocation>
---
```
Optional: `user-invocable: true`, `disable-model-invocation: true`, `allowed-tools: [...]`

### Adding a new skill
1. Create `plugins/<plugin>/skills/<skill-name>/SKILL.md` and `README.md`
2. Update `plugins/<plugin>/.claude-plugin/plugin.json` — add the skill, bump the plugin `version`
3. Update `.claude-plugin/marketplace.json` — bump `metadata.version`
4. Update `CHANGELOG.md` with the new entry
5. Update `README.md` skill table and count claims
6. Run `scripts/lint.sh` — must pass before committing
7. Commit, tag, and cut a GitHub release (see versioning below)

### Adding a new plugin
1. Create `plugins/<name>/` with `.claude-plugin/plugin.json`, a dispatcher skill, and sub-skills
2. Add the plugin entry to `.claude-plugin/marketplace.json`
3. Add agents in `agents/` if the plugin has an orchestration persona
4. Update `README.md` plugin table
5. Bump both `plugin.json` version and `marketplace.json` `metadata.version`

### Versioning (mandatory on every shipped change)
1. Bump `version` in the relevant `plugin.json` (patch / minor / major per semver)
2. Bump `metadata.version` in `.claude-plugin/marketplace.json`
3. Add a `CHANGELOG.md` entry: `## [X.Y.Z] - YYYY-MM-DD`
4. Commit all three files together with the actual change
5. Tag: `git tag -a vX.Y.Z <sha> -m "vX.Y.Z — short description"` and `git push origin vX.Y.Z`
6. Cut a GitHub release: `gh release create vX.Y.Z --title "vX.Y.Z — …" --notes-file <changelog-section>`

**All three artifacts must move together: JSON version field + git tag + GitHub release.**

### Hooks — marker-driven enforcement
Hooks check for HTML comment markers emitted by skills, not for keywords. Example:
```html
<!-- architecture-studio:requires-disclaimer -->
```
Do not use keyword sniffing in hooks — it causes false positives on READMEs and false negatives on terse regulatory replies.

### Hard rules from real bugs
When a bug is found in production, encode the fix as an explicit rule in the relevant `SKILL.md` body — not just a code patch. Rules survive refactors. See `PATTERNS.md` section 10 for examples.

## CI Lint

`scripts/lint.sh` (also runs via `.github/workflows/lint.yml` on push) validates:
1. No tracked `.DS_Store` files
2. Valid JSON in all manifest files (`marketplace.json`, `plugin.json`, `.mcp.json`)
3. `SKILL.md` frontmatter has `name` and `description`
4. Skill count consistency between README claims, `marketplace.json`, and actual file count
5. No broken internal markdown links
6. `shellcheck` on `hooks/*.sh`

Run `bash scripts/lint.sh` locally before committing any change.

## Rules Reference

These apply to every skill output automatically:

| Rule file | What it governs |
|-----------|----------------|
| `units-and-measurements.md` | Imperial/metric, GSF/USF/RSF area types, dimension formatting |
| `code-citations.md` | Building code references, edition years, jurisdiction awareness |
| `professional-disclaimer.md` | Required disclaimer language on regulatory outputs |
| `csi-formatting.md` | MasterFormat 2018 section numbers, three-part spec structure |
| `terminology.md` | AEC standard terms, abbreviations, material names |
| `output-formatting.md` | Tables, source attribution, file naming, list structure |
| `transparency.md` | Link sources, expose inputs, make outputs verifiable |

## Related Resources

- ALPA website: https://alpa.llc
- PATTERNS.md authoring conventions: `./PATTERNS.md`
- Claude Code docs: https://docs.anthropic.com/en/docs/claude-code
- Article — Claude Code Cheat Sheet for Architects: https://alpa.llc/articles/claude-code-cheat-sheet
- Article — Distributing Skills to Teams: https://alpa.llc/articles/distributing-skills-to-teams
- Upstream repo (canonical source): https://github.com/AlpacaLabsLLC/skills-for-architects
