# Holo Cote

Classeur de cartes Pokémon : scan, centrage, état, collection (brutes et gradées), ventes et cotes Cardmarket, avec courbes de prix.

En ligne : https://ethrniity.github.io/Pokemon/ (installable sur le téléphone : menu du navigateur › « Ajouter à l’écran d’accueil »).

## Ce que fait l’appli

| Écran | Contenu |
|---|---|
| **Récap** | Valeur totale (brutes + gradées), prix d’achat, plus-value latente, courbe de la valeur, plus belles cartes, ventes, répartition par langue, cartes suivies |
| **Chercher** | Recherche dans toutes les cartes FR, EN et JP : sans accents, tolère les fautes, `4/102` pour le numéro imprimé, mots du nom de série, `fr` / `en` / `jp` |
| **Fiche** | Cote Cardmarket (tendance, moyennes 1/7/30 j, plus bas, reverse), TCGplayer, courbe de prix, liens Cardmarket et ventes eBay, ajout, suivi |
| **Scanner** | 1. photo dans un cadre (ou depuis la galerie) · 2. centrage gauche/droite et haut/bas, recto et verso, lignes réglables avec loupe · 3. coins, bords, surface et note PSA estimée · 4. lecture du numéro imprimé et choix de la carte |
| **Collection** | Filtres (brutes, gradées, langue), tri (valeur, plus-value, série…), modification, vente totale ou partielle |
| **Ventes** | Chiffre de ventes, frais, bénéfice net, détail par mois |
| **Réglages** | Coefficients de valeur selon l’état, export JSON et CSV, compte |

## Valeurs

- **Carte brute** : cote Cardmarket du jour (colonne « reverse » pour une reverse) × coefficient de l’état (NM 100 %, EX 80 %, GD 65 %, LP 50 %, PL 35 %, PO 20 %, modifiables), ou la valeur saisie à la main.
- **Carte gradée** : la valeur saisie (lien vers les ventes réelles eBay pour la même note).
- **Bénéfice d’une vente** : prix de vente × quantité − frais − prix d’achat × quantité.

## Sources et données

- Cartes et cotes : [TCGdex](https://tcgdex.dev), cotes Cardmarket en euros (mises à jour chaque jour) et TCGplayer en dollars.
- Base : Supabase, projet « Séance musculation », schéma `pokemon` (séparé de Muscu et Coût Réel). Tables `cartes`, `suivis`, `prix`, `valeurs`, `reglages` ; vue `ma_collection` qui calcule la valeur.
- Les cotes sont relevées **par le serveur** (extension `http` de Postgres) : chaque nuit à 4 h 35 (tâche `pokemon-releve-prix`, pg_cron) pour toutes les cartes possédées ou suivies, et à l’ajout d’une carte. L’appli ne peut pas écrire de cote elle-même.
- Au premier relevé d’une carte, trois points « estimés » (moyennes 30, 7 et 1 jours) amorcent la courbe ; ils sont en pointillés.

## Limites connues

- Le centrage se mesure sur les cartes avec bordure ; les full art n’en ont pas. Coins, bords et surface sont évalués par toi : la note est une estimation, pas une gradation.
- La lecture automatique repose sur le numéro imprimé (`025/198`). Les numéros avec lettres (`TG05/TG30`, promos) se cherchent à la main.
- Les cartes japonaises ont leur nom en japonais : cherche-les par numéro (`25/165 jp`).
- La cote Cardmarket mélange les états et, pour une même impression, les langues FR et EN.
