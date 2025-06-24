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

2. Сумма выручки по каждому представлению
```
SELECT 
    p.title AS Представление,
    SUM(t.price) AS Выручка
FROM Tickets t
JOIN Performances p ON t.performance_id = p.performance_id
GROUP BY p.performance_id;
```

---

3. Все животные и их тренеры
```
SELECT 
    a.name AS Животное,
    a.species AS Вид,
    ar.full_name AS Тренер
FROM Animals a
LEFT JOIN Artists ar ON a.trainer_id = ar.artist_id;
```

---

4. Посетители и количество купленных билетов
```
SELECT 
    v.full_name AS Зритель,
    COUNT(t.ticket_id) AS Кол_билетов
FROM Visitors v
JOIN Tickets t ON v.visitor_id = t.visitor_id
GROUP BY v.visitor_id;
```
