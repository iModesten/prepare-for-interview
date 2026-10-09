# SQL — основы и продвинутые запросы

## 1. Что такое SQL?

**SQL (Structured Query Language)** — декларативный язык для работы с реляционными базами данных.

С помощью SQL можно:

- Получать данные.
- Добавлять, изменять и удалять записи.
- Создавать и изменять структуру БД.
- Управлять доступом.
- Управлять транзакциями.

SQL является декларативным языком: мы описываем, какой результат хотим получить, а СУБД самостоятельно выбирает способ выполнения запроса.

### Краткий ответ для собеседования

> SQL — язык структурированных запросов для управления данными в реляционных базах данных. Он позволяет выполнять выборку, добавление, изменение и удаление данных, управлять структурой таблиц, транзакциями и правами доступа.

---

## 2. Основные группы SQL-команд

| Группа | Назначение | Команды |
|---|---|---|
| DDL | Определение структуры БД | CREATE, ALTER, DROP, TRUNCATE |
| DML | Изменение данных | INSERT, UPDATE, DELETE |
| DQL | Получение данных | SELECT |
| DCL | Управление доступом | GRANT, REVOKE |
| TCL | Управление транзакциями | COMMIT, ROLLBACK, SAVEPOINT |

DQL часто рассматривают как часть DML, поэтому классификация может немного различаться.

### DDL — Data Definition Language

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE
);
```

```sql
ALTER TABLE users
ADD COLUMN status VARCHAR(20);
```

```sql
DROP TABLE users;
```

### DML — Data Manipulation Language

```sql
INSERT INTO users (id, name)
VALUES (1, 'Anna');
```

```sql
UPDATE users
SET name = 'Maria'
WHERE id = 1;
```

```sql
DELETE FROM users
WHERE id = 1;
```

**Важно:** особенности транзакционности DDL различаются между СУБД. Например, многие DDL-операции в PostgreSQL транзакционны, а MySQL обычно выполняет неявный commit при ряде DDL-команд.

---

## 3. Тестовые таблицы

Для примеров будем использовать три связанные таблицы.

### users

| id | name | city |
|---|---|---|
| 1 | Anna | Minsk |
| 2 | Bob | Moscow |
| 3 | Kate | Minsk |
| 4 | John | NULL |

### orders

| id | user_id | amount | status |
|---|---|---|---|
| 101 | 1 | 100 | PAID |
| 102 | 1 | 200 | NEW |
| 103 | 2 | 300 | PAID |
| 104 | 3 | 150 | PAID |
| 105 | 3 | 250 | CANCELLED |

### payments

| id | order_id | amount |
|---|---|---|
| 1001 | 101 | 100 |
| 1002 | 103 | 300 |
| 1003 | 104 | 150 |

Пользователь может иметь несколько заказов, а заказ может иметь несколько платежей.

Последнее важно учитывать при JOIN: связь «один ко многим» может приводить к размножению строк результата.

---

## 4. SELECT — получение данных

### Получить все строки

```sql
SELECT *
FROM users;
```

### Выбрать конкретные колонки

```sql
SELECT id, name
FROM users;
```

### Использовать псевдонимы

```sql
SELECT
    id AS user_id,
    name AS user_name
