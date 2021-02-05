
# BACKEND (NodeJS) - ***Update : 5 Février 2021***

**NodeJS** va nous permettre d'exécuter le code JavaScript sur la base du moteur **V8** de Google.

## Node

Téléchargez la version LTS (Long-term support) sur le site [https://nodejs.org/](https://nodejs.org/).

Pour connaitre la version de **git**

```zsh
node --version
```

Pour initialiser un projet **node**

```zsh
npm init -y
```

## Git

Pour initialiser un répertoire git

```zsh
git init
```

## Installation des dépendances

Installer les dépendances pour le projet

```
npm install typescript ts-node nodemon concurrently
```

## Package.json

Modifier le fichier **package.json** en insérant les lignes suivantes:

```
"scripts": {
    "dev": "concurrently -k -n \"Typescript,Node\" -p \"[{name}]\" -c \"blue,green\" \"tsc --watch\" \"NODE_ENV=dev nodemon dist/server.js\"",
    "start": "tsc && NODE_ENV=prod nodemon dist/server.js"
  },
```

## Création dossiers et fichiers

Créer un fichier **.gitignore** et copier le contenu suivant

```
node_modules
dist
```

Créer un fichier **tsconfig.json** et copier le contenu suivant

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "esModuleInterop": true,
    "target": "es6",
    "noImplicitAny": true,
    "moduleResolution": "node",
    "sourceMap": true,
    "outDir": "dist",
    "baseUrl": "."
  },
  "include": ["src/**/*"]
}
```

Créer un fichier **server.ts** dans un dossier **src**

```zsh
src/server.ts
```

## Lancement de l'application

Pour lancer l'application en mode production

```
npm start
```

Pour lancer l'application en mode développement
```
npm run dev
```

