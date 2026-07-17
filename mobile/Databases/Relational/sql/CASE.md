```sql
SELECT first_name, last_name,
CASE
  WHEN TIMESTAMPDIFF(YEAR, birthday, NOW()) >= 18 THEN 'Совершеннолетний'
  ELSE 'Несовершеннолетний'
END AS status
FROM Student
```
