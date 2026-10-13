# ADR 0012 — Règles de composition validées côté serveur

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Le front empêche déjà d'ajouter deux fois la même espèce ou deux Méga-Évolutions. Mais n'importe qui peut appeler l'API directement (par exemple avec `curl`).

## Décision

Les règles sont **vérifiées dans Équipes**, qui fait foi : 1 à 6 Pokémon, **un seul exemplaire par espèce** (*Species Clause* ; une Méga ou une forme compte comme son espèce de base, Dracaufeu + Méga-Dracaufeu X est refusé), **une seule Méga-Évolution**. Les contrôles du front ne sont qu'un **confort d'affichage** (pastille ✓, Pokémon grisés). L'espèce de base est fournie par le Pokédex (`baseSpecies`), Équipes n'ayant pas à connaître l'organisation des formes dans PokéAPI.

## Conséquences

- ✅ Impossible de contourner les règles en appelant l'API directement (testé).
- ⚠️ Règles exprimées à deux endroits (front pour l'affichage, serveur pour la validation) ; seule la version serveur est garante.

## Alternatives écartées

- **Validation uniquement dans le front** : contournable.
