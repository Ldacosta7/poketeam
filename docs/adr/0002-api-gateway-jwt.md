# ADR 0002 — API Gateway, authentification centralisée et services non exposés

- **Statut** : acceptée
- **Date** : 2026-10-13

## Contexte

Le front doit appeler trois services. Sans point d'entrée unique, il faudrait exposer chaque service, dupliquer la vérification du JWT et la configuration CORS dans chacun, et le front connaîtrait la topologie interne.

## Décision

Une **API Gateway** en Node/Express est le **seul conteneur exposé** (port 3000). Elle enchaîne quatre middlewares :
1. **CORS** pour le front (`http://localhost:4200`) ;
2. **suppression de tout en-tête `X-User-Id`** envoyé par le client ;
3. **vérification du JWT** (HS256) sauf sur les routes publiques (connexion, inscription, lecture du Pokédex), puis ajout de `X-User-Id` = identifiant du token ;
4. **routage** par préfixe vers les services, avec réécriture des anciennes routes `/pokeapi/*`.

Les services font confiance à `X-User-Id`, ce qui n'est acceptable **que parce qu'ils ne sont pas joignables de l'extérieur**. L'**authentification** (qui es-tu ?) est à la gateway ; l'**autorisation** (as-tu le droit sur cette équipe ?) reste dans chaque service, seul à connaître ses données.

## Conséquences

- ✅ Un seul endroit pour la sécurité d'entrée ; les services restent simples.
- ✅ Testé : sans token, avec un token invalide, ou en usurpant `X-User-Id`, l'accès est refusé.
- ⚠️ La gateway est un point unique de défaillance (acceptable ici, réplicable en production).
- ⚠️ Si un service était exposé par erreur, `X-User-Id` permettrait d'usurper une identité : c'est pourquoi les ports internes sont retirés du `docker-compose.yml`.

## Alternatives écartées

- **Chaque service vérifie le JWT** : duplication du secret et du code dans 3 services, dans 2 langages.
- **Appel à Dresseurs à chaque requête** pour valider le token : couplage synchrone et latence inutiles, le JWT étant auto-porteur.
- **Nginx comme gateway** : très performant, mais la vérification JWT y est moins simple ; Express suffit et reste lisible.
