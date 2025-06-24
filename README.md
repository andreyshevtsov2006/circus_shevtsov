# Курсовая работа на тему: Разработка базы данных для цирка

![image](https://github.com/user-attachments/assets/3b55e39a-7933-4577-bed7-5adf4603d22f)

## SQL-запросы

```
1. Список представлений и количества участвующих артистов
SELECT 
    p.title AS Представление,
    p.date AS Дата,
    COUNT(ap.artist_id) AS Кол-во_артистов
FROM Performances p
LEFT JOIN Artist_Performance ap ON p.performance_id = ap.performance_id
GROUP BY p.performance_id;
```

---

2. **Сумма выручки по каждому представлению**
```
SELECT 
    p.title AS Представление,
    SUM(t.price) AS Выручка
FROM Tickets t
JOIN Performances p ON t.performance_id = p.performance_id
GROUP BY p.performance_id;
```

---

3. **Все животные и их тренеры**
```
SELECT 
    a.name AS Животное,
    a.species AS Вид,
    ar.full_name AS Тренер
FROM Animals a
LEFT JOIN Artists ar ON a.trainer_id = ar.artist_id;
```

---

4. **Посетители и количество купленных билетов**
```
SELECT 
    v.full_name AS Зритель,
    COUNT(t.ticket_id) AS Кол_билетов
FROM Visitors v
JOIN Tickets t ON v.visitor_id = t.visitor_id
GROUP BY v.visitor_id;
```

## Процедуры

1. **Добавление нового артиста**
```
CREATE PROCEDURE AddArtist(
    IN fullName VARCHAR(100),
    IN genre VARCHAR(50),
    IN experience INT,
    IN phone VARCHAR(20)
)
BEGIN
    INSERT INTO Artists (full_name, genre, experience_years, phone)
    VALUES (fullName, genre, experience, phone);
END
```
**Описание:** процедура добавляет нового артиста в базу данных.
Это удобно, если регистрацией занимается администратор цирка — достаточно ввести имя, жанр (акробат, дрессировщик и т.д.), опыт и телефон.

2. **Добавление билета**
```
CREATE PROCEDURE AddTicket (
    IN p_performance_id INT,
    IN p_visitor_id INT,
    IN p_seat_number VARCHAR(10),
    IN p_price DECIMAL(10,2)
)
BEGIN
    DECLARE seat_taken INT;

    SELECT COUNT(*) INTO seat_taken
    FROM Tickets
    WHERE performance_id = p_performance_id AND seat_number = p_seat_number;

    IF seat_taken > 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Место уже занято';
    ELSE
        INSERT INTO Tickets (performance_id, visitor_id, seat_number, price, purchase_date)
        VALUES (p_performance_id, p_visitor_id, p_seat_number, p_price, NOW());
    END IF;
END
```
**Описание: Эта процедура добавляет новый билет в базу данных.

---

## Триггер
```
CREATE TRIGGER check_ticket_price 
BEFORE INSERT ON Tickets
FOR EACH ROW
BEGIN
    IF NEW.price < 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Цена билета не может быть отрицательной';
    END IF;
END
```
**Описание: этот триггер срабатывает до вставки новой записи в таблицу `Tickets`. Он проверяет, что значение поля `price` (цена билета) не отрицательное. Если цена отрицательна, триггер отменяет вставку и возвращает ошибку с сообщением:
«Цена билета не может быть отрицательной».

---

## Views
1. **Список представлений с артистами и ролями**
```
CREATE VIEW vw_performance_artists AS
SELECT
    p.performance_id,
    p.title,
    a.full_name AS artist_name,
    ap.role_description
FROM Performances p
JOIN Artist_Performance ap ON p.performance_id = ap.performance_id
JOIN Artists a ON ap.artist_id = a.artist_id;
```
**Oписание:** Тут данные берутся из трёх таблиц: `Performances`, `Artist_Performance`, `Artists`. И с помощью этого можно удобно получить список всех артистов с их ролями для каждой постановки.

2. **Сводка по билетам: представление, кол-во проданных билетов и общая выручка**
```
CREATE VIEW vw_ticket_summary AS
SELECT
    p.performance_id,
    p.title,
    COUNT(t.ticket_id) AS tickets_sold,
    SUM(t.price) AS total_revenue
FROM Performances p
LEFT JOIN Tickets t ON p.performance_id = t.performance_id
GROUP BY p.performance_id, p.title;
```
**Oписание:** Тут данные берутся из таблиц `Performances` и `Tickets` с использованием левого соединения (LEFT JOIN), чтобы учитывать постановки даже без проданных билетов, удобно для анализа популярности постановок и дохода.

