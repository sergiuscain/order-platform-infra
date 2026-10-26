mermaid
graph TD
```
    Client[Клиент / Swagger UI] -->|HTTP| Gateway[API Gateway<br/>YARP + JWT]
    
    Gateway -->|/api/orders| OrderService[OrderService]
    Gateway -->|/api/products| InventoryService[InventoryService]
    Gateway -->|/auth| AuthService[AuthService]
    
    OrderService -->|HTTP Reserve/Release| InventoryService
    OrderService -->|Publish Events| RabbitMQ[RabbitMQ]
    InventoryService -->|Consume Events| RabbitMQ
    NotificationService[NotificationService] -->|Consume Events| RabbitMQ
    
    OrderService --> DB1[(PostgreSQL<br/>order_db)]
    InventoryService --> DB2[(PostgreSQL<br/>inventory_db)]
    AuthService --> DB3[(PostgreSQL<br/>auth_db)]
```
