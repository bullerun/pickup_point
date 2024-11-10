# Pickup point

Pickup point — это сервис на Golang для управления процессами пункта выдачи заказов (ПВЗ), позволяющий отслеживать и
обрабатывать движение посылок: от поступления на склад до выдачи и возврата.

## Стек технологий

Проект использует следующие технологии и инструменты:

1. **gRPC** — для организации взаимодействия между клиентом и сервером.
2. **Prometheus** — для мониторинга метрик сервиса и его производительности.
3. **Кеширование** — для повышения скорости обработки запросов.
4. **Kafka** — для асинхронной обработки событий и обмена сообщениями.
5. **Goose** — для управления миграциями базы данных, позволяя легко изменять структуру данных.


Использование кэша
---
У меня есть два LRU кэша. Один кэш используется для заказов, которые вернули, другой для обычных заказов.

### returnedCache

Данный кэш достаточно тривиальный, когда заказы приходят с PostgreSQL при вызове метода `GetReturned` они формируются в
массив и попадают в кэш с ключом `returned:<offset>:<limit>`, после вызова метода `AcceptReturn`
или `ReturnOrderToCourier`, кэш полностью очищается, т.к. нет возможности понять изменились ли закэшированные данные или
нет.

### orderCache

Данный кэш используется для кэширования заказов, которые пришли после выполнения метода `ListOrders`, на самом деле в
нашем проекте нет смысла держать в кэше весь заказ достаточно только держать `userID`, ключом для которого
будет `orderID`. Благодаря такому кэшу мы можем валидировать:

`AddOrder` проверяя на то, есть ли у нас заказ с таким id или нет, если есть то нам нет смысла выполнять запрос в бд,
если нету, то делаем запрос в бд, если удачный запрос, то добавляем в кэш.

`UpdateIssue` пробегая по всем переданным `OrderID`пытаясь узнать у всех ли заказов один и тот же получатель (UserID),
если нет, то нам нет смысла выполнять запрос в бд.

