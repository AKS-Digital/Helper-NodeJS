# Tests unitaire (JEST + Supertest)

- **Jest** est un framework de test développé par Facebook
- **Supertest** est une bibliothèque de test des requêtes https

## Installation

```zsh
npm install jest ts-jest supertest @types/supertest
```

## package.json

Insérer dans le fichier **package.json** le code suivant pour executer les tests sur tous les fichiers avec l'extension ***\*.test.js|ts*** se trouvant dans le répertoire **\_test\_**

```ts
"scripts": {
    //...
    "test": "jest",
    //...
  },
"jest": {
    "transform": {
      "^.+\\.ts$": "ts-jest"
    },
    "moduleFileExtensions": [
      "js",
      "ts"
    ],
    "testMatch": [
      "**/_test_/**/*.test.(ts|js)"
    ],
    "testEnvironment": "node"
  },
```
