## Оценка атрибутов качества

В задании отсутствут бизнес-контекст и требования пользователей. Критические характеристики не выявлены. Исходя из существующих условий, для оценки качества архитектурного решения выбраны наиболее общие атрибуты качества:

### Надежность
Разделение функционала системы на сервисы позволяет изолировать и обрабатывать сбои и ошибки в каждом сервисе отдельно, обеспечивая надежность всей системы.
Система использует очередь сообщений (MQ), что способствует более надежной обработке запросов и позволяет избежать потери данных в случае сбоев в одном из сервисов.

*Критичный сценарий:*
- Billing Service недоступен.  
    1) Операция создания пользователя. Order Service создает пользователя и ставит сообщение Create User в очередь MQ. Когда Billing Service станет доступен, он обработает очередь и создаст billing accaunt для пользователя.
    2) Операция создания заказа. Order Service обрабатывает ошибку по таймауту и не создает заказ. Отправляет соответствующий ответ пользователю.
- Notification Service недоступен.
    1) Операция создания заказа. Order Service и Billing Service выполняют операцию и ставят сообщения в очередь MQ. Когда Notification Service станет доступен, он обработает очередь и отправит соответствующие сообщения пользователю.


### Производительность и Масштабируемость
Использование микросервисной архитектуры позволяет масштабировать систему горизонтально. Каждый сервис может быть развернут на отдельных серверах или контейнерах. Это позволяет обрабатывать большой объем запросов и увеличивать производительность системы, в случае необходимости, за счет увеличения колличества инстансов необходимого сервиса.
Использование очереди сообщений для асинхронного взаимодействия позволяет выровнить нагрузку и обеспечивает более высокую производительность, так как запросы не блокируют друг друга.

### Модифицируемость
Каждый сервис можно изменять и масштабировать независимо от других. Например, можно добавить дополнительные функции в сервис биллинга без влияния на другие компоненты системы.

### Безопасность
Использование Фасада позволяет обеспечить централизованное управление доступом к функциональности сервисов, мониторинг безопасности, что повышает общий уровень безопасности.
Фасад может обеспечивать аутентификацию и авторизацию для всех входящих запросов от клиентов (использование JWTили OAuth2) или же возможно вынести данный функционал в отдельный сервис авторизации.
Доступ к API сервисов должен быть защищен авторизацией и аутентификацией. Политика безопасности может настраиваться ля каждого сервиса отдельно.