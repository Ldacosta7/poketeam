# ADR 0013 — Secrets et mots de passe

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Le monolithe stockait les mots de passe en clair et le secret JWT était écrit dans le code source.

## Décision

- Mots de passe **hachés** avec PBKDF2-SHA256 (870 000 itérations, sel unique) via `make_password` / `check_password` de Django.
- Secrets (`JWT_SECRET`, `DJANGO_SECRET_KEY`) dans un fichier **`.env`** ignoré par Git, lu par Docker Compose. Un `.env.example` documente les variables attendues.
- `JWT_SECRET` est le seul secret partagé : Dresseurs signe, la gateway vérifie. Durée de validité du token : 6 heures.

## Conséquences

- ✅ Une fuite de la base ne révèle pas les mots de passe.
- ✅ Aucun secret dans l'historique Git.
- ⚠️ Régénérer `JWT_SECRET` déconnecte tous les utilisateurs (les comptes ne sont pas affectés).
- ⚠️ En production, un gestionnaire de secrets (Vault, secrets Docker/Kubernetes) remplacerait le fichier `.env`.

## Alternatives écartées

- **bcrypt** : équivalent en sécurité ; PBKDF2 est fourni par Django sans dépendance supplémentaire.
