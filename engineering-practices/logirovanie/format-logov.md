# Формат логов

### Общие принципы

*   Формат — **JSON-log per line** (JSONL)

    Каждое событие = один JSON-объект в одной строке (без переносов). Внутренние переносы должны экранироваться (`\\n`).
* Поля должны быть совместимы с [**OpenTelemetry semantic conventions**](https://opentelemetry.io/docs/concepts/semantic-conventions/)
* Временная метка — **UTC, ISO-8601 с миллисекундами** (2025-09-09T12:34:56.789Z)
* Уровень логирования — дублируется в `level` и `severity_number` (OTel)
* Сообщение — всегда строка (`message` → `Body`)

### JSON-формат

```json
{
  "timestamp": "2025-09-09T12:34:56.789Z",
  "level": "error",
  "severity_number": 17,
  "service.name": "billing-api",
  "deployment.environment": "prod",
  "host.name": "win-app-01",
  "process.pid": 1234,
  "thread.name": "9",
  "message": "Payment processing failed",
  "exception.type": "AggregateException",
  "exception.message": "One or more errors occurred",
  "exception.stacktrace": "...",
  "http.method": "POST",
  "http.url": "/api/payments",
  "http.status_code": 500,
  "http.response_time_ms": 350,
  "user.id": "u-12345",
  "client.address": "192.168.1.25",
  "trace_id": "abc123abc123abc123abc123abc123ab",
  "span_id": "def456def456def4",
  "tags": ["billing", "retry", "external:bank", "timeout"]
}
```

_Для наглядности JSON представлен в виде pretty print. В хранилище событие хранится в виде одной строки._

### Теги

* `tags: string[]`, максимум 8 тегов
* `^[a-z0-9][a-z0-9:_\-]{1,31}$`
* Только категории, без PII и уникальных значений
* Общий словарь: `billing`, `auth`, `timeout`, `external:bank` и т.п.

### Минимальный набор полей

```json
{
  "timestamp": "2025-09-09T12:34:56.789Z",
  "service.name": "nginx-proxy",
  "deployment.environment": "prod",
  "host.name": "linux-proxy-01",
  "level": "info",
  "severity_number": 9,
  "message": "HTTP access",
  "trace_id": "8f4c1be0d8a1b7a0a5b2f3c4d5e6f789",
  "http.method": "GET",
  "http.url": "/api/orders?limit=10",
  "http.status_code": 200,
  "http.response_time_ms": 15,
  "client.address": "203.0.113.45"
}
```

### Специфика источников

* **.NET/IIS** → Serilog/NLog JSON sink с маппингом на OTel
* **Nginx** → `log_format json` + Fluent Bit/OTel Collector parser
* **Windows Event Log** → Fluent Bit input winlog → нормализация к OTel-полям
* **Kubernetes stdout/stderr** → kubelet JSON → Collector добавляет `service.name`, `k8s.pod.name`, `k8s.namespace.name`

### Для ClickStack

* **timestamp** → `Timestamp`
* **message** → `Body`
* **level** → `SeverityText`
* **severity\_number** → `SeverityNumber` (1–24)
* **service.name, deployment.environment, host.name, k8s.\*, trace\_id, span\_id** → отдельные колонки
* Остальное → `Attributes` JSON-колонка

### Пример события

```json
{
  "timestamp": "2025-09-09T12:34:56.789Z",
  "level": "error",
  "severity_number": 17,
  "service.name": "billing-api",
  "deployment.environment": "prod",
  "host.name": "win-app-01",
  "message": "Payment processing failed",
  "trace_id": "8f4c1be0d8a1b7a0a5b2f3c4d5e6f789",
  "span_id": "a1b2c3d4e5f6a7b8",
  "http.method": "POST",
  "http.url": "/api/payments",
  "http.status_code": 500,
  "http.response_time_ms": 350,
  "exception.type": "SqlException",
  "exception.message": "Timeout expired",
  "tags": ["billing", "retry", "external:bank", "timeout"]
}
```

### JSON Schema (адаптированная)

* `timestamp` — ISO 8601
* `level` — enum (`trace` … `fatal`)
* `severity_number` — integer (1–24)
* `service.name`, `deployment.environment`, `host.name` — обязательные
* `trace_id`/`span_id` — hex (32/16)
* `message` — обязательное текстовое поле
* Допустимы `http.*`, `exception.*`, `k8s.*`, `tags`
