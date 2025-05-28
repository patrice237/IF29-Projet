# Détection de profils suspects sur Twitter

## Objectif

Ce projet vise à identifier automatiquement les profils Twitter potentiellement suspects à partir de leurs comportements, caractéristiques de compte et contenus postés. Pour cela, deux approches de Machine Learning sont comparées :

- **Supervisée** : basée sur un jeu de données labellisé par nos soins selon des critères définis.
- **Non-supervisée** : basée sur la détection automatique de groupes ou comportements atypiques.

Les données sont stockées dans MongoDB et traitées par un pipeline Python qui extrait les variables utiles à l’analyse.

---

## Données

- **Source** : 2285 fichiers JSON contenant des tweets autour de la Coupe du Monde.
- **Format** : Semi-structuré (MongoDB)
- **Objectif** : Enrichir chaque utilisateur avec un ensemble de variables caractérisant son activité, son profil et ses interactions.

---

## Variables exploitées

Les variables suivantes ont été retenues pour leur fort **pouvoir prédictif** dans la détection de bots et de comportements anormaux. Elles sont issues soit directement des métadonnées Twitter, soit calculées à partir de celles-ci.

### Popularité & Réseau

- **`user.followers_count`** : Nombre de personnes qui suivent l’utilisateur. Un très faible nombre ou, au contraire, un chiffre démesuré peut trahir un compte créé artificiellement.
- **`user.friends_count`** : Nombre de comptes suivis. Permet de calculer le **ratio followers/friends**, utile pour détecter les comptes déséquilibrés (suivent beaucoup, peu suivis).
- **`user.listed_count`** : Nombre de fois où le compte a été ajouté à une liste. Les vrais comptes sont souvent listés, contrairement aux bots.

### Activité Historique

- **`user.statuses_count`** : Total de tweets envoyés. Un bot spammeur peut produire des centaines de tweets en peu de temps.
- **`user.created_at`** : Date de création du compte. Permet de calculer l’âge du compte et d’évaluer si son activité est normale pour son ancienneté.
- **`avg_tweets_per_day`** : Dérivé du volume de tweets et de l’ancienneté ; utilisé pour quantifier l’**agressivité** (cf. SPOT).

### Engagement Utilisateur

- **`user.favourites_count`** : Total de tweets aimés. Les bots "likent" rarement. À l’inverse, un excès de likes peut indiquer un **like-bot**.
- **`likes_per_day`** : Indicateur d’activité d’engagement sur le long terme.

### Configuration du Profil

- **`user.default_profile`** et **`user.default_profile_image`** : Les comptes frauduleux gardent souvent l’apparence par défaut (avatar œuf, thème standard).
- **`user.verified`** : Être vérifié (badge bleu) réduit fortement la probabilité que le compte soit un bot.
- **`user.description`** : Un champ vide ou trop court indique un faible effort de personnalisation → souvent le cas chez les bots.
- **`user.location`** : L’absence de localisation ou une incohérence géographique peut trahir un profil douteux.

### Contenu des Tweets

- **`retweet_count`, `favorite_count`** : Permettent de mesurer la réception du contenu. Les bots ont souvent des RT sans likes.
- **`retweeted`** : Détecte les comptes qui ne font **que** retweeter (activité très robotisée).
- **`entities.hashtags`, `entities.user_mentions`** : Densité d’éléments dans les tweets, liée à la visibilité. Les bots en abusent souvent.
- **`entities.urls`** : L’usage excessif d’URLs, notamment raccourcies, est un indicateur de **dangerosité** (ex. phishing, spam).
- **`possibly_sensitive`** : Indique si le tweet contient du contenu sensible. Une fréquence élevée est un indicateur de risque.
- **`avg_retweets_per_tweet`**, **`avg_favorites_per_tweet`**, **`percent_retweets`**, **`avg_hashtags_per_tweet`**, **`avg_mentions_per_tweet`**, **`avg_urls_per_tweet`**, **`sensitive_tweet_ratio`** : Mesures agrégées du comportement par tweet.

### Langue & Technologie utilisée

- **`lang`** : La langue principale utilisée par le compte. Une alternance excessive peut trahir un réseau automatisé international.
- **`source`** : Outil utilisé pour publier le tweet (navigateur, API, service tierce). Certains outils comme *IFTTT*, *dlvr.it*, ou *twittbot.net* sont typiques des bots.

### Variables dérivées clés

- **`ratio_followers_friends`** : Détecte les comptes déséquilibrés socialement.
- **`likes_per_day`**, **`tweets_per_day`** : Quantifie la pression d’activité sur la plateforme.
- **`has_description`, `has_location`** : Simplifie la vérification d’informations de base.
- **`has_default_avatar`** : Détecte le manque de personnalisation.
- **`user_account_age`** : Ancienneté du compte, clé pour repérer les profils jetables.

---

## Modèles Machine Learning

### Apprentissage supervisé

- **Modèle choisi** : SVM (Support Vector Machine)
- **Justification** : Efficace pour séparer deux classes (suspect / non suspect), performant même avec peu de données labellisées. Déjà utilisé dans SPOT.
- **Features utilisées** : Toutes les variables listées ci-dessus.
- **Label** : Créé manuellement par le groupe selon des seuils sur les métriques dérivées.

### Apprentissage non-supervisé

- **Modèle choisi** : ACP (PCA) + KMeans
- **Justification** : ACP pour réduire la dimension, KMeans pour regrouper les comptes selon leurs comportements. Permet d’évaluer la pertinence du labelling supervisé.
- **Features utilisées** : Variables normalisées, centrées, issues des indicateurs SPOT.

