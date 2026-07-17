
**ACID** — это аббревиатура, описывающая требования к транзакционной системе, гарантирующие надежность и предсказуемость работы с данными.

## 📚 Определение ACID

| Буква | Термин | Значение |
|:---|:---|:---|
| **A** | **Atomicity** (Атомарность) | "Все или ничего" |
| **C** | **Consistency** (Согласованность) | Данные всегда целостны |
| **I** | **Isolation** (Изоляция) | Транзакции не мешают друг другу |
| **D** | **Durability** (Долговечность) | Зафиксированные данные не пропадают |

---

## 1️⃣ Atomicity (Атомарность)

Транзакция выполняется **целиком или не выполняется вообще**. Нет понятия "частичного выполнения".

### Пример

```sql
-- Банковский перевод: списать с одного счета, зачислить на другой
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT; -- или ROLLBACK при ошибке
```

Если после первого UPDATE произойдет сбой, второй не выполнится — все откатится.

### В JDBC
```java
connection.setAutoCommit(false);

try {
    // Операции
    pstmt1.executeUpdate();
    pstmt2.executeUpdate();
    
    connection.commit(); // Фиксируем всё сразу
} catch (SQLException e) {
    connection.rollback(); // Откатываем всё при ошибке
}
```

---

## 2️⃣ Consistency (Согласованность)

Транзакция переводит базу данных из **одного целостного состояния в другое**. Все ограничения, правила, триггеры должны соблюдаться.

### Виды ограничений
```sql
-- 1. Первичные ключи (PRIMARY KEY)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,  -- уникальность гарантирована
    email VARCHAR(255) UNIQUE  -- дубликаты запрещены
);

-- 2. Внешние ключи (FOREIGN KEY)
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id)  -- нельзя сослаться на несуществующего пользователя
);

-- 3. Проверки (CHECK)
CREATE TABLE products (
    price DECIMAL CHECK (price > 0),  -- цена не может быть отрицательной
    status VARCHAR(20) CHECK (status IN ('active', 'inactive'))
);

-- 4. NOT NULL
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
```

### Нарушение согласованности
```sql
-- Ошибка! Нарушает CHECK (price > 0)
INSERT INTO products (price) VALUES (-100);  

-- Ошибка! Нарушает FOREIGN KEY
INSERT INTO orders (user_id) VALUES (99999);  
```

---

## 3️⃣ Isolation (Изоляция)

Транзакции не должны "видеть" промежуточные состояния друг друга. Уровни изоляции решают, какие "аномалии" допустимы.

### Проблемы параллельного доступа

| Проблема | Описание | Пример |
|:---|:---|:---|
| **Dirty Read** (грязное чтение) | Чтение незафиксированных данных другой транзакции | Транзакция А видит данные, которые Б потом откатит |
| **Non-repeatable Read** (неповторяющееся чтение) | При повторном чтении те же строки дают разные значения | Между двумя SELECT-ами данные изменили |
| **Phantom Read** (фантомное чтение) | При повторном запросе появляются новые строки | Добавили новые записи, соответствующие условию |

### Уровни изоляции (ANSI SQL)

| Уровень | Dirty Read | Non-repeatable Read | Phantom Read |
|:---|::---:|:---:|:---:|
| **READ UNCOMMITTED** | Возможно | Возможно | Возможно |
| **READ COMMITTED** | Невозможно | Возможно | Возможно |
| **REPEATABLE READ** | Невозможно | Невозможно | Возможно |
| **SERIALIZABLE** | Невозможно | Невозможно | Невозможно |

### Настройка в PostgreSQL

```sql
-- Установка уровня изоляции для сессии
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Для конкретной транзакции
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- операции
COMMIT;
```

### В JDBC
```java
// Установка уровня изоляции для соединения
connection.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);

// Значения:
// Connection.TRANSACTION_NONE = 0
// Connection.TRANSACTION_READ_UNCOMMITTED = 1
// Connection.TRANSACTION_READ_COMMITTED = 2
// Connection.TRANSACTION_REPEATABLE_READ = 4
// Connection.TRANSACTION_SERIALIZABLE = 8
```

### Пример проблемы и решения

```sql
-- Транзакция 1                |  -- Транзакция 2
BEGIN;                         |  BEGIN;
                               |
SELECT sum(balance) FROM accounts;
-- результат: 1000             |
                               |  UPDATE accounts SET balance = balance - 100 WHERE id=1;
                               |  -- не закоммичено!
SELECT sum(balance) FROM accounts;
-- при READ UNCOMMITTED: 900   |  
-- при READ COMMITTED: 1000    |
                               |  COMMIT;
SELECT sum(balance) FROM accounts;
-- при READ COMMITTED: 900     |
COMMIT;                        |
```

---

## 4️⃣ Durability (Долговечность)

После подтверждения транзакции (`COMMIT`) данные сохраняются **даже при сбоях** (отключение питания, крах системы).

### Как обеспечивается
- **WAL (Write-Ahead Logging)** — сначала запись в журнал, потом в данные
- Репликация
- Резервное копирование

### В PostgreSQL
```sql
-- Настройка durability (в postgresql.conf)
fsync = on                -- сброс на диск
synchronous_commit = on   -- синхронный коммит
full_page_writes = on     -- защита от частичной записи
```

### Исключения (для производительности)
Можно ослабить durability ради скорости:
```sql
-- Для не критичных данных
SET synchronous_commit TO OFF;
-- Данные могут потеряться при сбое, но быстрее
```

---

## 🔄 Пример полной ACID транзакции

