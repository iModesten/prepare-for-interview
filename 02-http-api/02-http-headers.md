# HTTP Headers — заголовки запросов и ответов

**Уровень:** Middle / Senior QA Engineer  
**Темы:** HTTP Headers, Origin, Referer, Host, Authorization, Cookies, CORS, Caching, Security

---

## 1. Что такое HTTP Headers?

**HTTP Headers** — поля HTTP-сообщения, передающие дополнительную информацию о запросе, ответе, представлении ресурса и правилах взаимодействия клиента с сервером.

С помощью заголовков можно передавать:

- Учётные данные для аутентификации.
- Информацию о формате содержимого.
- Параметры кэширования.
- Cookies.
- Информацию об источнике запроса.
- Условия выполнения запроса.
- Ограничения безопасности.
- Данные для трассировки запросов.

### Пример запроса

```http
POST /api/orders HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJ...
Origin: https://shop.example.com
Content-Type: application/json
Accept: application/json

{
  "productId": 100,
  "quantity": 2
}
```

### Пример ответа

```http
HTTP/1.1 201 Created
Content-Type: application/json
Cache-Control: no-store
Location: /api/orders/501
X-Request-ID: req-123

{
  "id": 501,
  "status": "CREATED"
}
```

### Краткий ответ для собеседования

> HTTP-заголовки — это поля запроса и ответа, которые передают метаданные и управляющую информацию. Например, Authorization используется для передачи учётных данных, Content-Type определяет формат содержимого, Accept сообщает предпочтения клиента, Cookie передаёт сохранённые cookie, а Cache-Control управляет кэшированием. Некоторые заголовки, например Origin и Access-Control-Allow-Origin, участвуют в механизмах браузерной безопасности.

---

## 2. Request Headers и Response Headers

### Request Headers

Передаются клиентом серверу.

Примеры:

| Заголовок | Назначение |
|---|---|
| Host | Целевой хост HTTP/1.1-запроса |
| Authorization | Учётные данные |
| Accept | Приемлемые форматы ответа |
| Content-Type | Формат содержимого запроса |
| Cookie | Cookies, отправляемые серверу |
| Origin | Origin, инициировавший запрос |
| Referer | Информация об источнике перехода |
| User-Agent | Информация о клиенте |
| Accept-Encoding | Поддерживаемые кодирования содержимого |
| If-None-Match | Условная проверка версии ресурса |

### Response Headers

Передаются сервером клиенту.

Примеры:

| Заголовок | Назначение |
|---|---|
| Content-Type | Формат представления |
| Set-Cookie | Установка cookie |
| Cache-Control | Управление кэшированием |
| Access-Control-Allow-Origin | Разрешённый origin для CORS |
| Location | URI ресурса или перенаправления |
| ETag | Идентификатор версии представления |
| Retry-After | Рекомендация о времени повторной попытки |
| WWW-Authenticate | Требования HTTP-аутентификации |
| Strict-Transport-Security | Политика использования HTTPS |

### Важные особенности

Имена HTTP-заголовков регистронезависимы:

```http
Content-Type: application/json
content-type: application/json
CONTENT-TYPE: application/json
```

Они обозначают одно и то же поле.

Но значения отдельных заголовков могут быть регистрозависимыми.

Также не все поля можно произвольно объединять через запятую: правила зависят от конкретного заголовка. Особенно важно это для `Set-Cookie`.

---

# Часть 1. Заголовки, описывающие запрос и содержимое

## 3. Host

**Host** определяет имя хоста и при необходимости порт целевого сервера.

Пример:

```http
GET /api/users HTTP/1.1
Host: api.example.com
```

### Зачем нужен?

На одном IP-адресе могут обслуживаться несколько сайтов.

Например:

```text
IP: 192.0.2.10

api.example.com
admin.example.com
shop.example.com
```

Сервер или Reverse Proxy использует информацию о целевом хосте для выбора нужного виртуального сервера.

### HTTP/2 и HTTP/3

В HTTP/2 и HTTP/3 обычно используется псевдозаголовок:

```text
:authority: api.example.com
```

Он выполняет соответствующую роль в определении целевого authority.

### Host vs Origin

**Host** — куда направлен запрос.

**Origin** — из какого веб-контекста запрос был инициирован.

Пример:

```http
GET /api/orders HTTP/1.1
Host: api.example.com
Origin: https://shop.example.com
```

Запрос идёт на `api.example.com`, но инициирован страницей `shop.example.com`.

---

## 4. Content-Type

**Content-Type** сообщает MIME-тип содержимого HTTP-сообщения.

Примеры:

```http
Content-Type: application/json
```

```http
Content-Type: application/xml
```

```http
Content-Type: text/html; charset=utf-8
```

```http
Content-Type: multipart/form-data; boundary=abc123
```

### Зачем нужен?

Серверу важно понимать, как интерпретировать тело запроса.

Например:

```http
POST /api/users
Content-Type: application/json

{
  "name": "Anna"
}
```

Если API принимает только JSON, а клиент отправит неподдерживаемый формат, сервер может вернуть:

