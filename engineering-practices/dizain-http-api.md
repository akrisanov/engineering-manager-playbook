# Дизайн HTTP API

### Используемые стандарты

* [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) и [RFC 9111: HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)
* [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html)
* [RFC 6902: JSON Patch](https://www.rfc-editor.org/rfc/rfc6902) и [RFC 7386: JSON Merge Patch](https://www.rfc-editor.org/rfc/rfc7386)
* [RFC 6648: Deprecating the "X-" Prefix](https://datatracker.ietf.org/doc/html/rfc6648)
* IETF drafts: [Idempotency‑Key header](https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header/); RateLimit headers

### Ресурсы и URL

* URL **идентифицирует** ресурс. Имена – **существительные во мн. числе**: `/users`, `/articles`
* Для единственного «персонального» ресурса допустимы формы: `/profile`, `/basket`
* **Вложенные ресурсы** стараемся не использовать. Предпочтение – фильтрации:
  * ✅ `/products?category=food&category=health`
  * ⛔ `/category/food/products`
* Множественная выборка по id: `GET /contents?id=1&id=2&id=3`

#### Параметры запроса

* Невалидные значения параметров → 400 (Problem Details)
* Имя параметра одинаково при одиночном и множественном значении
* Операторы сравнения: `__gt`, `__gte`, `__lt`, `__lte`

### Методы и операции

| **Метод**  | **Коллекция `/resources`**                 | **Элемент `/resources/{id}`**                                                                                               |
| ---------- | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| **GET**    | Список, фильтрация, пагинация              | Чтение ресурса                                                                                                              |
| **POST**   | Создание                                   | Действия/подресурсы, если требуется                                                                                         |
| **PUT**    | Массовые опер. (редко), либо полная замена | **Полная замена** ресурса (idempotent)                                                                                      |
| **PATCH**  | –                                          | **Частичное обновление** (JSON Patch `application/json-patch+json` **или** JSON Merge Patch `application/merge-patch+json`) |
| **DELETE** | Массовое удаление (редко)                  | Удаление ресурса (idempotent)                                                                                               |

#### Действия над ресурсами

* По возможности моделируем **как ресурсы/состояния**. Если действие уместно – используем подпуть `/actions`:
  * `POST /users/{id}/actions/deactivate`
* Действие **должно быть идемпотентным** или клиент присылает `Idempotency-Key` (рекомендовано к поддержке) для безопасных повторов

### Контент

* Единый формат: `application/json; charset=utf-8`
* Клиент должен указывать `Accept`. Если запрошенный тип не поддержан → **406 Not Acceptable**
* На вход сервер требует корректный `Content-Type`; неподдерживаемый формат → **415 Unsupported Media Type**

### Пагинация

* **Стандарт для публичных API:** страничная пагинация параметрами: `page` (>=1), `perPage` (`1..100`, по умолчанию `20`)\
  Ответ:

```
{
    "totalCount": 13,
    "page": 1,
    "perPage": 20,
    "items": [...]
}
```

* **Для больших списков** – курсорная пагинация (рекомендация): `limit` (`1..100`), `after`/`before` (курсор), стабильная сортировка по ключу

### Выбор полей и расширение вложенных

Каждый из атрибутов – опционален и может быть реализован для удоства использования API клиентами.

* Параметры:
  * `fields` – whitelist полей → вернуть только нужные поля модели
  * `omit` – blacklist полей → опустить только указанные поля из модели
  * `expand` – раскрытие вложенных ресурсов, поддерживает вложенность через точку: `expand=tags.category` → вернуть вместо ID объект связанного ресурса
* Приоритет: `omit` > `fields` > `expand`

### Идентификаторы и числа

* Идентификаторы в JSON – **строки** (во избежание потерь точности в клиентах)
* Денежные суммы: либо **целые в минорных единицах** (рекоменд.), либо **строки** с фиксированной точностью

### Даты и время

* Формат – ISO‑8601. В ответах **UTC в `Z`**: `YYYY-MM-DDThh:mm:ssZ`
* Если необходимо отдать локальное время – используйте явный смещённый формат `±hh:mm` и это должно быть задокументировано для конкретного поля

### Коды статусов

| **Статус** | **Код**                | **Пояснение**                                                   |
| ---------- | ---------------------- | --------------------------------------------------------------- |
| 200        | OK                     | Успешный ответ                                                  |
| 201        | Created                | Создание ресурса; указывать `Location` на новый ресурс          |
| 204        | No Content             | Действие без тела ответа                                        |
| 400        | Bad Request            | Синтаксическая/структурная ошибка                               |
| 401        | Unauthorized           | Отсутствует/невалиден токен. **Обязательно** `WWW-Authenticate` |
| 403        | Forbidden              | Аутентифицирован, но нет прав                                   |
| 404        | Not Found              | Ресурс не найден                                                |
| 405        | Method Not Allowed     | Метод не поддержан; **обязательно** `Allow` со списком методов  |
| 406        | Not Acceptable         | Тип из `Accept` не поддержан                                    |
| 409        | Conflict               | Конфликт состояния, например, дубликат                          |
| 412        | Precondition Failed    | Нарушены предусловия (`If-Match`, `If-Unmodified-Since`)        |
| 413        | Content Too Large      | Превышен размер тела запроса                                    |
| 415        | Unsupported Media Type | Неподдерживаемый `Content-Type`/`Content-Encoding`              |
| 422        | Unprocessable Content  | Валидация не пройдена, бизнес‑ошибки ввода                      |
| 428        | Precondition Required  | Сервер **требует** предусловий (например, `If-Match`)           |
| 429        | Too Many Requests      | Превышен rate limit; указывать `Retry-After`                    |
| 451        | Unavailable For Legal  | Метод или API недоступно для текущего региона/локации           |
| 500–504    |                        | Ошибки сервера/шлюза                                            |

### Ошибки: Problem Details (RFC 9457/7807)

* Формат содержимого: `application/problem+json`.
* Базовая схема ответа:

```
{
  "type": "https://api.example.com/problems/validation-error",
  "title": "Ошибка валидации",
  "status": 422,
  "detail": "Некоторые поля не прошли проверку",
  "instance": "/contents",
  "requestId": "c91a0f8e-5f1a-4d1f-9f4f-2f7a3a...",
  "errors": [
    {
      "title": "Минимальная длина – 2",
      "detail": "Поле должно содержать не менее 2 символов",
      "pointer": "/body/key"
    },
    {
      "title": "Неверное значение",
      "detail": "Недопустимое значение параметра",
      "parameter": "sort"
    }
  ]
}
```

* Поля `errors[].pointer` – JSON Pointer до поля тела; `errors[].parameter` – имя query‑параметра
* Для 401 добавляем `WWW-Authenticate`
* Для 405 – `Allow`
* Для 429 – `Retry-After`

### Версионирование API

* Клиент **явно** отправляет версию в каждом запросе\
  **Заголовок:** `API-Version: YYYY-MM-DD`
* Альтернатива (по проектному решению): через `Accept`-тип, например:\
  `Accept: application/vnd.planfact.v2025-08-11+json`
* CHANGELOG фиксирует только **ломающие** изменения

### Кэширование и конкурентный доступ

* Возвращаем валидаторы: `ETag` (сильный/слабый) и/или `Last-Modified`
* Клиент использует `If-None-Match`/`If-Modified-Since` для GET; ответ **304 Not Modified**
* Для защиты от потерянных обновлений клиент отправляет `If-Match` при `PUT`/`PATCH`/`DELETE`; при расхождении → **412**
* **Vary** проставляем **минимально необходимый** (обычно `Accept`, иногда `Accept-Language`, `Accept-Encoding`)\
  Не используем глобально `Vary: Authorization, Cookie` – это раздувает ключи кэша; приватные ответы помечаем `Cache-Control: private`

### Сжатие

* Сервер поддерживает компрессию ответов согласно `Accept-Encoding` (обычно `gzip`, `br`)
* Если клиент отправляет сжатое тело, он обязан указать `Content-Encoding`. При неподдерживаемом кодировании → **415** (и можно вернуть `Accept-Encoding` со списком поддерживаемых кодировок).

### Ограничение запросов (рейт-лимит)

* Рекомендуемые заголовки семейства **RateLimit** (IETF draft):
  * `RateLimit-Limit: 100`
  * `RateLimit-Remaining: 42`
  * `RateLimit-Reset: 1733923200` (unix ts, секунды)
* При превышении → **429** + `Retry-After` (секунды или дата RFC‑1123)

### CORS

* Разрешаем домены партнёров/продукта
* Preflight (OPTIONS) должен возвращать: `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, `Access-Control-Max-Age`, при необходимости `Access-Control-Allow-Credentials`
* Если `Allow-Credentials: true`, значение `Allow-Origin` **не может** быть `*`

### Безопасность и транспорт

* Все запросы – только через HTTPS. Небезопасные запросы → **403** с Problem Details.
* Аутентификация: `Authorization: Bearer <token>`
* В каждый ответ включаем `Request-Id` (корреляция логов)

### Требования к ответам

* JSON «pretty printed» только в dev/staging (в prod – компактно)
* Поля и их типы должны быть стабильны; новые поля – обратно‑совместимые

### Требования к документации

* Для каждого метода: URI, параметры, схема запроса/ответа, коды статусов, примеры ошибок (Problem Details)
* Поддерживаем живую спецификацию OpenAPI (JSON/YAML) как «источник истины»
