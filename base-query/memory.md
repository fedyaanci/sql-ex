### Базовые запросы SQL.

##### Порядок выполнения отличается от того, в какой последовательности указываются: FROM, SELECT, ORDER BY, LIMIT 

```sql
CREATE TABLE employees
(
    id INTEGER PRIMARY KEY,
    first_name VARCHAR(50),
    LAST_NAME VARCHAR(50),
    hire_date DATE,
    salary DEC(10,2) NOT NULL CHECK(SALARY > 0)
    bonus DEC(10,2) 
)

SELECT 
    CONCAT(first_name, ' ', last_name),
    DATE_PART('year', hire_date) as hire_year,
    salary + COALESCE(bonus, 0) as total_income,
    CASE
        WHEN salary + COALESCE(bonus, 0) >= 100000 THEN 'высокий'
        WHEN salary + COALESCE(bonus, 0) >= 60000 THEN 'средний'
        ELSE 'низкий'
    END AS income_level
FROM employees
ORDER BY total_income desc
LIMIT 3
```