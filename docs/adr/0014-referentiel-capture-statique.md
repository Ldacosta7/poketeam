# ADR 0014 — Référentiel statique pour le mode Capture

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Le mode Capture a besoin, pour chacune des 1 025 espèces, de son stade d'évolution, de sa chaîne d'évolution, de l'espèce dont elle évolue, de ses stats de base et de ses Méga-Évolutions. Via l'API de PokéAPI, il faudrait plus de 1 500 appels (une page d'espèce et une page de Pokémon par espèce, plus les chaînes) avant de pouvoir répartir les Pokémon en paliers.

## Décision

Ces données sont **figées dans un fichier** `pokedex/app/data/capture.json` (~120 Ko), **généré** par `pokedex/scripts/build_capture_data.py` à partir des **CSV officiels** que PokéAPI publie sur GitHub (les mêmes données que l'API). Le fichier est commité et chargé au démarrage du Pokédex. Les profils détaillés (faiblesses, sprites) continuent de venir de PokéAPI via la façade ([ADR 0006](0006-pokedex-acl-sans-etat.md)).

## Conséquences

- ✅ Paliers, rencontres et seuils d'évolution calculés instantanément, sans réseau.
- ✅ Résultats reproductibles et testables sur les vraies données (stades, branches d'Évoli, Méga de Dracaufeu…).
- ✅ Le Pokédex reste sans état : ce fichier est du code, pas des données modifiables.
- ⚠️ À régénérer (une commande) à la sortie de nouveaux Pokémon.
- ⚠️ Deux sources de données Pokémon coexistent (le référentiel et PokéAPI) ; elles sont alignées car issues du même projet.

## Alternatives écartées

- **Appels PokéAPI à la demande** : 1 500 appels au premier usage, plusieurs dizaines de secondes, et une charge inutile sur un service gratuit.
- **API GraphQL bêta de PokéAPI** : un seul appel, mais dépendance à une API en bêta.
- **Base de données dédiée** : une troisième base, contraire au sujet.
