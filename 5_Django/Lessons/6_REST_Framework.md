# Django REST Framework

1. [Що таке API і навіщо воно потрібне](#що-таке-api-і-навіщо-воно-потрібне)
2. [Встановлення та налаштування DRF](#встановлення-та-налаштування-drf)
3. [Серіалізатори](#серіалізатори)
4. [API Views](#api-views)
5. [Маршрутизація API](#маршрутизація-api)
6. [Permissions та Authentication](#permissions-та-authentication)
7. [Browsable API](#browsable-api)
8. [Повний приклад: API бібліотеки](#повний-приклад-api-бібліотеки)
9. [Довідкові матеріали](#довідкові-матеріали)

---

## Що таке API і навіщо воно потрібне

**API (Application Programming Interface)** — це інтерфейс, через який одна програма спілкується з іншою. Веб-API дозволяє клієнту (браузер, мобільний застосунок, інший сервер) отримувати та змінювати дані на сервері через HTTP-запити.

### Різниця між звичайним Django-проєктом та API

| Звичайний Django | Django API |
|---|---|
| Повертає готовий HTML | Повертає дані у форматі JSON |
| Шаблони рендеряться на сервері | Шаблони рендеряться на клієнті |
| Один тип клієнта — браузер | Будь-який клієнт: браузер, iOS, Android |
| View повертає `render()` | View повертає `Response(data)` |

### REST — підхід до побудови API

**REST (Representational State Transfer)** — архітектурний стиль, де кожен ресурс (книга, користувач, замовлення) має свій URL, а дії над ним виражаються HTTP-методами:

| HTTP-метод | Дія | Приклад |
|---|---|---|
| `GET` | Отримати дані | `GET /api/books/` — список книг |
| `POST` | Створити запис | `POST /api/books/` — нова книга |
| `PUT` | Замінити запис повністю | `PUT /api/books/1/` |
| `PATCH` | Оновити частково | `PATCH /api/books/1/` |
| `DELETE` | Видалити запис | `DELETE /api/books/1/` |

### Що таке Django REST Framework

**Django REST Framework (DRF)** — бібліотека, яка розширює Django і значно спрощує створення REST API. Надає:

- **Серіалізатори** — перетворення Python-об'єктів ↔ JSON
- **API Views** — обробники запитів з вбудованою валідацією
- **Роутери** — автоматична генерація URL
- **Browsable API** — зручний веб-інтерфейс для тестування
- **Permissions** — гнучка система прав доступу
- **Authentication** — вбудована підтримка токенів, сесій, JWT

---

## Встановлення та налаштування DRF

### Встановлення

```bash
pip install djangorestframework
```

### Підключення у `settings.py`

```python
# project_name/settings.py

INSTALLED_APPS = [
    ...
    'rest_framework',   # ← додати
    'app_name',
]
```

### Глобальні налаштування DRF (необов'язково)

```python
# project_name/settings.py

REST_FRAMEWORK = {
    # Формат відповіді за замовчуванням
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
    ],
    # Пагінація за замовчуванням
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
    # Права доступу за замовчуванням
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',
    ],
}
```

---

## Серіалізатори

**Серіалізатор (Serializer)** — це клас, який перетворює складні Python-об'єкти (моделі Django) у прості типи даних (словники), які потім легко конвертуються в JSON. І навпаки — валідує вхідні JSON-дані і перетворює їх у Python-об'єкти.

```
Модель Django  →  Serializer  →  JSON     (відповідь клієнту)
JSON           →  Serializer  →  Модель   (збереження в БД)
```

### `Serializer` — базовий клас

Поля описуються вручну, аналогічно до `forms.Form`:

```python
# app_name/serializers.py

from rest_framework import serializers


class BookSerializer(serializers.Serializer):
    id     = serializers.IntegerField(read_only=True)
    title  = serializers.CharField(max_length=200)
    author = serializers.CharField(max_length=200)
    year   = serializers.IntegerField()

    def create(self, validated_data):
        return Book.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.title  = validated_data.get('title',  instance.title)
        instance.author = validated_data.get('author', instance.author)
        instance.year   = validated_data.get('year',   instance.year)
        instance.save()
        return instance
```

### `ModelSerializer` — серіалізатор на основі моделі

Найпоширеніший підхід — аналог `ModelForm`, але для API:

```python
# app_name/serializers.py

from rest_framework import serializers
from .models import Book


class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model  = Book
        fields = ['id', 'title', 'author', 'year', 'genre', 'is_available']
        # або fields = '__all__'   — усі поля
        # або exclude = ['created_at']
```

### Параметри полів серіалізатора

```python
class BookSerializer(serializers.ModelSerializer):
    # read_only — поле лише для читання, не приймається при POST/PUT
    id = serializers.IntegerField(read_only=True)

    # write_only — поле лише для запису, не повертається у відповіді
    password = serializers.CharField(write_only=True)

    # source — брати значення з іншого атрибута моделі
    name = serializers.CharField(source='title')

    # SerializerMethodField — обчислюване поле
    full_info = serializers.SerializerMethodField()

    def get_full_info(self, obj):
        return f"{obj.title} — {obj.author} ({obj.year})"

    class Meta:
        model  = Book
        fields = ['id', 'name', 'full_info']
```

### Вкладені серіалізатори

```python
class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model  = Author
        fields = ['id', 'name']


class BookSerializer(serializers.ModelSerializer):
    author = AuthorSerializer()   # ← вкладений серіалізатор

    class Meta:
        model  = Book
        fields = ['id', 'title', 'author', 'year']
```

```json
{
  "id": 1,
  "title": "1984",
  "author": {
    "id": 2,
    "name": "George Orwell"
  },
  "year": 1949
}
```

### Валідація у серіалізаторі

```python
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model  = Book
        fields = ['title', 'author', 'year']

    def validate_year(self, value):
        if value < 868 or value > 2025:
            raise serializers.ValidationError("Некоректний рік.")
        return value

    def validate(self, data):
        if data['title'] == data['author']:
            raise serializers.ValidationError("Назва і автор не можуть збігатися.")
        return data
```

---

## API Views

DRF надає кілька підходів до написання view, від низькорівневих до максимально автоматизованих.

### `@api_view` — функціональний підхід

Найпростіший спосіб — декоратор над звичайною функцією:

```python
# app_name/views.py

from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from .models import Book
from .serializers import BookSerializer


@api_view(['GET', 'POST'])
def book_list(request):
    if request.method == 'GET':
        books      = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)

    if request.method == 'POST':
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['GET', 'PUT', 'DELETE'])
def book_detail(request, pk):
    try:
        book = Book.objects.get(pk=pk)
    except Book.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = BookSerializer(book)
        return Response(serializer.data)

    if request.method == 'PUT':
        serializer = BookSerializer(book, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    if request.method == 'DELETE':
        book.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### `APIView` — класовий підхід

Більш структурований варіант — кожен HTTP-метод є окремим методом класу:

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from django.shortcuts import get_object_or_404


class BookListView(APIView):
    def get(self, request):
        books      = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


class BookDetailView(APIView):
    def get_object(self, pk):
        return get_object_or_404(Book, pk=pk)

    def get(self, request, pk):
        book       = self.get_object(pk)
        serializer = BookSerializer(book)
        return Response(serializer.data)

    def put(self, request, pk):
        book       = self.get_object(pk)
        serializer = BookSerializer(book, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def patch(self, request, pk):
        book       = self.get_object(pk)
        serializer = BookSerializer(book, data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk):
        book = self.get_object(pk)
        book.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### `ViewSet` + Router — максимальна автоматизація

`ViewSet` об'єднує всю логіку для одного ресурсу в один клас. `Router` автоматично генерує URL.

```python
# app_name/views.py

from rest_framework import viewsets
from .models import Book
from .serializers import BookSerializer


class BookViewSet(viewsets.ModelViewSet):
    queryset           = Book.objects.all()
    serializer_class   = BookSerializer
```

Це — повний CRUD для моделі `Book`. Всього два рядки замість кількох десятків.

`ModelViewSet` автоматично реалізує:

| Метод ViewSet | HTTP | URL |
|---|---|---|
| `list` | `GET` | `/api/books/` |
| `create` | `POST` | `/api/books/` |
| `retrieve` | `GET` | `/api/books/1/` |
| `update` | `PUT` | `/api/books/1/` |
| `partial_update` | `PATCH` | `/api/books/1/` |
| `destroy` | `DELETE` | `/api/books/1/` |

### Коди HTTP-статусів у DRF

```python
from rest_framework import status

status.HTTP_200_OK            # успішний GET
status.HTTP_201_CREATED       # успішний POST
status.HTTP_204_NO_CONTENT    # успішний DELETE
status.HTTP_400_BAD_REQUEST   # помилка валідації
status.HTTP_401_UNAUTHORIZED  # не авторизований
status.HTTP_403_FORBIDDEN     # немає прав
status.HTTP_404_NOT_FOUND     # не знайдено
```

---

## Маршрутизація API

### Ручна маршрутизація (для `@api_view` та `APIView`)

```python
# app_name/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('books/',      views.BookListView.as_view(),   name='book-list'),
    path('books/<int:pk>/', views.BookDetailView.as_view(), name='book-detail'),
]
```

```python
# project_name/urls.py

from django.urls import path, include

urlpatterns = [
    path('api/', include('app_name.urls')),
]
```

### Автоматична маршрутизація через `Router` (для `ViewSet`)

```python
# app_name/urls.py

from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register('books', views.BookViewSet, basename='book')

urlpatterns = router.urls
```

`DefaultRouter` автоматично створює:

```
GET    /api/books/        → list
POST   /api/books/        → create
GET    /api/books/1/      → retrieve
PUT    /api/books/1/      → update
PATCH  /api/books/1/      → partial_update
DELETE /api/books/1/      → destroy
GET    /api/             → API root (список ендпоінтів)
```

---

## Permissions та Authentication

### Permissions — права доступу

Дозволяють обмежити доступ до окремих view.

**Вбудовані класи:**

| Клас | Поведінка |
|---|---|
| `AllowAny` | Доступ для всіх (за замовч.) |
| `IsAuthenticated` | Лише авторизовані користувачі |
| `IsAdminUser` | Лише адміністратори |
| `IsAuthenticatedOrReadOnly` | Читання — всім, запис — лише авторизованим |

```python
from rest_framework.permissions import IsAuthenticated, IsAuthenticatedOrReadOnly
from rest_framework import viewsets


class BookViewSet(viewsets.ModelViewSet):
    queryset         = Book.objects.all()
    serializer_class = BookSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
    # GET /api/books/ — доступно всім
    # POST /api/books/ — лише авторизованим
```

**Глобально через `settings.py`:**

```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
}
```

### Token Authentication

DRF надає вбудовану автентифікацію через токени.

**Підключення:**

```python
# settings.py
INSTALLED_APPS = [
    ...
    'rest_framework.authtoken',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
}
```

```bash
python manage.py migrate   # створює таблицю токенів
```

**Ендпоінт для отримання токена:**

```python
# project_name/urls.py

from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    path('api/token/', obtain_auth_token, name='api-token'),
    path('api/', include('app_name.urls')),
]
```

**Використання:**

```bash
# Отримати токен
POST /api/token/
{"username": "admin", "password": "1234"}
→ {"token": "9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b"}

# Використати токен у запитах
GET /api/books/
Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b
```

---

## Browsable API

DRF автоматично генерує зручний HTML-інтерфейс для тестування API прямо у браузері. Він доступний за тими самими URL що й API — просто відкрийте їх у браузері.

```
http://127.0.0.1:8000/api/books/
```

Browsable API дозволяє:
- переглядати структуру відповіді
- надсилати GET, POST, PUT, DELETE запити через форму
- бачити повідомлення про помилки валідації

Для продакшн-середовища Browsable API зазвичай вимикають:

```python
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
        # BrowsableAPIRenderer — прибрати для продакшн
    ],
}
```

---

## Повний приклад: API бібліотеки

### Структура проєкту

```
project_name/
├── project_name/
│   ├── settings.py
│   └── urls.py
│
└── app_name/
    ├── migrations/
    ├── models.py
    ├── serializers.py
    ├── views.py
    └── urls.py
```

### `models.py`

```python
# app_name/models.py

from django.db import models


class Author(models.Model):
    name = models.CharField(max_length=200, verbose_name="Ім'я")
    bio  = models.TextField(blank=True, verbose_name="Біографія")

    def __str__(self):
        return self.name


class Book(models.Model):
    GENRE_CHOICES = [
        ('fiction',     'Художня'),
        ('non_fiction', 'Науково-популярна'),
        ('poetry',      'Поезія'),
    ]

    title        = models.CharField(max_length=200, verbose_name="Назва")
    author       = models.ForeignKey(
                       Author,
                       on_delete=models.CASCADE,
                       related_name='books',
                   )
    year         = models.IntegerField(verbose_name="Рік")
    genre        = models.CharField(max_length=20, choices=GENRE_CHOICES)
    is_available = models.BooleanField(default=True)
    created_at   = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['-year']

    def __str__(self):
        return self.title
```

### `serializers.py`

```python
# app_name/serializers.py

from rest_framework import serializers
from .models import Author, Book


class AuthorSerializer(serializers.ModelSerializer):
    books_count = serializers.SerializerMethodField()

    def get_books_count(self, obj):
        return obj.books.count()

    class Meta:
        model  = Author
        fields = ['id', 'name', 'bio', 'books_count']


class BookSerializer(serializers.ModelSerializer):
    author_name = serializers.CharField(
        source='author.name',
        read_only=True,
    )

    class Meta:
        model  = Book
        fields = [
            'id', 'title', 'author', 'author_name',
            'year', 'genre', 'is_available', 'created_at',
        ]
        read_only_fields = ['created_at']

    def validate_year(self, value):
        if value < 868 or value > 2025:
            raise serializers.ValidationError("Некоректний рік.")
        return value
```

### `views.py`

```python
# app_name/views.py

from rest_framework import viewsets, filters
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from .models import Author, Book
from .serializers import AuthorSerializer, BookSerializer


class AuthorViewSet(viewsets.ModelViewSet):
    queryset           = Author.objects.all()
    serializer_class   = AuthorSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]


class BookViewSet(viewsets.ModelViewSet):
    queryset           = Book.objects.select_related('author')
    serializer_class   = BookSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
    filter_backends    = [filters.SearchFilter, filters.OrderingFilter]
    search_fields      = ['title', 'author__name']
    ordering_fields    = ['year', 'title']

    # Кастомний action — GET /api/books/available/
    @action(detail=False, methods=['get'])
    def available(self, request):
        books      = self.queryset.filter(is_available=True)
        serializer = self.get_serializer(books, many=True)
        return Response(serializer.data)

    # Кастомний action — POST /api/books/1/toggle_availability/
    @action(detail=True, methods=['post'])
    def toggle_availability(self, request, pk=None):
        book              = self.get_object()
        book.is_available = not book.is_available
        book.save()
        return Response({'is_available': book.is_available})
```

### `urls.py`

```python
# app_name/urls.py

from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register('authors', views.AuthorViewSet, basename='author')
router.register('books',   views.BookViewSet,   basename='book')

urlpatterns = router.urls
```

```python
# project_name/urls.py

from django.contrib import admin
from django.urls import path, include
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    path('admin/',     admin.site.urls),
    path('api/',       include('app_name.urls')),
    path('api/token/', obtain_auth_token, name='api-token'),
]
```

### Приклади запитів до API

```bash
# Список усіх книг
GET /api/books/

# Пошук
GET /api/books/?search=orwell

# Сортування
GET /api/books/?ordering=-year

# Доступні книги (кастомний action)
GET /api/books/available/

# Одна книга
GET /api/books/1/

# Створити книгу
POST /api/books/
Content-Type: application/json
{"title": "1984", "author": 1, "year": 1949, "genre": "fiction"}

# Оновити частково
PATCH /api/books/1/
{"is_available": false}

# Видалити
DELETE /api/books/1/
```

### Приклад JSON-відповіді

```json
[
  {
    "id": 1,
    "title": "1984",
    "author": 2,
    "author_name": "George Orwell",
    "year": 1949,
    "genre": "fiction",
    "is_available": true,
    "created_at": "2025-01-15T10:30:00Z"
  }
]
```

---

## Довідкові матеріали

- [Django REST Framework — офіційна документація (EN)](https://www.django-rest-framework.org/) — повний опис усіх компонентів
- [DRF — Serializers (EN)](https://www.django-rest-framework.org/api-guide/serializers/) — усі типи серіалізаторів і параметрів
- [DRF — Views та ViewSets (EN)](https://www.django-rest-framework.org/api-guide/viewsets/) — `APIView`, `ViewSet`, `ModelViewSet`
- [DRF — Routers (EN)](https://www.django-rest-framework.org/api-guide/routers/) — `DefaultRouter`, `SimpleRouter`
- [DRF — Permissions (EN)](https://www.django-rest-framework.org/api-guide/permissions/) — вбудовані та кастомні права доступу
- [DRF — Authentication (EN)](https://www.django-rest-framework.org/api-guide/authentication/) — токени, сесії, JWT
- [DRF — Filtering (EN)](https://www.django-rest-framework.org/api-guide/filtering/) — пошук, фільтри, сортування
- [Real Python — Django REST Framework (EN)](https://realpython.com/django-rest-framework-quick-start/) — покроковий вступний туторіал
- [Django Girls Tutorial українською](https://tutorial.djangogirls.org/uk/) — основи Django для контексту
