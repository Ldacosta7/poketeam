# ADR 0004 — Persistance polyglotte : PostgreSQL et MongoDB

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Deux services ont besoin de stocker des données de nature différente : des comptes (structure fixe, unicité de l'adresse mail) et des équipes (un agrégat avec 1 à 6 Pokémon imbriqués, lus et écrits d'un bloc).

## Décision

- **Dresseurs → PostgreSQL** : contraintes d'intégrité (mail unique), schéma stable, migrations Django.
- **Équipes → MongoDB** : une équipe est **un document** contenant ses membres et son summary ; aucune jointure nécessaire.

Chaque base appartient à **un seul service** (*database per service*). La clé étrangère `Team → User` du monolithe disparaît : Équipes ne stocke que `userId`.

## Conséquences

- ✅ Chaque modèle de données est adapté à son usage.
- ✅ Les services peuvent évoluer et être déployés indépendamment.
- ⚠️ Plus d'intégrité référentielle entre dresseurs et équipes : supprimer un compte ne supprime pas ses équipes (à gérer par un événement si besoin).
- ⚠️ Deux technologies de base à opérer.

## Alternatives écartées

- **Une seule base partagée** : anti-pattern microservices (couplage par le schéma).
- **MySQL comme dans le monolithe** : le driver demandait une compilation native, fragile sur Mac M1 ; PostgreSQL s'installe en une ligne (`psycopg[binary]`).
