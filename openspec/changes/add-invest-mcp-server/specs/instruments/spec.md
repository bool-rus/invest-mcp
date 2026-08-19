## Purpose

Поиск и листинг инструментов T-Bank Invest API через MCP-инструменты, включая единый резолв ссылок на инструмент в FIGI и форматирование денежных величин.

## ADDED Requirements

### Requirement: find_instruments
Сервер SHALL предоставлять инструмент find_instruments с параметрами query (обязательный), instrument_kind (необязательный: share, bond, etf, currency, future, option) и api_trade_available_flag (необязательный). Результат — список кандидатов: name, ticker, class_code, figi, uid, instrument_type, currency, api_trade_available_flag. Пустой или пустые пробелы query SHALL отклоняться ошибкой валидации.

#### Scenario: Поиск по подстроке
- **WHEN** вызывается find_instruments с query=T и instrument_kind=share
- **THEN** возвращается список акций, у которых имя или тикер содержит T, с figi, тикером и class_code

#### Scenario: Пустой запрос
- **WHEN** вызывается find_instruments с пустой строкой query
- **THEN** инструмент возвращает ошибку валидации

### Requirement: list_instruments
Сервер SHALL предоставлять инструмент list_instruments с параметрами kind (share, bond, etf, currency, future) и instrument_status (необязательный: base, trading, all; по умолчанию base). Результат — полный список инструментов вида: figi, ticker, class_code, name, currency, lot, min_price_increment.

#### Scenario: Полный список акций
- **WHEN** вызывается list_instruments с kind=share и instrument_status=trading
- **THEN** возвращается полный список торгуемых акций с базовыми полями

### Requirement: get_instrument
Сервер SHALL предоставлять инструмент get_instrument с параметрами by (figi, ticker, uid) и id, а также class_code (требуется при by=ticker). Результат — детальная информация инструмента по его типу. Если инструмент не найден или class_code неоднозначен — SHALL возвращаться ошибка с пояснением.

#### Scenario: Получение по тикеру
- **WHEN** вызывается get_instrument с by=ticker, id=T и class_code=TQBR
- **THEN** возвращается детальная информация об акции T

#### Scenario: Неизвестный инструмент
- **WHEN** вызывается get_instrument с by=figi и несуществующим figi
- **THEN** инструмент возвращает ошибку о ненайденном инструменте

### Requirement: Резолв ссылок на инструмент
Все инструменты, принимающие instrument (place_order, place_stop_order, get_last_price, get_candles, get_order_book, правила стратегий), SHALL принимать ссылку в одном из форматов: {ticker,class_code}, {figi}, {uid}. Сервер SHALL резолвить ссылку в figi для торговых операций и кэшировать результат; неразрешимая ссылка SHALL возвращать ошибку.

#### Scenario: Резолв тикера в figi
- **WHEN** торговый инструмент принимает {ticker:T,class_code:TQBR}
- **THEN** сервер определяет figi инструмента и использует его при запросе

#### Scenario: Неразрешимая ссылка
- **WHEN** торговый инструмент принимает несуществующий тикер
- **THEN** возвращается ошибка резолва без обращения к торговому API

### Requirement: Формат денежных величин
Сервер SHALL отдавать все денежные величины (цены, суммы) как десятичные строки (например "340.50") с указанием валюты: value и currency. Никакие величины SHALL NOT округляться молчаливо: дробная часть воспроизводится из Quotation без потерь. Цены заявок, стоп-заявок и уровней правил стратегий SHALL нормализовываться к шагу цены (min_price_increment) инструмента: значение приводится к ближайшему допустимому уровню, некратная величина отклоняется ошибкой валидации.

#### Scenario: Дробная цена
- **WHEN** инструмент возвращает цену 340.5 рублей
- **THEN** в ответе value="340.5" и currency="rub"

#### Scenario: Цена не кратна шагу цены
- **WHEN** цена заявки не кратна min_price_increment инструмента
- **THEN** сервер возвращает ошибку валидации с указанием допустимого шага цены