```http
415 Unsupported Media Type
```

### Что проверять QA?

- Поддерживаемый формат.
- Неподдерживаемый формат.
- Отсутствующий Content-Type.
- Несоответствие тела заявленному типу.
- Корректность кодировки.
- Корректность multipart boundary при загрузке файлов.

---

## 5. Accept

**Accept** сообщает серверу, какие форматы представления клиент готов принять.

Пример:

```http
Accept: application/json
```

Клиент предпочитает JSON.

Можно передать несколько значений:

```http
Accept: application/json, application/xml;q=0.5
```

Здесь JSON имеет более высокий приоритет.

### Content-Type vs Accept

| Заголовок | Что означает |
|---|---|
| Content-Type | Какой формат у передаваемого содержимого |
| Accept | Какие форматы ответа клиент готов принять |

### Пример

```http
POST /api/orders
Content-Type: application/json
Accept: application/xml
```

Клиент отправляет JSON, но хочет получить XML.

Если сервер не может предоставить подходящее представление, возможен:

```http
406 Not Acceptable
```

### Типичный вопрос

**В чём разница между Content-Type и Accept?**

> Content-Type описывает формат содержимого текущего сообщения, а Accept — форматы ответа, которые клиент готов получить.

---

## 6. Accept-Encoding и Content-Encoding

### Accept-Encoding

Клиент сообщает, какие кодирования содержимого поддерживает.

Например:

```http
Accept-Encoding: gzip, br
```

### Content-Encoding

Сервер сообщает, какое кодирование применено к содержимому ответа.

```http
Content-Encoding: gzip
```

### Как это работает?

```text
Server JSON
    ↓
Compression
    ↓
HTTP Response
    ↓
Browser decompression
    ↓
JSON
```

### Зачем используется?

- Сокращение размера передаваемых данных.
- Уменьшение сетевой нагрузки.
- Потенциальное ускорение загрузки.

### Важно

`Content-Encoding` не следует путать с кодировкой текста `charset`.

Например:

```http
Content-Type: text/plain; charset=utf-8
Content-Encoding: gzip
```

Первое описывает текстовое представление, второе — применённое кодирование содержимого.

---

## 7. Accept-Language

Сообщает языковые предпочтения клиента.

Пример:

```http
Accept-Language: ru-RU,ru;q=0.9,en;q=0.7
```

Это означает предпочтение русского языка с возможностью использовать английский.

### Что проверять QA?

- Локализацию ответов.
- Язык сообщений об ошибках.
- Форматирование дат и чисел, если приложение учитывает локаль.
- Fallback при неподдерживаемом языке.
- Согласованность языка в разных сервисах.

**Важно:** Accept-Language — предпочтение, а не обязательное указание серверу. Конкретное поведение зависит от приложения.

---

# Часть 2. Origin и браузерная безопасность

## 8. Что такое Origin?

**Origin** — заголовок HTTP-запроса, который сообщает, из какого origin был инициирован запрос.

Origin определяется комбинацией:

```text
scheme + host + port
```

Пример:

```http
Origin: https://shop.example.com
```

При нестандартном порте:

```http
Origin: https://shop.example.com:8443
```

### Что не входит в Origin?

- URL path.
- Query parameters.
- Fragment.
- Логин и пароль.

Например, для страницы:

```text
https://shop.example.com/catalog/products?id=10
```

Origin будет:

```text
https://shop.example.com
```

### Краткий ответ для собеседования

> Origin — заголовок запроса, указывающий источник запроса в виде схемы, хоста и порта. Он используется браузерами и серверами в механизмах безопасности, прежде всего CORS, а также может помогать при проверке CSRF. В отличие от Referer, Origin не содержит путь и параметры страницы.

---

## 9. Same-Origin и Cross-Origin

Два URL относятся к одному origin, если совпадают:

- Схема.
- Хост.
- Порт.

### Пример

Текущая страница:

```text
https://shop.example.com
```

| Адрес запроса | Same Origin? | Причина |
|---|---|---|
| https://shop.example.com/api | Да | Совпадают схема, хост и порт |
| https://shop.example.com:443/api | Да | 443 — стандартный HTTPS-порт |
| http://shop.example.com/api | Нет | Другая схема |
| https://api.example.com/api | Нет | Другой хост |
| https://shop.example.com:8443/api | Нет | Другой порт |

### Same-Origin Policy

**Same-Origin Policy (SOP)** — набор браузерных ограничений на взаимодействие документов и скриптов из разных origin.

Один из примеров: JavaScript с сайта `shop.example.com` не может свободно читать ответы API другого origin без соответствующих разрешений.

### Важный нюанс

SOP не означает, что браузер вообще запрещает любые сетевые запросы на другие origin.

Браузер может отправить определённые cross-origin запросы, но ограничить доступ JavaScript к ответу.

Для некоторых запросов браузер сначала выполняет CORS preflight.

---

## 10. Когда браузер отправляет Origin?

Это особенно важный вопрос.

В типичных ситуациях браузер добавляет Origin:

