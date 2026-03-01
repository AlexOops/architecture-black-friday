# architecture-black-friday
Проект демонстрирует эволюцию архитектуры MongoDB и приложения:
1. Sharding
2. Sharding + Replica Sets
3. Sharding + Replica + Redis
4. Service Discovery + API Gateway
5. CDN

`Каждый вариант реализован в отдельной директории.`

## Структура проекта

| Директория            | Назначение                                            |
| --------------------- | ----------------------------------------------------- |
| `mongo-sharding`      | Базовое шардирование                                  |
| `mongo-sharding-repl` | Шардирование + репликация                             |
| `sharding-repl-cache` | Финальная версия (Replica + Redis + Gateway + Consul) |
| `task1`               | C4 схема шардирования                                 |
| `task5`               | C4 схема с API Gateway + Consul                       |
| `task6`               | C4 схема с CDN                                        |

Схемы реализованы в формате PlantUML (.puml).
Дополнительно экспортированы изображения для удобного просмотра.

## Как запустить финальную реализацию
Как запустить финальную реализацию

`sharding-repl-cache`

Перейти в директорию

`cd sharding-repl-cache`

Поднять стенд

`docker compose up -d --build`

Проверить

http://localhost:8000/docs

### MongoDB

Используется:
* Config Replica Set
* 2 шарда
* По 3 реплики в каждом шарде
* Mongos router
* Redis для кеширования
* API Gateway для балансировки
* Consul для Service Discovery

Подробные шаги инициализации находятся в README соответствующих директорий.

## Архитектурные схемы

| Задание            | Файл  |
| -------------------| ---|
| `Sharding`   | `task1/Sharding.puml` |
| `Replica` | `task1/Sharding_Replica.puml`|
| `Replica + Redis` | `task1/Sharding_Replica_Redis.puml` |
| `Gateway + Consul`            | `task5/Sharding_Replica_Redis_Gateway_Consul.puml`|
| `CDN`            | `task6/Sharding_Replica_Redis_Gateway_Consul_CDN.puml`|


### Остановка

`docker compose down`
