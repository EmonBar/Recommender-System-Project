# Recommender System Project

## Présentation du Projet
Ce projet développe un système de recommandation de paires de films qui suggère des films. 
Il intègre des données de MovieLens et IMDb pour fournir des recommandations.

## Jeux de Données
Les jeux de données suivants ont été utilisés :
- [Jeu de données MovieLens](https://grouplens.org/datasets/movielens/)
- [Jeu de données IMDb](https://datasets.imdbws.com/)

## Collecte et Prétraitement des Données

### Téléchargement et Fusion des Jeux de Données
- Importation des jeux de données de MovieLens et IMDb.
- Fusion de ces jeux de données pour inclure les évaluations des films, les genres et les métadonnées pour une analyse complète.

### Méthodologie
Pour le jeu de données MovieLens :
- Les colonnes `timestamp`, `occupation` et `zipcode` ont été retirées car elles n'étaient pas pertinentes pour notre analyse.

Pour le jeu de données IMDb, les fichiers suivants ont été utilisés :
- `title.basics.tsv` et `title.ratings.tsv`

#### Détails des fichiers IMDb utilisés :
- **title.basics.tsv**
    - `tconst` (identifiant unique)
    - `titleType` (type de titre)
    - `primaryTitle` (titre principal)
    - `originalTitle` (titre original)
    - `isAdult` (titre adulte ou non)
    - `startYear` (année de début)
    - `endYear` (année de fin)
    - `runtimeMinutes` (durée en minutes)
    - `genres` (genres associés)

- **title.ratings.tsv**
    - `tconst` (identifiant unique)
    - `averageRating` (note moyenne)
    - `numVotes` (nombre de votes)

J'ai décidé de ne pas utiliser les fichiers suivants car ils n'étaient pas pertinents pour notre analyse :
- `title.akas.tsv`
- `title.crew.tsv`
- `title.episode.tsv`
- `title.principals.tsv`
- `name.basics.tsv`

Je voulais utiliser title.crew.tsv pour obtenir les directeurs mais je me suis dit que ce n'était pas utile pour le feature engineering car personnellement je ne m'intéresse pas aux directeurs pour voir un film.

### Nettoyage des Données
- Suppression des colonnes `endYear`, `runtimeMinutes`, `isAdult` et `numVotes` car elles n'étaient pas pertinentes.
- Remplacement des valeurs `\N` par `NaN` ou 'Unknown' selon le contexte.
- Création d'un titre propre avec l'année correcte pour IMDb et MovieLens.
- Fusion des deux ensembles de données en faisant correspondre le titre et l'année.
- Normalisation des ratings des deux ensembles de données sur une échelle de 1 à 10, puis calcul de la moyenne des deux notes pour obtenir une note composite.
- Suppression des colonnes inutiles après la fusion.

### Feature Engineering
- Utilisation des genres des films (Action, Horreur, etc.) comme caractéristiques principales pour le système de recommandation.
- Utilisation du genre des utilisateurs pour adapter les recommandations (par exemple, deux femmes pourraient préférer des films moins orientés "garçon").
- Prise en compte de l'âge des utilisateurs pour recommander des films adaptés à leur groupe d'âge.

### Création de la Matrice d'Interaction User-Item
- Création d'une table pivot avec `UserID`, `CleanTitle` et `CompositeRating` pour former une matrice d'interaction user-item.

## Modélisation
### Configuration du Modèle
- Utilisation de l'algorithme SVD de la bibliothèque Surprise pour prédire les évaluations non observées, avec des hyperparamètres spécifiques ajustés pour optimiser les performances.

### Division des Données
- Séparation des données en ensembles d'entraînement et de test pour évaluer la performance du modèle.

### Évaluation du Modèle
- Le modèle est évalué en utilisant le RMSE et le MAE pour quantifier les erreurs de prédiction. Efficace pour mesurer l'exactitude des prédictions du modèle ainsi qu'avoir une mesure directe de l'erreur moyenne dans les prédictions.
- J'ai aussi mis la précision, le rappel et le f1_score pour plus de précisions.

## Résultats et Discussion

Après avoir évalué notre système de recommandation, les résultats confirment une forte précision dans nos prédictions de films :

- **RMSE (Erreur Quadratique Moyenne) :** 0.8799 - Les erreurs entre les notes prédites et réelles sont faibles, en moyenne inférieures à 1 sur une échelle de 10, ce qui témoigne de la précision du modèle.

- **MAE (Erreur Absolue Moyenne) :** 0.6934 - L'erreur moyenne des prédictions est moins de 0.7 sur 10, indiquant que les prédictions sont proches des notes réelles.

- **Précision Moyenne :** 0.9933 - La majorité des recommandations sont jugées pertinentes, signifiant que les films suggérés sont susceptibles de plaire aux utilisateurs.

- **Rappel Moyen :** 0.6740 - Le système recommande environ 67% des films que les utilisateurs pourraient apprécier, indiquant une bonne couverture des intérêts des utilisateurs, bien que cela puisse être amélioré.

- **Score F1 à k :** 0.8031 - Ce score équilibré entre la précision et le rappel montre que le système fait un bon compromis entre recommander des films pertinents et couvrir les préférences des utilisateurs.

## Recommandations
- Génération des recommandations pour les couples en utilisant les scores moyens prévus pour sélectionner les 10 meilleurs films.
- Exemple de sortie des recommandations présentant une liste des films avec les scores moyens associés.

Exemple :
```
                        MovieTitle  Average_Prediction
0                 schindler's list            8.963083
1  one flew over the cuckoo's nest            8.836204
2            it's a wonderful life            8.821984
3                     pulp fiction            8.810514
4                     12 angry men            8.807151
5                      rear window            8.791502
6                       casablanca            8.764301
7                stop making sense            8.757113
8                          sanjuro            8.734830
9            to kill a mockingbird            8.734041
```