# Redis

Redis garde en mémoire une structures de données et est open source (sous licence BSD), utilisé comme base de données, cache et message broker.

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

Créer un fichiers ***src/helpers/redis.ts*** et copier

```ts
import redis from "redis";

//Default Redis config
//localhost : 127.0.0.1 & port 6379
const client = redis.createClient({ port: 6379, host: "127.0.0.1" });

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

export default { open, close };
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


