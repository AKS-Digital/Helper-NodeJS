# Base de données (MongoDB) - ***Update : 2 Mars 2021***

**MongoDB** est une base de données **noSQL**.

## Installation

```zsh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" # Installera le logiciel https://brew.sh/
brew update
brew tap mongodb/brew
brew install mongodb-community@4.4
brew services start mongodb-community@4.4
```

## MongoDB Compass (macOS et Windows)

MongoDB Compass est une interface graphique permettant de manipuler des bases de données MongoDB.

- Allez sur le site [MongoDB Compass](https://www.mongodb.com/products/compass) pour télécharger ce logiciel.
- Cliquez sur Try it Now puis Download. Vous devrez remplir le formulaire pour pouvoir lancer le téléchargement.

### Vérification

> Pour vérifier votre installation de MongoDB, lancez MongoDB Compass et cliquez sur “Connect”. Si jamais il y a une erreur, essayez la commande **brew services restart mongodb-community** dans le terminal pour redémarrer MongoDB.
> 

## Créer une instance de base de données

Se rendre sur le site de [https://cloud.mongodb.com/](https://cloud.mongodb.com/).

- Dans l'onglet ***Atlas***, appuyer sur le bouton ***Build a cluster***
- Choisir la zone géographique pour le plan gratuit (frankfurt pour AWS)
- Saisir un nom au cluster
- Une fois le cluster créé, appuyer sur ***connect***
- Dans la méthode de connexion, choisir ***connect your application***, ***Node.js***
- Copier le lien est le placer dans le fichier .env


## Installation des packages

```zsh
npm install mongoose
```

## Example

Ajouter les champs suivant dans le fichier **.env**

```
MONGODB_URI=<MY_DATABASE_URI>
MONGODB_URI_DEV=mongodb://localhost:27017
```
Créer le fichier **src/db/mongodb.ts** et copier :

```ts
import mongoose from "mongoose";

const config = {
  prod: {
    url: process.env.MONGODB_URI,
    dbName: process.env.APP_NAME,
  },
  dev: {
    url: process.env.MONGODB_URI_DEV,
    dbName: process.env.APP_NAME + "-dev",
  },
  test: {
    url: process.env.MONGODB_URI_DEV,
    dbName: process.env.APP_NAME + "-test",
  },
};

// @ts-ignore
const { url, dbName } = config[process.env.NODE_ENV ?? "dev"];

const connect = async () => {
  await mongoose.connect(url.toString(), {
    dbName,
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false,
    useCreateIndex: true,
  });
};

const close = async () => {
  await mongoose.connection.close();
};

const dropDatabase = async () => {
  const status = await mongoose.connection.db.dropDatabase();
  console.log("Database droped :", status);
};

mongoose.connection.on("connected", () =>
  console.log(`${dbName} mongoose connection is opened at ${url}`)
);

mongoose.connection.on("error", (err) =>
  console.log("Mongoose connection ERROR : ", err.message)
);

mongoose.connection.on("disconnected", () =>
  console.log("Mongoose connection is closed")
);

export default { connect, close, dropDatabase };
```

Créer un fichier **src/db/index.ts** et copier :

```ts
export { default } from "./mongodb";
```

Insérer dans le fichier **src/index.ts** le code :

```ts
//... autres imports
import mongoDB from "./db";

//... code
await mongoDB.connect();
//... code serveur

process.on("SIGINT", async () => {
  await mongoDB.close();
  process.exit(0);
});
```