FROM users;
```

### WHERE — фильтрация

```sql
SELECT *
FROM users
WHERE city = 'Minsk';
```

Основные операторы:

| Оператор | Назначение |
|---|---|
| = | Равно |
| `<>`, `!=` | Не равно |
| `>` | Больше |
| `<` | Меньше |
| `>=`, `<=` | Сравнение |
| BETWEEN | Диапазон |
| IN | Совпадение со списком |
| LIKE | Поиск по шаблону |
| IS NULL | Проверка NULL |
| AND, OR, NOT | Логические операции |

### IN

```sql
SELECT *
FROM users
WHERE city IN ('Minsk', 'Moscow');
```

### BETWEEN

```sql
SELECT *
FROM orders
WHERE amount BETWEEN 100 AND 200;
```

`BETWEEN` включает обе границы диапазона.

### LIKE

```sql
SELECT *
FROM users
WHERE name LIKE 'A%';
```

`%` — любое количество символов, включая ноль.

`_` — ровно один символ.

В PostgreSQL для регистронезависимого поиска также существует `ILIKE`.

### ORDER BY

```sql
SELECT *
FROM orders
ORDER BY amount DESC;
```

- `ASC` — по возрастанию.
- `DESC` — по убыванию.

Если не указать ORDER BY, порядок результата не гарантирован.

### LIMIT и OFFSET

```sql
SELECT *
FROM orders
ORDER BY id
LIMIT 10 OFFSET 20;
```

Пропускаем первые 20 строк и получаем следующие 10.

**Важно:** для стабильной пагинации нужен детерминированный порядок сортировки, желательно с уникальным дополнительным ключом.

Также при больших OFFSET запросы могут становиться медленнее. Для больших наборов часто используют keyset pagination.

---

## 5. NULL и трёхзначная логика SQL

`NULL` означает отсутствие известного значения. Это не то же самое, что ноль или пустая строка.

Неправильно:

```sql
SELECT *
FROM users
WHERE city = NULL;
```

Правильно:

```sql
SELECT *
FROM users
WHERE city IS NULL;
```

Чтобы найти заполненные значения:

```sql
SELECT *
FROM users
WHERE city IS NOT NULL;
```

### Почему = NULL не работает?

В SQL используется трёхзначная логика:

- TRUE.
- FALSE.
- UNKNOWN.

Сравнение с NULL через обычные операторы обычно возвращает UNKNOWN.

WHERE отбирает только строки, для которых условие TRUE.

### COALESCE

Возвращает первый аргумент, не равный NULL.

```sql
SELECT
    name,
    COALESCE(city, 'Unknown') AS city
FROM users;
```

### Типичная ловушка: NOT IN и NULL

```sql
SELECT *
FROM users
WHERE id NOT IN (
    SELECT user_id
    FROM orders
);
```

Если подзапрос вернёт NULL, результат может оказаться неожиданным: строки с подходящим обычным значением не будут выбраны, поскольку условие может стать UNKNOWN.

В подобных задачах часто безопаснее использовать `NOT EXISTS`:

```sql
SELECT *
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

---

## 6. JOIN — объединение таблиц

JOIN позволяет объединять строки из нескольких таблиц по заданному условию.

### INNER JOIN

Возвращает только совпадающие записи из обеих таблиц.

```sql
SELECT
    u.name,
    o.id AS order_id,
    o.amount
FROM users u
INNER JOIN orders o
    ON u.id = o.user_id;
```

Результат:

| name | order_id | amount |
|---|---|---|
| Anna | 101 | 100 |
| Anna | 102 | 200 |
| Bob | 103 | 300 |
| Kate | 104 | 150 |
| Kate | 105 | 250 |

John не попадёт в результат, потому что у него нет заказов.

### LEFT JOIN

Возвращает все строки левой таблицы и подходящие строки правой.

Если соответствия нет, значения правой таблицы будут NULL.

```sql
SELECT
    u.name,
    o.id AS order_id
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id;
```

Результат:

| name | order_id |
|---|---|
| Anna | 101 |
| Anna | 102 |
| Bob | 103 |
| Kate | 104 |
| Kate | 105 |
| John | NULL |

### RIGHT JOIN

Сохраняет все строки правой таблицы.

```sql
SELECT
    u.name,
    o.id
FROM users u
RIGHT JOIN orders o
    ON u.id = o.user_id;
```

Практически многие RIGHT JOIN можно переписать через LEFT JOIN, поменяв таблицы местами.

### FULL OUTER JOIN

Возвращает совпадающие строки, а также несовпадающие строки обеих таблиц с NULL на отсутствующей стороне.

```sql
SELECT
    u.name,
    o.id
FROM users u
FULL OUTER JOIN orders o
    ON u.id = o.user_id;
```

**PostgreSQL:** поддерживает FULL OUTER JOIN.

**MySQL 8.4:** не поддерживает его напрямую. Аналогичную логику можно получить через комбинацию LEFT JOIN, RIGHT JOIN и UNION ALL с корректной обработкой совпадений.