- К cross-origin запросам Fetch/XHR.
- К same-origin запросам с методами POST, PUT, PATCH, DELETE.
- К определённым запросам отправки форм.
- К WebSocket handshake.

При обычном same-origin GET или HEAD Origin часто отсутствует.

### Пример

Страница:

```text
https://shop.example.com
```

JavaScript отправляет:

```javascript
fetch("https://api.example.com/orders")
```

Браузер может сформировать:

```http
GET /orders HTTP/1.1
Host: api.example.com
Origin: https://shop.example.com
```

### Всегда ли Origin присутствует?

Нет.

Его наличие зависит от:

- Способа формирования запроса.
- Fetch mode.
- HTTP-метода.
- Браузерного контекста.
- Политик безопасности.
- Перенаправлений.

Значение также может быть:

```http
Origin: null
```

Например, для некоторых sandboxed-документов или opaque origins.

### Важный момент

Отсутствие Origin не доказывает, что запрос безопасен.

Также Origin нельзя считать надёжным способом аутентификации: небраузерный HTTP-клиент может самостоятельно установить такой заголовок.

---

## 11. Origin vs Referer

**Origin** сообщает origin инициатора запроса.

**Referer** может содержать информацию об адресе страницы, с которой был выполнен переход или инициирован запрос.

Пример:

```http
Origin: https://shop.example.com
Referer: https://shop.example.com/catalog/product/10
```

### Сравнение

| Характеристика | Origin | Referer |
|---|---|---|
| Scheme | Да | Может присутствовать |
| Host | Да | Может присутствовать |
| Port | Да | Может присутствовать |
| Path | Нет | Может присутствовать |
| Query | Нет | Может присутствовать |
| Управляется политиками браузера | Да | Да |
| Используется при CORS | Да | Нет, основной механизм CORS не строится на Referer |
| Может использоваться при CSRF-проверках | Да | Да |

### Referrer-Policy

Заголовок ответа, регулирующий объём информации Referer, передаваемой браузером.

Пример:

```http
Referrer-Policy: strict-origin-when-cross-origin
```

Это распространённая политика по умолчанию в современных браузерах.

### Почему заголовок называется Referer, а не Referrer?

Название `Referer` содержит историческую опечатку, закрепившуюся в стандарте.

`Referrer-Policy` при этом пишется с правильным английским словом.

---

## 12. Origin vs Host vs Referer

Это один из лучших сравнительных вопросов для интервью.

Представим:

Пользователь открыл страницу:

```text
https://shop.example.com/products/10
```

Страница отправляет запрос:

```text
https://api.example.com/orders
```

HTTP-заголовки могут выглядеть так:

```http
POST /orders HTTP/1.1
Host: api.example.com
Origin: https://shop.example.com
Referer: https://shop.example.com/products/10
Content-Type: application/json
```

### Объяснение

- **Host** — адрес целевого сервера.
- **Origin** — origin, инициировавший запрос.
- **Referer** — информация об адресе исходной страницы, ограниченная политикой браузера.

### Готовый ответ

> Host показывает, какому серверу адресован запрос. Origin показывает origin инициатора — схему, хост и порт. Referer может содержать более подробный адрес исходной страницы, включая путь и query, но его содержимое ограничивается Referrer-Policy.

---

## 13. CORS: Access-Control-Allow-Origin

**Access-Control-Allow-Origin (ACAO)** — заголовок ответа сервера, указывающий, какому origin браузер может предоставить доступ к ответу через CORS.

Пример:

```http
Access-Control-Allow-Origin: https://shop.example.com
```

### Пример полного обмена

Браузер отправляет:

```http
GET /api/products HTTP/1.1
Host: api.example.com
Origin: https://shop.example.com
```

Сервер отвечает:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://shop.example.com
Content-Type: application/json

{
  "products": []
}
```

Браузер проверяет CORS и разрешает JavaScript доступ к содержимому ответа.

### Access-Control-Allow-Origin: *

```http
Access-Control-Allow-Origin: *
```

Разрешает доступ из произвольного origin для сценариев, где wildcard допустим.

**Но:** wildcard нельзя использовать для предоставления доступа к credentialed CORS response с `Access-Control-Allow-Credentials: true`.

### Кто обеспечивает CORS?

Браузер.

Сервер отправляет заголовки политики, а браузер принимает решение о доступе JavaScript к ответу.

Postman и curl обычно не применяют браузерную Same-Origin Policy.

### Важный вопрос

**Почему через Postman API работает, а в браузере возникает CORS error?**

Потому что Postman не ограничивает доступ к ответам согласно браузерной CORS-модели. Браузер проверяет разрешения, возвращённые сервером.

---

## 14. CORS Preflight и OPTIONS

Для некоторых cross-origin запросов браузер сначала отправляет предварительный запрос OPTIONS.

Пример JavaScript:

```javascript
fetch("https://api.example.com/orders", {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({ productId: 10 })
});
```

Такой запрос обычно требует preflight, поскольку `application/json` не входит в CORS-safelisted значения Content-Type.

### Preflight Request

```http
OPTIONS /orders HTTP/1.1
Host: api.example.com
Origin: https://shop.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: content-type
```

### Preflight Response

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://shop.example.com
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: content-type
Access-Control-Max-Age: 600
```

