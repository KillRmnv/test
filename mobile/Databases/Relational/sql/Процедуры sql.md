Сравнительная таблица   [[Пользовательские функции SQL]] и процедуры 

| Критерий                        | Функции (FUNCTION) | Процедуры (PROCEDURE) |
| ------------------------------- | ------------------ | --------------------- |
| **Возврат значения**            | Обязательно        | Не обязательно        |
| **Использование в SELECT**      | Да                 | Нет                   |
| **Управление транзакциями**     | Нет                | Да                    |
| **OUT параметры**               | Редко              | Часто                 |
| **Вызов**                       | `SELECT func()`    | `CALL proc()`         |
| **Возврат нескольких значений** | Через TABLE        | Через OUT params      |
| **Изменение состояния БД**      | Осторожно          | Основная цель         |
1. ****Performance Optimization****: Since stored procedures are precompiled, they execute faster than running ad-hoc SQL queries. The database engine can reuse the execution plan, eliminating the need for repeated query parsing and optimization.  
    
2. ****Security and Data Access Control****: By using stored procedures, developers can restrict direct access to sensitive data. Users can execute procedures without accessing the underlying tables, helping to protect critical information.  
    
3. ****Code Reusability and Maintainability****: SQL stored procedures can be reused in multiple applications or different parts of an application. This reduces the need to rewrite complex queries repeatedly.  
    
4. ****Reduced Network Traffic****: Instead of sending multiple individual queries to the database server, stored procedures allow you to execute multiple operations in one go, reducing network load.  
    
5. ****Maintainability****: Stored procedures simplify code maintenance. Changes made to the procedure are automatically reflected wherever the procedure is used, making it easier to manage complex logic.





CREATE PROCEDURE _procedure_name_  
AS  
_sql_statement_  
GO;

EXEC _procedure_name_;



CREATE PROCEDURE SelectAllCustomers  
AS  
SELECT * FROM Customers  
GO;


С ПАРАМЕТРОМ
CREATE PROCEDURE SelectAllCustomers @City nvarchar(30)  
AS  
SELECT * FROM Customers WHERE City = @City  
GO;

EXEC SelectAllCustomers @City = 'London';




CREATE PROCEDURE get_employee_info(
    IN emp_id INT, 
    OUT emp_name VARCHAR, 
    OUT emp_salary DECIMAL
)
AS \$$
BEGIN
    SELECT name, salary INTO emp_name, emp_salary 
    FROM employees WHERE id = emp_id;
END;
\$$ LANGUAGE plpgsql;