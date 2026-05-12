# Встановлення та запуск веб-додатку на Django

- [Встановлення та запуск веб-додатку на Django](#встановлення-та-запуск-веб-додатку-на-django)
  - [Встановлення Python](#встановлення-python)
    - [Windows](#windows)
    - [macOS](#macos)
    - [Linux (Ubuntu / Debian)](#linux-ubuntu--debian)
    - [Перевірка встановлення](#перевірка-встановлення)
  - [Створення віртуального середовища](#створення-віртуального-середовища)
  - [Встановлення Django](#встановлення-django)
  - [Створення проєкту](#створення-проєкту)
  - [Створення застосунку](#створення-застосунку)
    - [Підключення застосунку до проєкту](#підключення-застосунку-до-проєкту)
  - [Структура проєкту](#структура-проєкту)
  - [Налаштування бази даних і міграції](#налаштування-бази-даних-і-міграції)
    - [Застосуйте початкові міграції](#застосуйте-початкові-міграції)
    - [Створення нових міграцій](#створення-нових-міграцій)
    - [Створення суперкористувача (адміністратора)](#створення-суперкористувача-адміністратора)
  - [Запуск сервера розробки](#запуск-сервера-розробки)
    - [Запуск на іншому порті](#запуск-на-іншому-порті)
    - [Запуск із доступом з інших пристроїв у мережі](#запуск-із-доступом-з-інших-пристроїв-у-мережі)
  - [Шпаргалка команд](#шпаргалка-команд)
  
## Встановлення Python

### Windows

1. Перейдіть на [python.org/downloads](https://www.python.org/downloads/)
2. Завантажте останній стабільний інсталятор
3. **Обов'язково** поставте галочку ☑ **«Add Python to PATH»** перед встановленням
4. Натисніть «Install Now»

### macOS

```bash
# Через Homebrew (рекомендовано)
brew install python
```

або завантажте інсталятор з [python.org](https://www.python.org/downloads/macos/).

### Linux (Ubuntu / Debian)

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

### Перевірка встановлення

```bash
python --version
# Python 3.12.x

pip --version
# pip 24.x from ...
```

---

## Створення віртуального середовища

Віртуальне середовище (venv) ізолює залежності вашого проєкту від системного Python. Це дозволяє мати різні версії Django в різних проєктах.

Для створення віртуального середовища перейдіть у папку, призначену для проєкту, та введіть у командний рядок наступну команду:
```bash
# Windows
python -m venv venv

# macOS / Linux
python3 -m venv venv
```

Далі активуйте середовище, щоб працювати саме через нього.

```bash
# Windows (Command Prompt)
venv\Scripts\activate.bat

# Windows (PowerShell)
venv\Scripts\Activate.ps1

# macOS / Linux
source venv/bin/activate
```

Після активації у терміналі з'явиться префікс `(venv)`:

```
(venv) C:\my_django_project>
```

> ⚠️ **Важливо:** активуйте venv щоразу, коли відкриваєте новий термінал для роботи з проєктом.

Якщо необхідно деактивувати віртуальне середовище, можна скористатися командою:

```bash
deactivate
```

---

## Встановлення Django

Переконайтесь, що `(venv)` активовано, і виконайте:

```bash
pip install django
```

Перевірити версію Django можна наступною командою:

```bash
django-admin --version
# 5.x.x
```

Бажано зберегайте залежності проєкту після кожного доданого модуля для більш простого розгортання проєкту на інших пристроях:

```bash
pip freeze > requirements.txt
```

Файл `requirements.txt` дозволяє іншим розробникам відтворити ваше середовище однією командою:

```bash
pip install -r requirements.txt
```

---

## Створення проєкту

Для завантаження всіх базових файлів, необхідних для роботи Django, введіть наступну команду:

```bash
django-admin startproject config .
```

> Крапка `.` в кінці — обов'язкова! Вона вказує Django створити файли проєкту в поточній папці, а не у вкладеній.

Після виконання структура папки виглядатиме так:

```
my_django_project/
├── config/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py      ← головні налаштування
│   ├── urls.py          ← кореневий маршрутизатор
│   └── wsgi.py
├── venv/
└── manage.py            ← головний інструмент управління
```

---

## Створення застосунку

Django-проєкт може містити кілька застосунків (apps). Кожен застосунок відповідає за окрему функціональну область.

```bash
python manage.py startapp library
```

Після цього з'явиться папка `library/`:

```
library/
├── migrations/
│   └── __init__.py
├── __init__.py
├── admin.py
├── apps.py
├── models.py       ← Model (дані)
├── tests.py
├── urls.py         ← створіть вручну
└── views.py        ← View (логіка обробки запитів)
```

### Підключення застосунку до проєкту

Відкрийте `config/settings.py` і додайте назву вашого застосунку до `INSTALLED_APPS`:

```python
# config/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'library',  # ← додайте цей рядок
]
```

---

## Структура проєкту

Повна структура після виконання попередніх кроків:

```
my_django_project/
│
├── config/                  ← налаштування проєкту
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
├── library/                 ← ваш застосунок
│   ├── migrations/          ← автозгенеровані міграції БД
│   ├── templates/           ← HTML-шаблони (створіть вручну)
│   │   └── library/
│   │       └── index.html
│   ├── models.py
│   ├── views.py
│   └── urls.py
│
├── venv/                    ← віртуальне середовище (не додавати в git!)
├── manage.py
└── requirements.txt
```

> **Порада:** додайте `venv/` до файлу `.gitignore`, щоб не завантажувати його в репозиторій.

Приклад `.gitignore`:

```
venv/
__pycache__/
*.pyc
db.sqlite3
.env
```

---

## Налаштування бази даних і міграції

За замовчуванням Django використовує **SQLite** — файлова база даних, яка не потребує додаткового встановлення.

### Застосуйте початкові міграції

```bash
python manage.py migrate
```

Ця команда створює файл `db.sqlite3` і базові таблиці Django (користувачі, сесії тощо).

### Створення нових міграцій 

Після змін у `models.py` — створіть і застосуйте нову міграцію

```bash
# Створити файл міграції
python manage.py makemigrations

# Застосувати міграцію до БД
python manage.py migrate
```

> **Правило:** будь-яка зміна в `models.py` → `makemigrations` → `migrate`.

### Створення суперкористувача (адміністратора)

```bash
python manage.py createsuperuser
```

Вам буде запропоновано ввести логін, email і пароль. Після запуску сервера адмін-панель доступна за адресою `http://127.0.0.1:8000/admin/`.

---

## Запуск сервера розробки

Для запуску сервера розробки скористайтеся наступною командою:

```bash
python manage.py runserver
```

Очікуваний вивід:

```
Django version 5.x.x, using settings 'config.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

Відкрийте браузер і перейдіть за адресою:

```
http://127.0.0.1:8000/
```

Ви побачите стартову сторінку Django із ракетою 🚀 — це означає, що все працює коректно.

### Запуск на іншому порті

```bash
python manage.py runserver 8080
```

### Запуск із доступом з інших пристроїв у мережі

```bash
python manage.py runserver 0.0.0.0:8000
```

> ⚠️ Сервер розробки (`runserver`) призначений **тільки для локальної розробки**. Для продакшн-розгортання використовують Gunicorn + Nginx.

Для зупинки сервера у терміналі натисніть:

```
Ctrl + C
```

---

## Шпаргалка команд

```bash
# Віртуальне середовище
python -m venv venv                  # створити
source venv/bin/activate             # активувати (macOS/Linux)
venv\Scripts\activate.bat            # активувати (Windows)
deactivate                           # деактивувати

# Django
pip install django                   # встановити Django
django-admin startproject config .   # створити проєкт
python manage.py startapp library    # створити застосунок
python manage.py runserver           # запустити сервер

# База даних
python manage.py makemigrations      # створити міграцію
python manage.py migrate             # застосувати міграції
python manage.py createsuperuser     # створити адміністратора

# Залежності
pip freeze > requirements.txt        # зберегти залежності
pip install -r requirements.txt      # встановити із файлу
```