### CROSS JOIN

Возвращает декартово произведение.

```sql
SELECT *
FROM users
CROSS JOIN orders;
```

Если в users 4 строки, а в orders 5, результат содержит 20 строк.

Это бывает полезно для генерации комбинаций, но случайный CROSS JOIN может создать огромный промежуточный результат.

---

## 7. ON vs WHERE при LEFT JOIN

Одна из самых частых ловушек на интервью.

Есть запрос:

```sql
SELECT
    u.id,
    o.id
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
WHERE o.status = 'PAID';
```

Несмотря на LEFT JOIN, пользователи без оплаченных заказов будут исключены: для них `o.status` равен NULL, а условие WHERE не является TRUE.

Фактически в данном случае запрос оставляет только пользователей с подходящими оплаченными заказами.

Если необходимо сохранить всех пользователей:

```sql
SELECT
    u.id,
    o.id
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
   AND o.status = 'PAID';
```

Теперь все пользователи сохранятся, а справа будут только оплаченные заказы либо NULL.

**Вывод:** перенос условия между ON и WHERE при OUTER JOIN способен изменить результат запроса.

---

## 8. GROUP BY и агрегатные функции

Агрегация используется для вычислений по группам строк.

Основные функции:

| Функция | Назначение |
|---|---|
| COUNT | Количество |
| SUM | Сумма |
| AVG | Среднее |
| MIN | Минимум |
| MAX | Максимум |

### COUNT(*)

```sql
SELECT COUNT(*)
FROM orders;
```

Возвращает 5.

### COUNT(column)

```sql
SELECT COUNT(city)
FROM users;
```

Возвращает 3, поскольку NULL не учитывается.

### COUNT(DISTINCT)

```sql
SELECT COUNT(DISTINCT user_id)
FROM orders;
```

Возвращает 3 — количество различных пользователей с заказами.

### GROUP BY

Посчитать число заказов каждого пользователя:

```sql
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id;
```

Посчитать сумму оплаченных заказов:

```sql
SELECT
    user_id,
    SUM(amount) AS total_amount
FROM orders
WHERE status = 'PAID'
GROUP BY user_id;
```

### HAVING

HAVING позволяет фильтровать группы после агрегации.

Найти пользователей, у которых больше одного заказа:

```sql
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 1;
```

### WHERE vs HAVING

- `WHERE` фильтрует отдельные строки до группировки.
- `HAVING` фильтрует результат группировки.

Пример:

```sql
SELECT
    user_id,
    SUM(amount) AS total
FROM orders
WHERE status = 'PAID'
GROUP BY user_id
HAVING SUM(amount) > 100;
```

Сначала выбираем оплаченные заказы, затем группируем их по пользователю, затем оставляем группы с суммой больше 100.

---

## 9. Логический порядок выполнения SELECT

Запрос записывается в следующем порядке:

```sql
SELECT ...
FROM ...
JOIN ... ON ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY ...
LIMIT ...;
```

Но логический порядок обработки отличается:

1. FROM / JOIN / ON.
2. WHERE.
3. GROUP BY.
4. HAVING.
5. SELECT.
6. DISTINCT.
7. ORDER BY.
8. LIMIT / OFFSET.

Это упрощённая логическая модель. Физический план СУБД может переупорядочивать операции, если результат останется эквивалентным.

### Почему это важно?

Например, агрегатная функция обычно не может использоваться непосредственно в WHERE:

```sql
-- Некорректно
SELECT user_id
FROM orders
WHERE COUNT(*) > 2
GROUP BY user_id;
```

Нужно:

```sql
SELECT user_id
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 2;
```

---

## 10. Подзапросы (Subqueries)

Подзапрос — SQL-запрос, вложенный в другой запрос.

### Подзапрос с IN

Получить пользователей, у которых есть заказы:

```sql
SELECT *
FROM users
WHERE id IN (
    SELECT user_id
    FROM orders
);
```

### Подзапрос с EXISTS

