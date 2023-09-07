# Архитектурное решение

## Facade
Фасад служит в качестве единой точки входа для клиента и скрывает детали внутренней реализации системы. Он координирует вызовы на соответствующие сервисы.

### Facade API

#### Создание нового пользователя
```
POST /user/create/
{
    "first_name": "John",
    "last_name": "Colinse",
    "email": "user@email.com", 
}

201 CREATED
{
    "user_id:" 93493453
}
```

####  Биллинг - пополнение счета
```
POST /money/put/
{
    "user_id:" 93493453,
    "amount": "1000",
}

202 ACCEPTED
{
    "balance:" 2000
}
```

#### Биллинг - проверка истории операций

```
GET /money/history?user_id=93493453

200 OK
[
    {
        "operation_id": "12345",
        "user_id": 93493453,
        "type": "refill",
        "amount": "1000",
        "timestamp": "2023-09-07T14:30:00Z"
    },
    {
        "operation_id": "12346",
        "user_id": 93493453,
        "type": "withdrawal",
        "amount": "500",
        "timestamp": "2023-09-08T09:45:00Z"
    }
]

```

#### Получение списка сообщений для пользователя

```
GET /notifications/messages?user_id=93493453

200 OK
[
    {
        "message_id": "67890",
        "user_id": 93493453,
        "status": "Sent",
        "message": "Ваш заказ #54321 ожидает подтверждения.",
        "timestamp": "2023-09-07T15:00:00Z"
    },
    {
        "message_id": "67891",
        "user_id": 93493453,
        "status": "Sent",
        "message": "Ваш заказ #54321 подтвержден и отправлен.",
        "timestamp": "2023-09-08T10:00:00Z"
    }
]
```

#### Создание нового заказа

```
POST /orders/create/
{
    "user_id": 93493453,
    "product_id": "12345",
    "quantity": 5
}

201 CREATED
{
    "order_id": "54321",
    "user_id": 93493453,
    "product_id": "12345",
    "quantity": 5,
    "status": "Pending"
}

```


## Order Service
Отвечает за создание и управление заказами. Order Service взаимодействует с Billing Service и Notification Service асинхронно через очередь сообщений MQ.

#### Создание нового пользователя
```
POST /user/create/
{
    "first_name": "John",
    "last_name": "Colinse",
    "email": "user@email.com", 
}

201 CREATED
{
    "user_id:" 93493453
}
```

#### Создание нового заказа

```
POST /orders/create/
{
    "user_id": 93493453,
    "product_id": "12345",
    "quantity": 5
}

201 CREATED
{
    "order_id": "54321",
    "user_id": 93493453,
    "product_id": "12345",
    "quantity": 5,
    "status": "Pending"
}

```

## Billing Service
Управляет деньгами на аккаунтах пользователей. 

####  Биллинг - пополнение счета
```
POST /money/put/
{
    "user_id:" 93493453,
    "amount": "1000",
}

202 ACCEPTED
{
    "balance:" 2000
}
```

#### Биллинг - проверка истории операций

```
GET /money/history?user_id=93493453

200 OK
[
    {
        "operation_id": "12345",
        "user_id": 93493453,
        "type": "refill",
        "amount": "1000",
        "timestamp": "2023-09-07T14:30:00Z"
    },
    {
        "operation_id": "12346",
        "user_id": 93493453,
        "type": "withdrawal",
        "amount": "500",
        "timestamp": "2023-09-08T09:45:00Z"
    }
]
```

## Notification Service
Отправляет уведомления по электронной почте.

#### Отправка уведомления
```
POST /notifications/send
{
    "user_id": "93493453",
    "message": "Ваш заказ #54321 подтвержден и отправлен.",
    "notification_type": "order_shipment",
    "timestamp": "2023-09-07T14:30:00Z"
}

202 ACCEPTED
```

#### Получение списка сообщений для пользователя

```
GET /notifications/messages?user_id=93493453

200 OK
[
    {
        "message_id": "67890",
        "user_id": 93493453,
        "status": "Sent",
        "message": "Ваш заказ #54321 ожидает подтверждения.",
        "timestamp": "2023-09-07T15:00:00Z"
    },
    {
        "message_id": "67891",
        "user_id": 93493453,
        "status": "Sent",
        "message": "Ваш заказ #54321 подтвержден и отправлен.",
        "timestamp": "2023-09-08T10:00:00Z"
    }
]
```

## MQ (Message Broker)
Брокер сообщений используется для асинхронного взаимодействия между сервисами и управления сообщениями.

#### API сервиса с использованием протокола AMQP (Advanced Message Queuing Protocol):
#### Создание пользователя и постановка в очередь
```
Operation: Create User
HTTP Method: POST
Endpoint: /mq/users
Request Body:
{
    "user_id": "unique_user_id",
    "first_name": "John",
    "last_name": "Doe",
    "email": "johndoe@example.com"
}
Response:
HTTP Status: 202 Accepted

```

#### Подписка на создание пользователя и чтение из очереди
```
Operation: Subscribe to User Creation
HTTP Method: POST
Endpoint: /mq/billing/subscribe
Request Body:
{
    "queue_name": "user_creation_queue"
}
Response:
HTTP Status: 200 OK
```

#### Создание заказа и постановка в очередь
```
Operation: Create Order
HTTP Method: POST
Endpoint: /mq/orders
Request Body:
{
    "order_id": "unique_order_id",
    "user_id": "user_id_associated_with_order",
    "order_details": {
        "product_name": "Product XYZ",
        "quantity": 3,
        "total_amount": 100.00
    }
}
Response:
HTTP Status: 202 Accepted
```

#### Списание средств для заказа и обработка из очереди
```
Operation: Withdraw Money for Order
HTTP Method: POST
Endpoint: /mq/withdrawals
Request Body:
{
    "order_id": "unique_order_id",
    "amount": 100.00
}
Response:
HTTP Status: 202 Accepted
```