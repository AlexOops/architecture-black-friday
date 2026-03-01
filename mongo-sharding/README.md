# Задание 2 — Шардирование MongoDB

## Архитектура

Используются:

- `mongos` — маршрутизатор
- `configSrv-1..3` — Config Servers (Replica Set `configRS`)
- `shard1` — Replica Set `shard1RS`
- `shard2` — Replica Set `shard2RS`
- БД: `somedb`
- Коллекция: `helloDoc`

---

## 1. Запуск

```bash
docker compose up -d --build
```

## 2. Инициализация Config Servers

```bash
docker compose exec -T configSrv-1 mongosh --port 27019 <<EOF
rs.initiate({
_id: "configRS",
configsvr: true,
members: [
    { _id: 0, host: "configSrv-1:27019" },
    { _id: 1, host: "configSrv-2:27019" },
    { _id: 2, host: "configSrv-3:27019" }
]
})
EOF
```

## 3. Инициализация шардов

```bash
docker compose exec -T shard1 mongosh --port 27018 <<EOF
rs.initiate({ _id: "shard1RS", members: [{ _id: 0, host: "shard1:27018" }] })
EOF

docker compose exec -T shard2 mongosh --port 27018 <<EOF
rs.initiate({ _id: "shard2RS", members: [{ _id: 0, host: "shard2:27018" }] })
EOF
```

## 4. Добавление шардов

```bash
docker compose exec -T mongos mongosh --port 27017 <<EOF
sh.addShard("shard1RS/shard1:27018")
sh.addShard("shard2RS/shard2:27018")
EOF
```

## 5. Включение шардинга

```bash
docker compose exec -T mongos mongosh --port 27017 <<EOF
sh.enableSharding("somedb")
sh.shardCollection("somedb.helloDoc", { _id: "hashed" })
EOF
```

## 6. Заполнение данными

```bash
docker compose exec -T mongos mongosh --port 27017 <<EOF
use somedb
for (let i = 0; i < 2000; i++) {
  db.helloDoc.insertOne({ value: i })
}
EOF
```

## Проверка

### Общее количество:

```bash
docker compose exec -T mongos mongosh --port 27017 --eval "use somedb; db.helloDoc.countDocuments()"
```

### Распределение по шардам:

```bash
docker compose exec -T mongos mongosh --port 27017 --eval "use somedb; db.helloDoc.getShardDistribution()"
```

## Остановка

```bash
docker compose down
```

-------------------