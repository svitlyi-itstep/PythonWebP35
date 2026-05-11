# Архітектурні патерни MVC та MVT

## Зміст
- [Що таке архітектурний патерн?](#що-таке-архітектурний-патерн)
- [Патерн MVC](#патерн-mvc)
- [Патерн MVT](#патерн-mvt)
- [Порівняння MVC та MVT](#порівняння-mvc-та-mvt)
- [Фреймворки, що використовують ці патерни](#фреймворки-що-використовують-ці-патерни)
- [Довідкові матеріали](#довідкові-матеріали)

---

## Що таке архітектурний патерн?

> **Архітектурний патерн** — це перевірений шаблон організації коду, який вирішує типову задачу проєктування: _як розподілити відповідальність між частинами програми_.

Без патерну весь код швидко перетворюється на «спагеті»: логіка, дані й відображення переплутані в одному місці. Це ускладнює тестування, розширення і командну роботу.

**Навіщо потрібні патерни:**

- розподіл відповідальності (Separation of Concerns)
- полегшення тестування кожного шару окремо
- можливість змінювати один шар, не зачіпаючи інші
- спільна мова для команди

---

## Патерн MVC

**MVC (Model – View – Controller)** — архітектурний патерн, який поділяє застосунок на три незалежні шари.

![alt text](https://media.geeksforgeeks.org/wp-content/uploads/20240704102850/MVC-Architecture.webp)
<sup><sup>Джерело: https://www.geeksforgeeks.org/system-design/mvc-architecture-system-design/</sup></sup>

### Компоненти MVC

| Шар | Відповідає за | Що НЕ робить |
|---|---|---|
| **Model** | Дані, бізнес-логіка, збереження | Нічого не виводить на екран |
| **View** | Відображення даних користувачу | Не містить логіки обробки |
| **Controller** | Прийом команд, координація M і V | Не зберігає дані, не рендерить |

### Потік даних у MVC

1. Користувач вводить команду → **Controller** отримує її
2. Controller звертається до **Model** (читає або змінює дані)
3. Controller передає результат у **View**
4. **View** виводить результат користувачу

### Консольний приклад на Python: «Бібліотека»

Проєкт складається з чотирьох файлів:

```
library/
├── model.py
├── view.py
├── controller.py
└── main.py
```

#### `model.py` — дані та бізнес-логіка

```python
class Book:
    def __init__(self, title: str, author: str, year: int):
        self.title = title
        self.author = author
        self.year = year


class LibraryModel:
    def __init__(self):
        self._books = [
            Book("Kobzar", "Shevchenko", 1840),
            Book("1984", "Orwell", 1949),
            Book("Dune", "Herbert", 1965),
        ]

    def get_all(self) -> list[Book]:
        return list(self._books)

    def add(self, title: str, author: str, year: int) -> None:
        self._books.append(Book(title, author, year))

    def search(self, query: str) -> list[Book]:
        q = query.lower()
        return [b for b in self._books
                if q in b.title.lower() or q in b.author.lower()]

    def delete(self, title: str) -> bool:
        before = len(self._books)
        self._books = [b for b in self._books if b.title != title]
        return len(self._books) < before  # True якщо книгу знайдено і видалено
```

#### `view.py` — лише відображення

```python
class LibraryView:
    def show_menu(self) -> None:
        print("\n=== Бібліотека ===")
        print("1. Показати всі книги")
        print("2. Додати книгу")
        print("3. Знайти книгу")
        print("4. Видалити книгу")
        print("0. Вийти")

    def show_books(self, books: list) -> None:
        if not books:
            print("  (список порожній)")
            return
        for i, b in enumerate(books, 1):
            print(f"  {i}. {b.title} — {b.author} ({b.year})")

    def show_message(self, msg: str) -> None:
        print(f"\n  ✔  {msg}\n")

    def show_error(self, msg: str) -> None:
        print(f"\n  ✘  {msg}\n")

    def get_input(self, prompt: str) -> str:
        return input(f"  {prompt}: ").strip()
```

#### `controller.py` — координатор

```python
from model import LibraryModel
from view import LibraryView


class LibraryController:
    def __init__(self):
        self.model = LibraryModel()
        self.view = LibraryView()

    def run(self) -> None:
        while True:
            self.view.show_menu()
            choice = self.view.get_input("Оберіть дію")

            if choice == "1":
                self.view.show_books(self.model.get_all())

            elif choice == "2":
                title  = self.view.get_input("Назва")
                author = self.view.get_input("Автор")
                year   = self.view.get_input("Рік")
                if not year.isdigit():
                    self.view.show_error("Рік має бути числом")
                else:
                    self.model.add(title, author, int(year))
                    self.view.show_message("Книгу додано!")

            elif choice == "3":
                query = self.view.get_input("Пошуковий запит")
                self.view.show_books(self.model.search(query))

            elif choice == "4":
                title = self.view.get_input("Назва книги для видалення")
                if self.model.delete(title):
                    self.view.show_message("Книгу видалено")
                else:
                    self.view.show_error("Книгу не знайдено")

            elif choice == "0":
                print("До побачення!")
                break
```

#### `main.py` — точка входу

```python
from controller import LibraryController

if __name__ == "__main__":
    LibraryController().run()
```

> **Ключова ідея:** якщо замінити `LibraryView` на `FancyView` з іншим форматуванням — `Model` і `Controller` залишаться незмінними. Саме це і є «розподіл відповідальності».

---

## Патерн MVT

> **MVT (Model – View – Template)** — варіант MVC, який використовує фреймворк **Django**. Назви шарів змінені, але принцип той самий.

### Відмінності від MVC

| MVC | MVT (Django) | Що відбувається |
|---|---|---|
| Model | **Model** | Те саме: дані і ORM |
| Controller | **View** | Обробка запиту, логіка |
| View | **Template** | HTML-шаблон із змінними |

>⚠️ **Часта плутанина:** у Django `View` — це не відображення, а обробник запиту (те, що в класичному MVC є Controller). Відображення — це `Template`.

### Потік даних у MVT

![alt text](https://codefinity-content-media.s3.eu-west-1.amazonaws.com/f9161f23-e81e-4082-9642-0a0c970c68aa/899e5b82-04d9-4bb5-b3a8-e18ae1cf90b8_MVT.png)
<sup><sup>Джерело: https://codefinity.com/courses/v2/f9161f23-e81e-4082-9642-0a0c970c68aa/0ce97d0f-ac61-4db5-9ea5-9590e9c7f14c/899e5b82-04d9-4bb5-b3a8-e18ae1cf90b8</sup></sup>

### Компоненти MVT у Django

| Файл | Роль |
|---|---|
| `models.py` | Структура даних, ORM-запити до БД |
| `views.py` | Обробка запиту, виклик моделі, передача в шаблон |
| `templates/*.html` | HTML із тегами Django (`{{ змінна }}`, `{% блок %}`) |
| `urls.py` | Маршрутизація: який URL → яка View |

### Приклад на Django

Реалізуємо ту саму бібліотеку, але як веб-застосунок.

#### `models.py`

```python
from django.db import models


class Book(models.Model):
    title  = models.CharField(max_length=200)
    author = models.CharField(max_length=200)
    year   = models.IntegerField()

    def __str__(self):
        return f"{self.title} — {self.author} ({self.year})"
```

#### `views.py`

```python
from django.shortcuts import render, redirect
from .models import Book


def book_list(request):
    """Показати всі книги (GET) або додати нову (POST)."""
    if request.method == "POST":
        Book.objects.create(
            title=request.POST["title"],
            author=request.POST["author"],
            year=int(request.POST["year"]),
        )
        return redirect("book_list")

    books = Book.objects.all().order_by("year")
    return render(request, "library/book_list.html", {"books": books})
```

#### `templates/library/book_list.html`

```html
<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <title>Бібліотека</title>
</head>
<body>
    <h1>Каталог книг</h1>

    <ul>
      {% for book in books %}
        <li>{{ book.title }} — {{ book.author }} ({{ book.year }})</li>
      {% empty %}
        <li>Книг ще немає.</li>
      {% endfor %}
    </ul>

    <h2>Додати книгу</h2>
    <form method="post">
      {% csrf_token %}
      <input name="title"  placeholder="Назва"  required>
      <input name="author" placeholder="Автор"  required>
      <input name="year"   placeholder="Рік"    required type="number">
      <button type="submit">Додати</button>
    </form>
</body>
</html>
```

#### `urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.book_list, name="book_list"),
]
```

> **Зверніть увагу:** `models.py` не знає про HTTP. `views.py` не знає про HTML. `templates/` не містить Python-логіки. Розподіл дотриманий так само, як і в консольному MVC.

---

## Порівняння MVC та MVT

| Критерій | MVC | MVT (Django) |
|---|---|---|
| Хто приймає запит | Controller | View (+ urls.py) |
| Хто рендерить | View | Template |
| Хто зберігає дані | Model | Model |
| Маршрутизація | Частина Controller | Окремий `urls.py` |
| Типове середовище | Десктоп, консоль, веб | Виключно веб (Django) |
| Шаблонізатор | Зовнішній або View | Вбудований Django Templates |

**Головна різниця** — у MVT фреймворк (Django) сам бере на себе частину роботи Controller: маршрутизацію, CSRF-захист, рендеринг шаблонів. Розробник пише лише бізнес-логіку у `views.py`.

---

## Фреймворки, що використовують ці патерни

### MVC-фреймворки

| Мова | Фреймворк | Особливості |
|---|---|---|
| **Python** | [Flask](https://flask.palletsprojects.com/) | Мікрофреймворк, MVC без жорсткої структури |
| **Ruby** | [Ruby on Rails](https://rubyonrails.org/) | Канонічна реалізація MVC, «Convention over Configuration» |
| **PHP** | [Laravel](https://laravel.com/) | Елегантний MVC, Eloquent ORM |
| **Java** | [Spring MVC](https://spring.io/projects/spring-framework) | Enterprise-рівень, анотації `@Controller`, `@Model` |
| **C#** | [ASP.NET Core MVC](https://learn.microsoft.com/aspnet/core/mvc/overview) | Потужний MVC від Microsoft |
| **JavaScript** | [Express.js](https://expressjs.com/) | Мінімалістичний, MVC організується вручну |
| **Go** | [Gin](https://gin-gonic.com/) | Швидкий, MVC-подібна структура |
| **Swift** | [Vapor](https://vapor.codes/) | MVC для серверного Swift |

### MVT-фреймворки

| Мова | Фреймворк | Особливості |
|---|---|---|
| **Python** | [Django](https://www.djangoproject.com/) | Єдиний великий фреймворк із «чистим» MVT |

### MVVM та інші варіанти

| Патерн | Мова / Платформа | Фреймворк |
|---|---|---|
| **MVVM** | JavaScript | [Vue.js](https://vuejs.org/), [Angular](https://angular.io/) |
| **MVVM** | C# / WPF | [.NET MAUI](https://learn.microsoft.com/dotnet/maui/) |
| **MVP** | Android (Java) | Класична Android-архітектура |

---

## Довідкові матеріали:
- Модель-вид-контролер — https://uk.wikipedia.org/wiki/%D0%9C%D0%BE%D0%B4%D0%B5%D0%BB%D1%8C-%D0%B2%D0%B8%D0%B4-%D0%BA%D0%BE%D0%BD%D1%82%D1%80%D0%BE%D0%BB%D0%B5%D1%80
- Розділяй та володарюй: що таке патерни MVC і MVP, та як їх використовувати — https://highload.tech/uk/blogs/shho-take-mvc-ta-mvp-patterni/
- MVC Architecture - System Design — https://www.geeksforgeeks.org/system-design/mvc-architecture-system-design/
- MVC — https://developer.mozilla.org/en-US/docs/Glossary/MVC
- Django Project MVT Structure — https://www.geeksforgeeks.org/django-project-mvt-structure/

