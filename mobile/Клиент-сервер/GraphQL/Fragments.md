## 🔷 Что такое Fragments в GraphQL

**Fragment** — это **переиспользуемый кусок запроса** с набором полей.

> Позволяет не дублировать одинаковые поля в разных запросах.

---

## 🔷 Проблема без fragments

Есть тип `User`:

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  avatar: String!
}
```

И в разных местах ты пишешь:

```graphql
user {
  id
  name
  email
  avatar
}
```

И так в 10 запросах.

Если добавится поле — нужно менять везде.

---

## 🔷 Решение — Fragment

```graphql
fragment UserFields on User {
  id
  name
  email
  avatar
}
```

Теперь используем:

```graphql
query {
  user(id: 10) {
    ...UserFields
  }
}
```

`...` — это «подставь fragment сюда».

---

## 🔷 Можно использовать в нескольких местах

```graphql
query {
  user(id: 10) {
    ...UserFields
  }
  admins {
    ...UserFields
  }
}
```

---

## 🔷 Главное правило

Fragment всегда объявляется:

```
fragment <Name> on <Type>
```

То есть он **привязан к типу из Schema**.

---

## 🔷 Зачем это реально нужно

1. Переиспользование
    
2. Поддерживаемость
    
3. Чистота запросов
    
4. Удобно во фронтенде (React/Apollo это активно используют)
    

---

## 🔷 Более продвинутый пример (вложенность)

```graphql
fragment OrderFields on Order {
  id
  price
}

fragment UserFields on User {
  id
  name
  orders {
    ...OrderFields
  }
}
```

---

## 🔷 Коротко для экзамена

> Fragment в GraphQL — это переиспользуемый набор полей, привязанный к определённому типу схемы, который позволяет избегать дублирования кода в запросах и упрощает их поддержку.