```sql
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

`EXISTS` проверяет наличие хотя бы одной строки в результате подзапроса.

### IN vs EXISTS

`IN` проверяет принадлежность значения набору.

`EXISTS` проверяет существование хотя бы одной подходящей строки.

В современных СУБД оптимизатор часто может преобразовывать оба варианта в эффективные планы.

Поэтому неправильно утверждать, что EXISTS всегда быстрее IN.

### Коррелированный подзапрос

Подзапрос ссылается на данные внешнего запроса:

```sql
SELECT
    u.id,
    u.name
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
      AND o.amount > 200
);
```

---

## 11. CTE — Common Table Expressions

CTE — именованный временный набор результатов, определённый с помощью WITH и доступный в пределах одного SQL-оператора.

CTE помогает разбивать сложный запрос на логические части.

```sql
WITH paid_orders AS (
    SELECT
        user_id,
        SUM(amount) AS total
    FROM orders
    WHERE status = 'PAID'
    GROUP BY user_id
)
SELECT
    u.name,
    p.total
FROM users u
JOIN paid_orders p
    ON u.id = p.user_id;
```

### Зачем нужны CTE?

- Улучшение читаемости.
- Декомпозиция сложных запросов.
- Повторное использование промежуточных результатов.
- Рекурсивные запросы через WITH RECURSIVE.

**Важно:** CTE не гарантирует ускорение. Оптимизатор может встроить его в основной запрос или материализовать результат — зависит от СУБД, версии и самого запроса.

PostgreSQL и MySQL 8.4 поддерживают CTE.

---

## 12. Оконные функции (Window Functions)

Оконные функции выполняют вычисления по набору строк, связанных с текущей строкой, при этом не сворачивая весь набор в одну строку, как обычный GROUP BY.

Они используются для:

- Нумерации строк.
- Ранжирования.
- Поиска предыдущего или следующего значения.
- Накопительных сумм.
- Сравнения строки со средним значением группы.

### ROW_NUMBER

Пронумеровать заказы каждого пользователя:

```sql
SELECT
    id,
    user_id,
    amount,
    ROW_NUMBER() OVER (
        PARTITION BY user_id
        ORDER BY amount DESC, id
    ) AS row_num
FROM orders;
```

`PARTITION BY` разделяет строки на группы, а `ORDER BY` задаёт порядок внутри них.

### RANK и DENSE_RANK

При одинаковых значениях:

| amount | ROW_NUMBER | RANK | DENSE_RANK |
|---|---|---|---|
| 300 | 1 | 1 | 1 |
| 200 | 2 | 2 | 2 |
| 200 | 3 | 2 | 2 |
| 100 | 4 | 4 | 3 |

- ROW_NUMBER — уникальный порядковый номер.
- RANK — одинаковые ранги при совпадении, с пропусками.
- DENSE_RANK — одинаковые ранги, без пропусков.

Для ROW_NUMBER при одинаковых значениях сортировки нужен дополнительный уникальный ключ, если важен воспроизводимый порядок.

### Последний заказ каждого пользователя

Добавим к таблице orders колонку `created_at`.

```sql
WITH ranked_orders AS (
    SELECT
        id,
        user_id,
        created_at,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY created_at DESC, id DESC
        ) AS rn
    FROM orders
)
SELECT *
FROM ranked_orders
WHERE rn = 1;
```

### Накопительная сумма

```sql
SELECT
    id,
    user_id,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY id
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_total
FROM orders;
```

Оконные функции доступны в современных PostgreSQL и MySQL 8.4.

---

## 13. UNION и UNION ALL

Операторы используются для объединения результатов нескольких SELECT.

### UNION

```sql
SELECT user_id
FROM orders
WHERE status = 'PAID'

UNION

SELECT user_id
FROM orders
WHERE status = 'NEW';
```

UNION удаляет дубликаты итогового результата.

### UNION ALL

```sql
SELECT user_id
FROM orders
WHERE status = 'PAID'

UNION ALL

SELECT user_id
FROM orders
WHERE status = 'NEW';
```

UNION ALL сохраняет дубликаты.

UNION ALL часто дешевле, поскольку не требует устранения дубликатов.

В обоих случаях объединяемые запросы должны возвращать совместимое количество колонок и типы значений.

---

## 14. INSERT, UPDATE, DELETE

### INSERT

```sql
INSERT INTO users (id, name, city)
VALUES (5, 'Alex', 'Minsk');
```

Вставка нескольких записей:

```sql
INSERT INTO users (id, name, city)
VALUES
    (6, 'Maria', 'Moscow'),
    (7, 'Ivan', 'Minsk');
