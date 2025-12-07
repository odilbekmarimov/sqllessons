# Что такое VIEW (представление) в SQL

**VIEW** — это виртуальная таблица, созданная на основе результата запроса. Представление не хранит данные физически (за исключением материализованных представлений), оно просто сохраняет SQL‑логику, которую можно многократно использовать.

## Ключевые особенности VIEW

1. **Виртуальная таблица** — данные берутся из базовых таблиц при выполнении запроса.
2. **Упрощение сложных запросов** — один раз пишем сложный SELECT и используем как обычную таблицу.
3. **Безопасность данных** — можно скрыть чувствительные столбцы, предоставив пользователю доступ только к VIEW.
4. **Логическая абстракция** — слой между пользователем и реальными таблицами. Структура таблиц может меняться, но VIEW сохраняет совместимость.
5. **Повторное использование** — используется как переиспользуемый объект для отчётов, аналитики, интеграций.

## Когда использовать VIEW

* Когда нужно регулярно выполнять один и тот же сложный запрос.
* Когда нужно скрыть часть данных от определённых пользователей.
* Когда структуру исходных таблиц нельзя показывать внешним сервисам.
* Когда требуется подготовить данные для отчётности или представления в удобном формате.

## Когда лучше не использовать VIEW

* Когда требуется высокая производительность при больших данных. VIEW всегда пересчитывается.
* Когда логика слишком тяжёлая. Лучше создать таблицу в ETL.
* Когда требуются индексы внутри VIEW (кроме материализованных вариантов).

## Пример исходных таблиц

Ниже приведены тестовые данные для всех таблиц, которые используются в вопросах. Дубликаты CREATE TABLE были удалены, оставлены только версии с максимально полными колонками. (Sample Data для тестирования VIEW)
Ниже приведены тестовые данные, чтобы студенты могли выполнить примеры локально.

### Таблица Sales

Расширенная версия с дополнительными столбцами:

* customer_id
* product_id
* payment_method

```sql
CREATE TABLE Sales (
    sale_id INT,
    sale_date DATE,
    amount DECIMAL(10,2),
    customer_id INT,
    product_id INT,
    payment_method VARCHAR(20)
);

INSERT INTO Sales VALUES
(1, '2024-01-01', 150.00, 101, 501, 'Cash'),
(2, '2024-01-01', 200.00, 102, 502, 'Card'),
(3, '2024-01-02', 300.00, 103, 503, 'Online'),
(4, '2024-01-03', 400.00, 101, 504, 'Cash'),
(5, '2024-01-04', 120.00, 104, 505, 'Card'),
(6, '2024-01-05', 250.00, 105, 506, 'Online'),
(7, '2024-01-06', 310.00, 101, 507, 'Cash');
```


### Таблица Employees

Добавлены столбцы:

* department
* hire_date
* position

```sql
CREATE TABLE Employees (
    id INT,
    employee_name VARCHAR(100),
    salary DECIMAL(10,2),
    department VARCHAR(50),
    hire_date DATE,
    position VARCHAR(50)
);

INSERT INTO Employees VALUES
(1, 'Ali Karimov', 3000, 'Sales', '2020-03-01', 'Manager'),
(2, 'Dilshod Umarov', 3500, 'Logistics', '2019-07-15', 'Coordinator'),
(3, 'Madina Saidova', 4200, 'HR', '2021-02-10', 'HR Specialist'),
(4, 'Nodira Xolmatova', 4600, 'Sales', '2018-11-21', 'Senior Manager'),
(5, 'Baxtiyor Sodiqov', 2800, 'Support', '2022-04-12', 'Specialist');
```


### Таблица Orders

Добавлены столбцы:

* customer_id
* total_items
* order_status

```sql
CREATE TABLE Orders (
    order_id INT,
    order_date DATE,
    employee_id INT,
    customer_id INT,
    total_items INT,
    order_status VARCHAR(20)
);

INSERT INTO Orders VALUES
(101, '2024-01-05', 1, 101, 3, 'Completed'),
(102, '2024-01-06', 2, 102, 1, 'Pending'),
(103, '2024-01-07', 1, 103, 5, 'Completed'),
(104, '2024-01-08', 4, 101, 2, 'Completed'),
(105, '2024-01-09', 3, 105, 4, 'Cancelled');
```


### Таблица Schedule

Добавлены столбцы:

