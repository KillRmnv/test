## 🔷 Что такое gRPC

**gRPC** — это **фреймворк удалённых вызовов процедур (RPC) от Google**, который позволяет клиенту вызывать методы на сервере так, будто они локальные.

- Использует **HTTP/2** как транспорт
    
- Использует **Protocol Buffers (protobuf)** для сериализации данных
    
- Высокая производительность, меньше трафика, бинарный формат
    

> Проще говоря, это «RPC по сети с типами и контрактом».

---

## 🔷 Основные особенности gRPC

|Особенность|Пояснение|
|---|---|
|HTTP/2|Мультиплексирование, сжатие, низкая задержка|
|Протокол Buffers|Бинарный формат вместо JSON/XML, быстрый и компактный|
|Строгая типизация|Методы и данные описаны в `.proto` файле|
|Кодогенерация|Автоматически генерирует клиент и сервер|
|Поддержка потоков|Streaming запросы и ответы (uni/bi-directional)|

---

## 🔷 Типы вызовов в gRPC

1. **Unary RPC** — один запрос → один ответ (как обычный REST GET/POST)
    
2. **Server Streaming RPC** — один запрос → поток ответов от сервера
    
3. **Client Streaming RPC** — поток запросов → один ответ от сервера
    
4. **Bidirectional Streaming RPC** — поток запросов ↔ поток ответов (реальное время)
    

---

## 🔷 Пример `.proto` файла

```proto
syntax = "proto3";

service UserService {
  // Unary RPC
  rpc GetUser(UserRequest) returns (UserResponse);

  // Server streaming
  rpc ListUsers(Empty) returns (stream UserResponse);
}

// Сообщения
message UserRequest {
  int32 id = 1;
}

message UserResponse {
  int32 id = 1;
  string name = 2;
  string email = 3;
}
```

---

## 🔷 Пример вызова Unary RPC

**Клиент:**

```python
response = stub.GetUser(UserRequest(id=10))
print(response.name)
```

**Сервер:**

```python
def GetUser(self, request, context):
    return UserResponse(id=request.id, name="John", email="john@example.com")
```

---

## 🔷 Преимущества gRPC

- Высокая производительность и меньший трафик
    
- Кодогенерация клиента и сервера
    
- Чёткая типизация
    
- Поддержка потоков и real-time
    
- Подходит для микросервисной архитектуры
    

---

## 🔷 Отличия от REST, SOAP и GraphQL

![[Pasted image 20260202142023.png]]
---

