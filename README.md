# Analyse de la politique de réapprovisionnement d'un réseau de cinq entrepôts

---

## Sommaire

1. [Contexte et objectif](#1-contexte-et-objectif)
2. [Structure des données](#2-structure-des-données)
3. [Résumé](#3-résumé)
4. [Analyse approfondie](#4-analyse-approfondie)
   - [4.1 La demande est fortement saisonnière](#41-la-demande-est-fortement-saisonnière)
   - [4.2 Le stock détenu ignore cette saisonnalité](#42-le-stock-détenu-ignore-cette-saisonnalité)
   - [4.3 Les seuils ne suivent pas les délais fournisseurs](#43-les-seuils-ne-suivent-pas-les-délais-fournisseurs)
   - [4.4 Aucun produit ne domine le catalogue](#44-aucun-produit-ne-domine-le-catalogue)
5. [Recommandations](#5-recommandations)

---

## 1. Contexte et objectif

Un réseau de cinq entrepôts distribue cinquante références sur quatre régions, approvisionnées par dix fournisseurs. Chaque référence est suivie quotidiennement sur l'année 2024 : niveau de stock, ventes du jour, seuil de déclenchement de commande, délai du fournisseur.

Deux risques encadrent la gestion de ce type de réseau. **Trop peu de stock, et la vente est perdue. Trop de stock, et le capital dort en entrepôt**, avec un coût de stockage et un risque d'obsolescence. Tout l'enjeu d'une politique de réapprovisionnement est de tenir la ligne entre les deux.

**La question posée : la politique actuelle est-elle calibrée sur les contraintes réelles du réseau — la saisonnalité de la demande et les délais des fournisseurs ?**

Le classeur d'analyse et le détail des formules sont accessibles ici : [`supply_chain_dashboard.xlsx`](supply_chain_dashboard.xlsx) · [`formules.md`](formules.md) · [données source](https://www.kaggle.com/datasets/ziya07/high-dimensional-supply-chain-inventory-dataset).

---

## 2. Structure des données

**91 250 relevés quotidiens · 50 références · 5 entrepôts · 10 fournisseurs · 365 jours**

Le jeu forme une grille complète : chaque combinaison référence × entrepôt est observée chacun des 365 jours de 2024, soit **250 séries de 365 points**.

| Champ | Nature | Description |
|---|---|---|
| `Date` | — | Jour du relevé |
| `SKU_ID` | Dimension | Référence produit (SKU_1 à SKU_50) |
| `Warehouse_ID` | Dimension | Entrepôt (WH_1 à WH_5) |
| `Supplier_ID` | Dimension | Fournisseur (SUP_1 à SUP_10) |
| `Region` | Dimension | Zone de distribution (East, West, North, South) |
| `Units_Sold` | **Flux** | Unités vendues dans la journée |
| `Inventory_Level` | **État** | Niveau de stock en fin de journée |
| `Reorder_Point` | Paramètre | Seuil déclenchant une commande |
| `Supplier_Lead_Time_Days` | Paramètre | Délai de livraison du fournisseur |
| `Unit_Cost` / `Unit_Price` | Paramètre | Prix d'achat et de vente unitaires |
| `Promotion_Flag` | Contexte | Promotion active ce jour-là |
| `Demand_Forecast` | Contexte | Prévision de demande |

**La distinction flux / état conditionne tous les calculs qui suivent.** `Units_Sold` est un flux : les ventes de chaque jour s'additionnent pour donner le volume annuel. `Inventory_Level` est un état : c'est une photographie du stock à une date donnée. L'additionner sur 365 jours reviendrait à compter le même stock 365 fois.

Les indicateurs de stock présentés ici sont donc des **moyennes journalières**, obtenues en divisant le cumul par le nombre de dates distinctes.

---

## 3. Résumé

| Indicateur | Valeur |
|---|---|
| Stock moyen détenu par jour | **117 881 unités** |
| Valeur moyenne immobilisée | **1 439 025 €** |
| Unités vendues sur l'année | 1 829 979 |
| Chiffre d'affaires | 33 426 337 € |
| Taux de marge | 33,2% |
| Rotation de stock | **15,5 fois par an** — 24 jours de couverture moyenne |
| Relevés sous le seuil de réapprovisionnement | 5,2% |
| Ruptures effectives | **0** |

Le réseau n'a connu **aucune rupture** sur l'année, et la rotation de 15,5 se situe dans les standards de la distribution. Vu ainsi, la gestion paraît saine.

L'analyse qui suit montre que cette absence de rupture ne vient pas d'un pilotage fin, mais d'un stock uniformément élevé qui absorbe les variations sans jamais s'y adapter. Le réseau ne casse pas parce qu'il porte en permanence plus de stock que nécessaire — et il le paie pendant la moitié de l'année.

---

## 4. Analyse approfondie

### 4.1 La demande est fortement saisonnière

![Unités vendues par région et stock moyen par entrepôt](screenshots/graphiques_stock.png)

La demande quotidienne moyenne par référence et par entrepôt passe de **29,9 unités en avril à 10,2 unités en septembre**, soit un rapport de 2,9 entre le pic et le creux.

| Période | Demande moyenne | Profil |
|---|---|---|
| Mars – Avril | 29,8 u/jour | **Pic annuel** |
| Février, Mai | 27,2 u/jour | Haute saison |
| Janvier, Juin | 22,8 u/jour | Transition |
| Juillet, Décembre | 17,5 u/jour | Basse saison |
| **Septembre – Octobre** | **10,2 u/jour** | **Creux annuel** |

> **Insight.** Le réseau connaît deux régimes distincts : un semestre haut de février à juin où la demande dépasse 27 unités par jour, et un semestre bas d'août à décembre où elle tombe sous 18. L'écart entre les deux régimes dépasse un facteur 2 — ce n'est pas du bruit statistique, c'est une structure.

La répartition entre les quatre régions est en revanche parfaitement homogène, autour de 25% chacune. **Aucun déséquilibre géographique n'est à corriger** : le levier d'action n'est pas là.

**→ Recommandation A — Construire le plan d'approvisionnement sur deux saisons, pas sur une moyenne annuelle.** Une politique unique calibrée sur la moyenne de 20 unités par jour sera systématiquement trop basse six mois et trop haute les six autres. Deux régimes de commande, un par semestre, suffisent à épouser la structure réelle de la demande.

---

### 4.2 Le stock détenu ignore cette saisonnalité

![Évolution du stock moyen par mois](screenshots/graphiques_sku.png)

Face à une demande qui varie d'un facteur 2,9, le stock détenu varie d'un facteur **1,16**. Il reste ancré entre 454 et 478 unités toute l'année, à l'exception de janvier (527) qui sort d'un cycle de fin d'année.

La **corrélation entre le stock détenu et la demande du mois est de −0,27** : légèrement négative, là où une gestion pilotée par la demande produirait une corrélation fortement positive.

La conséquence se lit dans la couverture, c'est-à-dire le nombre de jours de vente que le stock permet de tenir :

| Mois | Demande | Stock | **Couverture** |
|---|---|---|---|
| Mars | 29,9 | 459 | **15 jours** |
| Avril | 29,8 | 454 | **15 jours** |
| Janvier | 22,8 | 527 | 23 jours |
| Juillet | 17,4 | 466 | 27 jours |
| **Septembre** | **10,2** | **472** | **46 jours** |
| Octobre | 10,3 | 476 | 46 jours |

> **Insight.** La couverture varie de 15 à 46 jours sans qu'aucune décision ne soit prise. Elle n'est pas pilotée, elle est subie : elle résulte mécaniquement d'un stock fixe divisé par une demande variable. En avril, le réseau tient deux semaines ; en septembre, un mois et demi.

En simulant une politique de couverture constante à 25 jours, l'excédent de capital immobilisé pendant les six mois de basse saison atteint **409 000 € en moyenne, avec un pic à 668 000 € en octobre**. À l'inverse, mars et avril tournent à 15 jours de couverture — une marge étroite si un fournisseur prend du retard au pire moment de l'année.

**→ Recommandation B — Piloter une couverture cible plutôt qu'un niveau de stock.** Fixer un objectif en jours (25 jours par exemple) et laisser le stock cible suivre la demande prévisionnelle du mois. L'effet est double : environ **400 000 € de capital libéré** en basse saison, et une couverture qui remonte de 15 à 25 jours en mars-avril, quand le réseau est le plus tendu.

---

### 4.3 Les seuils ne suivent pas les délais fournisseurs

![Délai moyen par fournisseur](screenshots/delai_fournisseur.png)

Le seuil de réapprovisionnement doit couvrir la consommation pendant le délai de livraison, plus une marge de sécurité. Mécaniquement, **un fournisseur plus lent exige un seuil plus haut**.

Les délais s'échelonnent de 6,96 jours pour le plus rapide à 8,64 jours pour le plus lent, soit un écart de 24%. Les seuils appliqués ne suivent pas cette hiérarchie :

| Fournisseur | Délai réel | Seuil appliqué | Seuil théorique* | Marge de sécurité |
|---|---|---|---|---|
| SUP_5 | 6,96 j | 298 | 209 | **+89** |
| SUP_7 | 7,56 j | 322 | 227 | +95 |
| SUP_10 | 7,41 j | 302 | 222 | +79 |
| SUP_6 | 7,90 j | 304 | 238 | +65 |
| SUP_1 | 8,11 j | 293 | 243 | +50 |
| SUP_9 | 8,19 j | 292 | 247 | +45 |
| SUP_8 | 8,50 j | 301 | 256 | +44 |
| **SUP_4** | **8,64 j** | **301** | **260** | **+41** |
| SUP_3 | 8,26 j | 285 | 248 | +37 |

*\* Délai × demande quotidienne × 1,5 (marge de sécurité de 50%)*

> **Insight.** La corrélation entre délai fournisseur et seuil appliqué est de **−0,36**, alors qu'elle devrait être fortement positive. Les fournisseurs les plus lents ont les seuils les plus bas. SUP_4 met 24% plus de temps à livrer que SUP_5, mais dispose d'une marge de sécurité **deux fois plus faible** — 41 unités contre 89.

Cette inversion est le défaut de calibrage le plus net du réseau. Les références approvisionnées par les fournisseurs les plus lents sont aussi les plus exposées en cas d'aléa, soit exactement l'inverse de ce que devrait produire une politique de réapprovisionnement.

**→ Recommandation C — Indexer chaque seuil sur le délai de son fournisseur.** Appliquer la formule standard `Seuil = Délai × Demande quotidienne × (1 + marge)` à chaque couple référence × fournisseur. Pour SUP_4, cela porterait le seuil de 301 à environ **390 unités** à marge égale avec SUP_5. Le réseau consomme un peu plus de stock sur les références lentes, mais supprime l'exposition asymétrique actuelle.

---

### 4.4 Aucun produit ne domine le catalogue

![Volumes vendus par référence](screenshots/graphiques_sku.png)

La classification ABC — concentrer le pilotage sur les 20% de références qui font 80% du volume — est le réflexe standard en gestion de stock. Elle ne s'applique pas ici.

Les cinquante références vendent entre **36 213 et 37 234 unités** sur l'année. L'écart entre la première et la dernière est de 2,8%, et l'écart-type représente 0,6% de la moyenne. Le catalogue est plat.

> **Insight.** Il n'existe pas de référence critique par le volume. Prioriser par les ventes n'apporterait aucune information, puisque toutes les références pèsent le même poids. Le critère de priorisation doit se trouver ailleurs — dans le délai du fournisseur qui approvisionne chaque référence, **seule dimension où le réseau présente une vraie dispersion**.

*Note méthodologique : une version antérieure de ce tableau de bord présentait « SKU_1 à SKU_10 » comme top 10 des ventes. Vérification faite, seules quatre de ces références figurent réellement dans les dix premières — une sélection par ordre d'identifiant avait été prise pour un classement. La correction a conduit au constat ci-dessus, plus utile que le classement lui-même.*

**→ Recommandation D — Surveiller par risque fournisseur, pas par volume de vente.** Concentrer le suivi sur les références approvisionnées par SUP_4, SUP_8 et SUP_2 : ce sont les seules dont le délai de reconstitution dépasse 8,4 jours, et donc les seules où un aléa se traduit rapidement en rupture.

---

## 5. Recommandations

Les quatre recommandations issues de l'analyse, ordonnées par priorité d'exécution.

### Priorité 1 — Corriger le calibrage des seuils *(recommandation C)*

**Le plus urgent, parce que la logique actuelle est inversée.** Les références les plus lentes à reconstituer sont aujourd'hui les moins protégées. La correction est mécanique — une formule appliquée à chaque couple référence × fournisseur — et ne demande aucun arbitrage de gestion.

| | |
|---|---|
| Effort | Faible — recalcul paramétrique |
| Gain | Suppression de l'exposition asymétrique sur 3 fournisseurs |
| Effet de bord | Légère hausse du stock moyen sur les références lentes |

### Priorité 2 — Passer à un pilotage par couverture *(recommandation B)*

**Le plus rentable.** Environ 400 000 € de capital libéré en moyenne sur la basse saison, et une couverture qui remonte à 25 jours pendant le pic de mars-avril. Demande en revanche de revoir le processus de commande, pas seulement un paramètre.

| | |
|---|---|
| Effort | Moyen — changement de logique de pilotage |
| Gain | ~400 000 € de capital libéré, couverture sécurisée au pic |
| Prérequis | Prévision de demande mensuelle fiable |

### Priorité 3 — Structurer l'approvisionnement en deux saisons *(recommandation A)*

Prolongement naturel de la priorité 2. Une fois la couverture pilotée, les régimes de commande se dessinent d'eux-mêmes : un régime haut de février à juin, un régime bas d'août à décembre.

| | |
|---|---|
| Effort | Moyen — refonte du calendrier d'approvisionnement |
| Gain | Alignement structurel de l'offre sur la demande |

### Priorité 4 — Réorienter la surveillance *(recommandation D)*

**À traiter en dernier car c'est un changement d'habitude, pas de système.** Le catalogue étant plat en volume, la vigilance doit porter sur le risque fournisseur. Le creux de septembre-octobre, où la couverture atteint 46 jours, offre la fenêtre naturelle pour déstocker, renégocier les conditions d'achat et concentrer les opérations promotionnelles — celles-ci génèrent 27,7% de ventes supplémentaires et ne couvrent aujourd'hui que 10% des jours.

| | |
|---|---|
| Effort | Faible — redéfinition des tableaux de suivi |
| Gain | Détection anticipée sur les références réellement exposées |
