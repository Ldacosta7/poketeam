# ADR 0015 — Progression du mode Capture : stockée par Équipes, décidée par le serveur

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Chaque dresseur a une progression : Pokémon débloqués, victoires par espèce (qui font évoluer), run en cours, choix d'évolution en attente. Il faut décider où la stocker, et empêcher la triche : le front ne doit pas pouvoir dire « j'ai gagné, débloque-moi Mewtwo ».

## Décision

- La progression est stockée par **Équipes**, dans une collection MongoDB `captures` (un document par dresseur). Équipes gère déjà « ce que possède le dresseur » ; la validation d'une équipe de capture (« tous ces Pokémon sont-ils débloqués ? ») se fait ainsi sans appel à un autre service de données.
- Les **règles** (paliers, tirage des adversaires, seuils 10 / 30 / 50, branches) sont dans le **Pokédex**, qui reçoit la progression à chaque appel et reste sans état.
- **Tout passe par le serveur** : les 3 adversaires sont tirés et enregistrés par le serveur ; on ne peut combattre que l'un d'eux ; le combat est calculé par le Pokédex ; on ne peut capturer qu'un Pokémon battu pendant la run ; une évolution n'est accordée que si le seuil est atteint.
- **Mises à jour atomiques** : victoires avec `$inc`, déblocages avec `$addToSet`, et passage à la manche suivante **conditionné au numéro de manche** (`find_one_and_update`). Un double clic ou deux requêtes simultanées ne comptent pas deux fois le même combat.
- Le Pokémon sauvage **s'adapte à la taille de l'équipe** (stats et poids des types multipliés par le nombre de membres) : on compare la moyenne de l'équipe au Pokémon, quelle que soit sa taille.

## Conséquences

- ✅ Impossible de tricher via l'API ; testé (adversaire non proposé, capture d'un Pokémon non battu, évolution non disponible).
- ✅ Aucun nouveau service ni nouvelle base.
- ⚠️ Équipes porte deux responsabilités proches (équipes libres et progression) ; à séparer en un service « Collection » si le mode grandissait.
- ⚠️ Chaque combat de capture fait 3 appels au Pokédex (analyse, combat, état) ; acceptable, le Pokédex répondant depuis son cache.

## Alternatives écartées

- **Progression dans Dresseurs (PostgreSQL)** : la progression est liée au joueur, mais chaque création d'équipe imposerait un appel synchrone Équipes → Dresseurs (nouveau couplage, et plus de création d'équipe si Dresseurs tombe).
- **Progression dans le navigateur** : falsifiable.
