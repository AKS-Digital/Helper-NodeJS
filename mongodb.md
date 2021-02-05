# Base de données (MongoDB) - ***Update : 5 Février 2021***

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

const mode = process.env.NODE_ENV;
const url =
  mode === "prod" ? process.env.MONGODB_URI : process.env.MONGODB_URI_DEV;
const dbName =
  mode === "prod" ? process.env.APP_NAME : process.env.APP_NAME + "-dev";

const connect = () => {
  mongoose
    .connect(url.toString(), {
      dbName,
      useNewUrlParser: true,
      useUnifiedTopology: true,
      useFindAndModify: false,
      useCreateIndex: true,
    })
    .then(() => console.log(`Database connected at ${url}`))
    .catch((err) => console.log(err.message));
};

const close = async () => {
  await mongoose.connection.close();
};

mongoose.connection.on("connected", () =>
  console.log("Mongoose connection is opened")
);

mongoose.connection.on("error", (err) => console.log(err.message));

mongoose.connection.on("disconnected", () =>
  console.log("Mongoose connection is closed")
);

process.on("SIGINT", async () => {
  await close();
  process.exit(0);
});

export default { connect, close };
```

Créer un fichier **src/db/index.ts** et copier :

```ts
import mongodb from "./mongodb";

export default mongodb;
```

Insérer dans le fichier **src/index.ts** le code :

```ts
//... autres imports
import mongoDB from "./db";

//... code
mongoDB.connect();
//... code serveur
```
