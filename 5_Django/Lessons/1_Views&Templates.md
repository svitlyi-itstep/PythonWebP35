# 1. Використання Views та Templates

## Зміст

1. [Що таке View?](#що-таке-view)
2. [Маршрутизація: як підключити View до URL](#маршрутизація-як-підключити-view-до-url)
3. [Що таке Template?](#що-таке-template)
4. [Як використати Template у View](#як-використати-template-у-view)
5. [Контекст: передача даних у шаблон](#контекст-передача-даних-у-шаблон)
6. [Статичні файли: CSS та зображення](#статичні-файли-css-та-зображення)
7. [Підсумок та шпаргалка](#підсумок-та-шпаргалка)

---

## Що таке View?

**View** — це звичайна Python-функція, яка:

1. отримує HTTP-запит (`request`)
2. виконує якусь логіку
3. повертає HTTP-відповідь (`response`)

Це серце обробки будь-якої сторінки. Користувач відкрив `/about/` → Django знайшов відповідну View-функцію → вона повернула HTML → браузер відобразив сторінку.

### Найпростіша View

```python
# app_name/views.py

from django.http import HttpResponse


def index(request):
    return HttpResponse("Привіт, світ!")
```

Тут:

- `request` — об'єкт із усією інформацією про запит (метод, заголовки, дані форми тощо)
- `HttpResponse(...)` — найпростіша відповідь: рядок тексту або HTML

### Повернення HTML напряму

```python
def index(request):
    html = """
    <html>
      <body>
        <h1>Бібліотека</h1>
        <p>Ласкаво просимо!</p>
      </body>
    </html>
    """
    return HttpResponse(html)
```

> Писати HTML прямо у Python — незручно і погана практика. Саме тому існують **Templates**, які ми розглянемо далі.

### Метод запиту

Через `request.method` можна дізнатися, яким методом прийшов запит:

```python
def contact(request):
    if request.method == "POST":
        return HttpResponse("Форму отримано!")
    return HttpResponse("Це сторінка контактів.")
```

---

## 2. Маршрутизація: як підключити View до URL

Щоб View була доступна за певною адресою, її треба зареєструвати в **маршрутизаторі** (`urls.py`).

### Структура файлів

У Django є два рівні маршрутизації:

```
project_name/
└── urls.py        ← головний маршрутизатор проєкту

app_name/
└── urls.py        ← маршрутизатор застосунку (створюється вручну)
```

### Крок 1 — Створіть `urls.py` всередині застосунку

```python
# app_name/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path("",       views.index,   name="index"),
    path("about/", views.about,   name="about"),
]
```

Синтаксис `path()`:

| Аргумент | Що означає |
|---|---|
| `""` | Порожній рядок — кореневий URL застосунку |
| `"about/"` | URL `/about/` |
| `views.index` | Функція-обробник (без дужок!) |
| `name="index"` | Ім'я маршруту для посилань у шаблонах |

### Крок 2 — Підключіть маршрути застосунку до головного `urls.py`

```python
# project_name/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("",       include("app_name.urls")),  # ← підключаємо застосунок
]
```

`include("app_name.urls")` означає: «усі URL з `app_name/urls.py` доступні від кореня сайту».

### Крок 3 — Додайте View-функції

```python
# app_name/views.py

from django.http import HttpResponse


def index(request):
    return HttpResponse("<h1>Головна сторінка</h1>")


def about(request):
    return HttpResponse("<h1>Про нас</h1>")
```

Після запуску сервера:

| URL | View | Відповідь |
|---|---|---|
| `http://127.0.0.1:8000/` | `index` | «Головна сторінка» |
| `http://127.0.0.1:8000/about/` | `about` | «Про нас» |

> **Зверніть увагу:** у `path()` функція передається **без дужок** — `views.index`, а не `views.index()`. Дужки означали б негайний виклик, а нам потрібне посилання на функцію.

---

## Що таке Template?

**Template (шаблон)** — це HTML-файл із спеціальними тегами Django, які дозволяють вставляти дані і додавати логіку прямо у розмітку.

### Чому не писати HTML у Python?

```python
# Погано — незручно, нечитабельно, важко підтримувати
def index(request):
    books = ["Kobzar", "1984", "Dune"]
    items = "".join(f"<li>{b}</li>" for b in books)
    return HttpResponse(f"<ul>{items}</ul>")
```

```html
<!-- Добре — HTML окремо, логіка окремо -->
<ul>
  {% for book in books %}
    <li>{{ book }}</li>
  {% endfor %}
</ul>
```

### Де зберігати шаблони?

Рекомендована структура:

```
app_name/
└── templates/
    └── app_name/          ← папка з назвою застосунку (щоб уникнути конфліктів)
        ├── index.html
        └── about.html
```

> Вкладена папка `app_name/` всередині `templates/` — це конвенція Django. Якщо два застосунки мають `index.html`, Django розрізнятиме їх за шляхом: `app_name/index.html` vs `blog/index.html`.

### Налаштування шаблонів у `settings.py`

За замовчуванням Django шукає шаблони у папці `templates/` кожного застосунку — якщо він є в `INSTALLED_APPS`. Перевірте, що ваш застосунок підключено:

```python
# project_name/settings.py

INSTALLED_APPS = [
    ...
    'app_name',   # ← застосунок має бути тут
]
```

І що в `TEMPLATES` є `'APP_DIRS': True`:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,   # ← Django шукає templates/ у кожному застосунку
        'OPTIONS': { ... },
    },
]
```

### Синтаксис шаблонів Django

Django Templates мають два типи спеціальних конструкцій:

| Конструкція | Синтаксис | Призначення |
|---|---|---|
| Змінна | `{{ змінна }}` | Вивести значення |
| Тег | `{% тег %}` | Логіка: цикли, умови, блоки |

```html
<!-- Виведення змінної -->
<h1>{{ title }}</h1>

<!-- Умова -->
{% if user_count > 0 %}
  <p>Користувачів: {{ user_count }}</p>
{% else %}
  <p>Користувачів ще немає.</p>
{% endif %}

<!-- Цикл -->
<ul>
  {% for book in books %}
    <li>{{ book }}</li>
  {% empty %}
    <li>Книг немає.</li>
  {% endfor %}
</ul>
```

---

## Як використати Template у View

Щоб рендерити шаблон, Django надає функцію `render()`.

### Функція `render()`

```python
render(request, template_name, context)
```

| Аргумент | Що передати |
|---|---|
| `request` | Об'єкт запиту (завжди першим) |
| `template_name` | Шлях до шаблону відносно `templates/` |
| `context` | Словник із даними для шаблону (необов'язковий) |

### Приклад

```python
# app_name/views.py

from django.shortcuts import render


def index(request):
    return render(request, "app_name/index.html")


def about(request):
    return render(request, "app_name/about.html")
```

```html
<!-- app_name/templates/app_name/index.html -->

<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8">
  <title>Бібліотека</title>
</head>
<body>
  <h1>Ласкаво просимо до бібліотеки!</h1>
  <nav>
    <a href="/">Головна</a> |
    <a href="/about/">Про нас</a>
  </nav>
</body>
</html>
```

> `render()` — це скорочення. Під капотом вона завантажує шаблон, підставляє контекст і повертає `HttpResponse` із готовим HTML. Імпортується з `django.shortcuts`.

---

## Контекст: передача даних у шаблон

**Контекст (context)** — це звичайний Python-словник, який View передає у шаблон. Ключі словника стають іменами змінних у шаблоні.

### Передача простих значень

```python
# app_name/views.py

def index(request):
    context = {
        "title":      "Бібліотека",
        "year":       2025,
        "is_open":    True,
    }
    return render(request, "app_name/index.html", context)
```

```html
<!-- app_name/templates/app_name/index.html -->

<h1>{{ title }}</h1>
<p>Рік заснування: {{ year }}</p>

{% if is_open %}
  <p>🟢 Бібліотека відчинена</p>
{% else %}
  <p>🔴 Бібліотека зачинена</p>
{% endif %}
```

### Передача списку

```python
def index(request):
    context = {
        "books": ["Kobzar", "1984", "Dune", "Гаррі Поттер"],
    }
    return render(request, "app_name/index.html", context)
```

```html
<ul>
  {% for book in books %}
    <li>{{ book }}</li>
  {% empty %}
    <li>Книг ще немає.</li>
  {% endfor %}
</ul>
```

### Передача словника / об'єкта

```python
def index(request):
    context = {
        "book": {
            "title":  "Dune",
            "author": "Frank Herbert",
            "year":   1965,
        }
    }
    return render(request, "app_name/index.html", context)
```

```html
<!-- Доступ до вкладених полів через крапку -->
<h2>{{ book.title }}</h2>
<p>Автор: {{ book.author }}</p>
<p>Рік: {{ book.year }}</p>
```

> **Крапкова нотація** у шаблонах Django — універсальна. `{{ book.title }}` працює і для словників (`book["title"]`), і для об'єктів (`book.title`), і для списків (`books.0`).

### Передача списку словників

```python
def index(request):
    context = {
        "books": [
            {"title": "Kobzar",  "author": "Shevchenko", "year": 1840},
            {"title": "1984",    "author": "Orwell",      "year": 1949},
            {"title": "Dune",    "author": "Herbert",     "year": 1965},
        ]
    }
    return render(request, "app_name/index.html", context)
```

```html
<table>
  <tr>
    <th>Назва</th><th>Автор</th><th>Рік</th>
  </tr>
  {% for book in books %}
  <tr>
    <td>{{ book.title }}</td>
    <td>{{ book.author }}</td>
    <td>{{ book.year }}</td>
  </tr>
  {% endfor %}
</table>
```

---

## Статичні файли: CSS та зображення

**Статичні файли** — це файли, що не змінюються динамічно: таблиці стилів (`.css`), зображення, JavaScript. Django має вбудований механізм їх підключення.

### Структура папок

```
app_name/
└── static/
    └── app_name/           ← папка з назвою застосунку (конвенція)
        ├── css/
        │   └── style.css
        └── img/
            └── logo.png
```

### Налаштування у `settings.py`

За замовчуванням Django вже налаштований для роботи зі статикою. Перевірте наявність у `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    ...
    'django.contrib.staticfiles',  # ← має бути підключено
    'app_name',
]
```

І наявність змінної `STATIC_URL`:

```python
STATIC_URL = "/static/"   # базовий URL для статичних файлів
```

### Підключення статики у шаблоні

У верхній частині HTML-файлу завжди потрібно завантажити тег `{% load static %}`, після чого використовувати тег `{% static %}` для формування URL.

```html
<!-- app_name/templates/app_name/index.html -->

{% load static %}
<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8">
  <title>Бібліотека</title>

  <!-- Підключення CSS -->
  <link rel="stylesheet" href="{% static 'app_name/css/style.css' %}">
</head>
<body>

  <!-- Підключення зображення -->
  <img src="{% static 'app_name/img/logo.png' %}" alt="Логотип бібліотеки">

  <h1>Бібліотека</h1>

</body>
</html>
```

### Що робить `{% static %}`?

Тег `{% static 'app_name/css/style.css' %}` генерує повний URL до файлу:

```
/static/app_name/css/style.css
```

Це важливо: ніколи не пишіть шлях до статики вручну рядком — якщо `STATIC_URL` зміниться, всі посилання зламаються. `{% static %}` завжди підставить правильний шлях автоматично.

### Приклад CSS-файлу

```css
/* app_name/static/app_name/css/style.css */

body {
    font-family: Arial, sans-serif;
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
}

h1 {
    color: #2c3e50;
    border-bottom: 2px solid #3498db;
    padding-bottom: 10px;
}

table {
    width: 100%;
    border-collapse: collapse;
}

td, th {
    border: 1px solid #ccc;
    padding: 8px 12px;
    text-align: left;
}

th {
    background-color: #3498db;
    color: white;
}
```

> **Якщо статика не відображається:** зупиніть сервер (`Ctrl+C`) і запустіть знову — іноді Django не «підхоплює» нові статичні файли без перезапуску.

---

## Підсумок та шпаргалка

### Схема взаємодії всіх компонентів

```
Браузер: GET /
    │
    ▼
project_name/urls.py
    │  path("", include("app_name.urls"))
    ▼
app_name/urls.py
    │  path("", views.index, name="index")
    ▼
app_name/views.py  →  def index(request):
    │                    context = {"title": "Бібліотека", ...}
    │                    return render(request, "app_name/index.html", context)
    ▼
app_name/templates/app_name/index.html
    │  {% load static %}
    │  <link href="{% static 'app_name/css/style.css' %}">
    │  <h1>{{ title }}</h1>
    ▼
Браузер отримує готовий HTML
```

### Шпаргалка

```python
# views.py — мінімальна View
from django.shortcuts import render

def my_view(request):
    context = {"key": "value"}
    return render(request, "app/template.html", context)
```

```python
# urls.py застосунку
from django.urls import path
from . import views

urlpatterns = [
    path("",      views.index, name="index"),
    path("about/", views.about, name="about"),
]
```

```python
# project_name/urls.py — підключення застосунку
from django.urls import path, include

urlpatterns = [
    path("", include("app_name.urls")),
]
```

```html
<!-- Шаблон: обов'язкові елементи -->
{% load static %}

{{ змінна }}
{% if умова %} ... {% endif %}
{% for item in list %} ... {% endfor %}

<link rel="stylesheet" href="{% static 'app/css/style.css' %}">
<img src="{% static 'app/img/photo.png' %}" alt="...">
```

### Структура застосунку після цього уроку

```
my_project/
├── project_name/
│   ├── settings.py
│   └── urls.py
│
└── app_name/
    ├── static/
    │   └── app_name/
    │       ├── css/
    │       │   └── style.css
    │       └── img/
    │           └── logo.png
    ├── templates/
    │   └── app_name/
    │       ├── index.html
    │       └── about.html
    ├── views.py
    └── urls.py
```
