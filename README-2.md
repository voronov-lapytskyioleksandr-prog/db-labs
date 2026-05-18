**Лабораторна робота 2: Перетворення ER-діаграми на схему PostgreSQL**

*1. Підсумок реляційної схеми*
У цій лабораторній роботі концептуальна ER-діаграма системи управління замовленнями в ресторані була перетворена на фізичну реляційну схему в PostgreSQL. Кожна сутність була реалізована як окрема таблиця.

**Список таблиць та їх стовпців:**

* category: category_id (PK), name, description.
* ingredient: ingredient_id (PK), name, is_allergen.
* dish: dish_id (PK), name, description, price, weight, category_id (FK).
* dish_ingredient (асоціативна таблиця): dish_id (FK), ingredient_id (FK). Складений первинний ключ: dish_id, ingredient_id.
* restaurant_order: order_id (PK), order_date, table_number, total_amount.
* order_item (асоціативна таблиця): order_id (FK), dish_id (FK), quantity. Складений первинний ключ: order_id, dish_id.

**2. Обмеження та типи даних**
Для забезпечення цілісності даних були використані такі обмеження:

* PRIMARY KEY (SERIAL): автоматично генерує унікальні ідентифікатори для кожної нової сутності. У PostgreSQL це автоматично означає UNIQUE NOT NULL.
* FOREIGN KEY: гарантує цілісність посилань (наприклад, що не можна додати страву в неіснуючу категорію).
* NOT NULL: застосовано до критично важливих полів, щоб запобігти пропуску даних (назви, ціна, вага, номер столика).
* CHECK: використано для перевірки логіки бізнес-правил (наприклад, ціна та кількість повинні бути більшими за нуль).
* DEFAULT: встановлено значення за замовчуванням (FALSE для алергенів, поточний час для дати замовлення).

**3. Тестування та результати**
Всі таблиці були успішно створені та заповнені тестовими даними за допомогою INSERT INTO (мінімум 3-5 рядків для кожної). Спочатку заповнювалися батьківські таблиці, а потім дочірні, щоб уникнути помилок зовнішніх ключів. Порушень обмежень не виявлено.
Докази виконання (скріншоти з pgAdmin із результатами запитів SELECT *) додані нижче.

**4. SQL-скрипт створення та наповнення бази даних**

```sql
-- Створення таблиці категорій
CREATE TABLE category (
    category_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT
);

-- Створення таблиці інгредієнтів
CREATE TABLE ingredient (
    ingredient_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    is_allergen BOOLEAN DEFAULT FALSE
);

-- Створення таблиці страв
-- Використовуємо NUMERIC для ціни та CHECK для перевірки додатного значення
CREATE TABLE dish (
    dish_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price NUMERIC(8,2) NOT NULL CHECK (price > 0),
    weight INTEGER NOT NULL CHECK (weight > 0),
    category_id INTEGER REFERENCES category(category_id)
);

-- Створення таблиці-зв'язку між стравами та інгредієнтами
CREATE TABLE dish_ingredient (
    dish_id INTEGER REFERENCES dish(dish_id),
    ingredient_id INTEGER REFERENCES ingredient(ingredient_id),
    PRIMARY KEY (dish_id, ingredient_id)
);

-- Створення таблиці замовлень
CREATE TABLE restaurant_order (
    order_id SERIAL PRIMARY KEY,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    table_number INTEGER NOT NULL CHECK (table_number > 0),
    total_amount NUMERIC(8,2) DEFAULT 0 CHECK (total_amount >= 0)
);

-- Створення таблиці-зв'язку між замовленнями та стравами
CREATE TABLE order_item (
    order_id INTEGER REFERENCES restaurant_order(order_id),
    dish_id INTEGER REFERENCES dish(dish_id),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    PRIMARY KEY (order_id, dish_id)
);

-- ЗАПОВНЕННЯ ТАБЛИЦЬ ДАНИМИ
-- Важливо: спочатку заповнюємо батьківські таблиці, потім дочірні

INSERT INTO category (name, description) VALUES
    ('Перші страви', 'Гарячі супи та борщі'),
    ('Основні страви', 'М''ясні та рибні страви'),
    ('Напої', 'Холодні та гарячі напої');

INSERT INTO ingredient (name, is_allergen) VALUES
    ('Свинина', FALSE),
    ('Буряк', FALSE),
    ('Молоко', TRUE),
    ('Арахіс', TRUE),
    ('Картопля', FALSE);

INSERT INTO dish (name, description, price, weight, category_id) VALUES
    ('Борщ український', 'Традиційний борщ зі свининою', 150.00, 300, 1),
    ('Стейк', 'Стейк зі свинини на грилі', 350.00, 250, 2),
    ('Пюре', 'Картопляне пюре з молоком', 80.00, 200, 2),
    ('Кава лате', 'Кава з молоком', 65.00, 250, 3);

INSERT INTO dish_ingredient (dish_id, ingredient_id) VALUES
    (1, 1), -- Борщ + Свинина
    (1, 2), -- Борщ + Буряк
    (1, 5), -- Борщ + Картопля
    (2, 1), -- Стейк + Свинина
    (3, 5), -- Пюре + Картопля
    (3, 3); -- Пюре + Молоко

INSERT INTO restaurant_order (table_number, total_amount) VALUES
    (5, 500.00),
    (12, 150.00),
    (3, 415.00);

INSERT INTO order_item (order_id, dish_id, quantity) VALUES
    (1, 1, 2), -- Замовлення 1: 2 борщі
    (1, 3, 1), -- Замовлення 1: 1 пюре
    (2, 4, 2), -- Замовлення 2: 2 лате
    (3, 2, 1), -- Замовлення 3: 1 стейк
    (3, 4, 1); -- Замовлення 3: 1 лате

```
<img width="1011" height="758" alt="image" src="https://github.com/user-attachments/assets/6e8f971e-d9b4-41eb-85bd-cb55fba7078c" />
<img width="1004" height="754" alt="image" src="https://github.com/user-attachments/assets/14ab325e-e738-4cb7-88be-81dac9b202de" />