```

### UPDATE

```sql
UPDATE users
SET city = 'Warsaw'
WHERE id = 5;
```

**Опасность:** UPDATE без WHERE может изменить все строки таблицы.

```sql
UPDATE users
SET city = 'Warsaw';
```

### DELETE

```sql
DELETE FROM users
WHERE id = 5;
```

DELETE без WHERE может удалить все строки:

```sql
DELETE FROM users;
```

### Как безопаснее менять данные?

1. Проверить условие через SELECT.
2. Убедиться в количестве подходящих строк.
3. По возможности использовать транзакцию.
4. Выполнить UPDATE или DELETE.
5. Проверить количество изменённых строк и результат.
6. Сделать COMMIT или ROLLBACK.

Пример:

```sql
BEGIN;

SELECT *
FROM orders
WHERE id = 101;

UPDATE orders
SET status = 'CANCELLED'
WHERE id = 101;

SELECT *
FROM orders
WHERE id = 101;

ROLLBACK;
```

Так можно проверить изменение без сохранения результата.

---

## 15. DELETE vs TRUNCATE vs DROP

| Команда | Что делает |
|---|---|
| DELETE | Удаляет строки |
| TRUNCATE | Быстро очищает таблицу |
| DROP | Удаляет сам объект таблицы |

### DELETE

Поддерживает WHERE:

```sql
DELETE FROM orders
WHERE status = 'CANCELLED';
```

### TRUNCATE

```sql
TRUNCATE TABLE orders;
```

Удаляет все строки без построчного условия WHERE.

Обычно эффективен для быстрой очистки таблицы, но может иметь особенности относительно блокировок, внешних ключей и счётчиков идентификаторов.

### DROP

```sql
DROP TABLE orders;
```

Удаляет таблицу как объект схемы.

### Важный нюанс PostgreSQL vs MySQL

В PostgreSQL TRUNCATE может быть отменён через ROLLBACK в рамках транзакции.

В MySQL TRUNCATE TABLE является DDL-операцией с неявным commit и не откатывается обычным ROLLBACK.

Это хороший пример того, почему нельзя безоговорочно переносить поведение SQL между СУБД.

---

## 16. Практические SQL-задачи для собеседования

Используем таблицы users, orders и payments из раздела 3.

### Задача 1. Найти пользователей без заказов

```sql
SELECT
    u.id,
    u.name
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
WHERE o.id IS NULL;
```

Результат — John.

Альтернатива:

```sql
SELECT
    u.id,
    u.name
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

### Задача 2. Найти пользователей с двумя и более заказами

```sql
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 2;
```

### Задача 3. Посчитать общую сумму оплаченных заказов

```sql
SELECT SUM(amount) AS total_paid
FROM orders
WHERE status = 'PAID';
```

Результат — 550.

### Задача 4. Найти заказы без платежей

```sql
SELECT
    o.id,
    o.amount
FROM orders o
LEFT JOIN payments p
    ON o.id = p.order_id
WHERE p.id IS NULL;
```

### Задача 5. Найти пользователей, у которых общая сумма оплаченных заказов больше 200

```sql
SELECT
    u.id,
    u.name,
    SUM(o.amount) AS total
FROM users u
JOIN orders o
    ON o.user_id = u.id
WHERE o.status = 'PAID'
GROUP BY u.id, u.name
HAVING SUM(o.amount) > 200;
```

Результат — Bob.

### Задача 6. Найти максимальную сумму заказа каждого пользователя

```sql
SELECT
    user_id,
    MAX(amount) AS max_amount
FROM orders
GROUP BY user_id;
```

Если нужно получить не только сумму, но и идентификатор соответствующего заказа, лучше использовать оконную функцию или другой запрос, выбирающий саму строку.

### Задача 7. Найти дубликаты email

Предположим, в users есть колонка email.

