# ADR 0006 — Pokédex sans état, façade PokéAPI (Anti-Corruption Layer) et cache

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

PokéAPI renvoie des documents très volumineux, avec des concepts propres (espèces, variétés, « pokemon-form » dont les identifiants chevauchent ceux des Pokémon). Le monolithe appelait PokéAPI ~150 fois successivement pour lister une génération.

## Décision

- Le Pokédex est le **seul** composant qui parle à PokéAPI. `pokeapi.py` traduit tout en **profils simples** (`id`, `name`, `types`, `stats`, `weaknesses`, `resistances`, `isMega`, `isForm`, `baseSpecies`). Le format de PokéAPI ne sort jamais de ce fichier.
- Un **index** de tous les Pokémon (formes de base, Méga, variétés, Arceus et Silvallié par type) est construit à partir des **18 pages de types et des 9 générations**, sans aucune requête par Pokémon.
- Les réponses sont **mises en cache en mémoire**. Le service reste **sans état** : un redémarrage vide le cache, sans perte de données.

## Conséquences

- ✅ Liste d'une génération : 1,37 s au premier appel, **0,025 s** ensuite.
- ✅ Si PokéAPI change, un seul fichier est à adapter.
- ✅ Les formes d'Arceus (simples « formes » dans PokéAPI) et les variétés comme Giratina-Origine (vrais Pokémon) sont présentées de la même façon au reste du système.
- ⚠️ Dépendance à la disponibilité de PokéAPI quand le cache est vide.
- ⚠️ Cache non partagé si le Pokédex est répliqué (Redis serait la suite logique).

## Alternatives écartées

- **Le front appelle PokéAPI directement** : chaque client dépendrait du format externe, sans cache commun.
- **Base de données locale pré-remplie** : aurait fait une 3ᵉ base, contraire au sujet.
