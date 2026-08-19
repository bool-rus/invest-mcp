## 1. Spike: транспорт и совместимость

- [ ] 1.1 Создать пустой crate invest-mcp с зависимостями yatis (git/path) и rmcp (features server, transport); собрать cargo tree, зафиксировать версии hyper/axum, при конфликте предложить изоляцию
- [ ] 1.2 Развернуть минимальный rmcp-сервер на Streamable HTTP (stateful) с одним инструментом и resource template; проверить по протоколу read_resource, subscribe и отправку resources/updated (уточнить API rmcp для нотификаций)

## 2. Scaffold сервера

- [ ] 2.1 CLI через clap: --env, --token/YATIS_TOKEN, --account, --host/--port, --confirm|--no-confirm, --hook, --hook-timeout; валидация и понятные ошибки запуска
- [ ] 2.2 config.rs: структура Config, парсинг CLI + env, права подтверждений
- [ ] 2.3 main.rs: инициализация config, резолв единственного аккаунта (get_accounts), запуск HTTP-сервера rmcp

## 3. Адаптер SDK

- [ ] 3.1 money.rs: конвертер Quotation/MoneyValue ↔ rust_decimal/строки с unit-тестами (отрицательные, nano-точность)
- [ ] 3.2 resolve.rs: InstrumentRef {ticker,class_code}|{figi}|{uid} → figi/uid через InstrumentsService, кэш, ошибки
- [ ] 3.3 sdk.rs: YatisAdapter над InvestApi, выбор Api/SandboxApi по --env, подключение и проверка соединения на старте

## 4. Инструменты: счета, инструменты, рыночные данные

- [ ] 4.1 get_accounts
- [ ] 4.2 find_instruments, list_instruments, get_instrument с форматированием по money.rs
- [ ] 4.3 get_last_price, get_order_book (валидация depth 1..50)
- [ ] 4.4 resources.rs: orderbook://{figi}/{depth}, read_resource, subscribe/unsubscribe, нотификации resources/updated

## 5. Инструменты: заявки и портфель

- [ ] 5.1 Гейтинг confirm для торговых инструментов (по config.confirm_orders)
- [ ] 5.2 place_order (идемпотентный ключ), replace_order, get_orders, get_order_state, cancel_order
- [ ] 5.3 place_stop_order, get_stop_orders, cancel_stop_order
- [ ] 5.4 get_portfolio, get_positions, get_withdraw_limits
- [ ] 5.5 get_operations, get_operations_by_cursor (пагинация, разбиение периода)

## 6. Реестр стримов

- [ ] 6.1 StreamRegistry: реф-каунтинг подписок order_state_stream на аккаунт, last_price/orderbook на figi; диспетчер событий по mpsc
- [ ] 6.2 Обработка переподключения стримов и фильтрация событий по чужим order_id

## 7. Движок стратегий

- [ ] 7.1 Модель стратегии (strategy, rules, triggers, actions) и валидация конфига (ref-уникальность, cross-ref, цена/тип/expiry) — общая для run/update
- [ ] 7.2 Задача движка: событийный цикл, матчер правил (порядок, once), дедупликация, контекст события
- [ ] 7.3 Исполнение действий: place/replace/cancel заявок и стопов, реф-маппинг ref→id, no-op cancel, stop
- [ ] 7.4 price-правила на last_price с сравнением rust_decimal
- [ ] 7.5 run_strategy, get_strategy, list_strategies, update_strategy, cancel_strategy (close_scope, закрытие позиций встречными рыночными)

## 8. Хук

- [ ] 8.1 hook.rs: сериализованная очередь, запуск sh через Command (stdin piped), таймаут с kill, логирование stderr, env YATIS_STRATEGY_ID/YATIS_EVENT_TYPE
- [ ] 8.2 События: strategy_started, rule_matched, strategy_updated, strategy_cancelled, strategy_finished; JSON-схема stdin; режим без --hook

## 9. Интеграция и проверка

- [ ] 9.1 Поведенческие сценарии со spec-ов на sandbox-токене: поиск/листинг инструментов, заявки и стоп-заявки, портфель и операции
- [ ] 9.2 Сквозной сценарий стратегии: лимитка → частичный fill → TP/SL → price-правило (отмена и новая заявка) → stop; проверка событий хука
- [ ] 9.3 Документация запуска (README): ключи, примеры тулов, контракт хука
