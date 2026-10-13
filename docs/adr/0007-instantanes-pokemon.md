# ADR 0007 — Instantané des Pokémon dans les équipes plutôt qu'une référence

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Une équipe pourrait ne stocker que des identifiants (`poke1` … `poke6`, comme le monolithe) et redemander les données au Pokédex à chaque affichage. Le monolithe faisait ainsi 6 appels HTTP par carte d'équipe, et les Pokémon s'affichaient dans le désordre.

## Décision

À la création ou modification d'une équipe, Équipes enregistre un **instantané** de chaque membre (nom, sprite, types, stats, faiblesses, résistances) et le **summary** de l'équipe. Les champs `poke1` … `poke6` restent renvoyés pour la compatibilité avec le front.

## Conséquences

- ✅ Affichage d'une équipe : 0 appel au Pokédex ; ordre des membres garanti.
- ✅ Les équipes restent lisibles et jouables même si le Pokédex ou PokéAPI sont indisponibles.
- ⚠️ Données dupliquées : si PokéAPI corrigeait une stat, les anciennes équipes garderaient l'ancienne valeur. Acceptable : les données Pokémon ne changent quasiment jamais, et modifier l'équipe rafraîchit l'instantané.

## Alternatives écartées

- **Référence seule** : données toujours fraîches, mais couplage fort à la disponibilité du Pokédex et 6 appels par équipe affichée.
