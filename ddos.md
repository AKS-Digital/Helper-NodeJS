# How to prevent a DDoS attack (or a Brute-force attack) - Update : 3 Mars 2021

Une attaque par déni de service (abr. DoS attack pour Denial of Service attack en anglais) est une attaque informatique ayant pour but de rendre indisponible un service, d'empêcher les utilisateurs légitimes d'un service de l'utiliser. À l’heure actuelle la grande majorité de ces attaques se font à partir de plusieurs sources, on parle alors d'attaque par déni de service distribuée (abr. DDoS attack pour Distributed Denial of Service attack).

## Installation des dépendances

```zsh
npm install express-rate-limit @types/express-rate-limit helmet
```

express-rate-limit permet de limiter le nombre de requête sur une période donnée.
Helmet peut aider à protéger votre application contre certaines vulnérabilités Web bien connues en définissant les en-têtes HTTP de manière appropriée.

## Création de limiters

dans un fichier ***src/limiters/global.limiter.ts***

```ts
import limiter from "express-rate-limit";
import createError from "http-errors";

export const globalLimiter = limiter({
  windowMs: 5000, // 5 seconds
  max: 5,
  message: new createError.TooManyRequests(),
});
```

Dans le fichier ***src/app.ts*** ajouter les lignes suivantes

```ts
import helmet from "helmet";

// Middlewares
app.use(globalLimiter);
app.use(helmet());
// ...
```
