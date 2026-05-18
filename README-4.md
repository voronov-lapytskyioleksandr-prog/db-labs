**Лабораторна робота 4: Аналітичні SQL-запити (OLAP)**

*1. Опис виконаних запитів*
У цій лабораторній роботі було написано 10 аналітичних SQL-запитів до бази даних ресторану для підсумовування тенденцій та створення звітів. Усі запити розділені на три категорії відповідно до вимог.

**Категорія 1: Агрегація та групування (4 запити)**

* **Запит 1.1 (COUNT):** Підраховує загальну кількість страв у меню ресторану. Допомагає зрозуміти обсяг меню.
* **Запит 1.2 (AVG):** Обчислює середню ціну страви. Дозволяє оцінити середній ціновий сегмент закладу.
* **Запит 1.3 (MIN, MAX, GROUP BY):** Знаходить мінімальну та максимальну ціну страви у кожній категорії. Допомагає проаналізувати розкид цін всередині розділів меню.
* **Запит 1.4 (SUM, GROUP BY, HAVING):** Підраховує загальну кількість замовлених порцій для кожної страви та фільтрує лише ті страви, яких замовили більше 1 порції. Визначає найпопулярніші позиції.

**Категорія 2: Об'єднання таблиць - JOIN (3 запити)**

* **Запит 2.1 (INNER JOIN):** Об'єднує таблиці страв та категорій, щоб вивести назву страви разом із текстовою назвою її категорії (а не просто ID).
* **Запит 2.2 (LEFT JOIN):** Виводить список усіх категорій та страв у них. Навіть якщо категорія порожня, вона все одно з'явиться у звіті.
* **Запит 2.3 (RIGHT JOIN):** Об'єднує деталі замовлень та таблицю страв, щоб показати, які страви замовляли, а навпаки – які страви є в меню, але їх ще жодного разу не замовили (для них кількість буде null).

**Категорія 3: Підзапити (3 запити)**

* **Запит 3.1 (Підзапит у WHERE):** Знаходить усі страви, ціна яких є вищою за середню ціну всіх страв у ресторані.
* **Запит 3.2 (Підзапит у SELECT):** Виводить назву страви, її ціну та обчислює різницю між ціною цієї страви та ціною найдорожчої страви в меню.
* **Запит 3.3 (Підзапит у WHERE для пошуку найдорожчого):** Знаходить номери замовлень, у яких клієнти замовили найдорожчу страву з меню.

*2. SQL-скрипт аналітичних запитів*

```sql
-- ==========================================
-- КАТЕГОРІЯ 1: Агрегація та групування
-- ==========================================

-- 1.1 Загальна кількість страв (COUNT)
SELECT COUNT(*) AS total_dishes 
FROM dish;

-- 1.2 Середня ціна страви (AVG)
SELECT AVG(price) AS average_price 
FROM dish;

-- 1.3 Мінімальна та максимальна ціна в кожній категорії (MIN, MAX, GROUP BY)
SELECT category_id, MIN(price) AS min_price, MAX(price) AS max_price 
FROM dish 
GROUP BY category_id;

-- 1.4 Загальна кількість проданих порцій кожної страви, де продано більше 1 (SUM, GROUP BY, HAVING)
SELECT dish_id, SUM(quantity) AS total_sold 
FROM order_item 
GROUP BY dish_id 
HAVING SUM(quantity) > 1;


-- ==========================================
-- КАТЕГОРІЯ 2: Об'єднання таблиць (JOIN)
-- ==========================================

-- 2.1 Назви страв та їх категорії (INNER JOIN)
SELECT d.name AS dish_name, c.name AS category_name 
FROM dish d 
INNER JOIN category c ON d.category_id = c.category_id;

-- 2.2 Усі категорії та їх страви, включаючи порожні категорії (LEFT JOIN)
SELECT c.name AS category_name, d.name AS dish_name 
FROM category c 
LEFT JOIN dish d ON c.category_id = d.category_id;

-- 2.3 Усі страви з меню та інформація про їх замовлення, навіть якщо їх не замовляли (RIGHT JOIN)
SELECT d.name AS dish_name, oi.quantity 
FROM order_item oi 
RIGHT JOIN dish d ON oi.dish_id = d.dish_id;


-- ==========================================
-- КАТЕГОРІЯ 3: Підзапити
-- ==========================================

-- 3.1 Страви, дорожчі за середню ціну по всьому меню (Підзапит у WHERE)
SELECT name, price 
FROM dish 
WHERE price > (SELECT AVG(price) FROM dish);

-- 3.2 Різниця в ціні кожної страви порівняно з найдорожчою (Підзапит у SELECT)
SELECT name, price, 
       (SELECT MAX(price) FROM dish) - price AS difference_from_max 
FROM dish;

-- 3.3 Знайти ID замовлень, де фігурує найдорожча страва меню (Підзапит у WHERE)
SELECT DISTINCT order_id 
FROM order_item 
WHERE dish_id = (
    SELECT dish_id 
    FROM dish 
    ORDER BY price DESC 
    LIMIT 1
);

```
1.
<img width="1004" height="751" alt="image" src="https://github.com/user-attachments/assets/f2fc330f-ebf0-4118-9bf2-ca9aa5e405d3" />
<img width="1010" height="757" alt="image" src="https://github.com/user-attachments/assets/af994f5c-c609-4264-8f31-27c332c0d901" />
<img width="1002" height="753" alt="image" src="https://github.com/user-attachments/assets/3ec50553-09fa-4579-8743-b5c35941e794" />
<img width="1011" height="755" alt="image" src="https://github.com/user-attachments/assets/27173c80-cef0-4d59-ab49-be19ec7a285c" />
2.
<img width="1002" height="754" alt="image" src="https://github.com/user-attachments/assets/e5199558-46e1-419a-981a-1e2d8458256b" />
<img width="1005" height="753" alt="image" src="https://github.com/user-attachments/assets/0c7e6f0f-6d23-49b5-86be-d664d1f99b91" />
<img width="1003" height="752" alt="image" src="https://github.com/user-attachments/assets/191a8b3c-6b0a-4811-b041-f2dd4ee3b086" />
3.
<img width="1010" height="754" alt="image" src="https://github.com/user-attachments/assets/18072b15-758f-4634-b173-6e23869c7f68" />
<img width="1001" height="756" alt="image" src="https://github.com/user-attachments/assets/899a3f64-9378-47bf-9c9c-79e0c54ab9c0" />
<img width="1003" height="749" alt="image" src="https://github.com/user-attachments/assets/ca34687c-5077-466c-8e9e-969ae3699aa0" />
