# Форми в Django

1. [Що таке форма і навіщо вона потрібна](#що-таке-форма-і-навіщо-вона-потрібна)
2. [Клас `Form` — незалежна форма](#клас-form--незалежна-форма)
3. [Клас `ModelForm` — форма на основі моделі](#клас-modelform--форма-на-основі-моделі)
4. [Обробка форми у View](#обробка-форми-у-view)
5. [Виведення форми у шаблоні](#виведення-форми-у-шаблоні)
6. [Валідація даних](#валідація-даних)
7. [Віджети](#віджети)
8. [Повний CRUD на прикладі бібліотеки](#повний-crud-на-прикладі-бібліотеки)
9. [Довідкові матеріали](#довідкові-матеріали)

---

## Що таке форма і навіщо вона потрібна

До цього моменту сторінки лише відображали дані. Форми дозволяють користувачеві **надсилати дані** на сервер: створювати записи, редагувати їх, виконувати пошук.

Django надає клас `Form`, який вирішує три задачі одразу:

- генерує HTML-розмітку полів введення
- валідує отримані дані (перевіряє типи, обмеження, обов'язковість)
- повертає очищені дані у зручному форматі Python

### GET vs POST

Форми можуть надсилати дані двома методами:

| Метод | Коли використовується | Де передаються дані |
|---|---|---|
| `GET` | Пошук, фільтрація — дані не змінюють БД | У рядку URL: `?q=python` |
| `POST` | Створення, редагування, видалення | У тілі запиту (не видно в URL) |

```html
<!-- GET-форма — для пошуку -->
<form method="get">
  <input name="q" placeholder="Пошук...">
  <button type="submit">Знайти</button>
</form>

<!-- POST-форма — для збереження даних -->
<form method="post">
  {% csrf_token %}
  ...
</form>
```

### CSRF-захист

Будь-яка POST-форма в Django **обов'язково** повинна містити тег `{% csrf_token %}`. Без нього Django поверне помилку 403.

**CSRF (Cross-Site Request Forgery)** — вид атаки, коли зловмисний сайт надсилає запит від імені авторизованого користувача. Токен підтверджує, що запит надійшов саме з вашого сайту.

```html
<form method="post">
  {% csrf_token %}  {# генерує приховане поле з унікальним токеном #}
  ...
</form>
```

### Як Django обробляє форму: загальна схема

```
Користувач відкрив сторінку (GET)
         │
         ▼
  View: form = BookForm()         ← порожня форма
  render() → HTML з порожньою формою
         │
  Користувач заповнив і натиснув «Зберегти» (POST)
         │
         ▼
  View: form = BookForm(request.POST)
         │
         ├── form.is_valid() == False
         │         │
         │         ▼
         │   render() → форма з повідомленнями про помилки
         │
         └── form.is_valid() == True
                   │
                   ▼
             form.save() або обробка cleaned_data
             redirect() → інша сторінка
```

---

## Клас `Form` — незалежна форма

`Form` — це форма, поля якої описуються вручну і не пов'язані з жодною моделлю. Підходить для пошуку, фільтрів, форм зворотного зв'язку.

### Створення файлу `forms.py`

За конвенцією форми описуються у файлі `forms.py` всередині застосунку:

```
app_name/
├── forms.py      ← форми
├── models.py
├── views.py
└── urls.py
```

### Опис форми

```python
# app_name/forms.py

from django import forms


class BookForm(forms.Form):
    title       = forms.CharField(max_length=200, label="Назва")
    author      = forms.CharField(max_length=200, label="Автор")
    year        = forms.IntegerField(label="Рік видання")
    description = forms.CharField(
        widget=forms.Textarea,
        required=False,
        label="Опис",
    )
    is_available = forms.BooleanField(
        required=False,
        label="Доступна",
        initial=True,
    )
```

### Параметри полів форми

| Параметр | Тип | Призначення |
|---|---|---|
| `label` | `str` | Підпис поля у HTML |
| `required` | `bool` | Чи є поле обов'язковим (за замовч. `True`) |
| `initial` | значення | Початкове значення |
| `help_text` | `str` | Підказка під полем |
| `widget` | `Widget` | Який HTML-елемент генерувати |
| `error_messages` | `dict` | Кастомні повідомлення про помилки |
| `validators` | `list` | Додаткові функції валідації |

### Основні типи полів `Form`

| Поле | HTML-елемент | Тип `cleaned_data` |
|---|---|---|
| `CharField` | `<input type="text">` | `str` |
| `IntegerField` | `<input type="number">` | `int` |
| `FloatField` | `<input type="number">` | `float` |
| `DecimalField` | `<input type="number">` | `Decimal` |
| `BooleanField` | `<input type="checkbox">` | `bool` |
| `EmailField` | `<input type="email">` | `str` |
| `URLField` | `<input type="url">` | `str` |
| `DateField` | `<input type="text">` | `date` |
| `DateTimeField` | `<input type="text">` | `datetime` |
| `ChoiceField` | `<select>` | `str` |
| `MultipleChoiceField` | `<select multiple>` | `list` |
| `FileField` | `<input type="file">` | `File` |
| `ImageField` | `<input type="file">` | `Image` |
| `SlugField` | `<input type="text">` | `str` |
| `TypedChoiceField` | `<select>` | вказаний тип |

> Повний перелік полів з описом параметрів: [docs.djangoproject.com — Form fields](https://docs.djangoproject.com/en/stable/ref/forms/fields/)

---

## Клас `ModelForm` — форма на основі моделі

`ModelForm` генерує поля форми автоматично на основі полів моделі. Це найпоширеніший підхід при роботі з базою даних, оскільки дозволяє уникнути дублювання: поля описані один раз у моделі.

### Порівняння з `Form`

```python
# Form — поля дублюються з моделі вручну
class BookForm(forms.Form):
    title  = forms.CharField(max_length=200)
    author = forms.CharField(max_length=200)
    year   = forms.IntegerField()


# ModelForm — поля генеруються автоматично
class BookModelForm(forms.ModelForm):
    class Meta:
        model  = Book
        fields = ["title", "author", "year"]
```

### Клас `Meta`

Вкладений клас `Meta` керує тим, яка модель використовується і які поля включаються:

```python
# app_name/forms.py

from django import forms
from .models import Book


class BookForm(forms.ModelForm):
    class Meta:
        model  = Book
        fields = ["title", "author", "year", "description", "is_available"]
```

**Два підходи до вибору полів:**

```python
# Явно вказати потрібні поля (рекомендовано)
fields = ["title", "author", "year"]

# Або виключити непотрібні (всі інші будуть включені)
exclude = ["created_at", "updated_at"]
```

> Рекомендується завжди використовувати `fields` явно, а не `exclude` — щоб нові поля моделі не потрапляли у форму автоматично.

### Налаштування `Meta`: `labels`, `help_texts`, `error_messages`

```python
class BookForm(forms.ModelForm):
    class Meta:
        model  = Book
        fields = ["title", "author", "year", "description", "is_available"]
        labels = {
            "title":        "Назва книги",
            "author":       "Автор",
            "year":         "Рік видання",
            "description":  "Короткий опис",
            "is_available": "Доступна для читання",
        }
        help_texts = {
            "year": "Введіть рік у форматі РРРР",
        }
        error_messages = {
            "title": {
                "required": "Назва книги є обов'язковим полем.",
                "max_length": "Назва не може перевищувати 200 символів.",
            }
        }
```

### Метод `save()`

Головна перевага `ModelForm` — метод `save()`, який одразу зберігає дані в базу:

```python
form = BookForm(request.POST)
if form.is_valid():
    book = form.save()          # створює новий запис і повертає об'єкт
    # або
    book = form.save(commit=False)  # створює об'єкт, але НЕ зберігає в БД
    book.some_field = "value"       # можна змінити поля перед збереженням
    book.save()                     # тепер зберегти
```

`commit=False` корисний коли потрібно заповнити поля, яких немає у формі (наприклад, автор запису — поточний користувач).

---

## Обробка форми у View

### Стандартний патерн — створення запису

```python
# app_name/views.py

from django.shortcuts import render, redirect, get_object_or_404
from .forms import BookForm
from .models import Book


def book_create(request):
    if request.method == "POST":
        form = BookForm(request.POST)    # форма з даними від користувача
        if form.is_valid():              # перевірка валідності
            form.save()                  # збереження в БД
            return redirect("app_name:list")
    else:                                # GET-запит
        form = BookForm()                # порожня форма

    return render(request, "app_name/book_form.html", {"form": form})
```

### Стандартний патерн — редагування запису

```python
def book_edit(request, pk):
    book = get_object_or_404(Book, pk=pk)

    if request.method == "POST":
        form = BookForm(request.POST, instance=book)   # форма з поточними даними
        if form.is_valid():
            form.save()
            return redirect("app_name:detail", pk=book.pk)
    else:
        form = BookForm(instance=book)   # заповнена поточними даними

    return render(request, "app_name/book_form.html", {"form": form, "book": book})
```

Аргумент `instance=book` — ключовий: він наповнює форму поточними значеннями (GET) і оновлює саме цей запис при збереженні (POST).

### Стандартний патерн — видалення запису

```python
def book_delete(request, pk):
    book = get_object_or_404(Book, pk=pk)
    if request.method == "POST":
        book.delete()
        return redirect("app_name:list")
    return render(request, "app_name/book_confirm_delete.html", {"book": book})
```

### Обробка `Form` (не `ModelForm`)

Коли форма не пов'язана з моделлю, дані беруться вручну через `cleaned_data`:

```python
def book_search(request):
    results = []
    form = SearchForm(request.GET)   # GET-форма — дані в URL

    if form.is_valid():
        query = form.cleaned_data["query"]
        results = Book.objects.filter(title__icontains=query)

    return render(request, "app_name/search.html", {
        "form": form,
        "results": results,
    })
```

`cleaned_data` — словник із валідованими даними у правильних Python-типах. Доступний лише після успішного `is_valid()`.

---

## Виведення форми у шаблоні

### Автоматичне виведення

Django вміє рендерити форму автоматично трьома способами:

```html
{{ form.as_p }}      {# кожне поле у тегу <p> #}
{{ form.as_div }}    {# кожне поле у тегу <div> — рекомендований з Django 4.1 #}
{{ form.as_table }}  {# поля у рядках <tr> таблиці (потрібен <table> зовні) #}
{{ form.as_ul }}     {# поля у тегах <li> #}
```

Повний шаблон із `as_div`:

```html
<!-- app_name/templates/app_name/book_form.html -->

{% extends "app_name/base.html" %}

{% block title %}
  {% if book %}Редагувати книгу{% else %}Додати книгу{% endif %}
{% endblock %}

{% block content %}
  <h1>
    {% if book %}Редагувати: {{ book.title }}{% else %}Додати книгу{% endif %}
  </h1>

  <form method="post">
    {% csrf_token %}
    {{ form.as_div }}
    <button type="submit">Зберегти</button>
    <a href="{% url 'app_name:list' %}">Скасувати</a>
  </form>
{% endblock %}
```

### Виведення полів вручну

Дає повний контроль над розміткою:

```html
<form method="post">
  {% csrf_token %}

  {# Загальні помилки форми (не пов'язані з конкретним полем) #}
  {% if form.non_field_errors %}
    <div class="errors">
      {{ form.non_field_errors }}
    </div>
  {% endif %}

  {# Поле title #}
  <div class="field {% if form.title.errors %}field--error{% endif %}">
    <label for="{{ form.title.id_for_label }}">{{ form.title.label }}</label>
    {{ form.title }}
    {% if form.title.help_text %}
      <small>{{ form.title.help_text }}</small>
    {% endif %}
    {% for error in form.title.errors %}
      <span class="error-msg">{{ error }}</span>
    {% endfor %}
  </div>

  {# Поле author #}
  <div class="field">
    <label for="{{ form.author.id_for_label }}">{{ form.author.label }}</label>
    {{ form.author }}
    {{ form.author.errors }}
  </div>

  {# Поле year #}
  <div class="field">
    <label for="{{ form.year.id_for_label }}">{{ form.year.label }}</label>
    {{ form.year }}
    {{ form.year.errors }}
  </div>

  <button type="submit">Зберегти</button>
</form>
```

### Корисні атрибути поля у шаблоні

| Змінна | Що містить |
|---|---|
| `{{ form.field_name }}` | HTML-елемент поля |
| `{{ form.field_name.label }}` | Підпис поля |
| `{{ form.field_name.errors }}` | Список помилок |
| `{{ form.field_name.help_text }}` | Підказка |
| `{{ form.field_name.id_for_label }}` | `id` поля для атрибута `for` у `<label>` |
| `{{ form.field_name.value }}` | Поточне значення |
| `{{ form.non_field_errors }}` | Помилки форми загалом |

### Перебір полів у циклі

```html
<form method="post">
  {% csrf_token %}
  {% for field in form %}
    <div>
      {{ field.label_tag }}  {# <label for="...">Назва</label> #}
      {{ field }}
      {{ field.errors }}
    </div>
  {% endfor %}
  <button type="submit">Зберегти</button>
</form>
```

### Шаблон підтвердження видалення

```html
<!-- app_name/templates/app_name/book_confirm_delete.html -->

{% extends "app_name/base.html" %}

{% block content %}
  <h1>Видалити книгу?</h1>
  <p>Ви впевнені, що хочете видалити <strong>«{{ book.title }}»</strong>?</p>
  <p>Цю дію неможливо скасувати.</p>

  <form method="post">
    {% csrf_token %}
    <button type="submit">Так, видалити</button>
    <a href="{% url 'app_name:detail' book.pk %}">Скасувати</a>
  </form>
{% endblock %}
```

---

## Валідація даних

### Вбудована валідація

Django автоматично перевіряє:

- тип даних (`IntegerField` приймає лише числа)
- обов'язковість (`required=True` за замовчуванням)
- обмеження (`max_length`, `min_value`, `max_value` тощо)

### Валідація окремого поля: `clean_<fieldname>()`

Метод викликається автоматично під час `is_valid()`:

```python
class BookForm(forms.ModelForm):
    class Meta:
        model  = Book
        fields = ["title", "author", "year"]

    def clean_year(self):
        year = self.cleaned_data["year"]
        if year < 868:     # рік першої відомої друкованої книги
            raise forms.ValidationError("Введіть коректний рік видання.")
        if year > 2025:
            raise forms.ValidationError("Рік не може бути у майбутньому.")
        return year        # обов'язково повернути значення

    def clean_title(self):
        title = self.cleaned_data["title"]
        return title.strip()   # можна модифікувати — прибрати зайві пробіли
```

### Валідація кількох полів: `clean()`

```python
    def clean(self):
        cleaned_data = super().clean()           # обов'язково викликати super()
        title  = cleaned_data.get("title", "")
        author = cleaned_data.get("author", "")

        if title.lower() == author.lower():
            raise forms.ValidationError(
                "Назва книги не може збігатися з іменем автора."
            )
        return cleaned_data
```

### Валідатори — окремі функції

Валідатори можна виносити у функції і перевикористовувати:

```python
from django.core.exceptions import ValidationError


def validate_year(value):
    if value < 868 or value > 2025:
        raise ValidationError(f"Рік {value} є некоректним.")


class BookForm(forms.ModelForm):
    year = forms.IntegerField(validators=[validate_year])

    class Meta:
        model  = Book
        fields = ["title", "author", "year"]
```

---

## Віджети

**Віджет** — це клас, який визначає який HTML-елемент генерується для поля. Кожне поле має віджет за замовчуванням, але його можна замінити.

### Вбудовані віджети

| Віджет | HTML-елемент | Використовується за замовч. у |
|---|---|---|
| `TextInput` | `<input type="text">` | `CharField` |
| `NumberInput` | `<input type="number">` | `IntegerField`, `FloatField` |
| `EmailInput` | `<input type="email">` | `EmailField` |
| `URLInput` | `<input type="url">` | `URLField` |
| `PasswordInput` | `<input type="password">` | — |
| `HiddenInput` | `<input type="hidden">` | — |
| `DateInput` | `<input type="text">` | `DateField` |
| `DateTimeInput` | `<input type="text">` | `DateTimeField` |
| `TimeInput` | `<input type="text">` | `TimeField` |
| `Textarea` | `<textarea>` | `TextField` у ModelForm |
| `CheckboxInput` | `<input type="checkbox">` | `BooleanField` |
| `Select` | `<select>` | `ChoiceField` |
| `SelectMultiple` | `<select multiple>` | `MultipleChoiceField` |
| `RadioSelect` | `<input type="radio">` | — |
| `CheckboxSelectMultiple` | кілька `<input type="checkbox">` | — |
| `FileInput` | `<input type="file">` | `FileField` |
| `ClearableFileInput` | `<input type="file">` + кнопка очистки | `ImageField` |

> Повний перелік віджетів з описом атрибутів: [docs.djangoproject.com — Widgets](https://docs.djangoproject.com/en/stable/ref/forms/widgets/)

### Налаштування віджету через `attrs`

```python
class BookForm(forms.ModelForm):
    class Meta:
        model  = Book
        fields = ["title", "author", "year", "description"]
        widgets = {
            "title": forms.TextInput(attrs={
                "placeholder": "Наприклад: Кобзар",
                "class":       "form-input",
                "autofocus":   True,
            }),
            "description": forms.Textarea(attrs={
                "rows":        5,
                "placeholder": "Короткий опис книги...",
                "class":       "form-textarea",
            }),
            "year": forms.NumberInput(attrs={
                "min": 868,
                "max": 2025,
            }),
        }
```

### Зміна віджету у полі форми

```python
class BookForm(forms.ModelForm):
    # Замінюємо ChoiceField з Select на RadioSelect
    genre = forms.ChoiceField(
        choices=Book.GENRE_CHOICES,
        widget=forms.RadioSelect,
    )

    class Meta:
        model  = Book
        fields = ["title", "author", "genre"]
```

### GET-форма з пошуком

```python
# app_name/forms.py

class BookSearchForm(forms.Form):
    query = forms.CharField(
        required=False,
        label="",
        widget=forms.TextInput(attrs={
            "placeholder": "Пошук за назвою або автором...",
            "class":       "search-input",
        }),
    )
```

```python
# app_name/views.py

def book_list(request):
    form    = BookSearchForm(request.GET)
    books   = Book.objects.all()

    if form.is_valid():
        query = form.cleaned_data.get("query", "")
        if query:
            books = books.filter(
                title__icontains=query
            ) | books.filter(
                author__icontains=query
            )

    return render(request, "app_name/book_list.html", {
        "form":  form,
        "books": books,
    })
```

```html
<!-- Пошукова форма у шаблоні -->
<form method="get">
  {{ form.query }}
  <button type="submit">Знайти</button>
  {% if request.GET.query %}
    <a href="{% url 'app_name:list' %}">Скинути</a>
  {% endif %}
</form>
```

---

## Повний CRUD на прикладі бібліотеки

### Структура проєкту

```
project_name/
├── project_name/
│   ├── settings.py
│   └── urls.py
│
└── app_name/
    ├── templates/
    │   └── app_name/
    │       ├── base.html
    │       ├── book_list.html
    │       ├── book_detail.html
    │       ├── book_form.html
    │       └── book_confirm_delete.html
    ├── forms.py
    ├── models.py
    ├── views.py
    └── urls.py
```

### `models.py`

```python
# app_name/models.py

from django.db import models


class Book(models.Model):
    GENRE_CHOICES = [
        ("fiction",     "Художня"),
        ("non_fiction", "Науково-популярна"),
        ("poetry",      "Поезія"),
        ("children",    "Дитяча"),
    ]

    title        = models.CharField(max_length=200, verbose_name="Назва")
    author       = models.CharField(max_length=200, verbose_name="Автор")
    year         = models.IntegerField(verbose_name="Рік")
    genre        = models.CharField(
                       max_length=20,
                       choices=GENRE_CHOICES,
                       default="fiction",
                       verbose_name="Жанр",
                   )
    description  = models.TextField(blank=True, verbose_name="Опис")
    is_available = models.BooleanField(default=True, verbose_name="Доступна")

    class Meta:
        ordering         = ["-year"]
        verbose_name     = "Книга"
        verbose_name_plural = "Книги"

    def __str__(self):
        return f"{self.title} — {self.author}"
```

### `forms.py`

```python
# app_name/forms.py

from django import forms
from .models import Book


class BookForm(forms.ModelForm):
    class Meta:
        model  = Book
        fields = ["title", "author", "year", "genre", "description", "is_available"]
        labels = {
            "title":        "Назва книги",
            "author":       "Автор",
            "year":         "Рік видання",
            "genre":        "Жанр",
            "description":  "Опис",
            "is_available": "Доступна для читання",
        }
        widgets = {
            "title":       forms.TextInput(attrs={"placeholder": "Введіть назву"}),
            "author":      forms.TextInput(attrs={"placeholder": "Прізвище Ім'я"}),
            "description": forms.Textarea(attrs={"rows": 4}),
        }

    def clean_year(self):
        year = self.cleaned_data["year"]
        if year < 868 or year > 2025:
            raise forms.ValidationError("Введіть коректний рік (868–2025).")
        return year


class BookSearchForm(forms.Form):
    query = forms.CharField(
        required=False,
        label="",
        widget=forms.TextInput(attrs={"placeholder": "Пошук за назвою або автором..."}),
    )
```

### `views.py`

```python
# app_name/views.py

from django.shortcuts import render, redirect, get_object_or_404
from .models import Book
from .forms  import BookForm, BookSearchForm


def book_list(request):
    form  = BookSearchForm(request.GET)
    books = Book.objects.all()

    if form.is_valid():
        query = form.cleaned_data.get("query", "")
        if query:
            books = books.filter(title__icontains=query) \
                  | books.filter(author__icontains=query)

    return render(request, "app_name/book_list.html", {
        "books": books,
        "form":  form,
    })


def book_detail(request, pk):
    book = get_object_or_404(Book, pk=pk)
    return render(request, "app_name/book_detail.html", {"book": book})


def book_create(request):
    if request.method == "POST":
        form = BookForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("app_name:list")
    else:
        form = BookForm()
    return render(request, "app_name/book_form.html", {"form": form})


def book_edit(request, pk):
    book = get_object_or_404(Book, pk=pk)
    if request.method == "POST":
        form = BookForm(request.POST, instance=book)
        if form.is_valid():
            form.save()
            return redirect("app_name:detail", pk=book.pk)
    else:
        form = BookForm(instance=book)
    return render(request, "app_name/book_form.html", {
        "form": form,
        "book": book,
    })


def book_delete(request, pk):
    book = get_object_or_404(Book, pk=pk)
    if request.method == "POST":
        book.delete()
        return redirect("app_name:list")
    return render(request, "app_name/book_confirm_delete.html", {"book": book})
```

### `urls.py`

```python
# app_name/urls.py

from django.urls import path
from . import views

app_name = "app_name"

urlpatterns = [
    path("",                 views.book_list,    name="list"),
    path("<int:pk>/",        views.book_detail,  name="detail"),
    path("new/",             views.book_create,  name="create"),
    path("<int:pk>/edit/",   views.book_edit,    name="edit"),
    path("<int:pk>/delete/", views.book_delete,  name="delete"),
]
```

```python
# project_name/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("books/", include("app_name.urls")),
]
```

### Шаблони

```html
<!-- app_name/templates/app_name/book_list.html -->

{% extends "app_name/base.html" %}

{% block content %}
  <h1>Каталог книг</h1>

  <form method="get">
    {{ form.query }}
    <button type="submit">Знайти</button>
    {% if request.GET.query %}
      <a href="{% url 'app_name:list' %}">Скинути</a>
    {% endif %}
  </form>

  <a href="{% url 'app_name:create' %}">+ Додати книгу</a>

  {% for book in books %}
    <div>
      <h3><a href="{% url 'app_name:detail' book.pk %}">{{ book.title }}</a></h3>
      <p>{{ book.author }}, {{ book.year }} · {{ book.get_genre_display }}</p>
      {% if not book.is_available %}<span>Недоступна</span>{% endif %}
    </div>
  {% empty %}
    <p>Книг не знайдено.</p>
  {% endfor %}
{% endblock %}
```

```html
<!-- app_name/templates/app_name/book_detail.html -->

{% extends "app_name/base.html" %}

{% block title %}{{ book.title }}{% endblock %}

{% block content %}
  <h1>{{ book.title }}</h1>
  <p><strong>Автор:</strong> {{ book.author }}</p>
  <p><strong>Рік:</strong> {{ book.year }}</p>
  <p><strong>Жанр:</strong> {{ book.get_genre_display }}</p>
  <p><strong>Опис:</strong> {{ book.description|default:"Опис відсутній" }}</p>
  <p><strong>Доступність:</strong> {{ book.is_available|yesno:"Доступна,Недоступна" }}</p>

  <a href="{% url 'app_name:edit'   book.pk %}">Редагувати</a>
  <a href="{% url 'app_name:delete' book.pk %}">Видалити</a>
  <a href="{% url 'app_name:list' %}">← До каталогу</a>
{% endblock %}
```

---

## Довідкові матеріали

### Офіційна документація Django

- [Форми у Django — загальний огляд (EN)](https://docs.djangoproject.com/en/stable/topics/forms/) — як працюють форми, повний цикл обробки
- [Усі типи полів форм (EN)](https://docs.djangoproject.com/en/stable/ref/forms/fields/) — повний перелік полів із параметрами та прикладами
- [Усі вбудовані віджети (EN)](https://docs.djangoproject.com/en/stable/ref/forms/widgets/) — повний перелік віджетів із описом атрибутів
- [ModelForm — детальна документація (EN)](https://docs.djangoproject.com/en/stable/topics/forms/modelforms/) — `Meta`, `save()`, `instance`, `commit=False`
- [Валідація форм (EN)](https://docs.djangoproject.com/en/stable/ref/forms/validation/) — `clean()`, `clean_<field>()`, валідатори
- [CSRF-захист (EN)](https://docs.djangoproject.com/en/stable/ref/csrf/) — як працює токен і чому він необхідний