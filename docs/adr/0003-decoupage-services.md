# ADR 0003 — Découpage en trois microservices

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Le sujet impose 3 microservices et 2 bases. Le découpage doit suivre des frontières métier stables, pas des couches techniques.

## Décision

Trois contextes métier, alignés sur les modules du monolithe :
- **Dresseurs** : l'identité (comptes, connexion).
- **Équipes** : ce que possède le dresseur (ses équipes, qui peut les voir, les combats où elles sont engagées).
- **Pokédex** : la connaissance du domaine Pokémon (données, types, règles de calcul), **sans état et sans notion d'utilisateur**.

## Conséquences

- ✅ Chaque service a une raison unique de changer : nouvelle règle de type → Pokédex ; nouvelle règle de propriété → Équipes.
- ✅ Le Pokédex, sans état, est réplicable à volonté.
- ⚠️ Équipes dépend du Pokédex pour créer une équipe ou lancer un combat (voir [ADR 0010](0010-communication-synchrone.md)).

## Alternatives écartées

- **Un service « Combat » séparé** : aurait fait 4 services ; les combats sont des calculs (Pokédex) déclenchés sur des équipes (Équipes), sans données propres.
- **Fusionner Dresseurs et Équipes** : une seule base, mais mélange identité et contenu.
