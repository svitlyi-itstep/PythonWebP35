# Моделі та ORM

1. [Що таке Model і навіщо вона потрібна](#що-таке-model-і-навіщо-вона-потрібна)
2. [Створення моделі](#створення-моделі)
3. [Поля моделі](#поля-моделі)
4. [Міграції](#міграції)
5. [Адмін-панель](#адмін-панель)
6. [ORM: запити до бази даних](#orm-запити-до-бази-даних)
7. [Використання моделей у View](#використання-моделей-у-view)
8. [Відображення даних у шаблоні](#відображення-даних-у-шаблоні)

---

## Що таке Model і навіщо вона потрібна

До цього моменту дані у view були захардкоджені прямо у коді:

```python
def book_list(request):
    books = [
        {"id": 1, "title": "Kobzar",  "author": "Shevchenko"},
        {"id": 2, "title": "1984",    "author": "Orwell"},
    ]
    return render(request, "app_name/book_list.html", {"books": books})
```

Такий підхід має очевидні проблеми: дані зникають при перезапуску сервера, їх не можна редагувати без зміни коду, і вони недоступні іншим частинам застосунку.

**Model** вирішує цю проблему: це Python-клас, який описує структуру даних і автоматично пов'язується з таблицею в базі даних.

### Принцип ORM

**ORM (Object-Relational Mapping)** — це механізм, який дозволяє працювати з базою даних через Python-об'єкти, не пишучи SQL вручну.

```
Python-клас   →   таблиця в БД
атрибут класу →   стовпець таблиці
екземпляр     →   рядок таблиці
```

Порівняння підходів:

```python
# Без ORM — SQL вручну (так не робимо в Django)
cursor.execute("SELECT * FROM app_name_book WHERE year > 1900")
rows = cursor.fetchall()

# З ORM — Python-код (так робимо)
books = Book.objects.filter(year__gt=1900)
```

ORM-підхід читабельніший, безпечніший (захист від SQL-ін'єкцій) і не залежить від конкретної бази даних: той самий Python-код працює з SQLite, PostgreSQL і MySQL.

### База даних за замовчуванням

Django поставляється з налаштованою базою даних **SQLite** — файлова БД, яка не потребує окремого встановлення. Вона зберігається у файлі `db.sqlite3` у корені проєкту і ідеально підходить для навчання та розробки.

```python
# project_name/settings.py — налаштування за замовчуванням
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}
```

---

## Створення моделі

Моделі описуються у файлі `models.py` всередині застосунку. Кожна модель — це клас, що успадковується від `django.db.models.Model`.

```python
# app_name/models.py

from django.db import models


class Book(models.Model):
    title  = models.CharField(max_length=200)
    author = models.CharField(max_length=200)
    year   = models.IntegerField()
    description = models.TextField(blank=True)
    is_available = models.BooleanField(default=True)

    def __str__(self):
        return f"{self.title} — {self.author}"
```

### Що тут відбувається

| Елемент | Пояснення |
|---|---|
| `models.Model` | Базовий клас — Django знає, що це модель |
| `title = models.CharField(...)` | Поле — стовпець у таблиці |
| `def __str__` | Як об'єкт відображається у рядковому вигляді |

### Автоматичне поле `id`

Django автоматично додає до кожної моделі поле `id` — унікальний цілочисельний ідентифікатор, який автоматично збільшується. Писати його вручну не потрібно.

```python
book = Book.objects.get(id=1)
print(book.id)  # 1
print(book.pk)  # 1 — pk (primary key) є псевдонімом id
```

---

## Поля моделі

### Основні типи полів

| Поле | Опис | Аналог у SQL |
|---|---|---|
| `CharField(max_length=N)` | Рядок обмеженої довжини | `VARCHAR(N)` |
| `TextField()` | Довгий текст без обмежень | `TEXT` |
| `IntegerField()` | Ціле число | `INTEGER` |
| `FloatField()` | Число з плаваючою точкою | `REAL` |
| `BooleanField()` | `True` або `False` | `BOOLEAN` |
| `DateField()` | Дата (рік, місяць, день) | `DATE` |
| `DateTimeField()` | Дата і час | `DATETIME` |
| `EmailField()` | Рядок із валідацією email | `VARCHAR` |
| `URLField()` | Рядок із валідацією URL | `VARCHAR` |
| `ImageField()` | Шлях до зображення | `VARCHAR` |

### Часті параметри полів

| Параметр | Значення | Пояснення |
|---|---|---|
| `max_length=N` | число | Максимальна кількість символів (обов'язково для `CharField`) |
| `blank=True` | bool | Дозволити порожнє значення у формах |
| `null=True` | bool | Дозволити `NULL` у базі даних |
| `default=...` | значення | Значення за замовчуванням |
| `verbose_name="..."` | рядок | Людська назва поля (для адмін-панелі) |
| `unique=True` | bool | Значення має бути унікальним |
| `choices=...` | список кортежів | Обмежений набір допустимих значень |

### Приклади з параметрами

```python
class Book(models.Model):
    title = models.CharField(
        max_length=200,
        verbose_name="Назва",
    )
    description = models.TextField(
        blank=True,
        verbose_name="Опис",
    )
    year = models.IntegerField(
        default=2024,
        verbose_name="Рік видання",
    )
    rating = models.FloatField(
        null=True,
        blank=True,
        verbose_name="Оцінка",
    )
    is_available = models.BooleanField(
        default=True,
        verbose_name="Доступна",
    )
    created_at = models.DateTimeField(
        auto_now_add=True,   # встановлюється автоматично при створенні
    )
    updated_at = models.DateTimeField(
        auto_now=True,       # оновлюється автоматично при кожному збереженні
    )

    def __str__(self):
        return self.title
```

### Поле з вибором значень (`choices`)

```python
class Book(models.Model):
    GENRE_CHOICES = [
        ("fiction",    "Художня"),
        ("non_fiction","Науково-популярна"),
        ("poetry",     "Поезія"),
        ("children",   "Дитяча"),
    ]

    title = models.CharField(max_length=200)
    genre = models.CharField(
        max_length=20,
        choices=GENRE_CHOICES,
        default="fiction",
    )
```

### Метод `__str__`

Метод `__str__` визначає, як об'єкт буде відображатися у рядковому вигляді — в адмін-панелі, у виводі `print()` і де завгодно ще.

```python
def __str__(self):
    return f"{self.title} ({self.year})"
# Результат: "Kobzar (1840)"
```

Без цього методу Django показуватиме `Book object (1)` — незрозуміло і незручно.

### Клас `Meta`

Вкладений клас `Meta` дозволяє задати метаданні моделі:

```python
class Book(models.Model):
    title  = models.CharField(max_length=200)
    author = models.CharField(max_length=200)
    year   = models.IntegerField()

    class Meta:
        ordering = ["-year"]          # сортування за замовчуванням (новіші перші)
        verbose_name = "Книга"        # назва в однині для адмін-панелі
        verbose_name_plural = "Книги" # назва у множині

    def __str__(self):
        return self.title
```

---

## Міграції

**Міграція** — це файл, який описує зміни в структурі бази даних. Щоразу коли ви змінюєте модель, Django генерує міграцію і застосовує її до БД.

### Два кроки — завжди разом

```bash
# Крок 1 — Django аналізує models.py і створює файл міграції
python manage.py makemigrations

# Крок 2 — Django застосовує міграцію до бази даних
python manage.py migrate
```

> **Правило:** будь-яка зміна у `models.py` → `makemigrations` → `migrate`. Пропустити другий крок — база не оновиться.

### Що відбувається під капотом

```
models.py змінився
       │
       ▼
makemigrations   →   app_name/migrations/0001_initial.py
                              (файл із описом змін)
       │
       ▼
   migrate       →   CREATE TABLE app_name_book (...)
                              (SQL виконується до БД)
```

### Перегляд згенерованої міграції

```python
# app_name/migrations/0001_initial.py (генерується автоматично)

class Migration(migrations.Migration):
    operations = [
        migrations.CreateModel(
            name="Book",
            fields=[
                ("id",     models.AutoField(primary_key=True)),
                ("title",  models.CharField(max_length=200)),
                ("author", models.CharField(max_length=200)),
                ("year",   models.IntegerField()),
            ],
        ),
    ]
```

Цей файл редагувати вручну не потрібно — він генерується і застосовується автоматично.

### Перевірка статусу міграцій

```bash
python manage.py showmigrations
```

Виводить список усіх міграцій із позначкою `[X]` (застосована) або `[ ]` (не застосована).

### Ім'я таблиці в БД

Django називає таблицю автоматично: `назва_застосунку_назва_моделі` у нижньому регістрі.

```
Застосунок: app_name
Модель:     Book
Таблиця:    app_name_book
```

---

## Адмін-панель

Django автоматично генерує повноцінний інтерфейс адміністратора. Щоб модель з'явилась в адмін-панелі — її потрібно зареєструвати.

### Реєстрація моделі

```python
# app_name/admin.py

from django.contrib import admin
from .models import Book

admin.site.register(Book)
```

Після цього за адресою `http://127.0.0.1:8000/admin/` з'явиться повноцінний CRUD-інтерфейс: список записів, форми створення та редагування, кнопка видалення.

### Створення суперкористувача

Для входу в адмін-панель потрібен обліковий запис адміністратора:

```bash
python manage.py createsuperuser
```

Django запитає логін, email і пароль.

### Налаштування відображення в адмін-панелі

За допомогою класу `ModelAdmin` можна налаштувати список і форми:

```python
# app_name/admin.py

from django.contrib import admin
from .models import Book


@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    list_display  = ["title", "author", "year", "is_available"]  # стовпці в списку
    list_filter   = ["is_available", "year"]                     # фільтри праворуч
    search_fields = ["title", "author"]                          # рядок пошуку
    ordering      = ["-year"]                                    # сортування
```

| Параметр | Призначення |
|---|---|
| `list_display` | Які поля показувати у списку записів |
| `list_filter` | Фільтри у правій бічній панелі |
| `search_fields` | Поля, по яких працює пошук |
| `ordering` | Сортування за замовчуванням |

> Декоратор `@admin.register(Book)` — скорочення для `admin.site.register(Book, BookAdmin)`. Обидва варіанти рівнозначні.

---

## ORM: запити до бази даних

Усі запити до БД виконуються через менеджер `objects`, який є у кожній моделі.

### Отримання даних

```python
# Всі записи
books = Book.objects.all()
# SELECT * FROM app_name_book;

# Один запис за id — викидає виняток якщо не знайдено
book = Book.objects.get(pk=1)
# SELECT * FROM app_name_book WHERE id = 1;

# Один запис або None — безпечніший варіант
book = Book.objects.filter(pk=1).first()

# Кількість записів
count = Book.objects.count()
# SELECT COUNT(*) FROM app_name_book;
```

### Фільтрація

```python
# Точна відповідність
books = Book.objects.filter(author="Orwell")

# Кілька умов одночасно (AND)
books = Book.objects.filter(author="Orwell", is_available=True)

# Виключення (NOT)
books = Book.objects.exclude(is_available=False)
```

### Оператори пошуку (lookup)

Додаються до імені поля через подвійне підкреслення `__`:

```python
Book.objects.filter(year__gt=1900)       # year > 1900
Book.objects.filter(year__gte=1900)      # year >= 1900
Book.objects.filter(year__lt=2000)       # year < 2000
Book.objects.filter(year__lte=2000)      # year <= 2000
Book.objects.filter(year__range=(1900, 2000))  # 1900 <= year <= 2000

Book.objects.filter(title__contains="1984")    # містить підрядок
Book.objects.filter(title__icontains="dune")   # містить, без урахування регістру
Book.objects.filter(title__startswith="Ko")    # починається з
Book.objects.filter(title__endswith="ar")      # закінчується на

Book.objects.filter(rating__isnull=True)       # поле = NULL
```

### Сортування

```python
# За зростанням
books = Book.objects.order_by("year")

# За спаданням (мінус перед іменем поля)
books = Book.objects.order_by("-year")

# За кількома полями
books = Book.objects.order_by("author", "-year")
```

### Ланцюжки запитів

Методи ORM можна поєднувати в ланцюжок — кожен повертає новий QuerySet:

```python
books = (
    Book.objects
    .filter(is_available=True)
    .exclude(year__lt=1900)
    .order_by("-year")
)
```

### Створення, оновлення, видалення

```python
# Створення — спосіб 1: create()
book = Book.objects.create(
    title="Dune",
    author="Herbert",
    year=1965,
)

# Створення — спосіб 2: save()
book = Book(title="Dune", author="Herbert", year=1965)
book.save()

# Оновлення одного запису
book = Book.objects.get(pk=1)
book.title = "Нова назва"
book.save()

# Масове оновлення
Book.objects.filter(year__lt=1900).update(is_available=False)

# Видалення одного запису
book = Book.objects.get(pk=1)
book.delete()

# Масове видалення
Book.objects.filter(is_available=False).delete()
```

### `get_or_create()`

Отримати запис, або створити його якщо він не існує:

```python
book, created = Book.objects.get_or_create(
    title="Kobzar",
    defaults={"author": "Shevchenko", "year": 1840},
)
# created = True якщо запис був створений
# created = False якщо запис вже існував
```

### Обробка винятку `DoesNotExist`

Метод `get()` викидає виняток якщо запис не знайдено. Варто обробляти це явно:

```python
from django.shortcuts import get_object_or_404

# Варіант 1 — вручну через try/except
try:
    book = Book.objects.get(pk=pk)
except Book.DoesNotExist:
    book = None

# Варіант 2 — скорочення Django: повертає 404 якщо не знайдено
book = get_object_or_404(Book, pk=pk)
```

`get_object_or_404` — найпоширеніший підхід у view-функціях.

---

## Використання моделей у View

Після підключення моделі view-функція більше не потребує захардкодженого списку — дані беруться з бази.

### До та після

```python
# Було — захардкоджені дані
def book_list(request):
    books = [
        {"id": 1, "title": "Kobzar",  "author": "Shevchenko"},
        {"id": 2, "title": "1984",    "author": "Orwell"},
    ]
    return render(request, "app_name/book_list.html", {"books": books})


# Стало — дані з бази
from .models import Book

def book_list(request):
    books = Book.objects.all().order_by("-year")
    return render(request, "app_name/book_list.html", {"books": books})
```

Шаблон при цьому залишається незмінним — `{{ book.title }}` працює однаково і для словника, і для об'єкта моделі.

### Сторінка списку

```python
# app_name/views.py

from django.shortcuts import render, get_object_or_404
from .models import Book


def book_list(request):
    books = Book.objects.all().order_by("-year")
    return render(request, "app_name/book_list.html", {"books": books})
```

### Сторінка деталей

```python
def book_detail(request, pk):
    book = get_object_or_404(Book, pk=pk)
    return render(request, "app_name/book_detail.html", {"book": book})
```

### Маршрути

```python
# app_name/urls.py

from django.urls import path
from . import views

app_name = "app_name"

urlpatterns = [
    path("",          views.book_list,   name="list"),
    path("<int:pk>/", views.book_detail, name="detail"),
]
```

---

## Відображення даних у шаблоні

Шаблони працюють з об'єктами моделі так само, як із словниками — через крапкову нотацію.

### Список записів

```html
<!-- app_name/templates/app_name/book_list.html -->

{% extends "app_name/base.html" %}

{% block content %}
  <h1>Каталог книг</h1>
  <p>Всього книг: {{ books|length }}</p>

  {% for book in books %}
    <div>
      <h3>
        <a href="{% url 'app_name:detail' book.pk %}">{{ book.title }}</a>
      </h3>
      <p>{{ book.author }}, {{ book.year }}</p>

      {% if book.is_available %}
        <span>✅ Доступна</span>
      {% else %}
        <span>❌ Недоступна</span>
      {% endif %}
    </div>
  {% empty %}
    <p>Книг у каталозі ще немає.</p>
  {% endfor %}
{% endblock %}
```

### Сторінка деталей

```html
<!-- app_name/templates/app_name/book_detail.html -->

{% extends "app_name/base.html" %}

{% block title %}{{ book.title }}{% endblock %}

{% block content %}
  <h1>{{ book.title }}</h1>

  <table>
    <tr><td>Автор</td>     <td>{{ book.author }}</td></tr>
    <tr><td>Рік</td>       <td>{{ book.year }}</td></tr>
    <tr><td>Опис</td>      <td>{{ book.description|default:"Опис відсутній" }}</td></tr>
    <tr>
      <td>Доступність</td>
      <td>{{ book.is_available|yesno:"Доступна,Недоступна" }}</td>
    </tr>
  </table>

  <a href="{% url 'app_name:list' %}">← До каталогу</a>
{% endblock %}
```

### Повна структура проєкту після цього заняття

```
my_project/
├── project_name/
│   ├── settings.py
│   └── urls.py
│
└── app_name/
    ├── migrations/
    │   ├── __init__.py
    │   └── 0001_initial.py       ← генерується автоматично
    ├── templates/
    │   └── app_name/
    │       ├── base.html
    │       ├── book_list.html
    │       └── book_detail.html
    ├── __init__.py
    ├── admin.py                  ← реєстрація моделі
    ├── models.py                 ← опис моделі
    ├── urls.py
    └── views.py                  ← використання ORM
```

