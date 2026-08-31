# 📦 Analyse & Suivi des Stocks — Supply Chain Dashboard

> Projet portfolio Data Analyst · Secteur Logistique & Retail · 2024

---

## 🎯 Contexte & Problématique

### Contexte métier

La gestion des stocks est un enjeu critique pour toute entreprise logistique ou retail : un stock insuffisant entraîne des ruptures qui font fuir les clients, tandis qu'un stock excédentaire immobilise du capital inutilement. L'enjeu est de trouver le bon équilibre.

### Problématique

**"Comment optimiser la gestion des stocks d'un entrepôt multi-sites en détectant automatiquement les risques de rupture et de surstock, et en identifiant les leviers d'action prioritaires ?"**

### Objectifs du projet

- Analyser les niveaux de stocks sur **5 entrepôts** et **10 SKU produits** sur une année complète
- Construire un dashboard Google Sheets interactif avec **KPIs métier clés**
- Implémenter un **système d'alertes automatiques** pour la détection des anomalies
- Identifier les fournisseurs les plus performants et les tendances saisonnières

---

## 📊 Dashboard Google Sheets

### Structure du fichier

Le dashboard est organisé en **3 onglets** :

| Onglet | Description |
|--------|-------------|
| **Données** | Dataset brut — 91 250 lignes, 15 colonnes |
| **Dashboard** | KPIs + 5 graphiques interactifs |
| **Alertes** | Système de détection automatique des anomalies |

### Onglet Dashboard — KPIs

| KPI | Valeur | Description |
|-----|--------|-------------|
| 📦 Stock total | 43 026 411 unités | Inventaire global tous entrepôts |
| 🛒 Unités vendues | 1 829 979 | Volume total sur l'année 2024 |
| ⚠️ Stock sous seuil | 9.6% | Part des lignes en zone critique |
| 💰 Valeur totale du stock | 338 110 350 € | Valorisation à prix coûtant |
| 🚚 Délai fournisseur moyen | 7.98 jours | Performance moyenne des 10 fournisseurs |
| 💵 Valeur des ventes | 21 235 335 € | Chiffre d'affaires total (Prix × Quantités) |
| 📈 Marge brute totale | 6 969 348 € | (Prix vente − Prix achat) × Quantités |

### Onglet Dashboard — Graphiques

**1. Stock moyen par entrepôt (Histogramme)**
- 5 entrepôts : WH_1 à WH_5
- WH_2 affiche le stock le plus élevé → entrepôt à prioriser pour le contrôle

**2. Unités vendues par région (Anneau)**
- 4 régions : East, South, North, West
- Répartition équilibrée ~25% chacune → aucun déséquilibre régional majeur

**3. Évolution du stock moyen par mois (Courbe)**
- Tendance sur 12 mois (Janvier–Décembre 2024)
- Pic en Janvier, légère baisse en milieu d'année

**4. Top 10 SKU les plus vendus (Barres horizontales)**
- SKU_1 à SKU_10 : volumes similaires (~35–37K unités)
- Dataset bien équilibré — pas de produit dominant

**5. Délai fournisseur moyen par fournisseur (Histogramme)**
- SUP_5 le plus rapide (~6.9 jours)
- SUP_4 le plus lent (~8.5 jours) → levier d'amélioration identifié

### Onglet Alertes

Système d'alertes à **3 niveaux** basé sur la comparaison stock actuel vs. seuil de réapprovisionnement :

```
🔴 Rupture imminente → Stock < Seuil réapprovisionnement → Commander immédiatement
🟡 Surstock          → Stock > 2× le seuil             → Réduire les commandes
🟢 Normal            → Entre les deux seuils            → Aucune action requise
```

**Résultats sur 99 lignes analysées :**
- 🔴 8 ruptures imminentes à traiter en priorité
- 🟡 4 surstocks à réguler
- 🟢 86 lignes en situation normale

---

## 🗄️ Dataset

