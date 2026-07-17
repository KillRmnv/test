Коротко, чётко и по-экзаменационному 👇

---

## 🔷 Что такое OpenAPI

**OpenAPI (ранее Swagger Specification)** — это **стандарт описания REST API** в машиночитаемом виде (YAML/JSON).

> Позволяет формально описать, **какие есть эндпоинты, методы, параметры, ответы и модели данных**.

---

## 🔷 Зачем нужен OpenAPI

OpenAPI позволяет:

- 📄 Автоматически генерировать документацию (Swagger UI)
    
- 🧪 Генерировать тесты
    
- 💻 Генерировать клиентский код (Java, Python, JS…)
    
- 🧩 Генерировать серверные заглушки (stubs)
    
- 🤝 Договориться фронту и бэку «по контракту»
    

То есть это **контракт API**.

---

## 🔷 В каком виде хранится

Файл:

```
openapi.yaml
```

или

```
openapi.json
```

---

## 🔷 Что описывается в OpenAPI

- Базовый URL
    
- Эндпоинты (`/users`, `/orders`)
    
- HTTP методы
    
- Параметры (path, query, header, body)
    
- Формат запросов и ответов (JSON-схемы)
    
- Коды ответов
    
- Авторизация (Bearer, OAuth2 и т.д.)
    

---

## 🔷 Пример OpenAPI (упрощённый)

```yaml
openapi: 3.0.0
info:
  title: Users API
  version: 1.0.0

paths:
  /users/{id}:
    get:
      summary: Get user by id
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
```

---

## 🔷 Swagger и OpenAPI — в чём разница

|Swagger|OpenAPI|
|---|---|
|Старое название спецификации|Новое официальное название|
|Набор инструментов (UI, Editor)|Сам стандарт описания|

Swagger UI просто **рисует красивую документацию** из OpenAPI файла.

---

## 🔷 Коротко для экзамена

> OpenAPI — это стандарт описания REST API в формате YAML/JSON, который позволяет формально задать эндпоинты, методы, параметры, ответы и модели данных, на основе чего автоматически генерируется документация и код.