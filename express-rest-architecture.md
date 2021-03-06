# Architecture API REST (Express) - ***Update : 5 Février 2021***

Express est un cadre d'application Web Node.js minimal et flexible qui fournit un ensemble robuste de fonctionnalités pour les applications Web et mobiles.

## Installation des dépendances

```
npm install dotenv express @types/express cors @types/cors http-errors @types/http-errors
```

| Package             | Version   | Description |
| :---                |    :---:  | :---        |
| dotenv              | 8.2.0     | Dotenv est un module sans dépendance qui charge les variables d'environnement d'un fichier .env dans process.env.|
| express             | 4.17.1    | The Express philosophy is to provide small, robust tooling for HTTP servers, making it a great solution for single page applications, websites, hybrids, or public HTTP APIs. |
| @types/express      | 4.17.11    | Package de définition des types pour express |
| cors                | 2.8.5     |CORS est un package node.js pour fournir un middleware Connect / Express qui peut être utilisé pour activer CORS avec diverses options.|
| @types/cors         | 2.8.9    |Package de définition des types pour cors|
| http-errors         | 1.8.0    |Créer des erreurs HTTP pour Express|
| @types/http-errors  | 1.8.0    |Package de définition des types pour http-errors|



## dotenv

Insérer dans le fichier **.gitignore**

```
.env
```

Créer le fichier **.env** à la racine du projet et insérer le code suivant

```
APP_NAME=<MY_APP_NAME>
PORT=10000
```


## Exemple

Créer le fichier **src/index.ts** et copier :

```ts
import dotenv from "dotenv";
dotenv.config();

import server from "./server";

server();
```

Créer le fichier **src/server.ts** et copier :

```ts
import { app } from "./app";

app.listen(process.env.PORT, () => {
  console.log(
    `${process.env.APP_NAME} has started on port ${process.env.PORT}`
  );
});

export default () => app;
```

Créer le fichier **src/app.ts** et copier :

```ts
import express, { NextFunction, Request, Response } from "express";
import createError from "http-errors";
import cors from "cors";

// Import routes
import { defaultRoutes, userRoutes } from "./routes";

export const app = express();

// Middlewares
app.use(cors());
app.use(express.json());

// Router
app.use(userRoutes);
app.use(defaultRoutes);

// Error handler middleware
app.use((err: any, req: Request, res: Response, next: NextFunction) => {
  try {
    if (err.isJoi) {
      throw new createError.UnprocessableEntity(err.details[0].message);
    } else {
      throw createError(err.status, err.message);
    }
  } catch (err) {
    res.status(err.status ?? 500).send(err);
  }
});
```

Créer le fichier **src/routes/index.ts** et copier :

```ts
import defaultRoutes from "./default";

export { defaultRoutes };
```


Créer le fichier **src/routes/default.ts** et copier :

```ts
import { Router } from "express";
import createError from "http-errors";

const router = Router();

router.all("*", (req, res, next) => {
  try {
    throw new createError.NotFound("Route doesn't exists");
  } catch (err) {
    next(err);
  }
});

export default router;
```
