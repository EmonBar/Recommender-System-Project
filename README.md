# Système de Recommandation de Paires de Films

## Présentation du Projet
Ce projet développe un système de recommandation de paires de films qui suggère des films basés sur des genres, des thèmes ou des préférences des spectateurs complémentaires. Il intègre des données de MovieLens et IMDb pour fournir des recommandations.

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
- Remplacement des valeurs manquantes dans la matrice par la moyenne des évaluations.

### Division des Données pour l'Évaluation du Modèle
- Division de l'ensemble de données en ensembles d'entraînement et de test (80% pour l'entraînement et 20% pour le test).

### Configuration et Évaluation du Modèle
- Configuration de l'algorithme SVD avec des hyperparamètres spécifiques :
  ```python
  reader = Reader(rating_scale=(1, 10))
  data = Dataset.load_from_df(train_data[['UserID', 'MovieID', 'CompositeRating']], reader)
  algo = SVD(n_factors=100, n_epochs=20, lr_all=0.005, reg_all=0.02)
  ```

Entraînement et évaluation du modèle en utilisant une validation croisée à 5 plis pour mesurer les performances avec RMSE et MAE.

### Résultats du Modèle

Entraînement et test du modèle SVD ont donné un RMSE de 0.8812 sur le jeu de test, indiquant une bonne précision des prédictions.

### Calcul des Scores pour les Couples

Définition d'une fonction couple_score pour prédire les notes des films pour deux utilisateurs et retourner la moyenne des deux scores

### Génération de Recommandations de Films pour les Couples

Génération des recommandations de films pour un couple en calculant les scores moyens et en retournant les 10 meilleurs films recommandés sous ce format :

```
('forrest gump', 8.936833259016685)
('saving private ryan', 8.906167567026777)
('schindler's list', 8.905076156655655)
('gladiator', 8.874940290546995)
('star wars: episode iv - a new hope', 8.853283426124957)
('12 angry men', 8.767683654508183)
('to kill a mockingbird', 8.735653013551273)
('it's a wonderful life', 8.69999852243148)
('sanjuro', 8.637534118104906)
('stop making sense', 8.634620824176773)
```