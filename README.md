

# 🌍 Analyse du Changement Climatique dans les Alpes

## 📘 Description

Ce projet vise à étudier les effets du **changement climatique** dans les Alpes, en particulier dans la région du Mont-Blanc, à travers l’analyse :

1. Des **données météorologiques** issues de Météo France (températures, précipitations) depuis 1950 dans le département de la Haute-Savoie (74).
2. Des **données hydrométriques** simulées et observées concernant la rivière Arve, en lien avec la fonte des glaciers et les scénarios climatiques RCP4.5 et RCP8.5.

Ce travail mêle **statistiques, modélisation, prévisions climatiques** et **visualisation de données** pour fournir une réponse rigoureuse à la question :

> Le changement climatique est-il visible dans les Alpes ?

---

## 📂 Structure du projet

```
.
├── README.md               # Présentation du projet
├── analyse_climat_alpes.ipynb    # Notebook principal d’analyse
├── Donnees_Debits/        # Dossier contenant les données hydrométriques simulées
│   ├── Q_cum_RCP4-5_*.csv
│   ├── Q_cum_RCP8-5_*.csv
│   ├── Q_cum_hist_*.csv
│   └── Q_sd_*.csv
├── Fig1.png               # Carte de la zone d'étude
├── Fig2.png               # Graphiques issus de l’article de référence
└── Q_74_previous-1950-2023_RR-T-Vent.csv   # Données Météo France (Haute-Savoie)
```

---

## 🧪 Outils utilisés

* Python 3
* Pandas, NumPy
* Matplotlib, Seaborn
* Scikit-learn
* Jupyter Notebook

---

## 🧭 Objectifs

### Partie 1 – Données météorologiques :

* Nettoyage des données météo : températures minimales/maximales, précipitations, etc.
* Calcul de moyennes annuelles et saisonnières
* Régression linéaire pour identifier les tendances climatiques depuis 1950
* Prédiction de la température moyenne en 2100
* Analyse des épisodes de **fortes chaleurs** (>28°C)

### Partie 2 – Données hydrométriques (débit de l'Arve) :

* Chargement et traitement des simulations de débits (historiques et futurs)
* Reproduction de la **figure 3** de l’article scientifique de référence
* Comparaison entre les données **observées**, **historiques** et **scénarios climatiques RCP**
* Analyse de l’impact futur du réchauffement climatique sur les ressources en eau

---

## 📈 Résultats clés

* 📊 **Hausse significative des températures moyennes** depuis 1950 dans la région du Mont-Blanc.
* 🌧️ Variabilité des précipitations, mais tendance moins marquée.
* 🔥 **Augmentation des épisodes de fortes chaleurs**, en particulier après les années 2000.
* 🌊 Sous les scénarios RCP4.5 et RCP8.5, les débits estivaux futurs de la rivière Arve chuteraient significativement, accentuant les tensions sur les ressources hydriques.

---

## 🔮 Perspectives

* Intégration d'autres modèles climatiques pour affiner les prévisions.
* Étude de l’évolution de la couverture neigeuse et glaciaire.
* Utilisation de techniques de **modélisation non linéaire** (réseaux de neurones, forêts aléatoires).
* Extension à d’autres bassins alpins ou massifs montagneux.

---

## 📝 Livrables attendus

* 📄 Un **compte-rendu PDF (max 8 pages)** analysant les résultats
* 📓 Le **notebook Jupyter exécuté**
* 📁 Les données météo et les fichiers de simulation hydrologique

