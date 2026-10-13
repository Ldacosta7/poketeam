# ADR 0009 — Formule d'affrontement et place du calcul

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Il faut désigner un vainqueur entre deux équipes à partir de leurs stats de base et de leurs types. Compter les faiblesses ne suffit pas : une faiblesse au Sol ne compte que si l'adversaire possède du Sol.

## Décision

1. Pour chaque Pokémon, le multiplicateur de chacun des 18 types attaquants est le **produit** des effets sur ses types (Eau/Sol face à Électrik : 2 × 0 = immunité).
2. **Score défensif** = somme, sur les types offensifs de l'adversaire, de `résistances − faiblesses`.
3. **Avantage** = score défensif de l'équipe − score défensif de l'adversaire.
4. **Score** = total des stats × `clamp(1 + 0,1 × avantage, 0,5, 1,5)`.

Le calcul vit dans le **Pokédex** (connaissance du domaine) ; le point d'entrée est **Équipes** (seul à savoir si l'utilisateur peut voir ces équipes). Le résultat n'est **pas stocké** : il se recalcule en quelques millisecondes à partir des summaries.

## Conséquences

- ✅ Résultat cohérent avec le jeu (vérifié : Tortank bat Dracaufeu malgré des stats inférieures).
- ✅ Le détail du calcul est renvoyé, ce qui permet au front de l'expliquer.
- ✅ Le front ne peut pas tricher en envoyant de faux summaries.
- ⚠️ Formule déterministe : un même combat donne toujours le même résultat.
- ⚠️ Le coefficient 0,1 et les bornes sont des choix d'équilibrage ; sans borne, un avantage de ±15 écraserait les stats.

## Alternatives écartées

- **Simulation tour par tour avec attaques** : beaucoup plus complexe, hors délai.
- **Calcul dans le front** : falsifiable et dupliqué.
