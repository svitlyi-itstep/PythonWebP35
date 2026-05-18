# Роутинг та робота з URL

1. [Як Django обробляє запит](#як-django-обробляє-запит)
2. [Файл `urls.py` та функція `path()`](#файл-urlspy-та-функція-path)
3. [Підключення маршрутів застосунку через `include()`](#підключення-маршрутів-застосунку-через-include)
4. [Іменовані маршрути та тег `{% url %}`](#іменовані-маршрути-та-тег--url-)
5. [URL-параметри](#url-параметри)
6. [Простір імен (`app_name`)](#простір-імен-app_name)
7. [Довідкові матеріали](#довідкові-матеріали)

---

## Як Django обробляє запит

Коли користувач відкриває сторінку у браузері, Django виконує наступний ланцюжок дій:

```
Браузер: GET /books/3/
         │
         ▼
   project_name/urls.py          ← головний маршрутизатор проєкту
         │  path("", include("app_name.urls"))
         ▼
   app_name/urls.py         ← маршрутизатор застосунку
         │  path("books/<int:pk>/", views.book_detail, name="book-detail")
         ▼
   views.book_detail(request, pk=3)   ← виклик view-функції
         │
         ▼
   render(request, "app_name/book_detail.html", context)
         │
         ▼
   Браузер отримує готовий HTML
```

Django перебирає маршрути зверху донизу і зупиняється на першому збігу. Якщо жоден маршрут не підійшов — повертається відповідь **404 Not Found**.

---

## Файл `urls.py` та функція `path()`

### Структура `path()`

```python
path(route, view, kwargs=None, name=None)
```

| Аргумент | Тип | Призначення |
|---|---|---|
| `route` | `str` | URL-шаблон — рядок або шаблон із параметрами |
| `view` | функція | View-функція-обробник (без дужок) |
| `kwargs` | `dict` | Додаткові аргументи, що передаються у view (рідко) |
| `name` | `str` | Унікальне ім'я маршруту для використання у шаблонах |

### Базові приклади

```python
# app_name/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path("",        views.index,     name="index"),     # /
    path("books/",  views.book_list, name="book-list"), # /books/
    path("about/",  views.about,     name="about"),     # /about/
    path("contact/",views.contact,   name="contact"),   # /contact/
]
```

### Порядок маршрутів має значення

Django перевіряє маршрути **зверху донизу** і зупиняється на першому збігу. Більш специфічні маршрути мають бути вище:

```python
urlpatterns = [
    path("books/new/",      views.book_create, name="book-create"), # ← спочатку
    path("books/<int:pk>/", views.book_detail, name="book-detail"), # ← потім
    path("books/",          views.book_list,   name="book-list"),
]
```

> Якщо `books/<int:pk>/` стояв би вище — рядок `"new"` не є числом `int`, тому збігу не буде. Але якщо використовувати `<str:pk>` — рядок `"new"` міг би захопитися невірним маршрутом. Тому специфічні маршрути завжди розміщуються першими.

---

## Підключення маршрутів застосунку через `include()`

### Навіщо потрібен `include()`

У реальному проєкті може бути кілька застосунків, кожен зі своїми сторінками. Зручно тримати маршрути кожного застосунку в окремому файлі `urls.py`, а у головному файлі лише підключати їх.

### Структура файлів

```
my_project/
├── project_name/
│   └── urls.py        ← головний маршрутизатор (проєкт)
│
├── app_name/
│   └── urls.py        ← маршрути застосунку app_name
│
└── blog/
    └── urls.py        ← маршрути застосунку blog
```

### Головний `urls.py`

```python
# project_name/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/",   admin.site.urls),
    path("",         include("app_name.urls")), # маршрути app_name від кореня
    path("blog/",    include("blog.urls")),    # маршрути blog від /blog/
]
```

### Маршрути застосунку `app_name`

```python
# app_name/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path("",        views.index,     name="index"),
    path("books/",  views.book_list, name="book-list"),
    path("about/",  views.about,     name="about"),
]
```

### Маршрути застосунку `blog`

```python
# blog/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path("",          views.post_list,   name="post-list"),
    path("new/",      views.post_create, name="post-create"),
]
```

### Як формуються фінальні URL

| `project_name/urls.py` | `app_name/urls.py` | Фінальний URL |
|---|---|---|
| `path("", include("app_name.urls"))` | `path("books/", ...)` | `/books/` |
| `path("blog/", include("blog.urls"))` | `path("new/", ...)` | `/blog/new/` |
| `path("blog/", include("blog.urls"))` | `path("", ...)` | `/blog/` |

> **Принцип:** Django склеює префікс із `project_name/urls.py` і залишок із `urls.py` застосунку.

### Кілька `include()` для одного застосунку

Один `urls.py` можна підключити за різними префіксами:

```python
# project_name/urls.py

urlpatterns = [
    path("uk/app_name/", include("app_name.urls")),
    path("en/app_name/", include("app_name.urls")),
]
```

---

## Іменовані маршрути та тег `{% url %}`

### Навіщо іменувати маршрути

Якщо писати URL вручну у шаблонах — при зміні адреси доведеться шукати і правити кожен файл:

```html
<!-- Погано — URL захардкоджений -->
<a href="/books/">Всі книги</a>
<a href="/about/">Про нас</a>
```

```python
# Змінили URL у urls.py
path("all-books/", views.book_list, name="book-list"),
# Тепер усі /books/ у шаблонах — зламані посилання
```

Іменований маршрут вирішує цю проблему: URL змінюється в одному місці, шаблони оновлюються автоматично.

### Тег `{% url %}` у шаблоні

```html
<!-- Добре — посилання через ім'я маршруту -->
<a href="{% url 'book-list' %}">Всі книги</a>
<a href="{% url 'about' %}">Про нас</a>
```

Django знаходить маршрут з відповідним `name=` і підставляє його URL.

### Практичний приклад: навігація у `base.html`

```html
<!-- base.html -->

<nav>
  <a href="{% url 'index' %}">Головна</a>
  <a href="{% url 'book-list' %}">Книги</a>
  <a href="{% url 'about' %}">Про нас</a>
  <a href="{% url 'contact' %}">Контакти</a>
</nav>
```

### Передача параметрів у `{% url %}`

Якщо маршрут має URL-параметри — їх потрібно передати після імені:

```html
<!-- Один параметр -->
<a href="{% url 'book-detail' book.id %}">{{ book.title }}</a>
<!-- Результат: /books/3/ -->

<!-- Кілька параметрів -->
<a href="{% url 'some-view' arg1 arg2 %}">Посилання</a>
```

---

## URL-параметри

### Навіщо потрібні параметри в URL

Динамічні сторінки — сторінка товару, профіль користувача, деталі замовлення — потребують змінної частини в URL:

```
/books/          → список усіх книг
/books/1/        → сторінка книги з id=1
/books/42/       → сторінка книги з id=42
```

Без URL-параметрів довелось би створювати окремий маршрут для кожного запису.

### Синтаксис

```python
path("books/<int:pk>/", views.book_detail, name="book-detail")
#           ↑       ↑
#        тип    ім'я параметра
```

Параметр у кутових дужках автоматично витягується з URL і передається у view-функцію як аргумент із відповідним ім'ям.

### Вбудовані конвертери типів

| Конвертер | Що приймає | Тип у Python | Приклад URL |
|---|---|---|---|
| `<int:pk>` | Ціле невід'ємне число | `int` | `/books/42/` |
| `<str:name>` | Будь-який рядок без `/` | `str` | `/books/dune/` |
| `<slug:slug>` | Рядок із букв, цифр, `-`, `_` | `str` | `/books/my-book/` |
| `<uuid:uid>` | UUID-рядок | `uuid.UUID` | `/orders/550e8400-.../` |
| `<path:subpath>` | Рядок, може містити `/` | `str` | `/files/docs/report.pdf` |

### Приклад: сторінка деталей книги

```python
# app_name/urls.py

urlpatterns = [
    path("books/",          views.book_list,   name="book-list"),
    path("books/<int:pk>/", views.book_detail, name="book-detail"),
]
```

```python
# app_name/views.py

def book_list(request):
    books = [
        {"id": 1, "title": "Kobzar",  "author": "Shevchenko", "year": 1840},
        {"id": 2, "title": "1984",    "author": "Orwell",      "year": 1949},
        {"id": 3, "title": "Dune",    "author": "Herbert",     "year": 1965},
    ]
    return render(request, "app_name/book_list.html", {"books": books})


def book_detail(request, pk):           # ← pk приходить з URL автоматично
    books = [
        {"id": 1, "title": "Kobzar",  "author": "Shevchenko", "year": 1840},
        {"id": 2, "title": "1984",    "author": "Orwell",      "year": 1949},
        {"id": 3, "title": "Dune",    "author": "Herbert",     "year": 1965},
    ]
    book = next((b for b in books if b["id"] == pk), None)
    return render(request, "app_name/book_detail.html", {"book": book})
```

```html
<!-- app_name/book_list.html -->

{% extends "app_name/base.html" %}
{% block content %}
  <h1>Каталог книг</h1>
  <ul>
    {% for book in books %}
      <li>
        <a href="{% url 'book-detail' book.id %}">
          {{ book.title }}
        </a>
      </li>
    {% endfor %}
  </ul>
{% endblock %}
```

```html
<!-- app_name/book_detail.html -->

{% extends "app_name/base.html" %}
{% block content %}
  {% if book %}
    <h1>{{ book.title }}</h1>
    <p>Автор: {{ book.author }}</p>
    <p>Рік: {{ book.year }}</p>
    <a href="{% url 'book-list' %}">← Назад до списку</a>
  {% else %}
    <p>Книгу не знайдено.</p>
    <a href="{% url 'book-list' %}">← До каталогу</a>
  {% endif %}
{% endblock %}
```

### Кілька параметрів в одному URL

```python
# urls.py
path("authors/<str:author>/books/<int:pk>/", views.author_book, name="author-book"),
```

```python
# views.py
def author_book(request, author, pk):
    # author = "orwell", pk = 2
    ...
```

```html
<!-- шаблон -->
<a href="{% url 'author-book' 'orwell' 2 %}">Книга</a>
<!-- Результат: /authors/orwell/books/2/ -->
```

### Що відбувається при невірному типі

| URL запиту | Маршрут | Результат |
|---|---|---|
| `/books/42/` | `<int:pk>` | ✅ `pk = 42` |
| `/books/abc/` | `<int:pk>` | ❌ 404 — `"abc"` не є int |
| `/books/abc/` | `<str:name>` | ✅ `name = "abc"` |
| `/books/my book/` | `<slug:slug>` | ❌ 404 — пробіл не дозволений |

> Django **автоматично** повертає 404 якщо значення з URL не відповідає типу конвертера. Перевірку типів писати вручну не потрібно.

---

## Параметри запиту

> **Параметри запиту (query parameters)** — це іменовані параметри, які передаються у url після символа "?". Зазвичай вони є необов'язковими та можуть вказуватися у будь-якому параметрі

Параметри додаються в кінці URL-адреси після знака питання `?`. Вони складаються з імені та значення, з'єднаних знаком `=`. Декілька параметрів розділяються знаком `&`.

Приклад параметрів запиту:
```
/product?id=5
/catalog?max_price=2000&sort=price_asc
```

Отримати ці параметри можна з властивості `GET` у параметрі `request`, який отримує `view`:
```python
# app_name/views.py

def get_catalog(request):
  # Отримання параметру ?max_price=2000
  max_price = request.GET.get("max_price")

  # GET — це dict (словник), в якому
  # зберігаються всі передані парамери
  # запиту.
```

---

## Простір імен (`app_name`)

### Проблема: конфлікт імен маршрутів

Коли в проєкті кілька застосунків — імена маршрутів можуть збігатися:

```python
# app_name/urls.py        
path("", views.list, name="list")

# blog/urls.py
path("", views.list, name="list")
```

```html
{% url 'list' %}  ← Django не знає, який із двох маршрутів мається на увазі
```

### Рішення: `app_name`

Додайте змінну `app_name` у `urls.py` застосунку:

```python
# app_name/urls.py

from django.urls import path
from . import views

app_name = "app_name"    # ← оголошуємо простір імен

urlpatterns = [
    path("",          views.book_list,   name="list"),
    path("<int:pk>/", views.book_detail, name="detail"),
]
```

```python
# blog/urls.py

from django.urls import path
from . import views

app_name = "blog"       # ← свій простір імен

urlpatterns = [
    path("",      views.post_list,   name="list"),
    path("<int:pk>/", views.post_detail, name="detail"),
]
```

### Використання у шаблонах

Тепер до імені маршруту додається префікс простору імен через двокрапку:

```html
<!-- Без простору імен — неоднозначно -->
<a href="{% url 'list' %}">???</a>

<!-- З простором імен — зрозуміло -->
<a href="{% url 'app_name:list' %}">Всі книги</a>
<a href="{% url 'blog:list' %}">Всі пости</a>

<!-- З параметром -->
<a href="{% url 'app_name:detail' book.id %}">{{ book.title }}</a>
<a href="{% url 'blog:detail' post.id %}">{{ post.title }}</a>
```

### Підключення у головному `urls.py`

`app_name` у `urls.py` застосунку достатньо — нічого додаткового у `project_name/urls.py` не потрібно:

```python
# project_name/urls.py

from django.urls import path, include

urlpatterns = [
    path("",      include("app_name.urls")), # app_name="app_name" вже оголошено
    path("blog/", include("blog.urls")),    # app_name="blog" вже оголошено
]
```

### Альтернативний спосіб: `namespace` у `include()`

Простір імен можна задати і безпосередньо в `include()`, без `app_name`:

```python
# project_name/urls.py

urlpatterns = [
    path("", include(("app_name.urls", "app_name"))),
    path("blog/", include(("blog.urls", "blog"))),
]
```

> Рекомендований підхід — оголошувати `app_name` прямо у `urls.py` застосунку: це більш явно і зрозуміло.

---

## Довідкові матеріали

- URL dispatcher — https://docs.djangoproject.com/en/6.0/topics/http/urls/
- Django URL-адреси — https://w3schoolsua.github.io/django/django_urls.html#gsc.tab=0