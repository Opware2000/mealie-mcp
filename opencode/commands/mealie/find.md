---
description: Cherche des recettes dans Mealie, affiche le détail et propose d'en ajouter une aux courses ou au planning.
---

Cherche dans la bibliothèque Mealie de l'utilisateur : $ARGUMENTS

1. `search_recipes` avec la requête donnée (et les filtres `categories` / `tags` si l'argument en contient).
2. Présente les résultats en liste courte : titre, catégories, tags, temps total.
3. Si l'utilisateur en choisit une, `get_recipe` sur son slug et résume : ingrédients (quantité, unité, aliment), étapes, portions.
4. Propose ensuite explicitement, sans le faire d'office : ajouter les ingrédients à une liste de courses (`get_shopping_lists` puis `add_recipe_to_shopping_list`) ou poser la recette dans le planning (`create_meal_plan_entry`).

Réponds en français, sans inventer de recette : si la recherche ne rend rien, dis-le et propose une variante de termes.
