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

 Такой момент увидел, что если у нас присутсвуют индексы по каким-то колонкам, то операция = не приемлема, лучше использовать >= и если тип колонки about date, то используем INTERVAL

приведение типа NULL даёт NULL, а не текст 'NULL'

COALESCE — это как раз инструмент «поймать» NULL и подставить заглушку. 

CASE

```sql
-- 1) Простая: только проверка на равенство
CASE status
    WHEN 'completed' THEN 'Завершён'
    WHEN 'cancelled' THEN 'Отменён'
    ELSE 'В работе'
END

-- 2) Поисковая: любые условия
CASE
    WHEN total >= 100000 THEN 'VIP'
    WHEN total >= 10000  THEN 'Крупный'
    ELSE 'Обычный'
END
```