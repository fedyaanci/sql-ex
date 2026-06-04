#### Фильтрация данных
На данный момент обучения порядок таков:

SELECT колонка1, колонка2 
FROM таблица                            
WHERE условие                           
ORDER BY колонка ASC / DESC             
LIMIT N;                              

```sql
-- правильный диапазон дат(не анти-паттерн как с between)
WHERE time >= '2022-08-25' AND time < '2022-09-05' 

-- если нужно поработать с днями недели
EXTRACT(DOW FROM time) = 3 --День недели (0=Вс, 1=Пн, ..., 6=Сб): 

-- проверка на свопадение условий 
WHERE user_id IN (170, 200, 230)

-- в выборке проивзодить некоторые логические операции
CASE 
    WHEN условие_1 THEN результат_1
    WHEN условие_2 THEN результат_2
    ELSE результат_по_умолчанию -- Если ничего не подошло (вернет NULL, если не указать)
END AS имя_новой_колонки

-- INTERVAL — арифметика дат
-- INTERVAL позволяет прибавлять или вычитать периоды времени:
SELECT CURRENT_DATE - INTERVAL '30 days';   -- 30 дней назад
SELECT CURRENT_DATE + INTERVAL '1 month';   -- через месяц
SELECT CURRENT_DATE - INTERVAL '1 year';    -- год назад

SELECT *
FROM orders
WHERE created_at >= NOW() - INTERVAL '30 days'

--DATE_TRUNC — усечение даты
--Обрезает дату до нужной точности: дня, недели, месяца, квартала, года
--используется в основном при группировке

SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS revenue,
    COUNT(DISTINCT user_id) AS buyers
FROM orders
WHERE order_date >= DATE_TRUNC('year', CURRENT_DATE)
GROUP BY 1
ORDER BY 1
```