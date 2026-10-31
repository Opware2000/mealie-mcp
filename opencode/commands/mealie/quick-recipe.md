---
description: Crée une recette dans Mealie à partir d'une description, avec ingrédients structurés et étapes.
---

Crée dans Mealie la recette : $ARGUMENTS

1. Si l'argument est vide ou trop vague, demande le nom, les ingrédients et les étapes — n'invente rien.
2. Consulte les référentiels avant d'écrire : `get_categories`, `get_tags`, `get_units`, `search_foods` (pour réutiliser les aliments et unités existants plutôt que d'en créer).
3. Construis un JSON Mealie v3 complet : `name`, `description`, `recipeYield`, `servings` (numérique), `prepTime`/`cookTime` en texte court, `recipeCategory`, `tags`, `recipeIngredient` (`quantity`, `unit`, `food`, `note`, `original`), `instructions` (une étape par entrée, phrases complètes), `tools`.
4. Montre le brouillon et **attends validation** avant `create_recipe`.
5. Après création, renvoie le slug et propose d'ajouter les ingrédients à la liste de courses.

Réponds en français. Ne crée ni catégorie ni tag en double : vérifie d'abord avec `get_categories` et `get_tags`.
