# ADR 0016 — Les Pokémon capturés, seul droit d'utilisation

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Le mode Capture fait débloquer les Pokémon un par un. Pour que cette progression ait un sens,
elle doit s'appliquer à toute l'application : on ne compose une équipe classique qu'avec ses
Pokémon capturés. Il faut aussi décider du sort des équipes créées avant cette règle, et de
l'équipe utilisée en run.

## Décision

- **Création et modification d'équipe** : Équipes lit les Pokémon débloqués du dresseur
  (collection `captures`) et fait analyser l'équipe par `POST /capture/analyze`, qui renvoie la
  liste des Pokémon non débloqués. S'il y en a, la requête est refusée (**403**). Les formes
  alternatives suivent leur espèce ; une Méga doit être débloquée elle-même.
- **Équipes existantes** : elles restent lisibles, mais une équipe contenant un Pokémon que son
  dresseur n'a pas capturé est **inutilisable en combat** (versus, boss : **409**). Les listes
  d'équipes portent un champ `locked`, calculé en un seul appel (`POST /capture/locked`) pour
  toute la page ; le front grise l'équipe et désactive « Combattre ». En versus, l'équipe
  adverse est vérifiée avec les Pokémon débloqués de **son** dresseur.
- **Équipe de run** : composée au lancement de chaque run, puis figée (instantanés + summary
  stockés dans la run) jusqu'à la capture ou la défaite. Une évolution débloquée en cours de
  run est utilisable à la run suivante.
- **Pokédex du dresseur** : compteurs par génération calculés par le Pokédex
  (`/capture/state`), à partir du référentiel statique ([ADR 0014](0014-referentiel-capture-statique.md)).
- Une équipe compte désormais **de 1 à 6** Pokémon : en début de partie, on n'en possède qu'un.

## Conséquences

- ✅ Une seule règle (« est-ce débloqué ? »), écrite une fois dans le Pokédex et appliquée partout
  côté serveur : impossible de la contourner en appelant l'API directement.
- ✅ Aucune migration de données : le verrouillage des anciennes équipes est calculé à la lecture.
- ✅ Chaque combat de run n'appelle plus l'analyse : l'équipe figée garde son summary.
- ⚠️ Lister les équipes appelle maintenant le Pokédex. Si celui-ci ne répond pas, `locked` vaut
  `null` (liste lisible) et la vérification est refaite au moment du combat.
- ⚠️ Un nouveau dresseur ne peut rien créer avant d'avoir choisi son starter : le front l'oriente
  vers le mode Capture.

## Alternatives écartées

- **Supprimer ou modifier les anciennes équipes** : destructif, et irréversible pour le dresseur.
- **Vérifier uniquement dans le front** (silhouettes) : contournable par un simple appel à l'API.
- **Stocker un drapeau `locked` dans chaque équipe** : il deviendrait faux dès qu'un Pokémon est
  capturé ; le calculer à la lecture le garde toujours juste.
