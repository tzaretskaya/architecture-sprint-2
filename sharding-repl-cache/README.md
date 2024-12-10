# pymongo-api

Схема находится в корневой папке. Файл task6.drawio

## Как запустить

Запускаем mongodb и приложение:

```shell
docker compose up -d
```
Подключитесь к серверу конфигурации и сделайте инициализацию:

```shell
docker exec -it configSrv mongosh --port 27017

> rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);
> exit();
```
Инициализируйте шарды:
```shell
docker exec -it shard1-1 mongosh --port 27018

> rs.initiate(
    {
      _id : "shard1",
      members: [
        { _id : 0, host : "shard1-1:27018" },
        { _id : 1, host : "shard1-2:27021" },
        { _id : 2, host : "shard1-3:27023" },
      ]
    }
);
> exit();

docker exec -it shard2-1 mongosh --port 27019

> rs.initiate(
    {
      _id : "shard2",
      members: [
        { _id : 3, host : "shard2-1:27019" },
        { _id : 4, host : "shard2-2:27022" },
        { _id : 5, host : "shard2-3:27024" },
      ]
    }
  );
> exit();
```

Инцициализируйте роутер:
```shell
docker exec -it mongos_router mongosh --port 27020

> sh.addShard( "shard1/shard1-1:27018,shard1-2:27021,shard1-3:27023");
> sh.addShard( "shard2/shard2-1:27019,shard2-2:27022,shard2-3:27024");

> sh.enableSharding("somedb");
> sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )
> exit();
```

Заполняем mongodb данными

```shell
./scripts/mongo-init.sh
```

Получится результат — 1000 документов.

Сделайте проверку на шардах:

```shell
 docker exec -it shard1-1 mongosh --port 27018
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit();
```
Получится результат — 492 документа.

```shell
 docker exec -it shard1-2 mongosh --port 27021
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit();
```
Получится результат — 492 документа.

```shell
 docker exec -it shard1-3 mongosh --port 27023
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit();
```
Получится результат — 492 документа.

Сделайте проверку на втором шарде:
```shell
docker exec -it shard2-1 mongosh --port 27019
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit();
```
Получится результат — 508 документов.

```shell
docker exec -it shard2-2 mongosh --port 27022
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit();
```
Получится результат — 508 документов.

```shell
docker exec -it shard2-3 mongosh --port 27024
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit();
```
Получится результат — 508 документов.

### Настройка и проверка Redis:
```shell
docker exec -it redis_1 bash
echo "yes" | redis-cli --cluster create 173.17.0.2:6379 --cluster-replicas 1 
```

## Как проверить

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8080

### Если вы запускаете проект на предоставленной виртуальной машине

Узнать белый ip виртуальной машины

```shell
curl --silent http://ifconfig.me
```

Откройте в браузере http://<ip виртуальной машины>:8080

## Доступные эндпоинты

Список доступных эндпоинтов, swagger http://<ip виртуальной машины>:8080/docs