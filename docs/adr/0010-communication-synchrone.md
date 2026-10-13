# ADR 0010 — Communication synchrone REST entre services

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Équipes a besoin du Pokédex pour analyser une équipe et calculer un combat. Une architecture événementielle (RabbitMQ, Kafka) découplerait davantage les services.

## Décision

Les échanges internes sont des **appels REST synchrones directs** (Équipes → Pokédex), sans passer par la gateway, qui ne sert que le trafic entrant.

## Conséquences

- ✅ Simple à comprendre, à tester et à déboguer ; pas de composant d'infrastructure supplémentaire.
- ✅ Adapté à des opérations qui attendent une réponse immédiate (le résultat d'un combat).
- ⚠️ Si le Pokédex est indisponible, on ne peut plus créer d'équipe ni combattre (erreur 503 explicite) ; la lecture des équipes continue de fonctionner grâce aux instantanés ([ADR 0007](0007-instantanes-pokemon.md)).
- ⚠️ Évolution possible : un broker deviendrait pertinent pour des réactions différées (compter les victoires d'un dresseur, supprimer les équipes d'un compte supprimé).

## Alternatives écartées

- **RabbitMQ / Kafka** : écartés pour le délai et parce qu'aucun besoin actuel n'est réellement asynchrone.
- **Passer par la gateway pour les appels internes** : latence et couplage inutiles.
