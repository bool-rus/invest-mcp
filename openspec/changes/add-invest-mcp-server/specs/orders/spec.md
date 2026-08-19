## Purpose

Размещение, замена, отмена и просмотр заявок и стоп-заявок на счёте через MCP-инструменты с гейтингом режимом подтверждений.

## ADDED Requirements

### Requirement: place_order
Сервер SHALL предоставлять инструмент place_order с параметрами: instrument, direction (buy, sell), order_type (market, limit, bestprice), quantity (целые лоты, больше нуля), price (обязателен для limit), time_in_force (day, fill_and_kill, fill_or_kill; по умолчанию day), confirm (см. capability server). Сервер SHALL генерировать идемпотентный ключ заявки при каждом вызове, если ключ не передан. Результат — order_id, статус исполнения, исполнено/остаток, средняя цена. Ошибки валидации (отсутствие price для limit, quantity=0) и отклонения брокера SHALL возвращаться как ошибка инструмента с кодом статуса.

#### Scenario: Лимитная заявка без цены
- **WHEN** вызывается place_order с order_type=limit без price
- **THEN** инструмент возвращает ошибку валидации, заявка не размещается

#### Scenario: Успешное размещение
- **WHEN** вызывается place_order с валидными параметрами
- **THEN** возвращается order_id и статус новой заявки

### Requirement: replace_order
Сервер SHALL предоставлять инструмент replace_order с параметрами order_id, instrument, quantity, price. Результат — новый order_id, статус, исполнено/остаток. Идемпотентность: повторная замена с тем же ключом SHALL NOT создавать вторую заявку.

#### Scenario: Замена лимитной заявки
- **WHEN** вызывается replace_order для существующей заявки с новой ценой
- **THEN** заявка заменяется и возвращается новый order_id

### Requirement: Просмотр заявок
Сервер SHALL предоставлять get_orders (активные заявки счёта) и get_order_state с параметром order_id (полный статус: текущее состояние, исполненные объёмы, средняя цена, история). Несуществующий order_id SHALL возвращать ошибку.

#### Scenario: Активные заявки
- **WHEN** вызывается get_orders
- **THEN** возвращается список активных заявок с id и статусами

#### Scenario: Состояние заявки после исполнения
- **WHEN** вызывается get_order_state для исполненной заявки
- **THEN** статус фиксирует исполнение с объёмом и средней ценой

### Requirement: cancel_order
Сервер SHALL предоставлять инструмент cancel_order с параметром order_id и подтверждением по правилам capability server. Успех — статус отмены. Если заявка уже исполнена или отменена — SHALL возвращаться ошибка с фактическим состоянием заявки.

#### Scenario: Отмена активной заявки
- **WHEN** вызывается cancel_order для активной заявки
- **THEN** заявка отменяется и возвращается подтверждение отмены

### Requirement: Стоп-заявки
Сервер SHALL предоставлять place_stop_order с параметрами: instrument, direction, order_type (stop_loss, take_profit, stop_limit), quantity, stop_price, price (обязателен для stop_limit), expiration_type (good_till_cancel, good_till_date), expire_date (обязателен при good_till_date), confirm. Также get_stop_orders (список активных стоп-заявок) и cancel_stop_order (по stop_order_id, с подтверждением). Валидация и отклонения обрабатываются как в place_order.

#### Scenario: Стоп-лимит без цены лимитной части
- **WHEN** вызывается place_stop_order с order_type=stop_limit без price
- **THEN** инструмент возвращает ошибку валидации

#### Scenario: Список стоп-заявок
- **WHEN** вызывается get_stop_orders
- **THEN** возвращаются активные стоп-заявки с id, статусами и типами
