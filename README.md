# Mealie MCP for OpenCode

An [OpenCode](https://opencode.ai) port of the [Thrivewell Partners Mealie MCP
server](https://github.com/thrivewell-partners/mealie-mcp): a 27-tool MCP server
for the [Mealie](https://mealie.io/) recipe manager, plus four skills and three
commands that let you search recipes, plan the week, and fill shopping lists
from a conversation.

This is a **fork**, not an original project. The MCP server, its tool set, the
skills, and the commands were written by Thrivewell Partners; see
[Credits](#credits).

The server code is unchanged. What changed is the packaging: Claude Code plugin
manifests, hooks, and agents are dropped, MCP tool names in prose are plain
(`search_recipes` → `tools.mealie.search_recipes`), and the family-specific
dietary rules are removed.

Full French guide: [`opencode/USAGE.md`](opencode/USAGE.md). Port summary:
[`opencode/README.md`](opencode/README.md).

---

## Features

- **27-tool MCP server** — recipes, shopping lists, meal plans, organizers, foods, units
- **4 skills** — recipe creation, Foundation recipes, cooking techniques, meal planning
- **3 commands** — `/mealie/find`, `/mealie/weekly-menu`, `/mealie/quick-recipe`
- **Delete guard** — `permissions` denies `delete_recipe`, `delete_shopping_list`, `delete_meal_plan_entry`
- **Mealie v3.29 routes** — the client is patched for current Mealie (see below)

---

## Install

**1. Clone and install**

```sh
git clone https://github.com/<you>/mealie-mcp
cd mealie-mcp
uv sync --python 3.13
```

**2. Add your Mealie credentials**

Create `.env` in `opencode/` (read by the launcher):

```sh
MEALIE_URL=https://mealie.example.com
MEALIE_API_KEY=<token from Mealie → profile → API tokens>
```

**3. Configure OpenCode**

Copy [`opencode/opencode.json.example`](opencode/opencode.json.example) into your
project as `opencode.json` and replace the absolute path with your clone:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "mealie": {
        "type": "local",
        "command": ["/absolute/path/to/mealie-mcp/opencode/bin/mealie-mcp"],
        "timeout": { "startup": 60000, "catalog": 60000, "execution": 120000 }
      }
    }
  },
  "skills": ["./opencode/skills"],
  "permissions": [
    { "action": "mealie_delete_recipe", "resource": "*", "effect": "deny" },
    { "action": "mealie_delete_shopping_list", "resource": "*", "effect": "deny" },
    { "action": "mealie_delete_meal_plan_entry", "resource": "*", "effect": "deny" }
  ]
}
```

**4. Add commands and skills to your project**

```sh
cp -r opencode/commands opencode/skills <project>/.opencode/
```

The `skills: ["./opencode/skills"]` entry above works too if you prefer to keep
them in the clone.

**5. Check the connection**

```sh
opencode mcp list     # ✓ mealie connected
```

The launcher reads `MEALIE_URL` / `MEALIE_API_KEY` from the environment first,
then from `opencode/.env` (`MEALIE_BASE_URL` / `MEALIE_API_TOKEN` also accepted).

---

## Commands

| Command | Usage | Description |
|---------|-------|-------------|
| `/mealie/find` | `/mealie/find [query]` | Search, show detail, then offer to add to shopping list or meal plan |
| `/mealie/weekly-menu` | `/mealie/weekly-menu [YYYY-MM-DD]` | Plan a week of dinners, show the table, wait for approval, then create entries |
| `/mealie/quick-recipe` | `/mealie/quick-recipe [description]` | Build a structured Mealie v3 recipe, show the draft, wait for approval |

Commands answer in French.

## Skills

Skills are loaded explicitly (`opencode/autoinvoke: false`) — invoke them or let
OpenCode pick them up when the topic matches.

| Skill | Covers |
|-------|--------|
| `recipe-creation` | Structured Mealie v3 recipe authoring: categories, tags, units, foods, ingredients, instructions |
| `foundations` | "Foundation recipe" paradigm — teaching the why behind a technique, not just the steps |
| `cooking-tips` | Scratch cooking and food science: braising, stock, sourdough, fermentation, rendering fat, cast iron |
| `meal-planning` | Weekly planning workflow against the Mealie meal plan API |

---

## MCP Server Tools (27)

### Recipes

| Tool | Description |
|------|-------------|
| `search_recipes` | Search by keyword with category/tag filters |
| `get_recipe` | Full recipe detail including ingredients, instructions, nutrition, notes |
| `get_recipe_ingredients` | Ingredient list only |
| `create_recipe_from_url` | Import from URL (Mealie scrapes automatically) |
| `create_recipe` | Create manually with full structured content |
| `update_recipe` | Update any fields on an existing recipe |
| `delete_recipe` | Permanently delete a recipe |

### Shopping Lists

| Tool | Description |
|------|-------------|
| `get_shopping_lists` | List all shopping lists |
| `get_shopping_list` | View list with all items (checked/unchecked) |
| `create_shopping_list` | Create new empty list |
| `delete_shopping_list` | Delete a list |
| `add_shopping_list_item` | Add item to a list |
| `remove_shopping_list_item` | Remove item from a list |
| `check_shopping_list_item` | Check or uncheck an item |
| `add_recipe_to_shopping_list` | Add all recipe ingredients to a list |

### Meal Plans

| Tool | Description |
|------|-------------|
| `get_meal_plans` | View plans for a date range |
| `create_meal_plan_entry` | Add entry (recipe or freeform) for a date |
| `delete_meal_plan_entry` | Remove an entry |
| `generate_random_meal_plan` | Auto-fill date range with random recipes |

### Organizers

| Tool | Description |
|------|-------------|
| `get_categories` | List all recipe categories |
| `get_tags` | List all tags |
| `get_cookbooks` | List all cookbooks |
| `get_cookbook_recipes` | Recipes in a specific cookbook |
| `create_category` | Create a new category |
| `create_tag` | Create a new tag |

### Foods & Units

| Tool | Description |
|------|-------------|
| `search_foods` | Search Mealie's foods database |
| `get_units` | List all measurement units |

---

## Requirements

- **Python 3.11+** (`pyproject.toml`); the launcher pins **3.13**
- **[uv](https://docs.astral.sh/uv/)** package manager
- **[Mealie](https://mealie.io/) v3.29+** — self-hosted recipe manager
- **[OpenCode](https://opencode.ai)** with MCP support

---

## Development

```sh
uv run pytest
```

### Project structure

```
mealie-mcp/
├── opencode/
│   ├── bin/mealie-mcp           # launcher: .env, pins Python 3.13, runs the server
│   ├── commands/mealie/         # find, weekly-menu, quick-recipe
│   ├── skills/                  # cooking-tips, foundations, meal-planning, recipe-creation
│   ├── opencode.json.example    # MCP + skills + permissions, to copy into your project
│   ├── README.md                # port summary
│   └── USAGE.md                 # full guide (French)
├── src/mealie_mcp/              # MCP server source
│   ├── server.py                # FastMCP app and tool registration
│   ├── client.py                # Async Mealie API client (httpx)
│   ├── config.py                # Pydantic settings (MEALIE_URL, MEALIE_API_KEY)
│   ├── formatting.py            # Markdown output formatters
│   └── tools/                   # recipes (7), shopping (8), mealplan (4), organizers (6), foods (2)
├── patch/
│   └── mealie-routes-v3.29.patch
├── tests/                       # pytest + respx
└── pyproject.toml
```

Upstream Claude Code packaging (`.claude-plugin/`, `agents/`, `hooks/`) is still
in the tree but unused by the OpenCode setup. It is documented in
[`CHANGELOG.md`](CHANGELOG.md).

## Mealie v3.29 route fix

`src/mealie_mcp/client.py` used pre-v3 paths that 404 on current Mealie:
`/api/groups/{shopping,mealplans,cookbooks}` → `/api/households/…` and
`/api/recipes/create-url` → `/api/recipes/create/url`. The diff lives in
[`patch/mealie-routes-v3.29.patch`](patch/mealie-routes-v3.29.patch); apply it
with `git apply` if you start from upstream.

## Credits

**Upstream — Thrivewell Partners**

- Repository: <https://github.com/thrivewell-partners/mealie-mcp>
- Author: Thrivewell Partners (<https://github.com/thrivewell-partners>)
- Licensed MIT, © 2026 Thrivewell Partners

Everything in `src/mealie_mcp/`, the 27 MCP tools, the skills, and the original
commands come from that project. The `.claude-plugin/`, `agents/`, and `hooks/`
directories are theirs too and are kept as-is.

**This fork — Nicolas OGIER** (<https://github.com/Opware2000/mealie-mcp>)

- OpenCode packaging: `opencode/` launcher, `opencode.json` template, ported
  commands and skills, French documentation
- Mealie v3.29 route fixes in `src/mealie_mcp/client.py`
  (`patch/mealie-routes-v3.29.patch`)

Both copyright notices are in [`LICENSE`](LICENSE). If you reuse this work,
keep them.

## License

MIT — see [`LICENSE`](LICENSE). Original work by
[Thrivewell Partners](https://github.com/thrivewell-partners/mealie-mcp).
