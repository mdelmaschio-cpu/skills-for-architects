# skills-for-architects — Architecture Studio

Multi-plugin Claude marketplace for architects, designers, and AEC professionals. Built by [ALPA](https://alpa.llc).

**Current version:** `1.1.1` · See `CHANGELOG.md` for full history.

> **Start here:** Read `PATTERNS.md` before making any structural changes. It is the canonical reference for how this repo is built, with the WHY documented inline for every rule.

## Repository Purpose

Architecture Studio teaches Claude architecture-specific workflows across the full project lifecycle: site research, zoning analysis, space programming, specifications, materials research, sustainability, and presentations.

- **7 agents** — orchestration personas that assess intent and chain skills with judgment
- **37 skills** — single-purpose slash commands organized across 9 plugins
- **7 rules** — always-on output conventions (loaded automatically, not invoked)
- **3 hooks** — event-driven automations (opt-in via Claude Code settings)

## Repository Structure

```
skills-for-architects/
├── .claude-plugin/
│   └── marketplace.json           # Marketplace manifest: version, plugin list
├── .github/
│   └── workflows/
│       └── lint.yml               # CI: runs scripts/lint.sh on push to main + all PRs
├── .gitignore
├── CHANGELOG.md                   # Keep-a-Changelog format, semver
├── PATTERNS.md                    # Canonical build conventions — read before contributing
├── README.md
├── agents/                        # 7 agent definition .md files + README
│   ├── brand-manager.md
│   ├── ffe-designer.md
│   ├── nyc-zoning-expert.md
│   ├── product-and-materials-researcher.md
│   ├── site-planner.md
│   ├── sustainability-specialist.md
│   └── workplace-strategist.md
├── hooks/                         # 3 shell scripts + settings snippet
│   ├── post-output-metadata.sh
│   ├── post-write-disclaimer-check.sh
│   ├── pre-commit-spec-lint.sh
│   └── settings-snippet.json      # Paste into ~/.claude/settings.json to enable hooks
├── plugins/                       # 9 plugins, numbered by project lifecycle order
│   ├── 00-due-diligence/          # 7 skills: NYC property records
│   ├── 01-site-planning/          # 4 skills: site research
│   ├── 02-zoning-analysis/        # 2 skills: buildable envelope + 3D viewer
│   ├── 03-programming/            # 2 skills: space programming + occupancy
│   ├── 04-specifications/         # 1 skill:  CSI MasterFormat spec writing
│   ├── 05-sustainability/         # 4 skills: EPD / GWP / LEED
│   ├── 06-materials-research/     # 12 skills: FF&E research, schedules, SIF
│   ├── 07-presentations/          # 3 skills: slide decks, palettes, image resize
│   └── 08-dispatcher/             # 2 skills: /studio router + /skills menu
├── rules/                         # 7 always-on rule files + README
│   ├── code-citations.md
│   ├── csi-formatting.md
│   ├── output-formatting.md
│   ├── professional-disclaimer.md
│   ├── terminology.md
│   ├── transparency.md
│   └── units-and-measurements.md
└── scripts/
    └── lint.sh                    # Structural lint — run before every commit
```

### Plugin directory layout

Every plugin follows this internal structure:

```
plugins/<nn>-<name>/
├── .claude-plugin/
│   └── plugin.json                # Plugin manifest: name, version, skills list
└── skills/
    └── <skill-name>/
        ├── SKILL.md               # Skill definition: frontmatter + body
        └── README.md              # User-facing docs for this skill
```

## Component Types

| Component | Location | How loaded | Purpose |
|-----------|----------|-----------|--------|
| **Agent** | `agents/<name>.md` | Via `/studio` or model routing | Orchestrates multiple skills; applies judgment across a workflow |
| **Skill** | `plugins/<nn>/skills/<name>/SKILL.md` | Slash command or dispatcher | Single-purpose task — one verb, one concern |
| **Rule** | `rules/<name>.md` | Loaded automatically | Cross-cutting output conventions for every response |
| **Hook** | `hooks/<name>.sh` | Claude Code event system (opt-in) | Enforcement automation triggered by tool events |

## Skill Authoring Conventions

Key rules from `PATTERNS.md` §1–§2:

- **One skill = one verb.** If `SKILL.md` exceeds ~500 lines or handles multiple distinct workflows, split it.
- `name` matches the directory name, kebab-case.
- `description` is trigger-phrase-rich — the model selects skills based on description matching. Vague descriptions cause missed invocations.
- `allowed-tools` is scoped to what **this** skill actually needs, not the union of all plugin tools.
- Skills hand off via explicit cross-references in their bodies: "After this completes, run `/epd-compare`."

**Skill frontmatter:**
```yaml
---
name: skill-name
description: Trigger-phrase-rich description. Include the exact phrases and
  situations that should invoke this skill. Be specific.
allowed-tools: [WebSearch, Read]     # scope tightly
user-invocable: true                 # omit if slash-only
---
```

## Naming Conventions

| Layer | Format | Example |
|-------|--------|---------|
| Marketplace | kebab-case product family | `skills-for-architects` |
| Plugin | `<nn>-<kebab-name>` | `00-due-diligence` |
| Dispatcher skill | matches plugin family name | `studio` |
| Sub-skill | `<verb>` (already namespaced by plugin) | `nyc-landmarks`, `epd-parser` |

## Dispatcher Pattern

Plugin `08-dispatcher` is the single entry point. `/studio <task>` reads user intent, classifies it against a routing table, and hands off to the right agent or skill. For ambiguous requests, it falls back to working mode.

**Do not add routing logic to individual skills.** All routing belongs in the dispatcher.

## Rules (Always-On Conventions)

Rules in `rules/` apply to every output and are loaded automatically — they are not slash commands. Current rules:

| Rule | What it governs |
|------|-----------------|
| `units-and-measurements` | Imperial/metric, GSF/USF/RSF area types, dimension notation |
| `code-citations` | Building code references, edition years, jurisdiction awareness |
| `professional-disclaimer` | Disclaimer language and when regulatory output requires it |
| `csi-formatting` | MasterFormat 2018 section numbers, three-part specification structure |
| `terminology` | AEC standard terms, abbreviations, material names |
| `output-formatting` | Tables, source attribution, file naming, list structure |
| `transparency` | Show your work — link sources, expose inputs, make outputs verifiable |

## Hooks (Opt-In Automations)

Hooks are event-driven shell scripts. Install by adding the `hooks/settings-snippet.json` content to `~/.claude/settings.json`.

| Hook | Event | What it does |
|------|-------|--------------|
| `post-write-disclaimer-check.sh` | After Write | Checks for `<!-- architecture-studio:requires-disclaimer -->` marker + canonical disclaimer block |
| `post-output-metadata.sh` | After Write | Stamps markdown reports with YAML front matter |
| `pre-commit-spec-lint.sh` | Before commit | Flags malformed CSI MasterFormat section numbers |

**Hooks are marker-driven, not keyword-sniffed.** Skills that produce regulatory output emit an HTML comment marker; hooks check for the marker. This eliminates false positives on READMEs and changelogs that mention regulated terms in passing. See `PATTERNS.md` §5.

## Versioning — Three Artifacts Must Move Together

On **every shipped change**, update all three in a single commit:

1. **`plugin.json` `version`** (per-plugin) — drives `claude plugin marketplace update` detection
2. **`marketplace.json` `metadata.version`** — documentation trail; bump for any repo-level change
3. **Git tag** `git tag -a vX.Y.Z <sha> -m "vX.Y.Z — description"` + `git push origin vX.Y.Z`
4. **GitHub release** `gh release create vX.Y.Z --title "vX.Y.Z — ..." --notes-file <changelog-section>`

Semver rules: patch for fixes/docs, minor for new skills or non-breaking behavior, major for breaking layout changes.

**CHANGELOG entry** under `## [X.Y.Z] - YYYY-MM-DD` is required for every release.

Omitting any of the three creates gaps: without a `plugin.json` bump, Cowork reports "already up to date" even when new commits exist; without a tag, `git checkout vX.Y.Z` fails; without a release, there is no shareable URL. See `PATTERNS.md` §6 for the full incident history.

## CI / Lint

Always run before pushing:

```bash
bash scripts/lint.sh
```

The lint script fails on:
1. Tracked `.DS_Store` files
2. Invalid JSON in any manifest (`marketplace.json`, `plugin.json`, `.mcp.json`)
3. `SKILL.md` missing `name` or `description` in frontmatter
4. Count drift — README headline, details section, plugin table, marketplace.json plugin list, and actual file count must all agree
5. Broken internal markdown links
6. `shellcheck` errors in `hooks/*.sh`

CI runs the same script on push to `main` and on every PR.

## Adding a New Skill

1. Identify the right plugin (lifecycle order: 00 = earliest, 08 = dispatcher).
2. Create `plugins/<nn>-<name>/skills/<skill-name>/SKILL.md` with full frontmatter.
3. Add `README.md` to the skill directory.
4. Register the skill in `plugins/<nn>-<name>/.claude-plugin/plugin.json`.
5. Update the README skill table (plugin row count + details section + `/skills` menu in the dispatcher).
6. Bump `plugin.json` version and `marketplace.json` `metadata.version`.
7. Run `bash scripts/lint.sh` — fix any count/frontmatter errors before committing.
8. Commit, tag, push, create GitHub release.

## Adding a New Plugin

1. Create `plugins/<nn>-<new-name>/`.
2. Add `.claude-plugin/plugin.json` starting at version `0.1.0`.
3. Create skills following the structure above.
4. Add a plugin entry to `.claude-plugin/marketplace.json`.
5. Update README plugin table.
6. Run lint, bump versions, tag, release.

## Hard Rules (Captured from Production Bugs)

These are stated in the dispatcher skill body AND in each sub-skill that touches affected behavior — never assume the dispatcher was invoked first.

- **Audit always re-parses** — never trust cached values; re-run `parse_product_url` against the live URL on every audit invocation, even if the cache is recent. Surfaces drift between sheet ↔ catalog ↔ live.
- **Read before write** — always read current sheet state before writing; use exact header names verbatim, never guess or invent column names.
- **Update in place** — when a SKU match exists, use `update_row_by_match`; never append a new row.
- **No fabricated capabilities** — only describe what the tools just called actually did; do not claim background refresh, async workers, or features that don't exist.

See `PATTERNS.md` §10 for the incident histories that produced each rule.

## MCP Server Bundling (if needed)

Plugins that depend on an MCP server bundle it inside the plugin via `.mcp.json` at the plugin root:

```json
{
  "mcpServers": {
    "<plugin-name>": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp/dist/server.js"]
    }
  }
}
```

The MCP source lives under `<plugin>/mcp/` with `dist/` committed so a fresh clone runs without a rebuild. `node_modules/` is gitignored. Never require per-user config edits — plugin install = MCP install.
