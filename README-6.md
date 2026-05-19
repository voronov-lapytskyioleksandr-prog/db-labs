### Лабораторна робота 6: Міграції схем за допомогою Prisma ORM

**1. Опис процесу**

У цій лабораторній роботі я налаштував Prisma ORM для керування існуючою схемою бази даних ресторану. Оскільки я працюю на тому самому комп'ютері, де було створено базу даних у попередніх роботах, я виконав команду `npx prisma db pull`, яка успішно проаналізувала структуру існуючих таблиць у PostgreSQL та згенерувала початковий файл `schema.prisma`.

Після цього я послідовно створив та застосував три міграції для зміни схеми бази даних, імітуючи реальний процес еволюції структури даних у виробничих умовах.

**2. Історія міграцій**

**Міграція 1: `add-review-table**`**

* **Обґрунтування змін:** Я додав нову модель `review` для зберігання відгуків відвідувачів про страви ресторану. Це дозволяє пов'язати оцінки та коментарі клієнтів із конкретними позиціями в меню через зв'язок "один до багатьох".
* **Зміни у `schema.prisma` (Додана модель):**

```prisma
model review {
  id         Int     @id @default(autoincrement())
  rating     Int
  comment    String?
  dish_id    Int
  dish       dish    @relation(fields: [dish_id], references: [dish_id])
}

```

**Міграція 2: `add-spicy-flag**`**

* **Обґрунтування змін:** Я розширив модель `dish`, додавши новий логічний прапорець `is_spicy`. Це необхідно для маркування гострих страв у меню ресторану. За замовчуванням встановлено значення `false`.
* **Зміни у `schema.prisma` (Оновлена модель `dish`):**

```prisma
model dish {
  // існуючі поля...
  is_spicy Boolean @default(false)
}

```

**Міграція 3: `drop-category-desc**`**

* **Обґрунтування змін:** Я видалив стовпець `description` з моделі `category`, оскільки для поточної бізнес-логіки системи достатньо лише назви категорії, а текстовий опис виявився надлишковим.
* **Зміни у `schema.prisma` (Оновлена модель `category`):**

```prisma
model category {
  category_id Int      @id @default(autoincrement())
  name        String   @db.VarChar(100)
  dish        dish[]
  // поле description видалено
}
```

**3. Перевірка та результати**

Оскільки була вже створена стара база довдеться задля роботи її скинути.
<img width="981" height="799" alt="image" src="https://github.com/user-attachments/assets/9332bffa-282e-4d84-9473-680f2f7d4d18" />

<img width="982" height="800" alt="image" src="https://github.com/user-attachments/assets/924b3e74-614b-448a-9f80-f597fb8227ec" />

<img width="981" height="344" alt="image" src="https://github.com/user-attachments/assets/7b1c1c3d-e671-469a-9ff3-335af86dc4a5" />

<img width="978" height="345" alt="image" src="https://github.com/user-attachments/assets/3906bafb-e653-4ebf-b9d9-998f60df9ffc" />

<img width="1920" height="905" alt="image" src="https://github.com/user-attachments/assets/29dc77af-36b6-4cff-b907-f6af4611f860" />


