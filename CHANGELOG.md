# Changelog

All notable changes to this project are documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) and semantic versioning.

---

## [1.1.0] — 2026-11-01

### Added — OpenCode port

- `opencode/` — launcher (`bin/mealie-mcp`), `opencode.json.example`, `README.md`, `USAGE.md` (French guide)
- **Commands** ported to `/mealie/find`, `/mealie/weekly-menu`, `/mealie/quick-recipe` (frontmatter reduced to `description`)
- **Skills** ported (4): `recipe-creation`, `foundations`, `cooking-tips`, `meal-planning` — loaded explicitly (`opencode/autoinvoke: false`)
- `permissions` in the example config deny `delete_recipe`, `delete_shopping_list`, `delete_meal_plan_entry`

### Fixed

- Mealie v3.29 routes: `/api/groups/{shopping,mealplans,cookbooks}` → `/api/households/…`, `/api/recipes/create-url` → `/api/recipes/create/url` — see `patch/mealie-routes-v3.29.patch`

### Removed

- Claude Code plugin packaging: `.claude-plugin/`, `agents/`, `hooks/`, `.mcp.json`, and the root `skills/`/`commands/` superseded by their `opencode/` copies (including `dietary-management`, which encoded one family's rules)
- These files remain available in git history and in the upstream repository

### Documentation

- `README.md` rewritten for OpenCode, with an explicit Credits section
- `LICENSE` carries both copyright notices (Thrivewell Partners + fork author)

---

## [1.0.0] — 2026-02-24

### Added — Claude Code Plugin

- `.claude-plugin/plugin.json` manifest — plugin is now installable from GitHub Marketplace
- `marketplace.json` — Thrivewell Partners marketplace directory entry
- **Skills** (5 total):
  - `recipe-creation` — Mealie v3 recipe authoring with family guidelines
  - `foundations` — Foundation recipe paradigm (educational "why behind the how")
  - `dietary-management` — No-pork / no-seed-oil compliance and substitutions
  - `cooking-tips` — Homesteading techniques, scratch cooking, whole-food methods
  - `meal-planning` — Weekly menu planning via Mealie API
- **Agents** (3 total):
  - `recipe-advisor` — Expert recipe consultant, auto-triggered for recipe discussions
  - `meal-planner` — Proactive weekly meal builder
  - `cooking-teacher` — Technique educator connecting to Foundation recipes
- **Commands** (3 total):
  - `quick-recipe` — Fast structured recipe creation into Mealie
  - `weekly-menu` — Plan a week of meals with family preferences
  - `find-recipe` — Search and filter with family dietary context
- **Hooks** — `PreToolUse` dietary guardrail scans ingredient lists before saving
- `.mcp.json` updated to use `${CLAUDE_PLUGIN_ROOT}`, `${MEALIE_URL}`, `${MEALIE_API_KEY}` env vars

---

## [0.1.0] — Initial MCP Server

### Added

- FastMCP server with 27 tools across 5 categories
- **Recipes** (7): search, get, create, import URL, update, delete, get ingredients
- **Shopping Lists** (8): list, view, create, delete, add/remove/check items, add recipe
- **Meal Plans** (4): view, create, delete, generate random
- **Organizers** (6): categories, tags, cookbooks (list + create each)
- **Foods & Units** (2): search foods, list units
- Structured ingredient support — food/unit resolution, section headers, auto-create missing foods
- Foundation recipe support — instruction dicts with title/summary/text, named notes, `isFoundation` extras
- Mealie v3 API compatibility — PATCH updates, `referenceId` on ingredients, full category/tag objects
