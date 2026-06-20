# mongo-sharding

## Как запустить

Запускаем mongodb и приложение

```shell
docker compose up -d
```

Заполняем mongodb данными и конфигурации

```shell
./mongo-init.sh
```

### Настройка сервера конфигурации
```shell
docker compose -f compose.yaml exec -T configSrv mongosh --port 27017 --quiet <<EOF
rs.initiate({_id : "config_server",configsvr: true,members: [{ _id : 0, host : "configSrv:27017" }]})
EOF
```

### Настройка шардов
```shell
docker compose -f compose.yaml exec -T shard1 mongosh --port 27018 --quiet <<EOF
rs.initiate({_id : "shard1",members: [{ _id : 0, host : "shard1:27018" }]})
EOF
```

```shell
docker compose -f compose.yaml exec -T shard2 mongosh --port 27019 --quiet <<EOF
rs.initiate({_id : "shard2",members: [{ _id : 1, host : "shard2:27019" }]})
EOF
```

### Настройка шардов на роутере
```shell
docker compose -f compose.yaml exec -T router mongosh --port 27020 --quiet <<EOF
sh.addShard("shard1/shard1:27018")
sh.addShard("shard2/shard2:27019")
EOF
```

### Настройка шардирования БД
```shell
docker compose -f compose.yaml exec -T router mongosh --port 27020 --quiet <<EOF
sh.enableSharding("somedb")
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )
EOF
```
### Наполнение БД тестовыми данными
```shell
docker compose -f compose.yaml exec -T router mongosh --port 27020 --quiet <<EOF
use somedb
for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})
EOF
```

### Проверка наполнения БД тестовыми данными
```shell
docker compose -f compose.yaml exec -T router mongosh --port 27020 --quiet <<EOF
use somedb
db.helloDoc.countDocuments()
EOF
```

### Проверка состояния БД
```shell
docker compose -f compose.yaml exec -T router mongosh --port 27020 --quiet <<EOF
use somedb
db.helloDoc.countDocuments()
db.helloDoc.getShardDistribution()
EOF
```

###  Проверка приложения
http://localhost:8080/helloDoc/users

### Остановка контейнера
```shell
docker compose -f compose.yaml down --rmi all --volumes --remove-orphans
```
---