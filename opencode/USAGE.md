# Guide d'utilisation — mealie-mcp sous OpenCode

Tout ce qui suit concerne **ce fork** (`Opware2000/mealie-mcp`) : le serveur MCP
d'origine (Thrivewell Partners) porté pour OpenCode, avec les routes corrigées
pour Mealie v3.29.

---

## 1. Ce que contient le dépôt

| Chemin | Rôle |
| --- | --- |
| `src/mealie_mcp/` | serveur MCP (FastMCP) : client HTTP Mealie + 27 outils |
| `opencode/bin/mealie-mcp` | lanceur pour OpenCode (lit `.env`, épingle Python 3.13) |
| `opencode/commands/mealie/` | 3 commandes : `/mealie/find`, `/mealie/weekly-menu`, `/mealie/quick-recipe` |
| `opencode/skills/` | 4 skills : `recipe-creation`, `meal-planning`, `cooking-tips`, `foundations` |
| `opencode/opencode.json.example` | configuration MCP + skills + permissions à adapter |
| `patch/mealie-routes-v3.29.patch` | correction des routes Mealie (déjà appliquée ici) |

Ce qui n'a **pas** été porté depuis l'amont, et qui a été supprimé ici : le
plugin Claude Code (`.claude-plugin/`, `agents/`, `hooks/`, `.mcp.json`), le
skill `dietary-management` et les règles familiales (pas de porc, pas d'huiles
de graines, portions pour 6). Ces fichiers restent dans l'historique git et dans
le dépôt amont.

---

## 2. Prérequis

