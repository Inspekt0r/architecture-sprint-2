# pymongo-api

## Как запустить

Запускаем mongodb и приложение

```shell
docker compose up -d
```

Проверяем в redis каталоге что есть файл redis.conf с содержимым:
```
port 6379
cluster-enabled no
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

Инициализируем конфиг сервер:
```shell
 docker exec -it config-01 mongosh --port 27017
```

```shell
rs.initiate({_id: "rs-config-server", configsvr: true, version: 1, members: [ { _id: 0, host : 'configSrv01:27017' }, { _id: 1, host : 'configSrv02:27017' }, { _id: 2, host : 'configSrv03:27017' } ] });

exit();
```

Настраиваем шардирование 

Первый шард

```shell
docker exec -it shard01-node-a mongosh --port 27019
```

```shell
rs.initiate( { _id: "rs0", members: [ {_id: 0, host: "shard01-a:27019"} , { _id: 1, host: "shard01-b:27019"}, {_id: 2, host: "shard01-c:27019"}]}); 

exit();
```

Второй шард

```shell
docker exec -it shard02-node-a mongosh --port 27020
```

```shell
rs.initiate( { _id: "rs1", members: [ {_id: 0, host: "shard02-a:27020"} , { _id: 1, host: "shard02-b:27020"}, {_id: 2, host: "shard02-c:27020"}]}); 
exit();
```

Третий шард

```shell
docker exec -it shard03-node-a mongosh --port 27022
```

```shell
rs.initiate( { _id: "rs2", members: [ {_id: 0, host: "shard03-a:27022"} , { _id: 1, host: "shard03-b:27022"}, {_id: 2, host: "shard03-c:27022"}]}); 
exit();
```


Инициализация роутера, включение шардирования (хэшового) и наполнение его тестовыми данными:

```shell
docker exec -it router-01 mongosh --port 27021
```

```shell
sh.addShard("rs0/shard01-a:27019");
sh.addShard("rs1/shard02-a:27020");
sh.addShard("rs2/shard03-a:27022");

sh.enableSharding("somedb")
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

use somedb
for(var i = 0; i < 10000; i++) db.helloDoc.insert({age:i, name:"mango"+i})

exit();
```
Здесь нужно немного подождать на цикле for т.к. кол-во документов большое, выполняться команда будет какое-то время, если нет времени ждать
то просто 10000 уменьшить можно до 5000 или 1000 =)

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