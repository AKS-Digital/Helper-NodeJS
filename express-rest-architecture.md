# Architecture API REST (Express) - ***Update : 5 Février 2021***

Express est un cadre d'application Web Node.js minimal et flexible qui fournit un ensemble robuste de fonctionnalités pour les applications Web et mobiles.

## Installation des dépendances

```
npm install dotenv express @types/express cors express-formidable @types/express-formidable
```

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


## Code minimal

Créer le fichier **src/index.ts** et copier :

```
import dotenv from "dotenv";
dotenv.config();

import server from "./server";

server();
```

Créer le fichier **src/server.ts** et copier :

```
import express from "express";
import cors from "cors";
import expressFormidable from "express-formidable";

// Import routes
import { defaultRoutes } from "./routes";

const app = express();

// Middlewares
app.use(cors());
app.use(expressFormidable());

// Routes use
app.use(defaultRoutes);

// Application start
app.listen(process.env.PORT, () => {
  console.log(
    `${process.env.APP_NAME} has started on port ${process.env.PORT}`
  );
});

export default () => app;
```

Créer le fichier **src/routes/index.ts** et copier :

```
import defaultRoutes from "./default";

export { defaultRoutes };
```


Créer le fichier **src/routes/default.ts** et copier :

```
import { Router } from "express";

const router = Router();

router.all("*", (req, res) => {
  return res.status(404).json({ status: 404, message: "Not Found" });
});

export default router;

```
