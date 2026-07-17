

Отличный вопрос! Это одна из фундаментальных тем в SQL. На первый взгляд, два подхода делают одно и то же, но различия критически важны.

---

## 🔍 Что вы имеете в виду под "обычным указанием"

Старый синтаксис (SQL-89) — **неявное соединение (implicit join)**:

sql

SELECT *
FROM employees, departments
WHERE employees.dept_id = departments.id;

Современный синтаксис (SQL-92) — **явное соединение (explicit JOIN)**:

sql

SELECT *
FROM employees
JOIN departments ON employees.dept_id = departments.id;

---

## ⚖️ Сравнение подходов

### 1️⃣ **Читаемость и понятность**

|Неявный (старый)|Явный (JOIN)|
|---|---|
|`sql<br>SELECT *<br>FROM employees e, departments d, locations l, countries c<br>WHERE e.dept_id = d.id<br> AND d.location_id = l.id<br> AND l.country_id = c.id<br> AND e.salary > 50000<br> AND c.name = 'USA';<br>`|`sql<br>SELECT *<br>FROM employees e<br>JOIN departments d ON e.dept_id = d.id<br>JOIN locations l ON d.location_id = l.id<br>JOIN countries c ON l.country_id = c.id<br>WHERE e.salary > 50000<br> AND c.name = 'USA';<br>`|

**Победитель: JOIN (явный)** ✅

- Сразу видно, **как** связаны таблицы
    
- Условия соединения отделены от фильтров
    
- Логика чтения: "от employees через departments к locations"
    

### 2️⃣ **Случайное перекрестное соединение**

Неявный синтаксис — **бомба замедленного действия**:

sql

-- Забыли WHERE — получили декартово произведение!
SELECT * FROM employees, departments;
-- Если 1000 сотрудников и 20 отделов = 20000 строк
-- Никакой ошибки! Просто дико медленно
-- С JOIN так не получится:
SELECT * FROM employees
JOIN departments;  -- ОШИБКА! Нужно ON или USING

**Победитель: JOIN (явный)** ✅

- Требует явного указания условия соединения
    
- Невозможно случайно сделать CROSS JOIN
    

### 3️⃣ **Разные типы JOIN**

Неявный синтаксис **не поддерживает** многие типы соединений:

sql

-- LEFT JOIN (все сотрудники, даже без отдела)
SELECT * FROM employees
LEFT JOIN departments ON employees.dept_id = departments.id;
-- Как это сделать неявно? Очень криво через UNION или вообще никак
SELECT * FROM employees, departments 
WHERE employees.dept_id = departments.id
UNION
SELECT *, NULL, NULL FROM employees 
WHERE dept_id NOT IN (SELECT id FROM departments);
-- Ужасно, медленно, нечитаемо

**Победитель: JOIN (явный)** ✅

- FULL JOIN, LEFT JOIN, RIGHT JOIN — только с явным синтаксисом
    
- Естественные соединения (NATURAL JOIN)
    
- Кросс-соединения (CROSS JOIN) — явно и понятно
    

### 4️⃣ **Производительность**

**Сюрприз:** для простых INNER JOIN разницы **нет**!

Современные оптимизаторы (PostgreSQL, MySQL 8+, Oracle, SQL Server) превращают оба запроса в **один и тот же план выполнения**:

sql

-- Неявный
EXPLAIN SELECT * FROM employees, departments 
WHERE employees.dept_id = departments.id;
-- Явный
EXPLAIN SELECT * FROM employees 
JOIN departments ON employees.dept_id = departments.id;

План будет идентичным. Оптимизатор видит суть, а не синтаксис.

**Но для OUTER JOIN разница есть** — неявный синтаксис их просто не поддерживает.

### 5️⃣ **Сложные условия соединения**

sql

-- JOIN с дополнительными условиями
SELECT * FROM orders o
JOIN order_items i 
    ON o.id = i.order_id 
    AND i.quantity > 0  -- Условие именно на соединение
    AND i.discount < 0.5;
-- В WHERE это перемешивается с фильтрацией
SELECT * FROM orders o, order_items i
WHERE o.id = i.order_id 
  AND i.quantity > 0      -- Это условие соединения или фильтр?
  AND i.discount < 0.5    -- Непонятно
  AND o.total > 100;       -- А это точно фильтр

**Победитель: JOIN** — можно четко разделить:

- Условия в ON — как соединяем
    
- Условия в WHERE — что фильтруем после соединения