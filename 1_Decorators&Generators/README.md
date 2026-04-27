# 1. Декоратори та генератори

## Декоратори

> [**Декоратор** в Python](https://acode.com.ua/decorators-python/) — це шаблон проектування, який дозволяє модифікувати роботу функції, обернувши її в іншу функцію. Зовнішня функція називається декоратором, який приймає як аргумент вихідну функцію та повертає її модифіковану версію.

Простий приклад декоратора:
```python
# Функція-декоратор
def my_decorator(func):
    # Дії, що виконаються одразу
    print("Декоратор викликано!")

    # Обгортка функції, що декорується
    def wrapper():
        print("Wrapper")
        # Виклик декорованої функції
        func()

    # Повернення обгортки
    return wrapper

# Декорування функції
@my_decorator
def say_hello():
    print("Hello")

print("Програма запущена")

```

## Генератори

> [**Генератор** в Python](https://acode.com.ua/generators-python/) — це функція, що повертає [ітератор](https://acode.com.ua/iterators-python/), який під час ітерації генерує послідовність значень. Генератори корисні, коли нам потрібно отримати велику послідовність значень, але ми не хочемо зберігати їх всі в пам’яті відразу.

Приклад генератора:
```python
# Функція-генератор
def count_up_to(n):
    i = 1
    while i <= n:
        # ВАЖЛИВО: yield замість return
        yield i
        i += 1

# Створення ітератора
gen = count_up_to(5)

# Обробка ітератора
for num in gen:
    print(num)
```

## Довідкові матеріали:
- Декоратори в Python — https://acode.com.ua/decorators-python/
- Що таке декоратори у Python — https://www.it-notes.wiki/python/what-is-decorators-in-python/
- Python: знайомство з декораторами на прикладі FastAPI — https://dou.ua/forums/topic/53490/
- Генератори в Python — https://acode.com.ua/generators-python/
- Ітератори в Python — https://acode.com.ua/iterators-python/
- Що таке генератори у Python — https://www.it-notes.wiki/python/what-are-generators-in-python/