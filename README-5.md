### Лабораторна робота 5: Нормалізація бази даних

**1. Пошук надлишковості та початкова схема**

Під час аналізу бази даних було виявлено погано структуровану зведену таблицю `legacy_orders_data`, яка містить ознаки аномалій та дублювання даних. Одне й те саме значення (наприклад, назва категорії чи опис страви) зберігається в кількох місцях, що порушує принципи нормалізації.

Оригінальний дизайн таблиці:
`legacy_orders_data(order_id, order_date, table_number, dish_id, dish_name, category_name, category_description, quantity)`

**Функціональні залежності (ФЗ) для початкової схеми:**
Згідно з вимогами, визначено такі функціональні залежності:

* `order_id` -> `order_date`, `table_number`
* `dish_id` -> `dish_name`, `category_name`, `category_description`
* `category_name` -> `category_description`
* `(order_id, dish_id)` -> `quantity`

**Оцінка початкової нормальної форми:**
Початкова схема знаходиться у **Першій нормальній формі (1NF)**. Усі атрибути є атомарними (без повторюваних груп у стовпцях). Проте вона порушує правила вищих нормальних форм через наявність часткових та транзитивних залежностей.

**2. Покрокове пояснення нормалізації**

**Перехід до 2NF (Усунення часткових залежностей)**

* **Проблема:** Таблиця має складений первинний ключ `(order_id, dish_id)`. Атрибути `dish_name`, `category_name` та `category_description` залежать лише від частини ключа (`dish_id`), а не від усього складеного ключа. Це порушує 2NF.


* **Рішення:** Розділяємо таблицю на три нові. Дані про замовлення виносимо в `restaurant_order`, деталі кількості — в `order_item`, а інформацію про страви — у `legacy_dish`.
* **Результат (2NF):**
* `restaurant_order(order_id, order_date, table_number)`
* `order_item(order_id, dish_id, quantity)`
* `legacy_dish(dish_id, dish_name, category_name, category_description)`



**Перехід до 3NF (Усунення транзитивних залежностей)**

* 
**Проблема:** Таблиця `legacy_dish` знаходиться у 2NF, але атрибут `category_description` залежить від `category_name`, який не є первинним ключем. Це транзитивна залежність, яка порушує 3NF.


* 
**Рішення:** Видаляємо транзитивну залежність, виносячи категорії в окрему таблицю `category`.


* **Результат (3NF):**
* `category(category_id, name, description)`
* `dish(dish_id, name, category_id)`
* (Таблиці `restaurant_order` та `order_item` залишаються без змін).
Ця зміна усуває аномалії оновлення (тепер опис категорії зберігається лише один раз).





**3. SQL-скрипти реорганізації (ALTER TABLE та CREATE TABLE)**

```sql
-- 1. Створення таблиці категорій для усунення транзитивної залежності (3NF)
CREATE TABLE category (
    category_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT
);

-- 2. Створення фінальної таблиці страв (3NF)
CREATE TABLE dish (
    dish_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    category_id INTEGER REFERENCES category(category_id)
);

-- 3. Модифікація старих даних (імітація ALTER TABLE для зв'язування)
-- Додаємо зовнішній ключ до нашої нормалізованої таблиці страв
ALTER TABLE dish 
ADD CONSTRAINT fk_dish_category 
FOREIGN KEY (category_id) REFERENCES category(category_id);

-- 4. Створення таблиць замовлень (вже в 3NF)
CREATE TABLE restaurant_order (
    order_id SERIAL PRIMARY KEY,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    table_number INTEGER NOT NULL
);

CREATE TABLE order_item (
    order_id INTEGER REFERENCES restaurant_order(order_id),
    dish_id INTEGER REFERENCES dish(dish_id),
    quantity INTEGER NOT NULL,
    PRIMARY KEY (order_id, dish_id)
);

-- 5. Видалення початкової ненормалізованої таблиці
-- DROP TABLE legacy_orders_data;

```

---
<img width="1013" height="760" alt="image" src="https://github.com/user-attachments/assets/3930dcdf-5117-4f32-817e-40119d01817e" />

<img width="792" height="565" alt="er_diagram" src="https://github.com/user-attachments/assets/19e5154b-baad-4e20-ab60-58c8babc3336" />
