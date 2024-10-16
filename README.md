# Домашнее задание к занятию "SQL. Часть 2" - `Александра Бужор`

---

### Задание 1

Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию:
- фамилия и имя сотрудника из этого магазина;
- город нахождения магазина;
- количество пользователей, закреплённых в этом магазине.

### Решение:

```sql
SELECT 
    CONCAT(s.first_name, ' ', s.last_name) AS staff_name,
    c.city,
    COUNT(cu.customer_id) AS customer_count
FROM customer cu
JOIN store st ON cu.store_id = st.store_id
JOIN staff s ON st.manager_staff_id = s.staff_id
JOIN address a ON st.address_id = a.address_id
JOIN city c ON a.city_id = c.city_id
GROUP BY st.store_id
HAVING COUNT(cu.customer_id) > 300;
```
```
staff_name  |city      |customer_count|
------------+----------+--------------+
Mike Hillyer|Lethbridge|           326|
```

---

### Задание 2

Получите количество фильмов, продолжительность которых больше средней продолжительности всех фильмов.

### Решение:
```sql
SELECT COUNT(*) AS long_duration_movies
FROM film
WHERE length > (SELECT AVG(length) FROM film);
```
```
long_duration_movies|
--------------------+
                 489|
```
---

### Задание 3

Получите информацию, за какой месяц была получена наибольшая сумма платежей, и добавьте информацию по количеству аренд за этот месяц.

### Решение:

```sql
WITH MonthlyPayments AS (
  SELECT 
    EXTRACT(YEAR FROM payment_date) AS year,
    EXTRACT(MONTH FROM payment_date) AS month,
    SUM(amount) AS total_payments
  FROM payment
  GROUP BY year, month
  ORDER BY total_payments DESC
  LIMIT 1
)
SELECT 
  mp.year, 
  mp.month, 
  mp.total_payments,
  (SELECT COUNT(*) 
   FROM rental 
   WHERE EXTRACT(YEAR FROM rental_date) = mp.year 
     AND EXTRACT(MONTH FROM rental_date) = mp.month) AS rental_count
FROM MonthlyPayments mp;
```
```
year|month|total_payments|rental_count|
----+-----+--------------+------------+
2005|    7|      28368.91|        6709|
```

---

### Задание 4*

Посчитайте количество продаж, выполненных каждым продавцом. Добавьте вычисляемую колонку «Премия». Если количество продаж превышает 8000, то значение в колонке будет «Да», иначе должно быть значение «Нет».

### Решение:

```sql
SELECT 
    staff_id, 
    COUNT(payment_id) AS total_sales, 
    CASE 
        WHEN COUNT(payment_id) > 8000 THEN 'Yes'
        ELSE 'No' 
    END AS Bonus
FROM payment
GROUP BY staff_id;
```
```
staff_id|total_sales|Bonus|
--------+-----------+-----+
       1|       8054|Yes  |
       2|       7990|No   |
```

---

### Задание 5*

Найдите фильмы, которые ни разу не брали в аренду.

### Решение:

```sql
SELECT f.film_id, f.title
FROM film f
LEFT JOIN inventory i ON f.film_id = i.film_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
WHERE r.rental_id IS NULL
GROUP BY f.film_id;
```
```
film_id|title                 |
-------+----------------------+
      1|ACADEMY DINOSAUR      |
     14|ALICE FANTASIA        |
     33|APOLLO TEEN           |
     36|ARGONAUTS TOWN        |
     38|ARK RIDGEMONT         |
     41|ARSENIC INDEPENDENCE  |
     87|BOONDOCK BALLROOM     |
    108|BUTCH PANTHER         |
    128|CATCH AMISTAD         |
    144|CHINATOWN GLADIATOR   |
    148|CHOCOLATE DUCK        |
    171|COMMANDMENTS EXPRESS  |
    192|CROSSING DIVORCE      |
    195|CROWDS TELEMARK       |
    198|CRYSTAL BREAKING      |
    217|DAZED PUNK            |
    221|DELIVERANCE MULHOLLAND|
    318|FIREHOUSE VIETNAM     |
    325|FLOATS GARDEN         |
    332|FRANKENSTEIN STRANGER |
    359|GLADIATOR WESTWARD    |
    386|GUMP DATE             |
    404|HATE HANDICAP         |
    419|HOCUS FRIDA           |
    495|KENTUCKIAN GIANT      |
    497|KILL BROTHERHOOD      |
    607|MUPPET MILE           |
    642|ORDER BETRAYED        |
    669|PEARL DESTINY         |
    671|PERDITION FARGO       |
    701|PSYCHO SHRUNK         |
    712|RAIDERS ANTITRUST     |
    713|RAINBOW SHOCK         |
    742|ROOF CHAMPION         |
    801|SISTER FREDDY         |
    802|SKY MIRACLE           |
    860|SUICIDES SILENCE      |
    874|TADPOLE PARK          |
    909|TREASURE COMMAND      |
    943|VILLAIN DESPERATE     |
    950|VOLUME HOUSE          |
    954|WAKE JAWS             |
    955|WALLS ARTIST          |

 ```