### Ситуация: покупка билета
1. Проверить наличие
2. Забронировать место
3. Списать деньги
4. Подтвердить покупку

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- 1. Проверка места (с блокировкой)
SELECT * FROM seats 
WHERE id = 10 AND is_free = true 
FOR UPDATE;  -- блокируем строку

-- Если место занято -> ROLLBACK

-- 2. Бронируем место
UPDATE seats SET is_free = false, user_id = 123 WHERE id = 10;

-- 3. Списание денег
UPDATE accounts SET balance = balance - 500 WHERE user_id = 123;

-- 4. Создание билета
INSERT INTO tickets (user_id, seat_id, price) VALUES (123, 10, 500);

COMMIT;  -- ВСЁ ИЛИ НИЧЕГО
```

### Java код
```java
public void buyTicket(int userId, int seatId, BigDecimal price) 
        throws SQLException {
    
    String checkSeatSql = "SELECT is_free FROM seats WHERE id = ? FOR UPDATE";
    String bookSeatSql = "UPDATE seats SET is_free = false, user_id = ? WHERE id = ? AND is_free = true";
    String chargeSql = "UPDATE accounts SET balance = balance - ? WHERE user_id = ? AND balance >= ?";
    String createTicketSql = "INSERT INTO tickets (user_id, seat_id, price) VALUES (?, ?, ?)";
    
    connection.setAutoCommit(false);
    connection.setTransactionIsolation(Connection.TRANSACTION_SERIALIZABLE);
    
    try {
        // Проверка места с блокировкой
        try (PreparedStatement pstmt = connection.prepareStatement(checkSeatSql)) {
            pstmt.setInt(1, seatId);
            try (ResultSet rs = pstmt.executeQuery()) {
                if (!rs.next() || !rs.getBoolean("is_free")) {
                    throw new SQLException("Seat not available");
                }
            }
        }
        
        // Бронирование
        try (PreparedStatement pstmt = connection.prepareStatement(bookSeatSql)) {
            pstmt.setInt(1, userId);
            pstmt.setInt(2, seatId);
            if (pstmt.executeUpdate() == 0) {
                throw new SQLException("Failed to book seat");
            }
        }
        
        // Списание денег
        try (PreparedStatement pstmt = connection.prepareStatement(chargeSql)) {
            pstmt.setBigDecimal(1, price);
            pstmt.setInt(2, userId);
            pstmt.setBigDecimal(3, price);
            if (pstmt.executeUpdate() == 0) {
                throw new SQLException("Insufficient funds");
            }
        }
        
        // Создание билета
        try (PreparedStatement pstmt = connection.prepareStatement(createTicketSql)) {
            pstmt.setInt(1, userId);
            pstmt.setInt(2, seatId);
            pstmt.setBigDecimal(3, price);
            pstmt.executeUpdate();
        }
        
        connection.commit();  // ✅ Всё хорошо
        
    } catch (SQLException e) {
        connection.rollback();  // ❌ Откат всего
        throw e;
    } finally {
        connection.setAutoCommit(true);
    }
}
```

---

## 🎯 PostgreSQL особенности

### 1. **Уровни изоляции в PostgreSQL**

| Уровень | Реальная реализация | Защита |
|:---|:---|:---|
| READ UNCOMMITTED | READ COMMITTED | В PostgreSQL READ UNCOMMITTED = READ COMMITTED |
| READ COMMITTED | READ COMMITTED | Нет dirty read |
| REPEATABLE READ | Снапшот (Snapshot Isolation) | Нет dirty read, нет non-repeatable read |
| SERIALIZABLE | Serializable Snapshot Isolation (SSI) | Полная изоляция |

### 2. **Проверка текущего уровня**
```sql
SHOW transaction_isolation;
-- или
SELECT current_setting('transaction_isolation');
```

### 3. **Блокировки (FOR UPDATE и др.)**
```sql
-- Блокировка строк для обновления
SELECT * FROM products WHERE id = 1 FOR UPDATE;

-- Блокировка без ожидания (сразу ошибка)
SELECT * FROM products WHERE id = 1 FOR UPDATE NOWAIT;

-- Пропустить заблокированные
SELECT * FROM products WHERE id = 1 FOR UPDATE SKIP LOCKED;

-- Блокировка в режиме "ключ-значение" (для внешних ключей)
SELECT * FROM orders WHERE user_id = 1 FOR KEY SHARE;
```

---

## 📊 Сравнение с NoSQL

| Свойство | SQL (ACID) | NoSQL (BASE) |
|:---|:---|:---|
| **A** | Атомарность | **B** - Basically Available (всегда доступно) |
| **C** | Согласованность | **S** - Soft state (состояние может меняться) |
| **I** | Изоляция | **E** - Eventual consistency (итоговая согласованность) |
| **D** | Долговечность | |

**BASE** = **B**asically **A**vailable, **S**oft state, **E**ventual consistency

---

## 💡 Best Practices

1. **Всегда используйте транзакции** для связанных операций
2. **Выбирайте правильный уровень изоляции**:
   - READ COMMITTED — для большинства задач
   - SERIALIZABLE — для финансовых операций
3. **Не держите транзакции долго открытыми** (блокировки)
4. **Обрабатывайте deadlocks** (повторяйте транзакцию)
5. **Используйте savepoints** для частичного отката

```java
// Savepoint пример
Savepoint sp = connection.setSavepoint("after_insert");
try {
    // операции...
    connection.releaseSavepoint(sp);
} catch (SQLException e) {
    connection.rollback(sp);  // откат только до savepoint
    connection.releaseSavepoint(sp);
}
```