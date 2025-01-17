# pymongo-api

## Как запустить

Не забудьте переключится в текущий каталог через cd mongo-sharding

Запускаем mongodb и приложение

```shell
docker compose up -d
```
Инициализация сервера конфигураций:
```shell
docker exec -it config mongosh --port 27017
```

```shell
rs.initiate({_id: "rs-config-server", configsvr: true, version: 1, members: [ { _id: 0, host : 'configSrv:27017' } ] });

exit();
```

Настраиваем шардирование 
Первый шард

```shell
docker exec -it shard01 mongosh --port 27019
```

```shell
rs.initiate( { _id: "shard-01", members: [{ _id: 0, host: "shard01:27019" } ] } );
exit();
```

Второй шард

```shell
docker exec -it shard02 mongosh --port 27020
```

```shell
rs.initiate( { _id: "shard-02", members: [{ _id: 0, host: "shard02:27020" } ] } );
exit();
```

Инициализация роутера, включение шардирования (хэшового) и наполнение его тестовыми данными:

```shell
docker exec -it router mongosh --port 27021
```

```shell
sh.addShard("shard-01/shard01:27019");
sh.addShard("shard-02/shard02:27020");

sh.enableSharding("somedb")
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

use somedb
for(var i = 0; i < 3000; i++) db.helloDoc.insert({age:i, name:"mango"+i})

exit();
```

Для проверки данных на шардах можно подключиться через:
```shell
docker exec -it shard01 mongosh --port 27022
use somedb
db.helloDoc.countDocuments()
```
```shell
docker exec -it shard02 mongosh --port 27020
use somedb
db.helloDoc.countDocuments()
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