Если preflight разрешён, браузер может отправить основной POST.

### Важные заголовки

| Заголовок | Направление |
|---|---|
| Origin | Request |
| Access-Control-Request-Method | Preflight Request |
| Access-Control-Request-Headers | Preflight Request |
| Access-Control-Allow-Origin | Response |
| Access-Control-Allow-Methods | Preflight Response |
| Access-Control-Allow-Headers | Preflight Response |
| Access-Control-Allow-Credentials | Response |
| Access-Control-Max-Age | Preflight Response |

### Типичная ошибка

Разработчик настроил CORS для POST, но забыл корректно обработать OPTIONS.

В итоге основной POST даже не будет отправлен браузером.

---

# Часть 3. Аутентификация и Cookies

## 15. Authorization

**Authorization** — заголовок запроса, содержащий учётные данные для доступа к защищённому ресурсу.

### Bearer Token

```http
Authorization: Bearer eyJhbGciOi...
```

Используется при токенной аутентификации.

### Basic Authentication

```http
Authorization: Basic YWxhZGRpbjpvcGVuc2VzYW1l
```

Содержит Base64-кодированное представление учётных данных.

**Важно:** Base64 — кодирование, а не шифрование. Basic Authentication должен использоваться через защищённое соединение.

### Нужно ли серверу всегда возвращать 401, если токен неверен?

Типичный результат — 401 Unauthorized с подходящим `WWW-Authenticate` challenge в стандартной HTTP authentication модели.

Однако конкретное поведение API зависит от используемой схемы аутентификации и контракта.

### Что проверять QA?

- Отсутствующий токен.
- Неверный токен.
- Истёкший токен.
- Токен другого пользователя.
- Недостаточные права.
- Неверная схема Authorization.
- Дополнительные пробелы и некорректный формат.

---

## 16. Cookie

**Cookie** — заголовок запроса, передающий сохранённые браузером cookie, соответствующие условиям отправки.

Пример:

```http
GET /profile HTTP/1.1
Host: app.example.com
Cookie: sessionId=abc123; theme=dark
```

### Зачем нужны Cookies?

- Идентификаторы сессий.
- Пользовательские настройки.
- Локализация.
- Другие небольшие значения состояния.

### Cookie и аутентификация

После логина сервер может установить cookie с идентификатором сессии.

Браузер затем автоматически отправляет её в подходящих запросах.

Важно, что cookie не обязательно содержит данные пользователя или сам JWT. Часто она содержит только непрозрачный session ID.

---

## 17. Set-Cookie

**Set-Cookie** — заголовок ответа, позволяющий серверу попросить браузер сохранить cookie.

Пример:

```http
Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Lax; Path=/
```

### Основные атрибуты

**HttpOnly**

Запрещает доступ к cookie через `document.cookie`.

Но не запрещает браузеру отправлять cookie с HTTP-запросами.

**Secure**

Ограничивает передачу cookie защищёнными соединениями согласно правилам браузера.

**SameSite**

Регулирует отправку cookie в cross-site контекстах.

Возможные значения:

- Strict.
- Lax.
- None.

При `SameSite=None` обычно требуется `Secure`.

**Domain**

Определяет доменную область действия cookie.

**Path**

Определяет область действия по URL-пути для отправки cookie.

**Max-Age / Expires**

Управляют сроком действия cookie.

### Важный нюанс

`SameSite` не равно `Same-Origin`.

Например:

```text
https://shop.example.com
https://api.example.com
```

Это разные origin, но при обычных условиях один site.

### Готовый ответ для собеседования

> Cookie — заголовок запроса, через который браузер отправляет сохранённые cookie. Set-Cookie — заголовок ответа, через который сервер устанавливает cookie. HttpOnly ограничивает доступ JavaScript, Secure требует защищённой передачи, а SameSite регулирует отправку cookie в межсайтовых контекстах.

---

## 18. Cookie vs Authorization

| Характеристика | Cookie | Authorization |
|---|---|---|
| Обычно передаётся | Браузером автоматически при выполнении условий | Клиентом согласно логике авторизации |
| Типичное значение | Session ID или другое состояние | Bearer token, Basic credentials |
| Может использоваться для аутентификации | Да | Да |
| Может быть HttpOnly | Да, через Set-Cookie | Не применимо |
| Риск CSRF | Особенно актуален при автоматической cookie-аутентификации | Зависит от способа использования credentials |
| Использование браузером | Управляется cookie-механизмом | Часто устанавливается кодом или HTTP auth |

### Важно

Bearer token можно хранить в cookie, а не только передавать через Authorization.

Поэтому механизмы хранения токена и его передачи следует рассматривать отдельно.

---

## 19. WWW-Authenticate

**WWW-Authenticate** — заголовок ответа, описывающий схему HTTP-аутентификации, необходимую для доступа к ресурсу.

