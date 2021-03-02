# Docker

Docker est un conteneur d'image léger servant à encapsuler NodeJS, mongodb,...

## Installation de docker

Se rendre le site de [docker](https://docs.docker.com/docker-for-mac/install/) et installer docker.

## Création du fichier ***Dockerfile***

Créer un fichier ***.dockerignore*** et copier 

```
node_modules
Dockerfile
.dockerignore
```

Créer un fichier sans extension ***Dockerfile*** et copier:

```
FROM node:lastest

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm run dev"]
```

Lancer le conteneur docker avec la commande :

```zsh
docker build -t airpol
```
