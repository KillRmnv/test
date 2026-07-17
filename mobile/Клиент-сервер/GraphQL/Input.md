
## 🔷 Что такое `input` в GraphQL

`input` — это **тип данных, который используется для передачи данных ВНУТРЬ мутаций (и иногда запросов)**.

> Это способ **структурированно передать объект в аргументы**.

---

## 🔷 Зачем нужен `input`

Без `input` мутация выглядела бы так:

```graphql
mutation {
  createUser(
    name: "John",
    email: "john@example.com",
    password: "123"
  ) {
    id
  }
}
```

Если полей 10–15 — это превращается в ад.

---

## 🔷 Решение — input type

В схеме:

```graphql
input CreateUserInput {
  name: String!
  email: String!
  password: String!
}
```

И мутация:

```graphql
type Mutation {
  createUser(input: CreateUserInput!): User
}
```

---

## 🔷 Как выглядит запрос

```graphql
mutation {
  createUser(
    input: {
      name: "John"
      email: "john@example.com"
      password: "123"
    }
  ) {
    id
    name
  }
}
```

---

## 🔷 В чём разница между `type` и `input`

| `type`                                      | `input`                                   |
| ------------------------------------------- | ----------------------------------------- |
| Используется в ответах                      | Используется в запросах                   |
| Может содержать связи, методы               | Только поля                               |
| Описывает данные, которые возвращает сервер | Описывает данные, которые клиент передаёт |

Нельзя использовать `type User` как аргумент. Только `input`.

---

## 🔷 Можно вкладывать input в input

```graphql
input AddressInput {
  city: String!
  street: String!
}

input CreateUserInput {
  name: String!
  address: AddressInput!
}
```

---

## 🔷 Коротко для экзамена

> `input` в GraphQL — это специальный тип данных, предназначенный для передачи структурированных данных от клиента в мутации или запросы, позволяющий группировать аргументы в единый объект.