Пример:

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="api"
```

Он помогает клиенту понять, какая схема аутентификации ожидается.

### Для QA

При проверке 401 полезно учитывать не только статус, но и соответствующие заголовки аутентификации.

---

# Часть 4. Кэширование и условные запросы

## 20. Cache-Control

**Cache-Control** — заголовок, управляющий поведением HTTP-кэшей.

Основные директивы:

### max-age

```http
Cache-Control: max-age=300
```

Разрешает считать представление свежим в течение заданного времени в соответствующих условиях.

### no-cache

```http
Cache-Control: no-cache
```

Не означает полного запрета хранения.

Означает необходимость проверки валидности перед повторным использованием сохранённого ответа.

### no-store

```http
Cache-Control: no-store
```

Предписывает кэшам не сохранять сообщение.

### private

```http
Cache-Control: private
```

Предназначен для частных кэшей, но не общих shared caches.

### public

Позволяет хранить ответ в shared caches, если остальные требования к кэшированию выполнены.

### Для QA

Проверять:

- Не выдаются ли устаревшие данные.
- Не кэшируются ли чужие персональные данные.
- Корректно ли обновляются ресурсы после изменений.
- Соответствует ли политика кэша требованиям безопасности.

---

## 21. ETag и If-None-Match

**ETag** — идентификатор версии представления ресурса.

Пример ответа:

```http
HTTP/1.1 200 OK
ETag: "user-v5"
Content-Type: application/json

{
  "name": "Anna"
}
```

Клиент повторяет запрос:

```http
GET /api/users/100
If-None-Match: "user-v5"
```

Если представление не изменилось, сервер может вернуть:

```http
HTTP/1.1 304 Not Modified
ETag: "user-v5"
```

### Зачем это нужно?

Не требуется повторно передавать полное содержимое ресурса, если подходящее кэшированное представление всё ещё актуально.

### Важный нюанс

ETag не обязательно является хешем тела. Способ его формирования определяется сервером.

---

## 22. If-Match и оптимистическая конкурентность

`If-Match` позволяет выполнить операцию только при совпадении версии представления с указанным условием.

Пример:

Клиент получил:

```http
ETag: "version-10"
```

Затем отправляет:

```http
PUT /api/users/100
If-Match: "version-10"
Content-Type: application/json

{
  "name": "Maria"
}
```

Если ресурс уже изменился и условие не выполняется, сервер может вернуть:

```http
412 Precondition Failed
```

### Для чего используется?

- Защита от потерянных обновлений.
- Контроль конкурентных изменений.
- Optimistic Concurrency Control.

### Что проверять QA?

1. Успешное изменение при актуальном ETag.
2. Изменение ресурса другим клиентом.
3. Повторный PUT со старым ETag.
4. Получение ожидаемой ошибки.
5. Отсутствие перезаписи чужих изменений.

---

## 23. Last-Modified и If-Modified-Since

Сервер может вернуть:

```http
Last-Modified: Wed, 07 Oct 2026 12:00:00 GMT
```

Клиент позднее отправляет:

```http
If-Modified-Since: Wed, 07 Oct 2026 12:00:00 GMT
```

Если ресурс не изменился после указанного момента, сервер может вернуть 304.

### ETag vs Last-Modified

- ETag — валидатор версии представления.
- Last-Modified — временной валидатор.

ETag часто обеспечивает более точную проверку версии, поскольку изменение ресурса может произойти несколько раз за короткий промежуток.

---

## 24. Vary

**Vary** сообщает, какие поля запроса влияют на выбор представления для целей кэширования.

Пример:

```http
Vary: Accept-Language
```

Если сервер возвращает разные языковые версии, кэш должен учитывать значение `Accept-Language`.

### CORS и Vary

Если сервер динамически выбирает разрешённый Origin, часто используется:

```http
Vary: Origin
```

Это помогает кэшам не использовать некорректный ответ для другого Origin.

### Что проверять QA?

- Корректное разделение ответов по языку.
- Корректную работу CDN.
- Отсутствие использования чужого варианта представления.
- Правильность динамического CORS при кэшировании.

---

# Часть 5. Сетевые, диагностические и защитные заголовки

## 25. User-Agent

**User-Agent** описывает клиентское программное обеспечение.

Пример:

```http
User-Agent: Mozilla/5.0 ...
```

Может использоваться для:

- Диагностики.
- Аналитики.
- Совместимости с клиентами.
- Специальной обработки отдельных окружений.

**Важно:** значение User-Agent не является надёжным источником идентификации браузера или пользователя. Его можно подделать.

---

## 26. X-Forwarded-For и Forwarded

Если запрос проходит через Reverse Proxy или Load Balancer, backend может видеть IP промежуточного сервера.

Для передачи сведений об исходном клиенте используют, например:

```http
X-Forwarded-For: 203.0.113.10
```

Стандартизованный вариант:

```http
Forwarded: for=203.0.113.10;proto=https;host=api.example.com
```

### Применение

- Логирование.
- Диагностика.
- Rate limiting.
- Определение исходной схемы запроса.

### Важный вопрос безопасности

**Можно ли всегда доверять X-Forwarded-For?**

Нет.

Клиент способен прислать такой заголовок самостоятельно.

Backend должен доверять сведениям только согласно конфигурации проверенных прокси и правильно интерпретировать цепочку передачи запроса.

---

## 27. X-Request-ID и traceparent

В микросервисной архитектуре один HTTP-запрос может проходить через множество сервисов.

Пример:

```text
API Gateway
    |
    v
