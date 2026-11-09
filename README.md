# Holo Cote

Classeur de cartes Pokémon : scan, centrage, état, collection (brutes et gradées), ventes et cotes Cardmarket, avec courbes de prix.

En ligne : https://ethrniity.github.io/Pokemon/ (installable sur le téléphone : menu du navigateur › « Ajouter à l’écran d’accueil »).

## Ce que fait l’appli

| Écran | Contenu |
|---|---|
| **Récap** | Valeur totale (brutes + gradées), prix d’achat, plus-value latente, courbe de la valeur, plus belles cartes, ventes, répartition par langue, cartes suivies |
| **Chercher** | Recherche dans toutes les cartes FR, EN et JP : sans accents, tolère les fautes, `4/102` pour le numéro imprimé, mots du nom de série, `fr` / `en` / `jp`. Les noms de Pokémon marchent dans toutes les langues (« artikodin » trouve aussi Articuno et フリーザー), les séries japonaises anciennes répondent à leur nom français/anglais (« fossile »), et le n° du Pokédex imprimé sur les anciennes japonaises est reconnu (« artikodin jp fossil 144 ») |
| **Fiche** | Cote Cardmarket (tendance, moyennes 1/7/30 j, plus bas, reverse), TCGplayer, courbe de prix, liens Cardmarket et ventes eBay, ajout, suivi |
| **Scanner** | 1. photo dans un cadre : toucher pour la mise au point (Android/Chrome), zoom, photo pleine résolution, alerte si floue ; ou l’appareil photo du téléphone (le plus fiable sur iPhone) · 2. centrage gauche/droite et haut/bas, recto et verso, lignes réglables avec loupe · 3. coins, bords, surface et note PSA estimée · 4. lecture du numéro imprimé et choix de la carte |
| **Pépites** | Sélection de la semaine (chaque dimanche) : 15 cartes notées sur 100, parmi ~1 000 dragons, légendaires et Pokémon iconiques en raretés recherchées, dans le budget réglé (80–250 € par défaut) |
| **Collection** | Filtres (brutes, gradées, langue), tri (valeur, plus-value, série…), modification, vente totale ou partielle |
| **Ventes** | Chiffre de ventes, frais, bénéfice net, détail par mois |
| **Réglages** | Coefficients de valeur selon l’état, export JSON et CSV, compte |

## Valeurs

- **Carte brute** : cote Cardmarket du jour (colonne « reverse » pour une reverse) × coefficient de l’état (NM 100 %, EX 80 %, GD 65 %, LP 50 %, PL 35 %, PO 20 %, modifiables), ou la valeur saisie à la main.
- **Carte gradée** : la valeur saisie (lien vers les ventes réelles eBay pour la même note).
- **Note PSA estimée** (cartes brutes) : issue du scan (centrage + coins, bords, surface), modifiable, plafonnée par l’état (NM ≤ 10, EX ≤ 6, GD ≤ 4, LP ≤ 3, PL ≤ 2, PO ≤ 1). Sa valeur = médiane des ventes réelles à cette note ; sans vente à la note exacte, la note voisine (inférieure d’abord) est utilisée et signalée.
- **Cartes sans cote Cardmarket** (japonaises anciennes notamment) : cote brute TCGplayer quasi neuve × coefficient d’état.
- **Bénéfice d’une vente** : prix de vente × quantité − frais − prix d’achat × quantité.

## Sources et données

- Cartes et cotes : [TCGdex](https://tcgdex.dev), cotes Cardmarket en euros (mises à jour chaque jour) et TCGplayer en dollars.
- Base : Supabase, projet « Séance musculation », schéma `pokemon` (séparé de Muscu et Coût Réel). Tables `cartes`, `suivis`, `prix`, `valeurs`, `reglages` ; vue `ma_collection` qui calcule la valeur.
- Les cotes sont relevées **par le serveur** (extension `http` de Postgres) : chaque nuit à 4 h 35 (tâche `pokemon-releve-prix`, pg_cron) pour toutes les cartes possédées ou suivies, et à l’ajout d’une carte. L’appli ne peut pas écrire de cote elle-même.
- Au premier relevé d’une carte, trois points « estimés » (moyennes 30, 7 et 1 jours) amorcent la courbe ; ils sont en pointillés.

### Prix gradés (PokemonPriceTracker)

- Clé API (formule gratuite, usage personnel) rangée dans le Vault Supabase (`pokemonpricetracker_cle`), jamais dans l’appli.
- `pokemon._maj_gradees` : identifiant TCGplayer fourni par TCGdex → `GET /api/v2/cards?tcgPlayerId=…&includeEbay=true` (2 crédits) → médianes `ebay.salesByGrade.psaN`, prix brut `prices.market`, convertis en euros (taux BCE, `pokemon.taux`). Résultats dans `pokemon.prix_gradees`.
- Relevé de nuit (`pokemon-releve-gradees`, 4 h 50) : cartes possédées et suivies tous les 3 jours, 40 au plus, plafond 95 crédits / jour (`pokemon.ppt_conso`). À l’ajout d’une carte et depuis la fiche : `pokemon.rafraichir_gradees`.
- Les cartes françaises sont cotées sur leur version anglaise (les ventes gradées FR sont rares) ; les japonaises sur leur version japonaise.

### Analyse « vaut-elle le coup » (`pokemon._analyser`, RPC `pokemon.analyser`)

Score sur 100 : désirabilité 25 (légendaire, dragon, icône), rareté 20, ancienneté de la série 15, dynamique du prix Cardmarket 15 (tendance vs moyenne 30 j), fiabilité 15 (nombre de ventes, écart prix bas / moyen), potentiel gradé 10 (PSA 10 / brut, au moins 3 ventes). Verdict : ≥ 75 très intéressante, ≥ 62 intéressante, ≥ 48 à surveiller. Répartition des ventes PSA par note (ce n’est pas la population PSA, réservée aux offres payantes). Indice de collectionneur, pas une prédiction.

### Sélection de la semaine

- `pokemon.candidats` : cartes EN des raretés recherchées dont le nom contient un Pokémon dragon, légendaire, fabuleux ou iconique (`pokemon.especes`, depuis `noms.json`). Liste refaite le lundi.
- `pokemon.maj_candidats` (chaque nuit à 2 h 20, 170 cartes par roulement) : prix TCGdex et présélection, sans crédit PokemonPriceTracker.
- `pokemon.selection_hebdo` (dimanche 6 h 30) : 20 meilleures dans le budget, analyse complète avec ventes réelles (≈ 40 crédits), 15 retenues dans `pokemon.selections`.

## Limites connues

- Le centrage se mesure sur les cartes avec bordure ; les full art n’en ont pas. Coins, bords et surface sont évalués par toi : la note est une estimation, pas une gradation.
- Images : TCGdex ne liste qu’une partie des images (surtout en japonais). L’appli essaie l’adresse reconstruite, puis la même carte dans l’autre langue (mention « visuel EN »), puis ta photo du scan, puis un visuel neutre. Les anciennes séries japonaises (Fossile, Neo…) n’ont pas d’images chez TCGdex : le visuel de la carte anglaise équivalente est affiché.
- Noms des Pokémon en FR/EN/JA : `noms.json`, tiré de PokeAPI.
- La lecture automatique repose sur le numéro imprimé (`025/198`). Les numéros avec lettres (`TG05/TG30`, promos) se cherchent à la main.
- Les cartes japonaises ont leur nom en japonais : cherche-les par numéro (`25/165 jp`).
- La cote Cardmarket mélange les états et, pour une même impression, les langues FR et EN.