**Source :** [Kaggle — High-Dimensional Supply Chain Inventory Dataset](https://www.kaggle.com/datasets/ziya07/high-dimensional-supply-chain-inventory-dataset)

**Volume :** 91 250 lignes × 15 colonnes

| Colonne | Type | Description |
|---------|------|-------------|
| `Date` | Date | Journée d'enregistrement (2024) |
| `SKU_ID` | Catégorie | Identifiant produit (SKU_1–SKU_50) |
| `Warehouse_ID` | Catégorie | Entrepôt (WH_1–WH_5) |
| `Supplier_ID` | Catégorie | Fournisseur (SUP_1–SUP_10) |
| `Region` | Catégorie | Région géographique (East/West/North/South) |
| `Units_Sold` | Entier | Unités vendues dans la journée |
| `Inventory_Level` | Entier | Niveau de stock en fin de journée |
| `Supplier_Lead_Time_Days` | Entier | Délai de livraison fournisseur (jours) |
| `Reorder_Point` | Entier | Seuil de déclenchement de commande |
| `Order_Quantity` | Entier | Quantité commandée (0 si pas de commande) |
| `Unit_Cost` | Décimal | Prix d'achat unitaire |
| `Unit_Price` | Décimal | Prix de vente unitaire |
| `Promotion_Flag` | Booléen | 1 = Promotion active, 0 = Prix normal |
| `Stockout_Flag` | Booléen | 1 = Rupture de stock, 0 = OK |
| `Demand_Forecast` | Décimal | Prévision de demande |

---

## 📐 Formules Google Sheets avancées

### KPIs avec données mixtes (texte/nombre)

Certaines colonnes numériques sont stockées en texte — ces formules gèrent la conversion :

```
=SUMPRODUCT(SUBSTITUE(Données!G2:G91251;".";",")*1*SUBSTITUE(Données!K2:K91251;".";",")*1)
```

### Taux de stock sous seuil

```
=ROUND(COUNTIF(Données!G2:G91251;"<"&AVERAGE(Données!I2:I91251))/COUNTA(Données!G2:G91251)*100;1)&"%"
```

### Évolution mensuelle (AVERAGEIFS avec dates)

```
=AVERAGEIFS(Données!G$2:G$91251;Données!A$2:A$91251;">="&DATE(2024;1;1);Données!A$2:A$91251;"<"&DATE(2024;2;1))
```

### Système d'alertes (logique SI imbriquée)

```
=SI(C2<D2;"🔴 Rupture imminente";SI(C2>D2*2;"🟡 Surstock";"🟢 Normal"))
```

```
=SI(C2<D2;"Commander immédiatement";SI(C2>D2*2;"Réduire les commandes";"Aucune action"))
```

---

## 💡 Insights métier

| Insight | Impact |
|---------|--------|
| WH_2 détient le plus gros stock | Risque d'immobilisation de capital — audit recommandé |
| SUP_4 : délai le plus long (+1.5j vs SUP_5) | Négociation ou substitution fournisseur à envisager |
| 9.6% des lignes sous seuil de réapprovisionnement | Système d'alerte actif — actions préventives possibles |
| Répartition régionale équilibrée | Pas de risque de dépendance géographique |
| Pic de stock en Janvier | Cohérent avec saisonnalité post-fêtes |

---

## 🛠️ Stack technique

- **Google Sheets** — Dashboard interactif et alertes automatiques
- **Formules avancées** — AVERAGEIFS, SUMPRODUCT, COUNTIF, SI imbriqués
- **Kaggle** — Source de données publique réelle
- **GitHub** — Versioning et présentation portfolio

---

## 🚀 Reproduire le projet

1. Télécharger le dataset sur [Kaggle](https://www.kaggle.com/datasets/ziya07/high-dimensional-supply-chain-inventory-dataset)
2. Créer un Google Sheets avec 3 onglets : `Données`, `Dashboard`, `Alertes`
3. Importer le CSV dans l'onglet `Données` via **Fichier → Importer**
4. Reproduire les formules et graphiques décrits dans ce README

---

## 👤 Auteur

**Thierry** · Candidat Data Analyst Junior · Paris Île-de-France  
Master Big Data — Paris 8 · Licence Informatique — Sorbonne Paris Nord

> 💼 En reconversion depuis un poste opérationnel en logistique — ce projet reflète ma capacité à analyser des données métier réelles et à produire des insights actionables pour des équipes terrain.

---

## 📁 Fichiers du repo

```
supply-chain-sheets-dashboard/
├── README.md                          ← Ce fichier
├── supply_chain_dataset1.csv          ← Dataset source (Kaggle)
└── dashboard_screenshot.png           ← Aperçu du dashboard (optionnel)
```
