#  Задача 1: Найти числа, встречающиеся подряд минимум три раза

##  Входные данные
```sql
CREATE TABLE Logs (
    id INT PRIMARY KEY,
    num VARCHAR(10)
);

INSERT INTO Logs (id, num) VALUES
(1, '1'),
(2, '1'),
(3, '1'),
(4, '2'),
(5, '1'),
(6, '2'),
(7, '2');
```

##  Задание
Найдите все числа, которые встречаются **как минимум три раза подряд** (id подряд). Без оконных функций.

##  Ожидаемый вывод
```
+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
```

##  Решения
###  Вариант 1 (JOIN)
```sql
SELECT DISTINCT l1.num AS ConsecutiveNums
FROM Logs l1
JOIN Logs l2 ON l2.id = l1.id + 1 AND l2.num = l1.num
JOIN Logs l3 ON l3.id = l1.id + 2 AND l3.num = l1.num;
```

###  Вариант 2 (подзапросы)
```sql
SELECT DISTINCT num AS ConsecutiveNums
FROM Logs l
WHERE num = (SELECT num FROM Logs WHERE id = l.id + 1)
  AND num = (SELECT num FROM Logs WHERE id = l.id + 2);
```

---

#  Задача 2: Найти последовательности в Stadium (≥3 подряд id, people ≥ 100)

##  Входные данные
```sql
CREATE TABLE Stadium (
    id INT PRIMARY KEY,
    visit_date DATE,
    people INT
);

INSERT INTO Stadium (id, visit_date, people) VALUES
(1, '2017-01-01', 10),
(2, '2017-01-02', 109),
(3, '2017-01-03', 150),
(4, '2017-01-04', 99),
(5, '2017-01-05', 145),
(6, '2017-01-06', 1455),
(7, '2017-01-07', 199),
(8, '2017-01-09', 188);
```

##  Задание
Найти **все строки**, которые являются частью последовательности из **трёх или более подряд идущих id**, где people ≥ 100.

##  Ожидаемый вывод
```
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-09 | 188       |
+------+------------+-----------+
```

##  Решения
###  Вариант 1 (JOIN)
```sql
SELECT s1.*
FROM Stadium s1
JOIN Stadium s2 ON s2.id = s1.id + 1 AND s2.people >= 100
JOIN Stadium s3 ON s3.id = s1.id + 2 AND s3.people >= 100
WHERE s1.people >= 100
ORDER BY s1.visit_date;
```

###  Вариант 2 (подзапросы)
```sql
SELECT *
FROM Stadium s
WHERE s.people >= 100
  AND EXISTS (SELECT 1 FROM Stadium WHERE id = s.id + 1 AND people >= 100)
  AND EXISTS (SELECT 1 FROM Stadium WHERE id = s.id + 2 AND people >= 100)
ORDER BY visit_date;
```

---

#  Задача 3: Определить уровень сотрудника (без рекурсии)

##  Входные данные
```sql
CREATE TABLE Employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    manager_id INT,
    salary INT,
    department VARCHAR(100)
);

INSERT INTO Employees VALUES
(1, 'Alice', NULL, 12000, 'Executive'),
(2, 'Bob', 1, 10000, 'Sales'),
(3, 'Charlie', 1, 10000, 'Engineering'),
(4, 'David', 2, 7500, 'Sales'),
(5, 'Eva', 2, 7500, 'Sales'),
(6, 'Frank', 3, 9000, 'Engineering'),
(7, 'Grace', 3, 8500, 'Engineering'),
(8, 'Hank', 4, 6000, 'Sales'),
(9, 'Ivy', 6, 7000, 'Engineering'),
(10, 'Judy', 6, 7000, 'Engineering');
```

##  Задание
Определить **уровень сотрудника**:
- CEO = 1
- его подчинённые = 2
- подчинённые уровня 2 = 3
- и так далее

##  Решения
###  Вариант 1 (JOIN — без рекурсии)
```sql
SELECT 
    e.employee_id,
    e.employee_name,
    CASE
        WHEN m1.employee_id IS NULL THEN 1
        WHEN m2.employee_id IS NULL THEN 2
        WHEN m3.employee_id IS NULL THEN 3
        WHEN m4.employee_id IS NULL THEN 4
    END AS level
FROM Employees e
LEFT JOIN Employees m1 ON e.manager_id = m1.employee_id
LEFT JOIN Employees m2 ON m1.manager_id = m2.employee_id
LEFT JOIN Employees m3 ON m2.manager_id = m3.employee_id
LEFT JOIN Employees m4 ON m3.manager_id = m4.employee_id
ORDER BY level, employee_name;
```

###  Вариант 2 (подзапрос — считает глубину до NULL)
```sql
SELECT 
    e.employee_id,
    e.employee_name,
    1
      + (SELECT COUNT(*) FROM Employees x WHERE x.employee_id = e.manager_id)
      + (SELECT COUNT(*) FROM Employees x JOIN Employees y ON x.employee_id = y.manager_id WHERE y.employee_id = e.manager_id)
AS level
FROM Employees e;
```
(Работает только при ограниченной глубине, как в примере.)


