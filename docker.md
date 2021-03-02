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
FROM node:14

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 10000

ENTRYPOINT npm start
```

Créer un fichier sans extension ***docker-compose.yml*** et copier:

```
version: "3"
services:
  app:
    container_name: airpol
    restart: always
    build: .
    ports:
      - "10000:10000"
    links:
      - mongo
  mongo:
    container_name: mongo
    image: mongo
    ports:
      - "27017:27017"
```

Lancer le conteneur docker avec la commande :

La clé ***-d*** est utile pour détaché l'instance docker et avoir le terminal libre

```zsh
docker-compose up -d
```

Arrêter l'instance docker avec la commande :

```zsh
docker-compose down
