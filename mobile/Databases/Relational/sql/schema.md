## SQL Schema — Полное руководство

**Schema** (схема) в контексте SQL и баз данных — это логическая структура, которая определяет организацию данных. В зависимости от контекста, термин может означать разные вещи: от общего проекта базы данных до конкретного объекта в СУБД.

---

##  Что такое Schema?

### В широком смысле (Концептуально)

Schema — это **"чертеж" базы данных**, который описывает:

- Какие таблицы существуют
    
- Какие поля есть в каждой таблице (имена, типы данных)
    
- Связи между таблицами (первичные и внешние ключи)
    
- Ограничения (constraints)
    
- Индексы, представления (views), хранимые процедуры
    

### В узком смысле (В конкретных СУБД)

В разных системах управления базами данных термин "схема" имеет разное значение:

|СУБД|Значение schema|
|---|---|
|**MySQL**|Schema = Database (синонимы). `CREATE SCHEMA` = `CREATE DATABASE`|
|**PostgreSQL**|Schema — это пространство имен внутри базы данных. Одна БД может содержать несколько схем|
|**Oracle**|Schema привязана к пользователю. Каждый пользователь имеет свою схему|
|**SQL Server**|Schema — контейнер для объектов внутри базы данных, принадлежит пользователю|

---

##  Создание и управление схемами

### PostgreSQL (и другие, где схема ≠ БД)

sql

-- Создание схемы
CREATE SCHEMA IF NOT EXISTS sales;
-- Создание схемы с владельцем
CREATE SCHEMA hr AUTHORIZATION user_hr;
-- Удаление схемы (каскадно — удалит всё внутри)
DROP SCHEMA IF EXISTS sales CASCADE;
-- Список всех схем
\dn  -- в psql
SELECT schema_name FROM information_schema.schemata;
-- Установка схемы по умолчанию для сессии
SET search_path TO sales, public;

### Работа с объектами в конкретной схеме

sql

-- Создание таблицы в определенной схеме
CREATE TABLE sales.orders (
    id SERIAL PRIMARY KEY,
    customer_id INT,
    amount DECIMAL(10,2)
);
-- Обращение к таблице через схему
SELECT * FROM sales.orders;
-- Если схема в search_path, можно без префикса
SELECT * FROM orders;  -- если sales в search_path

### MySQL (где schema = database)

sql

-- Создание базы данных (схемы)
CREATE DATABASE IF NOT EXISTS my_store
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
-- Использование схемы
USE my_store;
-- Создание таблицы (автоматически в текущей схеме)
CREATE TABLE products (...);

---