* building
* class_type (lecture/lab/seminar)
* duration_minutes

```sql
CREATE TABLE Schedule (
    group_name VARCHAR(50),
    subject VARCHAR(100),
    teacher_name VARCHAR(100),
    room VARCHAR(10),
    time TIME,
    building VARCHAR(10),
    class_type VARCHAR(20),
    duration_minutes INT
);

INSERT INTO Schedule VALUES
('Group A', 'Matematika', 'X. Axmedov', 'A101', '09:00', 'Main', 'Lecture', 90),
('Group B', 'Fizika', 'M. Rasulova', 'B202', '10:30', 'Tech', 'Lab', 120),
('Group A', 'Ingliz tili', 'S. Jamshidov', 'A103', '12:00', 'Main', 'Seminar', 60),
('Group C', 'Biologiya', 'O. Sodiqova', 'C301', '08:30', 'Main', 'Lecture', 90),
('Group B', 'Kimyo', 'R. Qodirov', 'B204', '13:00', 'Tech', 'Seminar', 75);
```




# Решения для T-SQL (VIEW и практические задачи)

## 1. VIEW: Итоговые продажи по дням

```sql
CREATE VIEW sales_daily AS
SELECT sale_date, SUM(amount) AS total_sales
FROM Sales
GROUP BY sale_date;
```

## 2. VIEW: Информация о заказах без чувствительных данных

```sql
CREATE VIEW order_info AS
SELECT 
    o.order_id,
    o.order_date,
    e.employee_name,
    o.total_items,
    o.order_status
FROM Orders o
JOIN Employees e ON o.employee_id = e.id;
```

## 3. VIEW: Студенческое расписание (очищенный формат)

```sql
CREATE VIEW schedule_view AS
SELECT group_name, subject, teacher_name, room, time, class_type
FROM Schedule;
```

---

# Решения задач (DateTime, String, Math)

## DateTime решения

### Задача: стаж работы сотрудника

```sql
SELECT employee_name,
       DATEDIFF(YEAR, hire_date, GETDATE()) AS years_worked
FROM Employees;
```

### Заказы за последние 90 дней

```sql
SELECT order_date,
       DATENAME(WEEKDAY, order_date) AS weekday_name,
       DATEPART(WEEK, order_date) AS week_number
FROM Orders
WHERE order_date >= DATEADD(DAY, -90, GETDATE());
```

### Разница между первой и последней покупкой

```sql
SELECT customer_id,
       MIN(sale_date) AS first_buy,
       MAX(sale_date) AS last_buy,
       DATEDIFF(DAY, MIN(sale_date), MAX(sale_date)) AS diff_days
FROM Sales
GROUP BY customer_id
HAVING DATEDIFF(DAY, MIN(sale_date), MAX(sale_date)) > 180;
```

---

## String решения

### Разделение имени на фамилию и имя

```sql
SELECT full_name,
       LEFT(full_name, CHARINDEX(' ', full_name) - 1) AS last_name,
       SUBSTRING(full_name, CHARINDEX(' ', full_name) + 1, LEN(full_name)) AS first_name,
       CONCAT(LEFT(full_name, 1), '.', LEFT(SUBSTRING(full_name, CHARINDEX(' ', full_name) + 1, LEN(full_name)), 1), '.') AS initials
FROM Users;
```

### Домен из email

```sql
SELECT email,
       SUBSTRING(email, CHARINDEX('@', email) + 1, LEN(email)) AS domain
FROM Users;
```

### Удалить повторяющиеся пробелы

```sql
WITH cte AS (
    SELECT REPLACE(text_value, '  ', ' ') AS cleaned
    FROM Comments
)
SELECT cleaned
FROM cte;
```

(выполняется несколько раз, пока не исчезнут двойные пробелы)

---

## Math решения

### Итоговая цена после скидки

```sql
SELECT price,
       discount_percent,
       ROUND(price - price * discount_percent / 100.0, 2) AS final_price
FROM Products;
```

### Среднеквадратичное отклонение

```sql
SELECT customer_id,
       SQRT(AVG(POWER(amount - AVG(amount) OVER(PARTITION BY customer_id), 2))) AS stddev
FROM Sales;
```

### Рейтинг продавца

```sql
SELECT seller_id,
       5 - LOG(total_returns + 1) AS rating
FROM Sellers
WHERE 5 - LOG(total_returns + 1) < 3;
```

