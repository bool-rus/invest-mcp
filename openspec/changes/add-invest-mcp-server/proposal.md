## Why

Нужен MCP-сервер на Rust поверх собственного SDK `yatis` (T-Bank Invest API), чтобы LLM-агент мог управлять портфелем: искать инструменты, выставлять и корректировать заявки и стоп-заявки, видеть портфель и сделки, а также исполнять **декларативные торговые стратегии** (цепочки «при срабатывании — выстави…», «при цене — отмени…»), оповещая внешний sh-скрипт о каждом событии.

## What Changes

Новый автономный бинарник `invest-mcp` (crate в этом репозитории, зависит от `yatis`):

- **MCP-сервер** на официальном Rust SDK `rmcp`, транспорт **Streamable HTTP** (запущенный процесс), stateful-сессии.
- **CLI-ключи запуска**: `--env sandbox|prod` (по умолчанию sandbox), `--token`/`YATIS_TOKEN`, `--host/--port`, `--confirm|--no-confirm`, `--hook <script>`. Аккаунт не выбирается параметром: на токене ровно один счёт, сервер определяет его при старте и завершается с ошибкой, если счетов не один.
- **Инструменты (tools)**: поиск/листинг инструментов; заявки и стоп-заявки; портфель/позиции/лимиты/операции; прайс-данные (last price, стакан, свечи); ресурс `orderbook://{figi}/{depth}` с push-обновлениями `resources/updated`.
- **Движок стратегий**: `run_strategy` / `update_strategy` / `get_strategy` / `list_strategies` / `cancel_strategy`. Декларативный конфиг: `initial_actions` + одноразовые правила `when → do` (триггеры: fill/reject/cancel заявки, пересечение цены; действия: place/replace/cancel заявок и стоп-заявок, `stop`).
- **Глобальный sh-хук**: на каждое событие любой стратегии безусловно асинхронно запускается заданный при старте скрипт; событие — один JSON в stdin; без очереди и таймаута.
- Режим подтверждений `--confirm` действует только на прямые торговые tools (стратегии автономны всегда).

## Capabilities

### New Capabilities

- `server`: запуск сервера (CLI-ключи, transport, токен, выбор среды sandbox/prod, режим confirm), резолв единственного аккаунта на старте, tool `get_accounts`.
- `instruments`: поиск инструментов (`find_instruments`, `list_instruments`, `get_instrument`), резолв instrument → figi.
- `market-data`: `get_last_price`, `get_order_book`, `get_candles`, ресурс `orderbook://{figi}/{depth}` с серверными push-нотификациями.
- `orders`: размещение/замена/отмена/просмотр заявок и стоп-заявок, гейтинг confirm.
- `portfolio`: портфель, позиции, лимиты вывода, операции и сделки (включая пагинированную историю, без фильтрации — только операции счёта).
- `strategies`: декларативные стратегии — конфиг (валидация по JSON-схеме и простым правилам цен), движок (events → rules → actions), lifecycle-инструменты, семантика ref/once/триггеров.
- `hooks`: глобальный sh-хук — контракт вызова, stdin-JSON событий, гарантии (безусловность, асинхронный запуск, отсутствие влияния на движок).

### Modified Capabilities

_Нет_ — спецификаций в репозитории ещё нет, все возможности новые.

## Impact

- **Код**: новый crate `invest-mcp` (модули: `main`/CLI, `config`, `sdk` (адаптер yatis), `tools/*`, `engine/*`, `hook`). Код `yatis` не изменяется.
- **Зависимости**: `yatis` (git/path), `rmcp` (официальный Rust MCP SDK, features: server, transport), `tokio`, `clap`, `serde`/`serde_json`, `schemars`, `rust_decimal`, `tracing`, `uuid`.
- **Риск**: согласование версий в гипер-экосистеме (tonic 0.13 из `yatis` + HTTP-транспорт `rmcp` в одном бинарнике) — проверить `cargo tree`, при конфликте изолировать транспортный слой.
- **Внешние системы**: T-Bank Invest API (боевой `invest-public-api.tinkoff.ru`, sandbox `sandbox-invest-public-api.tinkoff.ru`), пользовательские sh-скрипты.