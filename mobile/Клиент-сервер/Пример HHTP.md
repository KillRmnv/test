### 📤 HTTP Request (raw)

```http
POST /api/v1/users HTTP/1.1
Host: example.com
Connection: keep-alive
Cache-Control: no-cache
Pragma: no-cache

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) CustomClient/1.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br

Content-Type: application/json; charset=UTF-8
Content-Length: 72

Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Cookie: sessionId=abc123; theme=dark

Origin: https://example.com
Referer: https://example.com/register

X-Request-ID: 123e4567-e89b-12d3-a456-426614174000
X-Correlation-ID: req-789456
X-Forwarded-For: 203.0.113.42
X-Forwarded-Proto: https

If-None-Match: "etag12345"
If-Modified-Since: Wed, 21 Oct 2025 07:28:00 GMT

{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "qwerty123"
}
```

---

### 🧠 Что тут показано (и зачем)

|Заголовок|Назначение|
|---|---|
|`Host`|Обязателен в HTTP/1.1|
|`Connection`|Управление соединением|
|`Cache-Control`, `Pragma`|Управление кэшированием|
|`User-Agent`|Кто отправляет запрос|
|`Accept*`|Какие ответы клиент готов принять|
|`Content-Type`|Тип тела|
|`Content-Length`|Размер тела|
|`Authorization`|Авторизация (JWT/Bearer)|
|`Cookie`|Сессии|
|`Origin`, `Referer`|Веб-контекст, CORS, безопасность|
|`X-Request-ID`, `X-Correlation-ID`|Трассировка в микросервисах|
|`X-Forwarded-*`|Когда есть прокси / nginx / gateway|
|`If-None-Match`, `If-Modified-Since`|Условные запросы, работа с кэшем (ETag)|

---

### Почему это «универсальный» пример

Такой набор ты встретишь в:

- браузере
    
- Postman
    
- curl
    
- мобильном приложении
    
- микросервисе за API Gateway
    
- за nginx / cloudflare
    
- в продакшене с логированием и трассировкой
    

То есть это **не выдумка**, а почти реальный «боевой» HTTP-запрос.

Если хочешь, могу следующим шагом показать:

- как этот же запрос выглядит в `curl`
    
- как в `Postman`
    
- как в `fetch()`
    
- как в `Java HttpClient` / `C++` / `Python requests`  
    (для экзамена/курса это вообще золото).

## Response

### 📥 HTTP Response (raw)

```http
HTTP/1.1 201 Created
Date: Tue, 02 Feb 2026 12:34:56 GMT
Server: nginx/1.24.0
Connection: keep-alive

Content-Type: application/json; charset=UTF-8
Content-Length: 128
Content-Encoding: gzip

Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Expires: 0

ETag: "etag12345"
Last-Modified: Tue, 02 Feb 2026 12:34:56 GMT

Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Credentials: true

Set-Cookie: sessionId=def456; Path=/; HttpOnly; Secure; SameSite=Strict

X-Request-ID: 123e4567-e89b-12d3-a456-426614174000
X-Correlation-ID: req-789456

Location: https://example.com/api/v1/users/101

{
  "id": 101,
  "username": "john_doe",
  "email": "john@example.com",
  "createdAt": "2026-02-02T12:34:56Z"
}
```

---

### 🧠 Что тут важно

|Заголовок|Зачем нужен|
|---|---|
|`201 Created`|Ресурс создан (правильный код для POST create)|
|`Date`, `Server`|Служебные серверные заголовки|
|`Content-Type`|Тип тела ответа|
|`Content-Encoding`|Сжатие|
|`Cache-Control`|Запрет кэша для чувствительных данных|
|`ETag`, `Last-Modified`|Для последующих условных запросов|
|`Access-Control-*`|CORS для браузеров|
|`Set-Cookie`|Создание сессии|
|`X-Request-ID`, `X-Correlation-ID`|Трассировка запроса в логах|
|`Location`|Где находится созданный ресурс (очень важно для REST)|

---

### Почему это «правильный» REST-ответ

Если на экзамене или в работе спросят **как должен выглядеть корректный ответ на POST create**, то эталон — это:

- `201 Created`
    
- `Location`
    
- JSON с созданной сущностью
    
- Cookie / Auth / CORS / ETag
    

Это прямо **каноничный продакшен**.