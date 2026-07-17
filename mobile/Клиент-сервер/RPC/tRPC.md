
## 🔷 Что такое tRPC

**tRPC** — это фреймворк для **создания типобезопасных API** без необходимости писать схемы отдельно (как в OpenAPI или GraphQL).

> Главная идея: **типизация клиента и сервера на TypeScript полностью синхронизирована**.

- Нет необходимости описывать API в JSON или YAML
    
- Компилятор TypeScript проверяет запросы и ответы **на этапе разработки**
    
- Отлично подходит для Next.js, Node.js, React и других фронтенд/бэкенд стэков на TypeScript
    

---

## 🔷 Как работает tRPC

1. **Создаёшь роуты на сервере**:
    

```ts
import { initTRPC } from '@trpc/server';

const t = initTRPC.create();

export const appRouter = t.router({
  getUser: t.procedure.input((val: number) => val).query((opts) => {
    const id = opts.input;
    return { id, name: "John" };
  }),
});
```

2. **На клиенте вызываешь эти методы напрямую**:
    

```ts
import { createTRPCProxyClient } from '@trpc/client';
import type { AppRouter } from './server';

const client = createTRPCProxyClient<AppRouter>({ url: '/trpc' });

const user = await client.getUser.query(10);
console.log(user.name); // John
```

⚡ Здесь **TypeScript автоматически проверяет** типы аргументов и возвращаемого значения.

---

## 🔷 Преимущества tRPC

|Преимущество|Пояснение|
|---|---|
|Полная типизация|Нет ошибок типов между фронтом и бэком|
|Без схем|Не нужно писать OpenAPI, GraphQL schemas|
|Простота|Прямой вызов методов как локальных функций|
|Быстро|Легковесный транспорт (HTTP/JSON)|
|Подходит для TypeScript|Только для TypeScript, JS тоже можно с any|

---

## 🔷 Отличие от GraphQL и REST

||REST|GraphQL|tRPC|
|---|---|---|---|
|Схема|URL + методы|Schema + Queries/Mutations|TypeScript types|
|Типизация|Нет|Да|Полная на TS|
|Эндпоинты|Много|Один `/graphql`|Много, но прозрачные|
|Кодогенерация|OpenAPI/Swagger|Необязательно|Не нужна|
|Overfetching/Underfetching|Да|Нет|Нет (методы возвращают что нужно)|

---

## 🔷 Примеры использования

- **Next.js приложения** — фронт и бэк вместе на TS
    
- **Микросервисы на Node.js**
    
- **Rapid prototyping** — быстро создать API без писанины схем
    

---

## 🔷 Коротко для экзамена

> tRPC — это TypeScript-фреймворк для создания типобезопасных API, позволяющий фронту вызывать серверные функции напрямую с проверкой типов на этапе компиляции без отдельной схемы, что устраняет ошибки типов и ускоряет разработку.
