
# Le changement climatique peut il être observé dans les Alpes?

## Consignes

Ce TP peut être réalisé sur [MyDocker]. XX TODO ADD
Vous devrez rendre ce notebook avec les fonctions/ votre script python ainsi qu'un compte-rendu de 8 pages maximum dans lequel vous répondrez aux questions.
**Le compte-rendu au format PDF et le notebook** sont à rendre par email avant le prochain cours soit **lundi 27 janvier 23h59**. J'ouvrirai mes mails au réveil, et à partir de là, -2 points par heure de retard.

## Introduction

En science des données vous devrez préparer les données, les analyser (statistiquement) et produire des figures pertinentes dans l'objectif de répondre à différentes questions.

Dans ce TP, on se demande si le changement climatique est visible dans les Alpes et nous mettrons en lien les observations avec les analyses des flux hydrométriques (= le force du courant dans les rivières) dans cette région. 

Pour ce faire nous allons commencer par étudier l'évolution de la météo au cours des dernières décennies. Météo France, l'organisme national de météorologie en France, a déposé des données climatologiques par département avec de nombreux paramètres disponibles sur le site [data.gouv](https://www.data.gouv.fr/fr/datasets/donnees-climatologiques-de-base-quotidiennes/).

Dans un second temps, nous étudierons l'évolution des débits de l'Arve (une rivière) en lien avec le changement climatique et la fonte des glaciers aux alentours du Mont-Blanc (un massif *relativement* connu des Alpes). Le papier a été publié dans Scientific Reports en 2020 et est disponible [ici](https://doi.org/10.1038/s41598-020-67379-7).

## Chargement des librairies


Voici quelques librairies dont vous aurez très probablement besoin, n'hésitez pas a en ajouter d'autres !

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline

from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression

```

## Optimisation du code

Pensez à optimiser votre code pour améliorer l'efficacité, en particulier lorsqu'il s'agit de manipuler de grandes quantités de données. Utilisez des fonctions pour que vous puissiez refaire la même analyse avec un autre département.


# PARTIE 1: Données climatiques de Météo France

## Chargement du jeu de données 

Commencez par récupérer sur le site des données publiques francaises, les données quotidiennes de météo entre 1950 et 2022 en Haute-Savoie où se situe le Mont-Blanc (département n°74) sur le site [data.gouv](https://www.data.gouv.fr/fr/datasets/donnees-climatologiques-de-base-quotidiennes/).

On ne s'intéresse pour le moment qu'aux données de température et de précipitation.

Quand vous êtes prête ou prêt, chargez la table d'intéret dans la variable `meteo`.

*Indication :* Pour trouver quel fichier télécharger, lisez la documentation!

```python
#Importation du dataset
meteo=pd.read_csv("Q_74_previous-1950-2023_RR-T-Vent.csv",delimiter=';')
# afficher le resultat
meteo

```

## Exploration des données

Décrivez la table. Par exempl,e répondez aux questions suivantes:

- Quelle sont les dimensions de `meteo` ?
- Combien y a-t-il de postes météorologiques ? 
    
Vos réponses sont à rédiger dans le compte-rendu pas dans le notebook, ici je n'évaluerai que les fonctions!

```python
# dimension du dataset
meteo.shape
```

```python
# Verifier si on a autant de valeur distinctes d'altitude que de numero de poste
# meteo['NUM_POSTE'].nunique() == meteo['ALTI'].nunique()

meteo['ALTI'].nunique()
```

```python
# Verifions le nombre de NaN par colonnes:
for col in meteo.columns:
    print(col," :", len(meteo[col][meteo[col].isna()].to_list()),'\n')
```

```python
#Trouver le nombre de poste meteorologique
nombre_poste=meteo['NUM_POSTE'].nunique()
print(f"nombre_poste non nulles :{nombre_poste}")
```

**Gestion des valeurs manquantes et filtration**

- Quelles colonnes allez vous sélectionner pour notre étude ? Pour rappel, on va étudier l'évolution de la température et des précipitations depuis les années 1950 dans les Alpes.
*Indication:* La localisation des postes pourra être utile.
- Créer la table `meteo_cleaned` avec les colonnes d'interet et sans données manquantes.
- Faites une première analyse sur les informations statistiques de base sur les données (moyenne, médiane, écart-type, etc.).


```python
#Votre code ici
colonne_interet=['NUM_POSTE','NOM_USUEL','LAT','LON','ALTI','AAAAMMJJ','RR','TN','TX','TNTXM'];
meteo_cleaned=meteo.loc[:,colonne_interet]
meteo_cleaned=meteo_cleaned.dropna()
meteo_cleaned

```

```python
#moyenne pour chaque colonne temperature et precipitation
colonne_etude = ['RR','TN','TX','TNTXM']
for col in colonne_etude:
    print ("Moyenne",col,":",meteo_cleaned[col].mean())
#meteo['ALTI'].unique()

```

```python
# meteo_cleaned['NUM_POSTE'].unique()
```

```python
# valeurs moyenne par postes
les_postes = meteo_cleaned['NUM_POSTE'].unique()
for col in colonne_etude:
    print("\n\n" "---- pour la colonne ----",col,":")
    for poste in les_postes:
        print("poste",poste,":",meteo_cleaned[col][meteo_cleaned['NUM_POSTE']==poste].mean())


```

- Combien de stations restent dans notre étude ? Où se situent-elles ?

```python

# Identifier les stations uniques
nbre_stations_unique = meteo_cleaned['NUM_POSTE'].nunique()
# Localiser les stations 
stations_info = meteo_cleaned[['NUM_POSTE', 'NOM_USUEL', 'LAT', 'LON']].drop_duplicates()

# Afficher le nombre de stations et leur localisation
print(f"Nombre de stations restantes dans mon  étude : {nbre_stations_unique}")
print("\nLocalisation des stations :")
print(stations_info)
```

## Analyse des données


### Tendances annuelles

Quelles sont les tendances annuelles dans les données météorologiques depuis 1950 ? La température moyenne a-t-elle changée ? Est-ce qu'il y a plus ou moins de précipitations ?

Faites une analyse (calcul de moyenne, tests de regressions etc) pour répondre à la question. Ci-dessous voici les principales étapes à effectuer:
- Transformez la colonne date pour que vous puissiez l'exploiter facilement
- Calculez les températures et précipitations moyennes annuelles
- Faites une régression linéaire pour estimer l'évolution de ces valeurs. Utilisez scikit-learn comme appris en L1 ISD.
- Evaluez la performance de vos modèles de prédiction. Pour cela, vous pourrez par exemple utiliser les métriques disponibles dans `sklearn.metrics`.
- La régression est-elle pertinente pour la température ? Pour les précipitations ?
- Auriez-vous un autre modèle plus pertinent qu'une régression linéaire à proposer ? N'hésitez pas à l'**ajouter**.
- Crééz également un ou plusieurs graphiques pour représenter les variations des paramètres météorologiques au fil du temps.

Si besoin vous pouvez charger de nouvelles librairies.

```python

meteo_cleaned2 = meteo_cleaned.copy()

# Convertir la colonne 'AAAAMMJJ' en format date
meteo_cleaned2['AAAAMMJJ'] = pd.to_datetime(meteo_cleaned2['AAAAMMJJ'], format='%Y%m%d')
# Extraire l'année
meteo_cleaned2['ANNEE'] = meteo_cleaned2['AAAAMMJJ'].dt.year
meteo_cleaned2
```

```python
#températures et précipitations moyennes annuelles
les_annees = meteo_cleaned2['ANNEE'].unique()
for col in colonne_etude:
    print("\n\n" "---- pour la colonne ----",col,":")
    for annee in les_annees:
        print("moyennes",annee,":",meteo_cleaned2[col][meteo_cleaned2['ANNEE']==annee].mean())
```

```python

```

### Prévisions

Quelle température fera-t-il en 2100 selon votre modèle ?


```python
from sklearn.metrics import mean_squared_error, r2_score

# Calcul des moyennes annuelles pour la température et les précipitations
donnee_annuelle  = meteo_cleaned2.groupby('ANNEE').agg({
    'TNTXM': 'mean',
    'RR': 'mean'
}).reset_index()

# Préparation des données pour la régression
X = donnee_annuelle ['ANNEE'].values.reshape(-1, 1)  # Année
y_temperature =donnee_annuelle ['TNTXM'].values  # Température moyenne
y_precipitations = donnee_annuelle ['RR'].values  # Précipitations

# Régression linéaire pour la température
reg_temp = LinearRegression()
reg_temp.fit(X, y_temperature)

# Régression linéaire pour les précipitations
reg_precip = LinearRegression()
reg_precip.fit(X, y_precipitations)

# Prédiction pour l'an 2100
annee_2100 = np.array([[2100]])
temperature_2100 = reg_temp.predict(annee_2100)[0]

# Évaluation des modèles
y_pred_temp = reg_temp.predict(X)
y_pred_precip = reg_precip.predict(X)

# Calcul des erreurs
rmse_temp = np.sqrt(mean_squared_error(y_temperature, y_pred_temp))
r2_temp = r2_score(y_temperature, y_pred_temp)

rmse_precip = np.sqrt(mean_squared_error(y_precipitations, y_pred_precip))
r2_precip = r2_score(y_precipitations, y_pred_precip)

# Affichage des résultats
print(f"Prédiction de la température pour l'an 2100 : {temperature_2100:.2f}°C")
print(f"Évaluation du modèle pour la température : RMSE = {rmse_temp:.2f}, R² = {r2_temp:.2f}")
print(f"Évaluation du modèle pour les précipitations : RMSE = {rmse_precip:.2f}, R² = {r2_precip:.2f}")

# Graphiques
plt.figure(figsize=(14, 6))

# Température
plt.subplot(1, 2, 1)
plt.scatter(donnee_annuelle['ANNEE'], donnee_annuelle['TNTXM'], color='tab:red', label='Température observée')
plt.plot(donnee_annuelle['ANNEE'], y_pred_temp, color='blue', label='Régression linéaire')
plt.title('Évolution de la température moyenne annuelle')
plt.xlabel('Année')
plt.ylabel('Température (°C)')
plt.legend()

# Précipitations
plt.subplot(1, 2, 2)
plt.scatter(donnee_annuelle['ANNEE'], donnee_annuelle['RR'], color='tab:blue', label='Précipitations observées')
plt.plot(donnee_annuelle['ANNEE'], y_pred_precip, color='green', label='Régression linéaire')
plt.title('Évolution des précipitations moyennes annuelles')
plt.xlabel('Année')
plt.ylabel('Précipitations (mm)')
plt.legend()

plt.tight_layout()
plt.show()
```

### Variabilité saisonnière 

Analysez la variabilité saisonnière des données. 

- Manipulez les données pour les regrouper mois par mois. Y a-t-il des tendances ? Correspondent-elles à vos connaissances ?
- N'oubliez pas de visualisez les données
- Enfin si vous êtes à l'aise, étudiez en plus les épisodes de fortes chaleurs (températures > 28°C). Est-ce qu'il y en a plus souvent plus récemment?

```python
# Regroupement par mois
meteo_cleaned2['mois'] = meteo_cleaned2['AAAAMMJJ'].dt.month  
meteo_cleaned2.sort_values(by='mois')
# Calcul des moyennes mensuelles (sur colonnes numériques)
variabilite_mensuelle = meteo_cleaned2.groupby('mois').mean(numeric_only=True)


# Visualisation
plt.figure(figsize=(15, 5))
for col in ['RR', 'TN', 'TX','TNTXM']:
    if col in variabilite_mensuelle.columns:
        plt.plot(variabilite_mensuelle.index, variabilite_mensuelle[col], label=col)
plt.xticks(ticks=range(1, 13))
plt.title("Variabilité mensuelle des données météorologiques")
plt.xlabel("Mois")
plt.ylabel("Valeurs moyennes")
plt.legend()
plt.grid()
plt.show()
```

```python
# Filtrer les épisodes de fortes chaleurs
fortes_chaleurs = meteo_cleaned2[meteo_cleaned2['TX'] > 28]
# Comptage annuel des épisodes de fortes chaleurs
fortes_chaleurs_par_annee = fortes_chaleurs.groupby('ANNEE').size()

# Visualisation
plt.figure(figsize=(10, 5))
plt.bar(fortes_chaleurs_par_annee.index, fortes_chaleurs_par_annee.values, color='orange')
plt.xlabel('Année')
plt.ylabel('Nombre d\'épisodes de fortes chaleurs')
plt.title('Évolution des épisodes de fortes chaleurs (> 28°C)')
plt.show()

```

```python
meteo_cleaned2
```

<!-- #region -->
# PARTIE 2: Evolution des débits de l'Arve


Dans cet [article](https://doi.org/10.1038/s41598-020-67379-7), les autrices et les auteurs examinent l'impact du changement climatique et de la perte de masse des glaciers sur l'hydrologie du massif du Mont-Blanc, en particulier sur le bassin versant de la rivière Arve, qui est alimentée par une série de glaciers dans la région. Ils ont utilisé des projections climatiques (scénarios RCP4.5 et RCP8.5) et des simulations de dynamique des glaciers (historiques et futures) combinées a un modèle hydrologique pour étudier l'évolution du débit des rivières à l'échelle du 21e siècle (vus en cours).

Vous avez représenté ci-dessous la zone d'étude de l'article avec en panneau (a), la localisation du bassin versant de Sallanches dans les Alpes françaises et la carte des bassins étudiés et en panneau (b), le massif du Mont-Blanc vu de Téléphérique de la Flégère (point noir avec angle de vue sur le plan).
![Zone étudiée: Fig 1 du papier](Fig1.png)

<!-- #endregion -->

<!-- #region -->
Dans leur article, ils commencent par étudier l'évolution des températures et des précipitations à Sallances. Ils montrent qu'il y a une nette augmentation des températures estivales et hivernales dans la région du Mont-Blanc et des modificaions de précipitations. Cela correspond à la figure 2 représentée ci-dessous. **Avez-vous obtenus les mêmes résultats avec votre analyse de la partie 1?**


![Fig 2 du papier](Fig2.png)


<!-- #endregion -->

Dans cette partie, nous allons reproduire la figure n°3 du papier qui illustre l'évolution des débits saisonniers de la rivière Arve en fonction des scénarios climatiques. La figure présente des courbes pour les débits moyens (en hiver et en été) simulées sous les scénarios RCP4.5 et RCP8.5, à partir de différents modèles climatiques pour la région du Mont-Blanc.


## Chargement des données

Dans le dossier `Donnees_Debits/`, vous disposez de données simulées historiques ainsi que de projections futures des débits cumulés (et de leur écart-type) de la rivière Arve (pour chaque scénario climatique). Ces données sont organisées en fonction des saisons (hiver, été ou moyenne annuelle) et des différents scénarios climatiques (historiques, RCP4.5, RCP8.5). 
Vous disposez également des séries temporelles de débits observés (`Debits_obs_Sal_sans_2902`) pour la saison d'hiver et d'été, qui servent de référence pour la comparaison des simulations.


**Regardez un peu les données. Quelles sont les dimensions des données historiques et des données de modélisation ?**

*Indications: Pour rappel, les deux scénarios RCP font partie des trajectoires d'émissions de gaz à effet de serre utilisées pour projeter les futurs changements climatiques. Pour chacun d'eux, plusieurs modèles climatiques régionaux ont été utilisés pour simuler l'évolution du climat et ensuite du débit de la rivière. Les différents modèles régionaux sont des modèles climatiques spécifiques, souvent *downscalés* (affinés à une échelle régionale) pour mieux simuler les conditions climatiques locales du Mont Blanc. Le processus de downscaling permet d'obtenir des projections climatiques à une résolution spatiale plus fine que celle des modèles climatiques globaux.* 


```python
#chagement des donnees historiques et RCP
data=rcp4_ete_cum=pd.read_csv("Donnees_Debits/Q_cum_RCP4-5_ete_multimodeles_OK")
data=rcp4_hiver_cum=pd.read_csv("Donnees_Debits/Q_cum_RCP4-5_hiver_multimodeles_OK")
data=rcp_annee_cum=pd.read_csv("Donnees_Debits/Q_cum_RCP4-5_annee_multimodeles_OK")
data=rcp8_ete_cum=pd.read_csv("Donnees_Debits/Q_cum_RCP8-5_ete_multimodeles_OK")
data=rcp8_hiver_cum=pd.read_csv("Donnees_Debits/Q_cum_RCP8-5_hiver_multimodeles_OK")
data=rcp8_annee_cum=pd.read_csv("Donnees_Debits/Q_cum_RCP8-5_annee_multimodeles_OK")

data=rcp4_ete_sd=pd.read_csv("Donnees_Debits/Q_sd_RCP4-5_ete_multimodeles_OK")
data=rcp4_hiver_sd=pd.read_csv("Donnees_Debits/Q_sd_RCP4-5_hiver_multimodeles_OK")
data=rcp4_annee_sd=pd.read_csv("Donnees_Debits/Q_cum_RCP4-5_annee_multimodeles_OK")
data=rcp8_ete_sd=pd.read_csv("Donnees_Debits/Q_sd_RCP8-5_ete_multimodeles_OK")
data=rcp8_hiver_sd=pd.read_csv("Donnees_Debits/Q_sd_RCP8-5_hiver_multimodeles_OK")
data=rcp8_annee_sd=pd.read_csv("Donnees_Debits/Q_sd_RCP8-5_annee_multimodeles_OK")

data=hist_ete_cum=pd.read_csv("Donnees_Debits/Q_cum_hist_ete_multimodeles_OK")
data=hist_annee_cum=pd.read_csv("Donnees_Debits/Q_cum_hist_annee_multimodeles_OK")
data=hist_hiver_cum=pd.read_csv("Donnees_Debits/Q_cum_hist_hiver_multimodeles_OK")
data=hist_ete_sd=pd.read_csv("Donnees_Debits/Q_sd_hist_ete_multimodeles_OK")
data=hist_hiver_sd=pd.read_csv("Donnees_Debits/Q_sd_hist_hiver_multimodeles_OK")
data=hist_annee_sd=pd.read_csv("Donnees_Debits/Q_sd_hist_annee_multimodeles_OK")
data

```

## Analyse et création de fonctions

**Etant donné une saison, faites une fonction qui pour chaque scenario (historique, ou RCP), calcule la moyenne et l'écart-type moyen du débit pour chaque année. Commencez par charger les données et transformer la date pour pouvoir extraire l'année. Dans un second temps, chargez les données observées et extrayez la valeur pour la saison considérée. Puis, reproduisez les graphiques montrant l'évolution du débit moyen pour les saisons d'hiver et d'été, sous les deux scénarios RCP4.5 et RCP8.5, comparés aux données historiques et aux données observées.**

Est-ce clair pour vous, quelle est la différence entre les données observées et historiques ?



**Discutez comment le changement climatique impacte les ressources en eau dans les régions montagneuses.**



```python

```

```python

# Calcul de la moyenne et de l'écart-type pour chaque série
def calcul_moy_std(data):
    """
    Fonction pour calculer la moyenne et l'écart-type
    des séries temporelles de chaque modèle.
    """
    moy = data.mean(axis=1)
    std = data.std(axis=1)
    return moy, std

# Calculs pour les différents scénarios
moy_hist_ete, std_hist_ete = calcul_moy_std(hist_ete_cum)
moy_hist_hiver, std_hist_hiver = calcul_moy_std(hist_hiver_cum)
moy_hist_annee, std_hist_annee = calcul_moy_std(hist_annee_cum)

moy_rcp4_ete, std_rcp4_ete = calcul_moy_std(RCP4_ete_cum)
moy_rcp4_hiver, std_rcp4_hiver = calcul_moy_std(RCP4_hiver_cum)
moy_rcp4_annee, std_rcp4_annee = calcul_moy_std(RCP4_annee_cum)

moy_rcp8_ete, std_rcp8_ete = calcul_moy_std(RCP8_ete_cum)
moy_rcp8_hiver, std_rcp8_hiver = calcul_moy_std(RCP8_hiver_cum)
moy_rcp8_annee, std_rcp8_annee = calcul_moy_std(RCP8_annee_cum)

def visualisation(Date, moy, std, label, color):
    """
    Fonction pour tracer les séries temporelles avec la moyenne et l'écart-type.
    """
    plt.plot(Date, moy, label=label, color=color)
    plt.fill_between(Date, moy - std,moy + std, color=color, alpha=0.3)

# Tracer les graphiques pour l'été, l'hiver et l'année
plt.figure(figsize=(15, 5))

# Été
plt.subplot(1, 3, 1)
visualisation(hist_ete_cum['Date'], moy_hist_ete, std_hist_ete, "Historique", "blue")
visualisation(RCP4_ete_cum['Date'], moy_rcp4_ete, std_rcp4_ete, "RCP4.5", "green")
visualisation(RCP8_ete_cum['Date'], moy_rcp8_ete, std_rcp8_ete, "RCP8.5", "red")
plt.title("Débits moyens en été")
plt.xlabel("Année")
plt.ylabel("Débit cumulé (m³/s)")
plt.legend()

# Hiver
plt.subplot(1, 3, 2)
visualisation(hist_hiver_cum['Date'], moy_hist_hiver, std_hist_hiver, "Historique", "blue")
visualisation(RCP4_hiver_cum['Date'], moy_rcp4_hiver, std_rcp4_hiver, "RCP4.5", "green")
visualisation(RCP8_hiver_cum['Date'], moy_rcp8_hiver, std_rcp8_hiver, "RCP8.5", "red")
plt.title("Débits moyens en hiver")
plt.xlabel("Année")
plt.ylabel("Débit cumulé (m³/s)")
plt.legend()

# Année
plt.subplot(1, 3, 3)
visualisation(hist_annee_cum['Date'], moy_hist_annee, std_hist_annee, "Historique", "blue")
visualisation(RCP4_annee_cum['Date'], moy_rcp4_annee, std_rcp4_annee, "RCP4.5", "green")
visualisation(RCP8_annee_cum['Date'], moy_rcp8_annee, std_rcp8_annee, "RCP8.5", "red")
plt.title("Débits moyens annuels")
plt.xlabel("Année")
plt.ylabel("Débit cumulé (m³/s)")
plt.legend()

plt.tight_layout()
plt.show()

```
