# ADR 0008 — Source unique de vérité des règles Pokémon

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Le summary d'une équipe (+1 par faiblesse et par résistance de chaque membre) était d'abord calculé dans Équipes, alors que le Pokédex en avait aussi besoin pour l'analyse en direct pendant la création. La même règle existait donc en deux exemplaires, dans deux services.

## Décision

Le Pokédex expose `POST /teams/analyze` : à partir d'une liste d'identifiants, il renvoie les profils des membres, le summary, la couverture des 18 types et les suggestions. **Équipes appelle cette route** à la création ou modification d'une équipe et stocke le résultat ; il ne contient plus aucune règle de type Pokémon. Il garde en revanche les règles de **composition d'équipe** (1 à 6 membres, une espèce une fois, une Méga au plus), qui relèvent de son domaine.

## Conséquences

- ✅ Une règle de calcul n'existe qu'à un endroit.
- ✅ Un seul appel au Pokédex par création d'équipe, au lieu de 6.
- ✅ L'analyse affichée pendant la création et le summary enregistré sont forcément identiques.

## Alternatives écartées

- **Garder le calcul dans Équipes** : duplication et risque de divergence entre l'analyse affichée et l'équipe enregistrée.
