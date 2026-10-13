# ADR 0001 — Migration progressive du monolithe (Strangler Fig)

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

L'application existante est un monolithe NestJS (modules `auth`, `users`, `teams`, `pokeapi`) avec une base MySQL, appelé par un front Angular sur `http://localhost:3000`. Le sujet impose une architecture microservices. Réécrire le front en même temps que le backend multiplierait les risques sur un délai de 3 jours.

## Décision

On applique le pattern **Strangler Fig** : une API Gateway prend la place du monolithe sur le **même port (3000)** et expose **les mêmes routes** (`/auth/login`, `/auth/profile`, `/users`, `/teams`, `/pokeapi/...`). Derrière, chaque module du monolithe devient un service : `auth` + `users` → Dresseurs, `teams` → Équipes, `pokeapi` → Pokédex.

## Conséquences

- ✅ Le front a continué de fonctionner pendant toute la migration ; seuls les formats de `/pokeapi/*` ont été nettoyés (tableaux imbriqués remplacés par des objets).
- ✅ Chaque service a pu être construit et testé indépendamment, l'un après l'autre.
- ✅ Les failles du monolithe ont été corrigées au passage (voir [architecture](../architecture.md)).
- ⚠️ La gateway porte une contrainte de compatibilité : changer une route publique impose de modifier aussi le front.

## Alternatives écartées

- **Réécriture complète front + back** : trop risquée pour le délai.
- **Nouvelles routes `/api/...`** : plus « propre », mais obligeait à modifier tous les appels du front.