---

#  Задача 4: Найти некорректные IP-адреса

##  Входные данные
```sql
CREATE TABLE logs (
    log_id INT PRIMARY KEY,
    ip VARCHAR(50),
    status_code INT
);

INSERT INTO logs VALUES
(1, '192.168.1.1', 200),
(2, '256.1.2.3', 404),
(3, '192.168.001.1', 200),
(4, '192.168.1.1', 200),
(5, '192.168.1', 500),
(6, '256.1.2.3', 404),
(7, '192.168.001.1', 200);
```

##  Задание
Найти все **некорректные IPv4-адреса**, где ошибка определяется по правилам:
1. Любой октет > 255
2. Ведущие нули в октете (например, 01, 002)
3. Количество октетов ≠ 4

Вернуть таблицу вида:
```
ip | invalid_count
```
отсортированную по **invalid_count DESC**, затем **ip DESC**.

##  Решения
###  Вариант 1 (CTE + SUBSTRING / LEFT / RIGHT)
```sql
WITH parts AS (
    SELECT
        ip,
        (LEN(ip) - LEN(REPLACE(ip, '.', ''))) AS dot_count,
        -- Первый октет
        LEFT(ip, CHARINDEX('.', ip) - 1) AS o1,

        -- Второй октет
        SUBSTRING(
            ip,
            CHARINDEX('.', ip) + 1,
            CHARINDEX('.', ip, CHARINDEX('.', ip) + 1) - CHARINDEX('.', ip) - 1
        ) AS o2,

        -- Третий октет
        SUBSTRING(
            ip,
            CHARINDEX('.', ip, CHARINDEX('.', ip) + 1) + 1,
            CHARINDEX('.', ip, CHARINDEX('.', ip, CHARINDEX('.', ip) + 1) + 1)
                - CHARINDEX('.', ip, CHARINDEX('.', ip) + 1) - 1
        ) AS o3,

        -- Четвёртый октет
        RIGHT(ip,
            LEN(ip) - CHARINDEX('.', ip, CHARINDEX('.', ip, CHARINDEX('.', ip) + 1) + 1)
        ) AS o4
    FROM logs
)
,
invalid AS (
    SELECT ip,
        CASE
            WHEN dot_count <> 3 THEN 1

            WHEN CAST(o1 AS INT) > 255 OR CAST(o2 AS INT) > 255 OR CAST(o3 AS INT) > 255 OR CAST(o4 AS INT) > 255 THEN 1

            WHEN (o1 LIKE '0%' AND o1 <> '0')
              OR (o2 LIKE '0%' AND o2 <> '0')
              OR (o3 LIKE '0%' AND o3 <> '0')
              OR (o4 LIKE '0%' AND o4 <> '0') THEN 1

            ELSE 0
        END AS is_invalid
    FROM parts
)
SELECT ip, COUNT(*) AS invalid_count
FROM invalid
WHERE is_invalid = 1
GROUP BY ip
ORDER BY invalid_count DESC, ip DESC;
```

###  Вариант 2 (JOIN-основанная версия без CTE)
```sql
SELECT ip, COUNT(*) AS invalid_count
FROM (
    SELECT l.ip,
        -- Количество октетов
        (LEN(l.ip) - LEN(REPLACE(l.ip, '.', ''))) AS dot_count,

        -- Разбор IP
        LEFT(l.ip, CHARINDEX('.', l.ip) - 1) AS o1,
        SUBSTRING(l.ip, CHARINDEX('.', l.ip) + 1,
            CHARINDEX('.', l.ip, CHARINDEX('.', l.ip) + 1) - CHARINDEX('.', l.ip) - 1) AS o2,
        SUBSTRING(l.ip, CHARINDEX('.', l.ip, CHARINDEX('.', l.ip) + 1) + 1,
            CHARINDEX('.', l.ip, CHARINDEX('.', l.ip, CHARINDEX('.', l.ip) + 1) + 1)
              - CHARINDEX('.', l.ip, CHARINDEX('.', l.ip) + 1) - 1
        ) AS o3,
        RIGHT(l.ip, LEN(l.ip) - CHARINDEX('.', l.ip, CHARINDEX('.', l.ip, CHARINDEX('.', l.ip) + 1) + 1)) AS o4
    FROM logs l
) x
WHERE 
      dot_count <> 3
   OR CAST(o1 AS INT) > 255 OR CAST(o2 AS INT) > 255 OR CAST(o3 AS INT) > 255 OR CAST(o4 AS INT) > 255
   OR (o1 LIKE '0%' AND o1 <> '0')
   OR (o2 LIKE '0%' AND o2 <> '0')
   OR (o3 LIKE '0%' AND o3 <> '0')
   OR (o4 LIKE '0%' AND o4 <> '0')
GROUP BY ip
ORDER BY invalid_count DESC, ip DESC;
```
