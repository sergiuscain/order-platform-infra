# Progress Log
## 23.10.26:
### Inventory Service 
Реализовал InventoryService:
- Настроена база данных PostgreSQL
- Реализованы сущности `Product` и `Stock` (связь один-к-одному)
- Реализован базовый CRUD для товаров
- Настроены миграции EF Core.
- Добавлен Dockerfile
### Compose
- Создал Docker Compose файл для Inventory Service и Базы данных для него
- Выгрузил контейнер с Inventory Service на GitHub (ghcr.io) 
- А потом запустил на сервере на Виртуальной машине

## 24.10.26:
### OrderService