```sql
SELECT
    email,
    COUNT(*) AS duplicates_count
FROM users
WHERE email IS NOT NULL
GROUP BY email
HAVING COUNT(*) > 1;
```

### Задача 8. Найти заказы, сумма платежей по которым не совпадает с суммой заказа

```sql
SELECT
    o.id,
    o.amount AS order_amount,
    COALESCE(SUM(p.amount), 0) AS paid_amount
FROM orders o
LEFT JOIN payments p
    ON p.order_id = o.id
GROUP BY o.id, o.amount
HAVING o.amount <> COALESCE(SUM(p.amount), 0);
```

Этот запрос полезен при тестировании интеграций с платёжными системами.

Но для реальных финансовых сценариев нужно учитывать статусы платежей, возвраты, частичные оплаты и бизнес-правила.

---

## 17. Типичные ошибки на собеседовании

**Ошибка:** «INNER JOIN и LEFT JOIN отличаются только производительностью».

Правильно: они различаются семантикой результата. LEFT JOIN сохраняет несовпавшие строки левой таблицы.

**Ошибка:** «COUNT(*) и COUNT(column) всегда возвращают одинаковое значение».

Правильно: COUNT(column) не считает NULL.

**Ошибка:** «WHERE и HAVING взаимозаменяемы».

Правильно: WHERE фильтрует строки до группировки, HAVING — группы после неё.

**Ошибка:** «UNION и UNION ALL одинаковые».

Правильно: UNION устраняет дубликаты, UNION ALL сохраняет их.

**Ошибка:** «Если запрос содержит ORDER BY, строки с одинаковым значением всегда идут в одном порядке».

Правильно: для детерминированного порядка нужно дополнительно сортировать по уникальному ключу.

**Ошибка:** «JOIN не изменяет количество строк относительно основной таблицы».

Правильно: связи один ко многим и многие ко многим могут существенно увеличивать количество строк.

**Ошибка:** «NULL можно сравнивать через =».

Правильно: обычно используются IS NULL, IS NOT NULL и соответствующие NULL-safe операции конкретной СУБД.

**Ошибка:** «CTE всегда ускоряет запрос».

Правильно: CTE в первую очередь способ структурирования запроса. Его влияние на производительность зависит от реализации оптимизатора.

---

## 18. Что нужно знать без подсказки

- [ ] Основные группы SQL-команд.
- [ ] SELECT, WHERE, ORDER BY, LIMIT, OFFSET.
- [ ] Как SQL обрабатывает NULL.
- [ ] Отличия INNER, LEFT, RIGHT, FULL и CROSS JOIN.
- [ ] Почему ON и WHERE отличаются при LEFT JOIN.
- [ ] Агрегатные функции COUNT, SUM, AVG, MIN, MAX.
- [ ] Отличие WHERE от HAVING.
- [ ] Логический порядок обработки SELECT.
- [ ] Подзапросы, IN, EXISTS, NOT EXISTS.
- [ ] Что такое CTE.
- [ ] ROW_NUMBER, RANK, DENSE_RANK.
- [ ] UNION vs UNION ALL.
- [ ] Безопасное выполнение UPDATE и DELETE.
- [ ] Отличия DELETE, TRUNCATE, DROP.
- [ ] Как найти дубликаты.
- [ ] Как найти записи без связанной сущности.
- [ ] Как проверить соответствие сумм заказов и платежей.
- [ ] Почему JOIN может дублировать данные.

---

## Источники

- [PostgreSQL — The SQL Language](https://www.postgresql.org/docs/current/tutorial-sql.html)
- [PostgreSQL — Queries](https://www.postgresql.org/docs/current/queries.html)
- [PostgreSQL — Window Functions](https://www.postgresql.org/docs/current/tutorial-window.html)
- [MySQL 8.4 — SELECT Statement](https://dev.mysql.com/doc/refman/8.4/en/select.html)
- [MySQL 8.4 — Common Table Expressions](https://dev.mysql.com/doc/refman/8.4/en/with.html)
- [MySQL 8.4 — Window Functions](https://dev.mysql.com/doc/refman/8.4/en/window-functions.html)