# Skills for Architects

Architecture Studio — agents, skills, rules, and hooks for architects, designers, and AEC (Architecture/Engineering/Construction) professionals. Works with Claude Desktop and Claude Code.

**Stats:** 7 agents · 37 skills · 7 rules · 3 hooks across 9 plugins. Built by [ALPA](https://alpa.llc).

## Repository Structure

```
skills-for-architects/
├── README.md                    # Full documentation, quick start, skill directory
├── PATTERNS.md                  # Naming, layout, dispatcher pattern, versioning, hard rules
├── CHANGELOG.md                 # Version history
├── LICENSE                      # MIT
├── .claude-plugin/              # Claude plugin manifest
├── .github/                     # CI workflows
├── agents/                      # 7 orchestration agents
│   ├── site-planner.md
│   ├── nyc-zoning-expert.md
│   ├── workplace-strategist.md
│   ├── product-and-materials-researcher.md
│   ├── ffe-designer.md
│   ├── sustainability-specialist.md
│   └── brand-manager.md
├── plugins/                     # 9 skill plugins by project lifecycle phase
│   ├── 00-due-diligence/        # 7 skills — NYC property data (landmarks, DOB, ACRIS, HPD, BSA)
│   ├── 01-site-planning/        # 4 skills — Environmental, mobility, demographics, history
│   ├── 02-zoning-analysis/      # 2 skills — NYC zoning envelope + 3D visualization
│   ├── 03-programming/          # 2 skills — Space programs + IBC occupancy loads
│   ├── 04-specifications/       # 1 skill — CSI MasterFormat outline specs
│   ├── 05-sustainability/       # 4 skills — EPD parsing, research, comparison, spec thresholds
│   ├── 06-materials-research/   # 12 skills — FF&E product research, schedules, SIF conversion
│   ├── 07-presentations/        # 3 skills — Slide decks, color palettes, image resizing
│   └── 08-dispatcher/           # 2 skills — /studio router + /skills help menu
├── rules/                       # 7 always-on output conventions (loaded automatically)
│   ├── units-and-measurements.md
│   ├── code-citations.md
│   ├── professional-disclaimer.md
│   ├── csi-formatting.md
│   ├── terminology.md
│   ├── output-formatting.md
│   └── transparency.md
├── hooks/                       # 3 event-driven shell automations
│   ├── post-write-disclaimer-check.sh
│   ├── post-output-metadata.sh
│   └── pre-commit-spec-lint.sh
└── scripts/                     # Utility scripts
```

## Architecture Layers

### Agents (orchestration)
Describe your task — the agent selects which skills to invoke, in what order, and applies domain judgment. Entry point: `/studio <your task>`

| Agent | Domain | Entry Point |
|-------|--------|-------------|
| `site-planner` | Site Planning | `/studio <address>` |
| `nyc-zoning-expert` | Due Diligence + Zoning | `/studio <address>` |
| `workplace-strategist` | Programming | `/studio <headcount + work style>` |
| `product-and-materials-researcher` | Materials Research | `/studio <product brief>` |
| `ffe-designer` | FF&E Design | `/studio <schedule task>` |
| `sustainability-specialist` | Sustainability | `/studio <material/EPD task>` |
| `brand-manager` | Presentations | `/studio <deck/palette task>` |

### Skills (slash commands)
Single-purpose tools invoked directly (e.g. `/environmental-analysis 123 Main St`). Grouped into plugins by lifecycle phase. See `README.md` for the full list of 37 skills.

### Rules (always-on conventions)
Govern every output. Loaded automatically by the plugin system — never invoked manually.

| Rule | What It Governs |
|------|-----------------|
| `units-and-measurements` | Imperial primary with metric parenthetical; GSF/USF/RSF area types |
| `code-citations` | Edition year and jurisdiction required (e.g. "IBC 2021 §1004.1") |
| `professional-disclaimer` | Mandatory on regulatory/code/zoning outputs |
| `csi-formatting` | MasterFormat 2018 — 6-digit section numbers, three-part structure |
| `terminology` | AEC standard terms, abbreviations, material names |
| `output-formatting` | Tables, source attribution, file naming, list structure |
| `transparency` | Link sources, show inputs, make outputs verifiable |

### Hooks (event-driven automations)
Opt-in via Claude Code settings.

| Hook | Event | Action |
|------|-------|--------|
| `post-write-disclaimer-check.sh` | After Write | Warns if regulatory output lacks disclaimer |
| `post-output-metadata.sh` | After Write | Stamps markdown with YAML front matter |
| `pre-commit-spec-lint.sh` | Before git commit | Flags malformed CSI section numbers |

## SKILL.md Structure

```yaml
---
name: skill-name          # kebab-case, matches directory name exactly
description: [trigger phrases for skill selection]
version: X.Y.Z
---

# Skill Title

[Domain knowledge, instructions, tables, reference data]
```

## Naming Conventions

- **Plugin directories:** `<number>-<name>` (e.g. `00-due-diligence`)
- **Skill directories:** kebab-case noun phrases (e.g. `epd-parser`)
- **Agent files:** kebab-case `.md` in `agents/` (e.g. `site-planner.md`)
- **Rule files:** kebab-case `.md` in `rules/` (e.g. `csi-formatting.md`)
- **Hook files:** `.sh` in `hooks/` with event prefix (e.g. `post-write-disclaimer-check.sh`)

See `PATTERNS.md` for the full naming, layout, dispatcher, and versioning conventions.

## Adding a New Skill

1. Choose the appropriate plugin folder (or propose a new plugin in the PR)
2. Create: `plugins/<plugin-name>/skills/<skill-name>/`
3. Add `SKILL.md` (required) + `README.md` (required) + any supporting data files
4. Register the skill in the plugin manifest
5. Update `plugins/08-dispatcher/` help output to include the new skill
6. Test against real AEC input (real addresses, real product URLs, real EPD PDFs)
7. Open a PR with: what the skill does, how tested, and sample output

## Domain Conventions (enforced by rules)

- **Units:** Imperial primary (SF, LF, inches) with metric in parentheses. Use GSF = gross, USF = usable, RSF = rentable
- **Code citations:** Always include edition year and jurisdiction
- **Specs:** MasterFormat 2018 — 6-digit section numbers (e.g. `09 21 16`), three-part format (Part 1 General / Part 2 Products / Part 3 Execution)
- **Professional disclaimer:** Required on any output referencing building codes, zoning, or regulatory data
- **Sources:** Always link sources. Expose inputs. Make outputs independently verifiable.

## NYC-Specific Skills

The `nyc-*` skills query live NYC public APIs:
- **DOB (Department of Buildings):** permits, violations
- **ACRIS:** property transaction records
- **HPD:** housing violations, complaints, registration
- **BSA (Board of Standards and Appeals):** variances, special permits
- **LPC (Landmarks Preservation Commission):** landmark status, historic districts
- **NYC PLUTO:** zoning district, FAR, height limits

Test against real NYC addresses. These skills require internet access.

## AI Assistant Guidelines

- Do **not** remove or weaken `professional-disclaimer.md` — it is legally required for regulatory outputs
- CSI formatting (`csi-formatting.md`) must be applied to all spec outputs — the pre-commit hook enforces this
- The dispatcher `/studio` routes to agents; direct slash commands bypass agents — both paths must work independently
- Skills must be single-purpose — if a SKILL.md handles multiple distinct scenarios, split it into separate skills
- Agent files include domain knowledge, handoff logic, and which skills to orchestrate — keep them current when skills are added
- The `transparency.md` rule is non-negotiable — always show sources and inputs, never output unverifiable claims