---

## Примеры реальных ситуаций

### 1) Отчёт для бухгалтерии

Бухгалтерии нужно видеть только итоговые суммы продаж по дням, но не видеть фамилий клиентов.
Создаём VIEW без персональных данных.

```sql
CREATE VIEW sales_daily AS
SELECT sale_date, SUM(amount) AS total_sales
FROM Sales
GROUP BY sale_date;
```

### 2) Онлайн‑магазин — скрытие колонок

Менеджеру заказов не нужно видеть зарплату сотрудников, но нужно знать кто обработал заказ.

```sql
CREATE VIEW order_info AS
SELECT o.order_id, o.order_date, e.employee_name
FROM Orders o
JOIN Employees e ON o.employee_id = e.id;
```

### 3) Университет — расписание занятий

Студенты должны видеть расписание, но без внутреннего кода преподавателя.

```sql
CREATE VIEW schedule_view AS
SELECT group_name, subject, teacher_name, room, time
FROM Schedule;
```

---

# ПРАКТИКА: Средние и сложные задачи по датам, строкам и математическим функциям

## 1. Задачи по DateTime (средний → продвинутый уровень)

### Задача 1

У сотрудника есть столбец `hire_date`. Нужно вывести стаж работы в годах, округлив вниз.

**Ожидаемый результат:** целое число лет.

### Задача 2

В таблице заказов `Orders` получить все заказы, сделанные в последние 90 дней, но вывести только:

* дату,
* день недели текстом,
* номер недели в году.

### Задача 3 (продвинутая)

Найти клиентов, у которых разница между первой и последней покупкой больше 180 дней.

Подсказка: используйте `MIN(order_date)` и `MAX(order_date)`.

---

# 2. Задачи по строкам (string functions)

### Задача 1

В таблице `Users(full_name)` вывести:

* фамилию (текст до пробела),
* имя (текст после пробела),
* инициалы вида: "Ф.И.".

Используйте `LEFT`, `RIGHT`, `CHARINDEX`, `SUBSTRING`.

### Задача 2

Есть колонка `email`. Требуется вывести только домен (часть после "@").

### Задача 3 (продвинутая)

В таблице комментариев убрать повторяющиеся пробелы в тексте, например:

```
"Это    пример   текста" → "Это пример текста"
```

Подсказка: используйте рекурсивный CTE или многоразовое `REPLACE`.

---

# 3. Задачи по математическим функциям (math functions)

### Задача 1

В таблице товаров есть `price` и `discount_percent`. Рассчитать:

```
final_price = price - price * discount_percent / 100
```

С округлением до двух знаков.


---

# Дополнительные таблицы для всех задач

## Таблица Users

```sql
CREATE TABLE Users (
    user_id INT,
    full_name VARCHAR(100),
    email VARCHAR(150)
);

INSERT INTO Users VALUES
(101, 'Karimov Alisher', 'alisher.karimov@mail.uz'),
(102, 'Umarova Dilnoza', 'dilnoza.umarova@gmail.com'),
(103, 'Rustamov Bekzod', 'bekzod_rustamov@yahoo.com'),
(104, 'Saidov Jamshid', 'jsaidov@mail.uz');
```

## Таблица Products

```sql
CREATE TABLE Products (
    product_id INT,
    product_name VARCHAR(100),
    price DECIMAL(10,2),
    discount_percent INT
);

INSERT INTO Products VALUES
(501, 'Notebook', 4500.00, 10),
(502, 'Mouse', 120.00, 5),
(503, 'Keyboard', 250.00, 8),
(504, 'Monitor', 1500.00, 12);
```

## Таблица Comments

```sql
CREATE TABLE Comments (
    comment_id INT,
    user_id INT,
    text_value VARCHAR(500)
);

INSERT INTO Comments VALUES
(1, 101, 'Bu   matn   misol   uchun'),
(2, 102, 'Ikkinchi    matn     ham    noto''g''ri'),
(3, 103, 'Oddiy     tozalash   talabi');
```

## Таблица Sellers

```sql
CREATE TABLE Sellers (
    seller_id INT,
    seller_name VARCHAR(100),
    total_returns INT
);

INSERT INTO Sellers VALUES
(1, 'TechMarket', 3),
(2, 'BestElectro', 10),
(3, 'SuperShop', 0),
(4, 'CityLine', 7);
```

---
