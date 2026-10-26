# Order Platform — Microservices Architecture

Учебный проект, демонстрирующий полноценную микросервисную архитектуру с единой точкой входа, асинхронным обменом сообщениями, разграничением ролей и изолированными базами данных.

## 📐 Архитектура

Проект построен по принципам Domain-Driven Design (DDD) и Clean Architecture. Каждый сервис изолирован, имеет собственную базу данных и общается с другими сервисами либо синхронно (через HTTP), либо асинхронно (через брокер сообщений).

# Сервисы
Сервис	Порт (внутренний)	Зона ответственности	База данных
ApiGateway	8080 (внешний)	Единая точка входа, маршрутизация (YARP), валидация JWT, агрегация Swagger UI.	—
OrderService	8080	Управление заказами, оркестрация резервирования товаров, публикация событий.	order_db
InventoryService	8080	Управление каталогом товаров и складскими остатками (резервы, списание).	inventory_db
AuthService	8080	Аутентификация, выдача JWT (Access + Refresh), управление пользователями и ролями.	auth_db
NotificationService	—	Асинхронная обработка событий (имитация отправки Email/Push).	—
RabbitMQ	5672 / 15672	Брокер сообщений (event-driven).	—
🛠 Технологический стек
Платформа: .NET 10 (C#)

## Фреймворк: ASP.NET Core Web API

## Базы данных: PostgreSQL 17 + Entity Framework Core 10

## API Gateway: YARP (Yet Another Reverse Proxy)

## Брокер сообщений: RabbitMQ + MassTransit 8.4.1

## Аутентификация: JWT (HS256), HttpOnly Cookies, Refresh Tokens

## Архитектурные паттерны: Clean Architecture, CQRS (MediatR), Event-Driven Architecture, Repository/Unit of Work (через EF Core)

## Инфраструктура: Docker, Docker Compose

🔄 Ключевые бизнес-потоки
1. Создание заказа (Saga / Orchestration)
Клиент отправляет POST /api/orders через Gateway.

OrderService валидирует запрос и запрашивает актуальные данные товаров у InventoryService.

OrderService вызывает InventoryService для резервирования остатков (ReserveStock).

OrderService сохраняет заказ в БД и публикует событие OrderCreatedEvent в RabbitMQ.

NotificationService ловит событие и "отправляет" уведомление клиенту.

2. Завершение заказа
Администратор отправляет POST /api/orders/{id}/complete.

OrderService меняет статус на Done и публикует OrderCompletedEvent.

InventoryService ловит событие и окончательно списывает зарезервированный товар со склада.

3. Отмена заказа
Клиент отправляет POST /api/orders/{id}/cancel.

OrderService меняет статус на Cancelled и вызывает InventoryService для снятия резерва (ReleaseStock).

🔐 Безопасность
Аутентификация: Реализована в AuthService с использованием JWT. Поддерживается передача токена как через заголовок Authorization: Bearer, так и через HttpOnly Cookie (для браузерных клиентов).

Авторизация: Ролевая модель (User / Admin).

API Gateway: Валидирует JWT на своей стороне. Если токен валиден, Gateway извлекает UserId и Roles и передает их внутренним сервисам через заголовки X-User-Id и X-User-Roles. Внутренние сервисы доверяют этим заголовкам (так как порты наружу не проброшены) и используют кастомный HeaderAuthenticationHandler.

🚀 Запуск проекта
Существует два способа запуска проекта в зависимости от ваших целей.

Вариант 1: Продакшн-подобный (из готовых образов)
Используется, если образы уже собраны и опубликованы (например, в GitHub Container Registry). Базы данных и сервисы поднимаются из готовых образов.

bash
# Перейдите в директорию инфраструктуры
cd repos/order-platform-infra

# Запустите контейнеры
docker compose -f docker-compose.yml up -d
Вариант 2: Тестирование локальных изменений (сборка из Dockerfile)
Используется во время разработки, когда нужно протестировать изменения до коммита. Docker Compose собирает образы из локальных исходников (Dockerfile).

Важно: Все репозитории с сервисами должны быть склонированы в одну папку (рядом с order-platform-infra).

bash
# Перейдите в директорию инфраструктуры
cd repos/order-platform-infra

# Запустите сборку и поднятие контейнеров
docker compose -f docker-compose.test.yml up --build -d
Доступные интерфейсы после запуска:
Агрегированный Swagger UI: http://localhost:8080/swagger

RabbitMQ Management: http://localhost:15672 (логин: guest, пароль: guest)

Тестовые учетные записи (Seed):
При первом запуске AuthService автоматически создает администратора:

Email: admin@local

Пароль: Admin123!

(Вы можете зарегистрировать обычного пользователя через POST /auth/register в Swagger).

📂 Структура репозиториев
Проект разделен на независимые репозитории (полирепозиторий):

[order-platform-infra] (текущий) — Docker Compose, конфигурация инфраструктуры, README.

[OrderService] — сервис заказов (Clean Architecture).

[InventoryService] — сервис склада (Clean Architecture).

[AuthService] — сервис аутентификации (Clean Architecture).

[ApiGateway] — шлюз на YARP.

[NotificationService] — консьюмер событий.

📝 Заметки по разработке
Docker контекст: Если вы добавляете новые NuGet-пакеты, перед сборкой Docker-образа убедитесь, что локальные папки bin/ и obj/ удалены, либо выполните "Пересобрать решение" в Visual Studio. Это предотвратит попадание устаревшего project.assets.json в контейнер.

События: Для общих контрактов событий (например, OrderCompletedEvent) используется идентичное пространство имен в публикующем и потребляющем сервисах, чтобы MassTransit мог корректно маршрутизировать сообщения без дополнительных настроек MessageTopology.
