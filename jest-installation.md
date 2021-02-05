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

## Example

Créer un fichier dans **\_test\_server.test.ts** le code :

```ts
import dotenv from "dotenv";
dotenv.config();
import supertest from "supertest";
import mongoDB from "../db";
import { app } from "../app";

const request = supertest(app);

beforeAll(() => {
  mongoDB.connect();
});

describe("Server", () => {
  it("should test server", async (done) => {
    const response = await request.get("/unknown-route");
    expect(response.status).toBe(404);
    expect(response.body).toStrictEqual({ status: 404, message: "Not Found" });
    done();
  });
});

afterAll(() => {
  mongoDB.close();
});
```

Lancer les tests

```zsh
npm run test
```
