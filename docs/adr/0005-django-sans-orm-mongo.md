# ADR 0005 — Django pour Dresseurs et Équipes, sans ORM côté MongoDB

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

L'équipe maîtrise Python. Django apporte l'ORM, les migrations et le hachage des mots de passe, mais son ORM est conçu pour les bases SQL ; les adaptateurs MongoDB sont abandonnés (djongo) ou très récents.

## Décision

- **Dresseurs** : Django + Django REST Framework + ORM, sur PostgreSQL. Seules les fonctions de hachage de `django.contrib.auth` sont utilisées, sur un modèle simple (pas d'admin, de sessions ni de permissions Django).
- **Équipes** : Django + DRF pour le routage et les vues, **`pymongo` directement** pour MongoDB (`DATABASES = {}`).
- L'authentification de DRF est désactivée dans les deux services : c'est la gateway qui l'assure ([ADR 0002](0002-api-gateway-jwt.md)).

## Conséquences

- ✅ Même framework, mêmes conventions dans les deux services Python métier.
- ✅ Pas de dépendance à un adaptateur MongoDB fragile.
- ⚠️ Côté Équipes, Django n'apporte « que » le routage et la sérialisation ; FastAPI aurait aussi convenu.

## Alternatives écartées

- **Garder NestJS** et réutiliser le code du monolithe : moins de réécriture, mais l'équipe est plus à l'aise en Python.
- **djongo / django-mongodb-backend** : risque de maintenance ou de maturité pour un projet de 3 jours.
