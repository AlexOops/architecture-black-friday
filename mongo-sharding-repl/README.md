# Задание 3 — Репликация MongoDB (mongo-sharding-repl)

Цель: настроить репликацию для каждого шарда в шардированном кластере MongoDB.
БД: `somedb`, коллекция: `helloDoc`.

---

## 1. Запуск

```bash
docker compose up -d --build
```

## 2. Инициализация config servers (Replica Set)

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

## 3. Инициализация Replica Set для шардов

### Shard 1 (shard1RS)

```bash
docker compose exec -T shard1-1 mongosh --port 27018 <<EOF
rs.initiate({
  _id: "shard1RS",
  members: [
    { _id: 0, host: "shard1-1:27018" },
    { _id: 1, host: "shard1-2:27018" },
    { _id: 2, host: "shard1-3:27018" }
  ]
})
EOF
```

### Shard 2 (shard2RS)

```bash
docker compose exec -T shard2-1 mongosh --port 27018 <<EOF
rs.initiate({
  _id: "shard2RS",
  members: [
    { _id: 0, host: "shard2-1:27018" },
    { _id: 1, host: "shard2-2:27018" },
    { _id: 2, host: "shard2-3:27018" }
  ]
})
EOF
```

## 4. Добавление шардов в кластер через mongos

```bash
docker compose exec -T mongos mongosh --port 27017 <<EOF
sh.addShard("shard1RS/shard1-1:27018,shard1-2:27018,shard1-3:27018")
sh.addShard("shard2RS/shard2-1:27018,shard2-2:27018,shard2-3:27018")
EOF
```

## 5. Включение шардинга и создание коллекции

```bash
docker compose exec -T mongos mongosh --port 27017 <<EOF
sh.enableSharding("somedb")
sh.shardCollection("somedb.helloDoc", { _id: "hashed" })
EOF
```

## 6. Заполнение данными (>= 1000)

```bash
docker compose exec -T mongos mongosh --port 27017 <<EOF
use somedb
for (let i = 0; i < 2000; i++) { db.helloDoc.insertOne({ value: i }) }
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

### Количество реплик (проверка RS)
```bash
docker compose exec -T shard1-1 mongosh --port 27018 --eval "rs.status().members.length"
docker compose exec -T shard2-1 mongosh --port 27018 --eval "rs.status().members.length"
```

## Остановка

```bash
docker compose down
```

-------------------