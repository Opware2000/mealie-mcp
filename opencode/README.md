# OpenCode port

This directory makes the Mealie MCP server usable from
[OpenCode](https://opencode.ai) instead of Claude Code: the same server, the
same skills (minus the Johnson family's dietary rules), and three commands.

```
opencode/
├── bin/mealie-mcp            # launcher: reads .env, pins Python 3.13, runs the server
├── commands/mealie/          # /mealie/find, /mealie/weekly-menu, /mealie/quick-recipe
├── skills/                   # cooking-tips, foundations, meal-planning, recipe-creation
└── opencode.json.example     # MCP + skills + permissions, to copy into your project
```

## Install

```sh
git clone https://github.com/<you>/mealie-mcp
cd mealie-mcp
uv sync --python 3.13
```

Create `.env` next to this README:

```sh
MEALIE_URL=https://mealie.example.com
MEALIE_API_KEY=<token from Mealie → profile → API tokens>
```

Then copy `opencode.json.example` into your project as `opencode.json`, replace
the absolute path with your clone, and copy `commands/` and `skills/` into the
project's `.opencode/` directory:

```sh
cp -r opencode/commands opencode/skills <project>/.opencode/
```

Check the connection:

```sh
opencode mcp list     # ✓ mealie connected
```

The project config `skills: ["./opencode/skills"]` also works if you prefer to
keep the skills in the clone instead of copying them.

## Differences from the Claude Code plugin

| Upstream | Here |
| --- | --- |
| `.claude-plugin/`, `agents/`, `hooks/` | dropped — OpenCode has no plugin hooks for MCP tool calls |
| 5 skills, including `dietary-management` | 4 skills — the family-specific one is removed, the rest are neutralised |
| `mcp__plugin_mealie_mealie__<tool>` in prose | `<tool>` — OpenCode exposes them as `tools.mealie.<tool>` |
| Tool allow-lists in command frontmatter | dropped — OpenCode commands only carry `description`/`agent`/`model` |

The dietary guardrail (no pork, no seed oils, serves 6) was one family's rules,
not the server's behaviour, so it is not ported. Add a `permissions` rule or a
skill of your own if you want a similar check.

## Mealie v3.29 fix

`src/mealie_mcp/client.py` used pre-v3 paths, which 404 on current Mealie:
`/api/groups/{shopping,mealplans,cookbooks}` → `/api/households/…` and
`/api/recipes/create-url` → `/api/recipes/create/url`. The patch is at
`patch/mealie-routes-v3.29.patch`; apply it with `git apply` if you start from
upstream.
