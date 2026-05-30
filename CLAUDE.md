# skills-for-architects

Comprehensive multi-plugin marketplace (37 skills across 9 plugins) for architects, designers, and AEC professionals. Built by ALPA (Alpaca Labs). Covers site analysis, NYC zoning, programming, materials research, specifications, sustainability, and presentations.

## Repository Purpose

A Claude Code plugin collection providing AI-powered tools for architecture practice. Includes skills, agents, rules, and hooks. Deployed via the Claude Code plugin marketplace.

## Structure

```
skills-for-architects/
├── README.md
├── PATTERNS.md                # Canonical design patterns reference
├── CHANGELOG.md               # Per-version history
├── LICENSE                    # MIT
├── .claude-plugin/
│   └── marketplace.json       # Plugin registry & metadata (v1.1.1)
├── agents/                    # 7 orchestrating agents
│   ├── site-planner.md
│   ├── nyc-zoning-expert.md
│   ├── workplace-strategist.md
│   ├── product-and-materials-researcher.md
│   ├── ffe-designer.md
│   ├── sustainability-specialist.md
│   └── brand-manager.md
├── plugins/                   # 9 numbered plugin directories
│   ├── 00-due-diligence/      # NYC property data (7 skills)
│   ├── 01-site-planning/      # Environmental, mobility, demographics (4 skills)
│   ├── 02-zoning-analysis/    # NYC buildable envelope, 3D viz (2 skills)
│   ├── 03-programming/        # Space programs, occupancy loads (2 skills)
│   ├── 04-specifications/     # CSI spec writing (1 skill)
│   ├── 05-sustainability/     # EPD parsing, GWP research (4 skills)
│   ├── 06-materials-research/ # Product research, spec extraction, PDF/image (12 skills)
│   ├── 07-presentations/      # Slide decks, color palettes, image resizing (3 skills)
│   └── 08-dispatcher/         # /studio router, /skills menu (2 skills)
├── rules/                     # 7 hard enforcement rules
│   ├── units-and-measurements.md
│   ├── code-citations.md
│   ├── professional-disclaimer.md
│   ├── csi-formatting.md
│   ├── terminology.md
│   ├── output-formatting.md
│   └── transparency.md
├── hooks/                     # 3 event-driven bash automations
│   ├── post-write-disclaimer-check.sh
│   ├── post-output-metadata.sh
│   └── pre-commit-spec-lint.sh
└── scripts/
    └── lint.sh                # YAML lint runner
```

## Plugin Architecture

### PATTERNS.md is the canonical reference

Before adding anything to this repo, read `PATTERNS.md`. It defines the authoritative conventions for building plugins at ALPA. Key patterns:

**Small, single-verb skills** (~500 lines max per SKILL.md). If a skill does two things, split it.

**Clear contracts**: Every SKILL.md body must document what goes in, what comes out, and what the model should do.

**Dispatcher pattern**: `08-dispatcher` is the single entry point. `/studio` routes to the right plugin. `/skills` lists available commands. Add new commands to the dispatcher routing table when adding skills.

**Orchestrator agents**: Agents in `agents/` call multiple skills in sequence. They do not contain skill logic themselves — they reference skill names and coordinate.

**Rule enforcement via HTML markers**: Skills embed markers like `<!-- architecture-studio:requires-disclaimer -->` in their output templates. The `post-write-disclaimer-check.sh` hook detects these and enforces the rule. Don't remove markers.

### Plugin Naming Convention

Plugins are numbered `00-` through `08-` to control load order. Skills within plugins use the pattern `<plugin-prefix>-<verb>-<noun>` in kebab-case (e.g., `zoning-calculate-envelope`, `materials-parse-epd`).

### Rules

Rules in `rules/` are hard constraints — the model must always follow them. Each rule file explains the WHY (usually from a past incident). Do not weaken rules without team discussion.

Key rules:
- **professional-disclaimer**: Every output involving code, zoning, or specifications must include a disclaimer
- **units-and-measurements**: Always use imperial + metric dual notation for AEC context
- **csi-formatting**: CSI division numbers must match MasterFormat 2020
- **code-citations**: All building code references must include edition year and jurisdiction

### Hooks

| Hook | Event | Purpose |
|------|-------|---------|
| `post-write-disclaimer-check.sh` | After write | Verifies disclaimer markers are present |
| `post-output-metadata.sh` | After output | Appends plugin version + timestamp metadata |
| `pre-commit-spec-lint.sh` | Before commit | Lints specification YAML for required fields |

Hooks are bash scripts. They must be executable (`chmod +x`) and should exit 0 on success, non-zero to block.

## Versioning

Follow semantic versioning in `marketplace.json` and `CHANGELOG.md`:
- **Patch** (x.x.X): Bug fixes, documentation updates, rule clarifications
- **Minor** (x.X.0): New skills, new agents, new hooks
- **Major** (X.0.0): Breaking changes to existing skill contracts, dispatcher interface changes

Always update `CHANGELOG.md` with the date and description. Tag the commit: `git tag v<version>`.

## CI/CD

GitHub Actions runs `./scripts/lint.sh` on push and pull requests to `main`. The linter validates YAML frontmatter in all skill files using Python's yaml parser. All PRs must pass lint before merge.

To run locally:
```bash
./scripts/lint.sh
```

## Development Workflow

1. Read `PATTERNS.md` before writing any new skill or agent
2. Identify the right plugin directory (or create a new one with the next number)
3. Write the SKILL.md following the contract pattern
4. Add the skill to the dispatcher routing table in `08-dispatcher/`
5. Update `marketplace.json` version (patch or minor)
6. Update `CHANGELOG.md`
7. Run `./scripts/lint.sh` locally
8. Open PR — describe the use case, the contract, and how you tested it

## NYC-Specific Context

Plugins `00-due-diligence` and `02-zoning-analysis` are NYC-specific and reference:
- NYC Zoning Resolution (ZR) with section numbers
- DOB NOW, ACRIS, HPD, BSA, LPC data sources
- Landmarks Preservation Commission regulations

For non-NYC adaptations, create new skills under the appropriate plugin rather than modifying NYC-specific ones.

## Key Conventions

- **No generic output** — every skill output must be actionable for an AEC professional
- **Source attribution** — cite the specific code section, dataset, or standard behind every fact
- **Disclaimer markers** — any output that could be relied upon professionally must include `<!-- architecture-studio:requires-disclaimer -->`
- **Structured output** — prefer tables and structured markdown over prose for data-heavy outputs
- **Dual units** — always provide both imperial and metric measurements
