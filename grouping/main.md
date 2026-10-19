# Группировка данных — мини-конспект

## Суть

Свернуть много строк в группы и посчитать агрегат **по каждой группе**.

```sql
SELECT col, agg(...)
FROM   t
WHERE  ...            -- фильтр строк ДО группировки
GROUP  BY col
HAVING ...            -- фильтр групп ПОСЛЕ группировки
ORDER  BY ...;
```

## Правило №1

**Все неагрегированные колонки из `SELECT` должны быть в `GROUP BY`.**

```sql
-- ❌ ошибка: order_id не в GROUP BY и не под агрегатом
SELECT user_id, order_id, COUNT(*) FROM user_actions GROUP BY user_id;

-- ✅
SELECT user_id, COUNT(*) FROM user_actions GROUP BY user_id;
```

## Порядок выполнения

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

Следствия:

- агрегат в `WHERE` **нельзя** → только `HAVING`;
- `WHERE` фильтрует строки, `HAVING` — группы;
- псевдоним из `SELECT` не виден в `WHERE`, но виден в `ORDER BY` (и в `GROUP BY` — послабление PostgreSQL).

## Что можно класть в `GROUP BY`

| Что | Пример |
|---|---|
| Колонку | `GROUP BY user_id` |
| Несколько колонок | `GROUP BY user_id, action` |
| Выражение | `GROUP BY DATE_TRUNC('month', time)::date` |
| Номер колонки | `GROUP BY 1` |
| Псевдоним (PG) | `GROUP BY month` |

## Агрегаты и NULL

| Что | Как ведёт себя |
|---|---|
| `COUNT(*)` | считает все строки |
| `COUNT(col)` | считает только не-NULL |
| `COUNT(DISTINCT col)` | уникальные не-NULL |
| `SUM` / `AVG` / `MIN` / `MAX` | игнорируют NULL |
| `SUM`/`AVG` на пустом множестве | `NULL`, а не `0` |

**NULL в группировке**: все строки с `NULL` схлопываются в **одну** группу.

## `HAVING`

```sql
GROUP BY user_id HAVING COUNT(*) > 3
```

- Можно агрегат, которого нет в `SELECT`.
- Без `GROUP BY` работает: вся таблица — одна группа.

## Условная агрегация (сводная таблица)

```sql
COUNT(*) FILTER (WHERE action = 'cancel_order')
COUNT(CASE WHEN action = 'cancel_order' THEN 1 END)
SUM(CASE WHEN action = 'cancel_order' THEN 1 ELSE 0 END)
```

## Шаблоны — копируй и меняй

```sql
-- топ-5 по количеству
SELECT user_id, COUNT(*) AS cnt
FROM   user_actions
GROUP  BY user_id
ORDER  BY cnt DESC, user_id
LIMIT  5;

-- разрез по месяцам
SELECT DATE_TRUNC('month', time)::date AS month, COUNT(*) AS cnt
FROM   user_actions
GROUP  BY 1
ORDER  BY 1;

-- сводка по статусам за один проход
SELECT DATE_TRUNC('month', time)::date AS month,
       COUNT(*) AS total,
       COUNT(*) FILTER (WHERE action = 'create_order') AS created,
       COUNT(*) FILTER (WHERE action = 'cancel_order') AS canceled
FROM   user_actions
GROUP  BY 1
ORDER  BY 1;

-- группы по условию
SELECT user_id, COUNT(*) AS cnt
FROM   user_actions
WHERE  action = 'create_order'
GROUP  BY user_id
HAVING COUNT(*) > 3
ORDER  BY cnt DESC, user_id;
```

## Грабли

1. Колонка не в `GROUP BY` → ошибка.
2. `COUNT(*)` ≠ `COUNT(col)` — второй теряет NULL.
3. `AVG(col)` игнорирует NULL → знаменатель меньше `COUNT(*)`.
4. Групп без данных не бывает: месяца без строк в результате не будет.
5. `SUM` после `JOIN` «один ко многим» раздувается.
6. `LIMIT` без `ORDER BY` → случайные строки.
7. Даты — диапазон `>= начало AND < следующий день`, а не `BETWEEN` по `timestamp`.

## Чек-лист перед отправкой запроса

- [ ] все неагрегированные колонки в `GROUP BY`
- [ ] фильтр строк — `WHERE`, фильтр групп — `HAVING`
- [ ] `ORDER BY` детерминирован (добавлен tie-breaker: `user_id`)
- [ ] NULL учтён (`COUNT(*)`, `COALESCE`)
- [ ] даты заданы диапазоном
