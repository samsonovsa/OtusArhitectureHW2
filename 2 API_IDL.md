# Описание API сервисов в формате IDL

```
openapi: 3.0.0
info:
  title: Архитектурное решение API
  version: 1.0.0
servers:
  - url: https://api.homework.otus.com
paths:
  /user/create:
    post:
      summary: Создание нового пользователя
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                first_name:
                  type: string
                last_name:
                  type: string
                email:
                  type: string
      responses:
        '201':
          description: Пользователь успешно создан
          content:
            application/json:
              schema:
                type: object
                properties:
                  user_id:
                    type: string

  /money/put:
    post:
      summary: Пополнение счета
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                user_id:
                  type: string
                amount:
                  type: string
      responses:
        '202':
          description: Запрос принят
          content:
            application/json:
              schema:
                type: object
                properties:
                  balance:
                    type: string

  /money/history:
    get:
      summary: Проверка истории операций
      parameters:
        - name: user_id
          in: query
          required: true
          schema:
            type: string
      responses:
        '200':
          description: История операций успешно получена
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    operation_id:
                      type: string
                    user_id:
                      type: string
                    type:
                      type: string
                    amount:
                      type: string
                    timestamp:
                      type: string

  /notifications/messages:
    get:
      summary: Получение списка сообщений для пользователя
      parameters:
        - name: user_id
          in: query
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Список сообщений успешно получен
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    message_id:
                      type: string
                    user_id:
                      type: string
                    status:
                      type: string
                    message:
                      type: string
                    timestamp:
                      type: string

  /notifications/send:
    post:
      summary: Отправка уведомления
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                user_id:
                  type: string
                message:
                  type: string
                notification_type:
                  type: string
                timestamp:
                  type: string
      responses:
        '202':
          description: Уведомление успешно отправлено
          content:
            application/json:
              schema:
                type: object
                properties:
                  message_id:
                    type: string

  /orders/create:
    post:
      summary: Создание нового заказа
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                user_id:
                  type: string
                product_id:
                  type: string
                quantity:
                  type: integer
      responses:
        '201':
          description: Заказ успешно создан
          content:
            application/json:
              schema:
                type: object
                properties:
                  order_id:
                    type: string
                  user_id:
                    type: string
                  product_id:
                    type: string
                  quantity:
                    type: integer
                  status:
                    type: string

  /mq/users:
    post:
      summary: Создание пользователя и постановка в очередь
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                user_id:
                  type: string
                first_name:
                  type: string
                last_name:
                  type: string
                email:
                  type: string
      responses:
        '202':
          description: Операция принята

  /mq/billing/subscribe:
    post:
      summary: Подписка на создание пользователя и чтение из очереди
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                queue_name:
                  type: string
      responses:
        '200':
          description: Подписка успешно оформлена

  /mq/orders:
    post:
      summary: Создание заказа и постановка в очередь
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                order_id:
                  type: string
                user_id:
                  type: string
                order_details:
                  type: object
                  properties:
                    product_name:
                      type: string
                    quantity:
                      type: integer
                    total_amount:
                      type: number
      responses:
        '202':
          description: Операция принята

  /mq/withdrawals:
    post:
      summary: Списание средств для заказа и обработка из очереди
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                order_id:
                  type: string
                amount:
                  type: number
      responses:
        '202':
          description: Операция принята

```


*Примечание. По данному крипту с помощью сервиса https://editor-next.swagger.io/ был сгенерировани проект сервиса на Asp.Net Core и проверена работа API через swqagger.*