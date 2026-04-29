# 2. Лямбда-функції


> [**Lambda expressions (лямбда вирази)**](https://www.it-notes.wiki/python/lambda-expressions-and-anonymous-functions-in-python/) – це короткий спосіб створення функцій (lambda functions), що є анонімними (тобто безіменні), та зазвичай створюються прямо в місці їх використання.
>
> У Python поняття **лямбда-функції**, **лямбда-вирази** та **анонімні функції** — це різні назви однієї концепції.

Синтаксис лямбда-функцій дуже простий:

```python
lambda [параметри]: вираз
```

Вони потрібні для того, щоб "на льоту" створити просту функцію, яка зазвичай буде використана десь далі. На відміну від анонімних функцій в інших мовах програмування, **лямбда-вирази в Python можуть містити тільки один вираз**.

Приклади лямбда-виразів:
```python
# Без параметрів
greet    = lambda: "Hello!"

# З одним параметром
square   = lambda x: x ** 2

# З двома параметрами
add      = lambda x, y: x + y

# З параметром за замовчанням
power    = lambda x, n=2: x ** n

# З умовним виразом
absolute = lambda x: x if x >= 0 else -x
```

Часто лямбда-функції використовуються для обробки, сортування та фільтрації данних, бо в них можна вказати умови та налаштування цих дій.

```python
data = [
    {"name": "Bob", "age": 25}, 
    {"name": "Alice", "age": 30}
]

# sorted() — сортування даних
sorted_data = sorted(data, key=lambda p: p["age"])

# map() — обробка кожного елементу набору даних
squares = list(map(lambda x: x**2, [1, 2, 3, 4]))

# filter() — фільтрація даних
evens = list(filter(lambda x: x % 2 == 0, [1, 2, 3, 4, 5, 6]))

# reduce() — виділення одного значення з набору значень
from functools import reduce
product = reduce(lambda acc, x: acc * x, [1, 2, 3, 4, 5])

# min() / max() — налаштування пошуку мінімуму/максимуму
oldest = max(data, key=lambda p: p["age"])
```

**ВАЖЛИВО:** використовуйте лямбда-функції тільки для простої логіки. Складні концепції краще реалізувати за допомогою `def`.


Окремо слід зазначити, що лямбда-функції дають можливість створювати **function factory** (фабрику функцій):

```python
# Повертає функцію, що буде множити отримане число
# на те, що вказано при створенні функції
def make_multiplier(n):
    return lambda x: x * n

double = make_multiplier(2)
triple = make_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15
```

## Довідкові матеріали:
- Лямбда (анонімні функції) в Python — https://acode.com.ua/lambda-python/
- Lambda вирази та анонімні функції в Python — https://www.it-notes.wiki/python/lambda-expressions-and-anonymous-functions-in-python/
- Функція map() в Python — https://acode.com.ua/function-map-python/
- Функція filter() в Python — https://acode.com.ua/function-filter-python/
- Функція sorted() в Python — https://acode.com.ua/function-sorted-python/