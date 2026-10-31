---
description: Planifie les dîners d'une semaine dans le planning Mealie, après validation explicite.
---

Planifie les dîners de la semaine qui commence à $ARGUMENTS (format AAAA-MM-JJ ; si l'argument est vide, prends le lundi suivant).

1. `get_meal_plans` sur les 7 jours pour voir ce qui est déjà posé.
2. `search_recipes` pour trouver des plats adaptés (varie : viande, poisson, végétarien, pâtes, soupe).
3. Propose un tableau lundi→dimanche avec titre et lien, et **attends validation**.
4. Après accord seulement : `create_meal_plan_entry` pour chaque jour (type `dinner`), en réutilisant les recettes existantes.
5. Demande ensuite s'il faut construire une liste de courses (`get_shopping_lists`, `create_shopping_list`, `add_recipe_to_shopping_list`).

Ne supprime aucune entrée de planning. Réponds en français, en une proposition compacte.