Order Service
    |
    v
Payment Service
    |
    v
Bonus Service
```

### X-Request-ID

Часто используемый нестандартизованный заголовок для корреляции запросов.

```http
X-Request-ID: req-123
```

### traceparent

Заголовок стандарта W3C Trace Context, используемый для распределённой трассировки.

Пример:

```http
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

### Для QA

Если API отвечает 500, correlation ID или trace ID помогает найти связанный запрос в логах и tracing-системе.

### Полезная практика

При описании сложного backend-дефекта указывать:

- Время запроса.
- Endpoint.
- Request ID / Trace ID.
- HTTP status.
- Ожидаемый результат.
- Фактический результат.

---

## 28. Retry-After

Заголовок, который может сообщать клиенту, когда рекомендуется повторить запрос.

Например:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
```

Клиенту рекомендуется повторить запрос не раньше чем через 60 секунд.

Также может использоваться с 503.

### Важно

Retry-After может задаваться:

- Количеством секунд.
- HTTP-датой.

### Для QA

Проверять:

- Правильность значения.
- Реальное поведение rate limiting.
- Корректность восстановления после ограничения.
- Отсутствие бесконечных повторов со стороны клиента.

---

## 29. Location

**Location** используется, например, для указания URI созданного ресурса или адреса перенаправления.

### После создания

```http
HTTP/1.1 201 Created
Location: /api/orders/501
```

### При Redirect

```http
HTTP/1.1 302 Found
Location: https://example.com/login
```

### Для QA

Проверять:

- Корректность URI.
- Отсутствие небезопасных перенаправлений.
- Правильность перехода между HTTP/HTTPS.
- Сохранение метода при 307/308.

---

## 30. Strict-Transport-Security (HSTS)

HSTS позволяет серверу сообщить браузеру, что к данному хосту следует обращаться через HTTPS в течение заданного времени.

Пример:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### Зачем нужен?

Снижает риск небезопасных переходов на HTTP для хостов, для которых браузер уже знает HSTS-политику.

### Важно

HSTS обычно принимается браузером через защищённое HTTPS-соединение.

Первое обращение до получения HSTS может оставаться отдельным риском, если домен не включён в предварительно загруженный HSTS-список браузера.

---

## 31. Content-Security-Policy (CSP)

**Content-Security-Policy** — заголовок, задающий ограничения на источники ресурсов и некоторые действия браузера.

Пример:

```http
Content-Security-Policy: default-src 'self'; script-src 'self'
```

Обычно это означает, что ресурсы и скрипты должны загружаться из разрешённых источников согласно политикам.

### Для чего нужен?

- Снижение риска отдельных видов XSS.
- Ограничение загрузки ресурсов.
- Контроль выполнения скриптов.
- Ограничение встраивания страниц и других возможностей.

### Важно

CSP — дополнительный механизм защиты, а не замена безопасной обработки пользовательского ввода.

---

## 32. X-Content-Type-Options

Пример:

```http
X-Content-Type-Options: nosniff
```

Помогает ограничивать MIME-sniffing для определённых типов ресурсов.

Полезен для предотвращения ситуаций, когда браузер ошибочно интерпретирует содержимое в более опасном формате.

---

## 33. Sec-Fetch-Site

**Sec-Fetch-Site** — браузерный заголовок, характеризующий отношение инициатора запроса к целевому ресурсу.

Возможные значения:

- `same-origin`
- `same-site`
- `cross-site`
- `none`

### Пример

```http
Sec-Fetch-Site: cross-site
```

### Для чего используется?

Сервер может учитывать Fetch Metadata при защите от нежелательных cross-site запросов.

Например, отклонять определённые state-changing запросы из недоверенных cross-site контекстов.

### Чем отличается от Origin?

Origin указывает конкретный origin инициатора.

Sec-Fetch-Site описывает отношение между источником и назначением запроса.

### Важно

Fetch Metadata — дополнительный уровень защиты. Он не заменяет аутентификацию, авторизацию и другие проверки безопасности.

---

# Часть 6. Практика QA

## 34. Как смотреть HTTP Headers в браузере?

### Chrome / Edge

1. Открыть DevTools.
2. Перейти во вкладку Network.
3. Выполнить действие в приложении.
4. Выбрать нужный запрос.
5. Открыть раздел Headers.
6. Изучить Request Headers и Response Headers.

### Safari

1. Включить инструменты разработчика Safari.
2. Открыть Web Inspector.
3. Перейти в Network.
4. Выбрать запрос.
5. Посмотреть его HTTP-заголовки.

### Что полезно проверять?

- URL и метод.
- Status code.
- Origin.
- Authorization.
- Cookie.
- Content-Type.
- Accept.
- Cache-Control.
- Set-Cookie.
- CORS-заголовки.
- Request ID / Trace ID.

### Важный нюанс

Некоторые заголовки могут быть добавлены браузером или инфраструктурой.

Клиентский JavaScript не может свободно менять все заголовки.

Например, Origin относится к управляемым браузером заголовкам.

---

## 35. Практический кейс: CORS Error

### Ситуация

Frontend находится на:

```text
https://shop.example.com
```

Backend:

```text
https://api.example.com
```

Frontend выполняет запрос:

```javascript
fetch("https://api.example.com/orders")
```

В браузере возникает CORS Error.

### Что проверить QA?

1. Присутствует ли Origin.
2. Какое значение Access-Control-Allow-Origin возвращает сервер.
3. Совпадает ли разрешённый origin.
4. Используются ли cookies или другие credentials.
5. Нужен ли preflight.
6. Успешен ли OPTIONS.
7. Разрешены ли нужные методы и заголовки.
8. Не изменяет ли CORS-заголовки Reverse Proxy.
9. Не возвращает ли CDN некорректно кэшированный вариант ответа.

### Важно

CORS Error не обязательно означает, что Backend не получил запрос.

В зависимости от сценария основной запрос мог дойти до сервера и даже изменить данные, но браузер отказал JavaScript в доступе к ответу.

---

## 36. Практический кейс: после логина Cookie не сохраняется

### Что проверить?

- Есть ли Set-Cookie в ответе.
- Корректен ли Domain.
- Корректен ли Path.
- Используется ли Secure.
- Не блокируется ли Cookie из-за SameSite.
- Включены ли credentials при cross-origin запросе.
- Корректны ли CORS-заголовки.
- Нет ли ограничений браузера для third-party cookies.
- Не истёк ли срок действия Cookie.

### Пример

```http
Set-Cookie: session=abc; SameSite=None; Secure; HttpOnly
```

Для cross-origin fetch с cookie может понадобиться:

```javascript
fetch("https://api.example.com/profile", {
  credentials: "include"
});
```

При этом сервер должен корректно разрешать credentialed CORS, если запрос cross-origin.

---

## 37. Практический кейс: API возвращает старые данные

### Возможные причины

- HTTP-кэш браузера.
- CDN-кэш.
- Reverse Proxy-кэш.
- Кэш приложения.
- Кэш БД или репликация.
- Неправильные Cache-Control.
- Некорректные ETag / Last-Modified.

### Что проверить?

1. Есть ли заголовки Cache-Control.
2. Возвращается ли 304.
3. Используется ли ETag.
4. Меняется ли ответ при условной валидации.
5. Проблема воспроизводится только в браузере или также через API-клиент.
6. Проходит ли запрос до Backend.
7. Не читает ли Backend данные с отстающей реплики.

**Важно:** HTTP-кэш и кэш бизнес-данных на Backend — разные уровни системы.

---

## 38. Практический кейс: другой пользователь видит чужие данные

Предположим:

```http
GET /api/profile
Cookie: session=userA
```

Возвращает профиль пользователя A.

После смены пользователя:

```http
GET /api/profile
Cookie: session=userB
```

Ошибочно возвращаются данные пользователя A.

### Возможные причины

- Некорректный серверный кэш.
- Неучёт пользователя в ключе кэша.
- Ошибка авторизации.
- Устаревшая клиентская модель данных.
- Некорректное использование shared cache.

### Что проверить?

- Данные обоих пользователей.
- Authorization / Cookie.
- Cache-Control.
- Vary и другие правила разделения кэша.
- Поведение после logout/login.
- Повтор запроса без кэширования.
- Наличие корректных server-side проверок доступа.

Это критичный дефект безопасности.

---

# Часть 7. Вопросы для собеседования

## 39. Что такое HTTP Header?

Поле HTTP-сообщения, передающее метаданные или управляющую информацию о запросе, ответе и его содержимом.

## 40. Чем Request Headers отличаются от Response Headers?

Request Headers передаются клиентом серверу, Response Headers — сервером клиенту.

## 41. Что такое Origin?

Origin — источник запроса, определяемый схемой, хостом и портом. Используется в браузерной безопасности, включая CORS.

## 42. Чем Origin отличается от Referer?

Origin содержит схему, хост и порт. Referer может содержать более подробный URL исходной страницы, но ограничивается политиками браузера.

## 43. Чем Origin отличается от Host?

Host определяет целевой сервер запроса, а Origin — источник, инициировавший запрос.

## 44. Что такое Same-Origin Policy?

Браузерная политика безопасности, ограничивающая определённые взаимодействия между документами и скриптами разных origin.

## 45. Что такое CORS?

Механизм, при котором сервер через HTTP-заголовки сообщает браузеру, каким origin можно предоставить доступ к ответу.

## 46. Что такое Preflight Request?

Предварительный OPTIONS-запрос браузера, позволяющий проверить разрешения CORS перед определёнными cross-origin запросами.

## 47. Чем Content-Type отличается от Accept?

Content-Type описывает формат содержимого, Accept — приемлемые форматы ответа.

## 48. Чем Cookie отличается от Set-Cookie?

Cookie передаётся клиентом серверу, Set-Cookie — сервером клиенту для установки cookie.

## 49. Что делают HttpOnly, Secure и SameSite?

HttpOnly ограничивает доступ JavaScript, Secure ограничивает передачу cookie защищёнными соединениями, SameSite регулирует отправку cookie в межсайтовых контекстах.

## 50. Чем Cache-Control: no-cache отличается от no-store?

no-cache требует проверки актуальности перед повторным использованием, no-store запрещает кэшам сохранять сообщение.

## 51. Что такое ETag?

Валидатор версии представления ресурса, используемый в кэшировании и условных запросах.

## 52. Для чего используется If-Match?

Для условного выполнения операции, например предотвращения перезаписи ресурса при конкурентном изменении.

## 53. Что такое X-Forwarded-For?

Заголовок, часто используемый прокси для передачи сведений об IP исходного клиента.

Нельзя доверять ему без корректной настройки доверенных прокси.

## 54. Что такое Retry-After?

Заголовок, сообщающий рекомендуемое время перед повторной попыткой запроса.

## 55. Почему Postman может выполнить API-запрос, а браузер получает CORS Error?

Потому что CORS контролируется браузером. Postman не применяет браузерную Same-Origin Policy к доступу к ответу.

---

## Частые ошибки на собеседовании

**Ошибка:** «Origin — это адрес сервера, куда отправляется запрос».

Правильно: Origin указывает источник инициирования запроса. Назначение определяется целевым URI и соответствующими полями вроде Host или :authority.

**Ошибка:** «Origin всегда содержит полный URL страницы».

Правильно: в Origin нет path и query.

**Ошибка:** «Referer и Origin — одинаковые заголовки».

Правильно: они отличаются составом данных и назначением.

**Ошибка:** «Если в запросе нет Origin, значит он безопасный».

Правильно: отсутствие Origin не является доказательством безопасности.

**Ошибка:** «CORS защищает Backend от любых чужих запросов».

Правильно: CORS прежде всего контролирует доступ браузерного JavaScript к ответам. Для защиты Backend нужны аутентификация, авторизация, CSRF-защита и другие меры.

**Ошибка:** «HttpOnly означает, что Cookie не отправляется браузером».

Правильно: HttpOnly ограничивает доступ к cookie через JavaScript, но браузер может отправлять её с запросами.

**Ошибка:** «no-cache означает, что браузер ничего не сохраняет».

Правильно: no-cache допускает хранение, но требует проверки актуальности.

**Ошибка:** «Access-Control-Allow-Origin: * разрешает credentialed requests».

Правильно: wildcard нельзя использовать для разрешения доступа к credentialed CORS responses.

**Ошибка:** «X-Forwarded-For всегда содержит реальный IP пользователя».

Правильно: его значение может быть подделано и должно интерпретироваться в соответствии с доверенной инфраструктурой.

---

## Что нужно знать без подсказки

### Основные заголовки

- [ ] Host.
- [ ] Content-Type.
- [ ] Accept.
- [ ] Accept-Encoding.
- [ ] Content-Encoding.
- [ ] Accept-Language.
- [ ] Authorization.
- [ ] Cookie.
- [ ] Set-Cookie.
- [ ] User-Agent.

### Браузерная безопасность

- [ ] Origin.
- [ ] Referer.
- [ ] Host vs Origin vs Referer.
- [ ] Same-Origin Policy.
- [ ] CORS.
- [ ] OPTIONS Preflight.
- [ ] Access-Control-Allow-Origin.
- [ ] Access-Control-Allow-Credentials.
- [ ] HttpOnly, Secure, SameSite.
- [ ] Sec-Fetch-Site.
- [ ] Content-Security-Policy.

### Кэширование и диагностика

- [ ] Cache-Control.
- [ ] ETag.
- [ ] If-None-Match.
- [ ] If-Match.
- [ ] Last-Modified.
- [ ] Vary.
- [ ] Retry-After.
- [ ] Location.
- [ ] X-Forwarded-For.
- [ ] X-Request-ID / traceparent.

### QA-практика

- [ ] Диагностировать CORS Error.
- [ ] Проверять Cookie после логина.
- [ ] Отличать HTTP-кэш от Backend-кэша.
- [ ] Проверять конкурентные изменения через If-Match.
- [ ] Анализировать Request и Response Headers в DevTools.
- [ ] Обнаруживать небезопасное кэширование приватных данных.

---

## Источники

- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)
- [MDN — HTTP Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)
- [MDN — Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Origin)
- [MDN — Referer](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Referer)
- [MDN — CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)
- [MDN — Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie)
- [MDN — Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Cache-Control)
- [MDN — Sec-Fetch-Site](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Site)
- [MDN — Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy)