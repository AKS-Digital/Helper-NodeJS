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

Connexion à la base de données.

```js
mongoose.connect("mongodb://localhost/exemple", {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});
```
