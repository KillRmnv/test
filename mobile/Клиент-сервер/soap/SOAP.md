Вот тут преподаватели любят ловить на отличиях от REST 👇

---

## 🔷 Что такое SOAP

**SOAP (Simple Object Access Protocol)** — это **протокол обмена сообщениями** между приложениями.

> В отличие от REST — это **строгий протокол**, а не архитектурный стиль.

Работает поверх:

- HTTP
    
- SMTP
    
- TCP и др.
    

---

## 🔷 Главная особенность SOAP

Всё общение идёт в формате **XML** и строго по стандарту.

---

## 🔷 Структура SOAP-сообщения

SOAP-сообщение — это XML с чёткой структурой:

```xml
<Envelope>
    <Header>
        <!-- служебная информация -->
    </Header>
    <Body>
        <!-- полезные данные -->
    </Body>
</Envelope>
```

---

## 🔷 Пример SOAP запроса

```xml
POST /UserService HTTP/1.1
Host: example.com
Content-Type: text/xml; charset=utf-8
SOAPAction: "getUser"

<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <getUser>
      <id>10</id>
    </getUser>
  </soap:Body>
</soap:Envelope>
```

---

## 🔷 Что такое WSDL (очень важно)

**WSDL (Web Services Description Language)** — это файл, который описывает:

- какие методы есть
    
- какие параметры принимают
    
- какие ответы возвращают
    

👉 То есть «контракт» SOAP-сервиса.

---

## 🔷 Особенности SOAP

|Особенность|Пояснение|
|---|---|
|Строгий стандарт|Всё по спецификации|
|Только XML|Никакого JSON|
|WSDL|Обязательное описание сервиса|
|Поддержка транзакций, безопасности (WS-Security)|Встроено в стандарт|
|Сложный и тяжёлый|Большие сообщения|

---

## 🔷 Где используется

- Банковские системы
    
- Государственные системы
    
- Корпоративные интеграции (enterprise)
    

Где важны:

- надёжность
    
- формальный контракт
    
- безопасность
    
- транзакционность
    

---

## 🔷 SOAP vs REST (любимая таблица)

|SOAP|REST|
|---|---|
|Протокол|Архитектурный стиль|
|XML|Обычно JSON|
|Строгий стандарт|Гибкость|
|WSDL|OpenAPI (необязательно)|
|Тяжёлый|Лёгкий|
|Безопасность встроена|Реализуется отдельно|
|Методы как функции|Ресурсы как сущности|


## Response

### 📥 HTTP + SOAP Response (raw)

```http
HTTP/1.1 200 OK
Date: Tue, 02 Feb 2026 12:40:00 GMT
Server: Apache-CXF/4.0.0
Content-Type: text/xml; charset=utf-8
Content-Length: 658
Connection: keep-alive
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope
    xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    
    <soap:Header>
        <auth:Security
            xmlns:auth="http://example.com/security">
            <auth:Token>abc123token</auth:Token>
        </auth:Security>
    </soap:Header>

    <soap:Body>
        <ns:getUserResponse
            xmlns:ns="http://example.com/userservice">
            
            <ns:user>
                <ns:id>10</ns:id>
                <ns:name>John Doe</ns:name>
                <ns:email>john@example.com</ns:email>
                <ns:createdAt>2026-02-02T12:39:50Z</ns:createdAt>
            </ns:user>

        </ns:getUserResponse>
    </soap:Body>
</soap:Envelope>
```

---

## 🧠 Что здесь показано

|Часть|Назначение|
|---|---|
|`HTTP/1.1 200 OK`|Успешный ответ|
|`Content-Type: text/xml`|SOAP всегда XML|
|`<Envelope>`|Обязательная оболочка SOAP|
|`<Header>`|Служебная инфа (безопасность, токены, транзакции)|
|`<Body>`|Полезные данные|
|`getUserResponse`|Ответ на вызов метода|
|`xmlns`|Пространства имён (обязательно в SOAP)|

---

## ⚠️ Важно понимать

В SOAP **методы выглядят как функции**, поэтому:

- Запрос: `<getUser>`
    
- Ответ: `<getUserResponse>`
    

Это принципиальное отличие от REST.
