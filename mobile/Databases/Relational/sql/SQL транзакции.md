An SQL transaction is a sequence of one or more SQL operations (e.g., `INSERT`, `UPDATE`, `DELETE`) executed as a single unit of work. Transactions ensure that either all operations succeed or none are applied, maintaining data integrity.

ПРИМЕР:
START TRANSACTION [transaction_name];

-- Списание денег с первого счета
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;

-- Зачисление денег на второй счет
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;

-- Проверка, что оба счета существуют и достаточно средств
IF (SELECT balance FROM accounts WHERE id = 1) < 0 THEN
    [[ROLLBACK]];
    [[RAISE EXCEPTION]] 'Недостаточно средств на счете 1';
ELSE
    [[COMMIT]];
END IF;




[[SAVEPOINT]] SP1;  
DELETE FROM Student WHERE AGE = 20;    
SAVEPOINT SP2;  

ROLLBACK TO SAVEPOINT SP1;