# Heroku

Heroku est notre hebergeur de serveur.

## CI/CD avec Github

- Appuyer sur ***New/create new app***
- Dans l'onglet ***Deploy***
  - Deployment Method : Github
  - App connected to GitHub : Repertoire du serveur backend (besoin de se connecter à Github avant)
  - Automatic deploys: Autoriser le déploiement automatique
  - Manual deploy: Choisir la branche 'Main' et cliquer sur ***Deploy***

à chaque commit sur la branche Main, le serveur recompilera le projet et sera actualisé.

## Intégration de Heroku Redis

Sous l'onglet ***ressource***, rechercher l'add-on ***Heroku Redis*** et l'intégrer au projet

## Variables de configuration

Cette partie est importante car il faut mettre toutes les clés utiles à notre application et qui se trouve dans le fichier .env

Aller dans l'onglet ***Settings*** et appuyer sur ***Reveal config vars*** et saisir les clés

- APP_NAME : <NOM_APP>

### Intégration de mongoDB Atlas

- MONGODB_URI : <LIEN_MONGODB_ATLAS>

### Intégration de Heroku Redis

- REDIS_TLS_URL : Modifier par Heroku automatiquement
- REDIS_URL : Modifier par Heroku automatiquement

### Clés de l'application

- ACCESS_TOKEN_SECRET : la clé SHA-256 générée
- REFRESH_TOKEN_SECRET : la clé SHA-256 générée
- ACCESS_TOKEN_EXPIRES_IN : 1d
- REFRESH_TOKEN_EXPIRES_IN : 7d