- macOS ou Linux, `git`
- [`uv`](https://docs.astral.sh/uv/) (`brew install uv`)
- **Python 3.13** — impératif : certains paquets (`rpds-py` via `pyo3`) ne
  compilent pas encore en 3.14/3.15
- une instance Mealie ≥ **3.21** (testé en 3.29)
- un token API Mealie : *Profil → API Tokens → Generate*

---

## 3. Installation

```sh
git clone https://github.com/Opware2000/mealie-mcp ~/workspaces/divers/mealie-mcp
cd ~/workspaces/divers/mealie-mcp
uv sync --python 3.13
```

Créer le fichier d'identifiants **à côté de `opencode/bin/`**, jamais dans le
dépôt :

```sh
cd ~/workspaces/divers/mealie-mcp/opencode
printf 'MEALIE_URL=https://mealie.example.com\nMEALIE_API_KEY=<token>\n' > .env
chmod 600 .env
```

Le lanceur accepte aussi `MEALIE_BASE_URL` / `MEALIE_API_TOKEN` (les noms
utilisés par l'app de recettes), et priorise toujours les variables déjà
présentes dans l'environnement.

Vérification rapide, sans OpenCode :

```sh
cd ~/workspaces/divers/mealie-mcp
set -a; . opencode/.env; set +a
uv run --python 3.13 python -c "
import asyncio
from mealie_mcp.client import MealieClient
async def main():
    c = MealieClient()
    print(await c.search_recipes('tarte', per_page=3))
    await c._client.aclose()
asyncio.run(main())"
```

---

## 4. Branchement dans OpenCode

Le serveur se configure **projet par projet** — ne le mettez pas en global si
vous ne le voulez pas dans toutes vos sessions.

### 4.1 Déclarer le serveur

Créer `opencode.json` à la racine du projet concerné :

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "mealie": {
        "type": "local",
        "command": ["/Users/<vous>/workspaces/divers/mealie-mcp/opencode/bin/mealie-mcp"],
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

> **Le chemin doit être absolu.** Un chemin relatif échoue avec
> `PermissionDenied: ChildProcess.spawn` : OpenCode ne résout pas les commandes
> MCP relativement au workspace.

### 4.2 Ajouter commandes et skills au projet

```sh
mkdir -p .opencode
cp -r ~/workspaces/divers/mealie-mcp/opencode/commands .opencode/
cp -r ~/workspaces/divers/mealie-mcp/opencode/skills   .opencode/
```

Les commandes deviennent `/mealie/find` etc. Les skills sont dans
`.opencode/skills/<id>/SKILL.md` (format V2).

### 4.3 Vérifier

```sh
opencode mcp list
# ✓ mealie  connected
```

Puis, dans une session, une demande du type « qu'est-ce que j'ai comme tarte ? »
doit déclencher `tools.mealie.search_recipes`. Les outils sont nommés
`mealie_<outil>` (`mealie_search_recipes`) et regroupés sous
`tools.mealie.*` quand Code Mode est actif.

---

## 5. Les 27 outils

### Recettes (7)
| Outil | Usage |
| --- | --- |
| `search_recipes` | recherche par mot-clé, filtres `categories` / `tags`, pagination |
| `get_recipe` | fiche complète (ingrédients, étapes, nutrition, notes) |
| `get_recipe_ingredients` | ingrédients seuls |
| `create_recipe_from_url` | import via le scraper Mealie |
| `create_recipe` | création structurée (création puis PATCH en interne) |
| `update_recipe` | mise à jour de champs existants |
| `delete_recipe` | suppression **définitivement** — bloqué par défaut (voir §7) |

### Listes de courses (8)
`get_shopping_lists`, `get_shopping_list`, `create_shopping_list`,
`delete_shopping_list`, `add_shopping_list_item`, `remove_shopping_list_item`,
`check_shopping_list_item`, `add_recipe_to_shopping_list`

### Planning (4)
`get_meal_plans`, `create_meal_plan_entry`, `delete_meal_plan_entry`,
`generate_random_meal_plan`

### Organisateurs (6)
`get_categories`, `get_tags`, `get_cookbooks`, `get_cookbook_recipes`,
`create_category`, `create_tag`

### Référentiels (2)
`search_foods`, `get_units`

> Les ingrédients doivent utiliser des unités existantes dans Mealie, sinon la
> création échoue. `search_foods` et `get_units` servent à les retrouver avant
> d'écrire.

---

## 6. Commandes

| Commande | Exemple | Effet |
| --- | --- | --- |
| `/mealie/find` | `/mealie/find tarte aux pommes` | cherche, résume, propose courses ou planning |
| `/mealie/weekly-menu` | `/mealie/weekly-menu 2026-11-02` | propose une semaine de dîners, **après validation** crée les entrées |
| `/mealie/quick-recipe` | `/mealie/quick-recipe gratin de courgettes` | brouillon structuré, création après accord |

Aucune commande ne supprime d'entrée de planning ni de recette.

---

## 7. Skills et permissions

Les 4 skills sont livrés avec `metadata.opencode/autoinvoke: false` : elles ne
sont **pas** proposées automatiquement au modèle, pour ne pas polluer toutes les
sessions. Deux façons de s'en servir :

- mention explicite dans le prompt : `@recipe-creation` (ou `@meal-planning`) ;
- chargement par l'agent via l'outil `skill` avec l'id.

Pour les rendre auto-actives dans un projet donné, retirer du frontmatter :

```yaml
metadata:
  opencode/autoinvoke: false
```

Les suppressions sont bloquées par les règles `permissions` du `opencode.json`
(§4.1). Pour autoriser, supprimer la ligne correspondante ; les règles sont
évaluées dans l'ordre, la dernière qui correspond l'emporte.

---

## 8. Dépannage

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `✗ mealie failed: PermissionDenied: ChildProcess.spawn` | chemin relatif dans `command` | mettre le chemin absolu du lanceur |
| `Failed to build rpds-py` / `pyo3` | Python 3.14+ | `uv sync --python 3.13` (le lanceur l'épingle déjà) |
| `Error fetching shopping lists: Expecting value: line 1 column 1` | routes pré-v3 (`/api/groups/...`) | appliquer `patch/mealie-routes-v3.29.patch` |
| `✓ connected` mais 0 résultat | token invalide ou mauvais `MEALIE_URL` | tester `curl -H "Authorization: Bearer $TOKEN" $MEALIE/api/users/self` |
| Liste de courses vide dans Mealie | l'instance n'a aucun article | normal : `get_shopping_lists` renvoie 0 item |
| Le modèle n'appelle jamais les outils | skills en `autoinvoke: false` et demande vague | nommer l'outil ou charger `@recipe-creation` |

Journal serveur : `~/.local/share/opencode/log/opencode.log` (`grep mealie`).

---

## 9. Mettre à jour depuis l'amont

```sh
git remote add upstream https://github.com/thrivewell-partners/mealie-mcp
git fetch upstream
git rebase upstream/master          # puis réappliquer le patch si conflit
git apply patch/mealie-routes-v3.29.patch   # si le rebase l'a perdu
```

Le patch ne concerne que `src/mealie_mcp/client.py` : `api/groups/*` →
`api/households/*` et `api/recipes/create-url` → `api/recipes/create/url`.

---

## 10. Limites connues

- **Pas de hook** : l'amont bloque les ingrédients indésirables via un hook
  Claude `PreToolUse` ; OpenCode n'a pas d'équivalent direct. Le contrôle se fait
  donc par les permissions (§7) ou par une consigne dans le prompt.
- **Pas d'agents** : `recipe-advisor`, `meal-planner`, `cooking-teacher`
  (format Claude) ne sont pas portés.
- **Suivi de l'amont** : ce fork est un portage ponctuel, pas un miroir ; les
  évolutions de Thrivewell ne sont pas suivies automatiquement.
- **Pas de gestion des comptes** : le serveur utilise le token fourni, avec ses
  droits (y compris écriture).
