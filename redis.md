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

soit ,l'url http://127.0.0.1:8081
