# Шаблони та Master-шаблон

1. [Теги та змінні у шаблонах](#теги-та-змінні-у-шаблонах)
2. [Умови: тег `{% if %}`](#умови-тег--if-)
3. [Цикли: тег `{% for %}`](#цикли-тег--for-)
4. [Фільтри](#фільтри)
5. [Master-шаблон: наслідування шаблонів](#master-шаблон-наслідування-шаблонів)
6. [Довідкові матеріали](#довідкові-матеріали)

---

## Теги та змінні у шаблонах

Django Templates мають два типи спеціальних конструкцій:

| Конструкція | Синтаксис | Призначення |
|---|---|---|
| **Змінна** | `{{ змінна }}` | Вивести значення змінної з контексту |
| **Тег** | `{% тег %}` | Логіка: цикли, умови, наслідування, завантаження |
| **Коментар** | `{# текст #}` | Коментар — не потрапляє у HTML |

### Змінні `{{ }}`

```html
<!-- Виведення простого значення -->
<h1>{{ title }}</h1>

<!-- Виведення вкладеного значення -->
<p>{{ author.name }}</p>

<!-- Якщо змінної немає в контексті — виводить порожній рядок -->
<p>{{ undefined_variable }}</p>
```

### Теги `{% %}`

Теги бувають одиночні та парні:

```html
<!-- Одиночний тег -->
{% load static %}

<!-- Парний тег — має закриваючу частину -->
{% block content %}
  ...
{% endblock %}

{% for item in list %}
  ...
{% endfor %}

{% if condition %}
  ...
{% endif %}
```

### Коментарі `{# #}`

```html
{# Цей рядок не потрапить у фінальний HTML #}

{# TODO: додати зображення #}
<h1>{{ title }}</h1>
```

---

## Умови: тег `{% if %}`

### Базовий синтаксис

```html
{% if умова %}
  <!-- виконується якщо умова True -->
{% elif інша_умова %}
  <!-- виконується якщо перша умова False, а ця True -->
{% else %}
  <!-- виконується якщо всі умови False -->
{% endif %}
```

### Оператори порівняння

```html
{% if age >= 18 %}
  <p>Повнолітній</p>
{% endif %}

{% if name == "Олексій" %}
  <p>Привіт, Олексій!</p>
{% endif %}

{% if score != 0 %}
  <p>Є результат</p>
{% endif %}
```

### Логічні оператори

```html
{% if age >= 18 and is_student %}
  <p>Повнолітній студент</p>
{% endif %}

{% if is_admin or is_moderator %}
  <p>Є права доступу</p>
{% endif %}

{% if not is_blocked %}
  <p>Доступ відкрито</p>
{% endif %}
```

### Перевірка наявності значення

```html
{% if user %}
  <p>Користувач: {{ user.name }}</p>
{% else %}
  <p>Гість</p>
{% endif %}

{% if skills %}
  <ul>
    {% for skill in skills %}
      <li>{{ skill }}</li>
    {% endfor %}
  </ul>
{% else %}
  <p>Навички не вказано.</p>
{% endif %}
```

### `{% if %}` всередині `{% for %}`

Умови і цикли можна вільно вкладати одне в одне:

```html
{% for project in projects %}
  <div class="
    {% if project.done %}
      project-done
    {% else %}
      project-wip
    {% endif %}
  ">
    <h3>{{ project.title }}</h3>
  </div>
{% endfor %}
```

---

## Цикли: тег `{% for %}`

### Базовий синтаксис

```html
{% for елемент in колекція %}
  <!-- тіло циклу -->
{% endfor %}
```

### Приклади

```html
<!-- Список рядків -->
{% for skill in skills %}
  <li>{{ skill }}</li>
{% endfor %}

<!-- Список словників -->
{% for project in projects %}
  <p>{{ project.title }} — {{ project.year }}</p>
{% endfor %}
```

### Тег `{% empty %}`

Виконується, якщо колекція порожня — замість окремої перевірки `{% if %}`:

```html
{% for project in projects %}
  <li>{{ project.title }}</li>
{% empty %}
  <p>Проєктів поки немає.</p>
{% endfor %}
```

### Змінна `forloop`

Django автоматично надає змінну `forloop` всередині циклу:

| Змінна | Значення |
|---|---|
| `forloop.counter` | Номер ітерації з 1 (1, 2, 3...) |
| `forloop.counter0` | Номер ітерації з 0 (0, 1, 2...) |
| `forloop.first` | `True` якщо перша ітерація |
| `forloop.last` | `True` якщо остання ітерація |
| `forloop.revcounter` | Зворотний лічильник |

```html
{% for skill in skills %}
  <p>
    {{ forloop.counter }}. {{ skill }}
    {% if forloop.first %}⭐{% endif %}
  </p>
{% endfor %}
```

Результат для `skills = ["Python", "Django", "HTML"]`:

```
1. Python ⭐
2. Django
3. HTML
```

### Вкладені цикли

```python
# views.py
context = {
    "schedule": [
        {"day": "Понеділок", "lessons": ["Математика", "Фізика"]},
        {"day": "Вівторок",  "lessons": ["Хімія", "Біологія"]},
    ]
}
```

```html
{% for day in schedule %}
  <h3>{{ day.day }}</h3>
  <ul>
    {% for lesson in day.lessons %}
      <li>{{ lesson }}</li>
    {% endfor %}
  </ul>
{% endfor %}
```

---

## Фільтри

**Фільтри** змінюють значення змінної безпосередньо у шаблоні. Застосовуються через символ `|`.

```html
{{ змінна|фільтр }}
{{ змінна|фільтр:аргумент }}
```

Фільтри можна поєднувати в ланцюжок:

```html
{{ name|lower|capfirst }}
```

### Найпоширеніші фільтри

**Текстові:**

| Фільтр | Приклад | Результат |
|---|---|---|
| `upper` | `{{ name\|upper }}` | `ОЛЕКСІЙ` |
| `lower` | `{{ name\|lower }}` | `олексій` |
| `capfirst` | `{{ name\|capfirst }}` | `Олексій` |
| `title` | `{{ name\|title }}` | `Олексій Іванов` |
| `truncatechars:N` | `{{ bio\|truncatechars:30 }}` | `Я розробник, який люби...` |
| `truncatewords:N` | `{{ bio\|truncatewords:5 }}` | `Я розробник який дуже...` |

**Числові:**

| Фільтр | Приклад | Результат |
|---|---|---|
| `floatformat:N` | `{{ price\|floatformat:2 }}` | `19.90` |
| `filesizeformat` | `{{ size\|filesizeformat }}` | `1.2 MB` |

**Списки та загальні:**

| Фільтр | Приклад | Результат |
|---|---|---|
| `length` | `{{ skills\|length }}` | `4` |
| `default:"текст"` | `{{ phone\|default:"не вказано" }}` | `не вказано` |
| `default_if_none` | `{{ value\|default_if_none:"—" }}` | `—` |
| `yesno:"так,ні"` | `{{ is_done\|yesno:"так,ні" }}` | `так` або `ні` |
| `join:", "` | `{{ skills\|join:", " }}` | `Python, Django, HTML` |
| `first` | `{{ skills\|first }}` | `Python` |
| `last` | `{{ skills\|last }}` | `CSS` |

### Практичний приклад із фільтрами

```python
# views.py
context = {
    "bio":    "я захоплююся розробкою вебзастосунків та відкритим кодом",
    "price":  19.9,
    "skills": ["python", "django", "html", "css"],
    "phone":  None,
}
```

```html
<p>{{ bio|capfirst|truncatechars:40 }}</p>
<!-- Я захоплюююся розробкою вебзасто... -->

<p>Ціна: {{ price|floatformat:2 }} грн</p>
<!-- Ціна: 19.90 грн -->

<p>Навички: {{ skills|join:", "|upper }}</p>
<!-- Навички: PYTHON, DJANGO, HTML, CSS -->

<p>Телефон: {{ phone|default:"не вказано" }}</p>
<!-- Телефон: не вказано -->
```

---

## Master-шаблон: наслідування шаблонів

### Проблема: дублювання коду

Без наслідування кожна сторінка містить однаковий код:

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>...</head>     ← однакові
  <nav>...</nav>       ← однакові
  <main>
    Головна сторінка   ← різні
  </main>
  <footer>...</footer> ← однакові
</html>
```

```html
<!-- about.html -->
<!DOCTYPE html>
<html>
  <head>...</head>     ← однакові
  <nav>...</nav>       ← однакові
  <main>
    Про мене           ← різні
  </main>
  <footer>...</footer> ← однакові
</html>
```

Якщо потрібно змінити навігацію — доводиться редагувати кожен файл окремо.

### Рішення: `base.html`

Наслідування шаблонів дозволяє винести спільну частину в один файл і перевизначати лише змінні блоки.

```
base.html
├── спільна частина (head, nav, footer) — один раз
└── {% block content %} ← «дірка» для унікального вмісту

index.html
└── {% extends "portfolio/base.html" %}
└── {% block content %} Головна {% endblock %}

about.html
└── {% extends "portfolio/base.html" %}
└── {% block content %} Про мене {% endblock %}
```

### Створення `base.html`

```html
<!-- portfolio/templates/portfolio/base.html -->

{% load static %}
<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8">
  <title>{% block title %}Мій сайт{% endblock %}</title>
  <link rel="stylesheet" href="{% static 'portfolio/css/style.css' %}">
</head>
<body>

  <nav>
    <a href="/">Головна</a>
    <a href="/about/">Про мене</a>
    <a href="/contact/">Контакти</a>
  </nav>

  <main>
    {% block content %}
    {% endblock %}
  </main>

  <footer>
    <p>© 2026 Моє портфоліо</p>
  </footer>

</body>
</html>
```

### Дочірні шаблони

```html
<!-- portfolio/templates/portfolio/index.html -->

{% extends "portfolio/base.html" %}

{% block title %}Головна — Портфоліо{% endblock %}

{% block content %}
  <h1>Привіт, я {{ name }}!</h1>
  <p>Ласкаво просимо на мій сайт.</p>
{% endblock %}
```

```html
<!-- portfolio/templates/portfolio/about.html -->

{% extends "portfolio/base.html" %}

{% block title %}Про мене{% endblock %}

{% block content %}
  <h1>Про мене</h1>
  <p>{{ bio|capfirst }}</p>
  <h2>Мої навички</h2>
  <ul>
    {% for skill in skills %}
      <li>{{ skill }}</li>
    {% empty %}
      <li>Навички не вказано.</li>
    {% endfor %}
  </ul>
{% endblock %}
```

### Правила наслідування

| Правило | Пояснення |
|---|---|
| `{% extends %}` — завжди перший рядок | Жодного тексту до нього |
| Дочірній шаблон визначає лише блоки | Увесь вміст поза `{% block %}` ігнорується |
| Блоки можна не перевизначати | Тоді використовується вміст з `base.html` |
| Блоків може бути скільки завгодно | `title`, `content`, `scripts`, `extra_css` тощо |
| `{{ block.super }}` | Вставити вміст батьківського блоку і розширити його |

### Кілька блоків

Зазвичай у `base.html` визначають кілька блоків під різні потреби:

```html
<!-- base.html -->
<head>
  <title>{% block title %}Сайт{% endblock %}</title>
  {% block extra_css %}{% endblock %}   {# місце для додаткових стилів #}
</head>
<body>
  {% block content %}{% endblock %}
  {% block extra_js %}{% endblock %}    {# місце для скриптів #}
</body>
```

```html
<!-- page.html -->
{% extends "portfolio/base.html" %}

{% block title %}Сторінка{% endblock %}

{% block extra_css %}
  <link rel="stylesheet" href="{% static 'portfolio/css/page.css' %}">
{% endblock %}

{% block content %}
  <h1>Вміст сторінки</h1>
{% endblock %}
```

### Тег `{% url %}` у шаблоні

Замість того щоб писати URL вручну, краще використовувати ім'я маршруту — тоді посилання не зламаються якщо URL зміниться:

```html
<!-- Погано — хардкодимо URL -->
<a href="/about/">Про мене</a>

<!-- Добре — використовуємо ім'я маршруту -->
<a href="{% url 'about' %}">Про мене</a>
```

`{% url 'about' %}` шукає в `urls.py` маршрут із `name="about"` і підставляє його URL автоматично.

Оновлена навігація у `base.html`:

```html
<nav>
  <a href="{% url 'index' %}">Головна</a>
  <a href="{% url 'about' %}">Про мене</a>
  <a href="{% url 'contact' %}">Контакти</a>
</nav>
```

---

## Довідкові матеріали
- Templates — https://docs.djangoproject.com/en/6.0/topics/templates/
- The Django template language — https://docs.djangoproject.com/en/6.0/ref/templates/language/
- 
- Django Add Master Template — https://w3schoolsua.github.io/django/django_master_template_en.html#gsc.tab=0