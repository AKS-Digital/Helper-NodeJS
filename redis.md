# Redis

Redis garde en mémoire une structures de données et est open source (sous licence BSD), utilisé comme base de données, cache et message broker.

:fire: Attention : Redis redémarre toutes les 5 minutes à cause du [timeout](https://devcenter.heroku.com/articles/heroku-redis#timeout). à vérifier si cela ne provient pas du free-plan.

## Installation

```zsh
brew install redis
brew services start redis
```

Nous allons installer ***redis-commander*** globalement car ce n'est pas un dépendance pour notre code. Il sert à visualiser nos données présent dans Redis.

```zsh
npm install -g redis-commander
redis-commander
```

Pour visuler Redis, saisir sur le navigateur l'url issue de la commande redis-commander

```zsh
tomaks@MacBook-Pro-de-Tomaks ~ % redis-commander
Using scan instead of keys
No Save: false
listening on 0.0.0.0:8081
access with browser at http://127.0.0.1:8081
Redis Connection localhost:6379 using Redis DB #0
```

soit l'url : http://127.0.0.1:8081

## Connecter un client Redis à NodeJS

### Installation des dépendances

```zsh
npm install redis @types/redis
```

### Intégration dans NodeJS

Ajouter dans le fichier .env 

```
REDIS_URL=redis://127.0.0.1
```

Créer un fichiers ***src/helpers/redis.ts*** et copier

```ts
import redis from "redis";

//Default Redis config
//localhost : 127.0.0.1 & port 6379
const client = redis.createClient(process.env.REDIS_URL);

const open = () => {
  client.on("connect", () => {
    console.log("Attempt to connect to Redis Client...");
  });

  client.on("ready", () => {
    console.log("Redis client ready");
  });

  client.on("error", (err) => {
    console.log(err.message);
  });

  client.on("end", () => {
    console.log("Client disconnect from redis...");
  });
};

const close = () => {
  client.quit();
};

export const Redis = { open, close, client };
```

Ajouter dans ***src/index.ts***

```ts
import { Redis } from "./helpers";

(async () => {
  // ...
  Redis.open();
  // ...
})();

process.on("SIGINT", async () => {
  // ...
  Redis.close();
  process.exit(0);
});
```

### Utilisation de Redis pour blacklister les refreshToken

Modifier le code dans ***src/helpers/auth.token.ts***

```ts
export const signRefreshToken = (userId: string) => {
  return new Promise((resolve, reject) => {
    const payload = { name: process.env.APP_NAME };
    const secret = process.env.REFRESH_TOKEN_SECRET;
    const options = {
      expiresIn: process.env.REFRESH_TOKEN_EXPIRES_IN,
      issuer: "jwt.io",
      audience: userId,
    };
    JWT.sign(payload, secret, options, (err, token) => {
      if (err) return reject(new createError.InternalServerError());
      Redis.client.SET(userId, token, "EX", 365 * 24 * 60 * 60, (err) => {
        if (err) {
          console.log(err.message);
          return reject(new createError.InternalServerError());
        }
        resolve(token);
      });
    });
  });
};

export const verifyRefreshToken = (refreshToken: string) => {
  return new Promise((resolve, reject) => {
    JWT.verify(
      refreshToken,
      process.env.REFRESH_TOKEN_SECRET,
      (err, decoded) => {
        if (err) return reject(new createError.Unauthorized());
        // @ts-ignore
        const userId = decoded.aud;

        Redis.client.GET(userId, (err, result) => {
          if (err) {
            console.log(err.message);
            return reject(new createError.InternalServerError());
          }
          if (refreshToken === result) return resolve(userId);
          reject(new createError.Unauthorized());
        });
      }
    );
  });
};
```


