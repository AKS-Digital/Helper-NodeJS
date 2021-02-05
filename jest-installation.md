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
      "**/_test_/**/*.test.(ts)"
    ],
    "testEnvironment": "node"
  },
```

## Example

Créer un fichier dans **\_test\_/setup.test.ts** le code :

```ts
import dotenv from "dotenv";
dotenv.config();
import { mongoDB } from "../db";

beforeAll(async () => {
  await mongoDB.connect();
});

afterAll(async () => {
  await mongoDB.close();
});

describe("initialization", () => {
  test("Database initialization", () => console.log("init mongoDB"));
});
```

Créer un fichier dans **\_test\_/server.test.ts** le code :

```ts
import supertest from "supertest";
import { app } from "../app";

const request = supertest(app);

describe("Server", () => {
  it("should test unknown get routes", async (done) => {
    const response = await request.get("/unknown-route");
    expect(response.status).toBe(404);
    expect(response.body).toStrictEqual({ message: "Route doesn't exists" });
    done();
  });
  it("should test unknown post route", async (done) => {
    const response = await request.post("/unknown-route").send({ foo: "bar" });
    expect(response.status).toBe(404);
    expect(response.body).toStrictEqual({ message: "Route doesn't exists" });
    done();
  });
});
```

Lancer les tests

```zsh
npm run test
```
