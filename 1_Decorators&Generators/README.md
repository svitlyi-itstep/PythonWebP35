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
    # ! ВАЖЛИВО: Обгортка повертається саме функцією
    #            my_decorator, а не wrapper
    return wrapper

# Декорування функції
@my_decorator
def say_hello():
    print("Hello")

print("Програма запущена")

```

Декоратори можна використовувати в ситуаціях, коли є певний функціонал, стандартний для деякої групи функцій. Наприклад, якщо потрібно додати декорування виведення певної функції в консолі, можна реалізувати це наступним чином.

```python
def decorate_with_lines(func):
    def wrapper():
        print("═"*20)
        func()
        print("═"*20)
    return wrapper

@decorate_with_lines
def say_hello():
    print("Hello")


say_hello()
```
У консоль буде виведено наступне:

```
    ════════════════════
    Hello
    ════════════════════
```

### Використання декораторів для функцій з параметрами

Спробуємо використати декоратор вище для функції, що приймає параметри:
```python
def decorate_with_lines(func):
    def wrapper():
        print("═"*20)
        func()
        print("═"*20)

    return wrapper

@decorate_with_lines
def sum(a, b):
    print(f"{a} + {b} = {a + b}")


sum(2, 3)
```

В результаті отримаємо помилку: `TypeError: decorate_with_lines.<locals>.wrapper() takes 0 positional arguments but 2 were given`.

Це відбувається, тому що функція `wrapper` фактично замінює собою функцію `sum`. При цьому `wrapper` не приймає параметрів, тож передані числа 2 і 3 записати нікуди, звідси і помилка. В даному випадку можна просто додати вказані параметри у функцію `wrapper` і передавати їх функції `sum`.

```python
def decorate_with_lines(func):
    # Додаємо параметри a та b
    def wrapper(a, b):
        print("═"*20)
        # Також не забуваємо передати їх 
        # у функцію, що декорується
        func(a, b)
        print("═"*20)
    
    return wrapper

@decorate_with_lines
def sum(a, b):
    print(f"{a} + {b} = {a + b}")


sum(2, 3)
```

Але більш універсальним способом буде вказання необов'язкових параметрів `*args, **kwargs` у `wrapper` та функцію, що декорується:

```python
def decorate_with_lines(func):
    # Буде працювати для функцій з будь-якими
    # параметрами
    def wrapper(*args, **kwargs):
        print("═"*20)
        # Також не забуваємо передати параметри 
        # у функцію, що декорується
        func(*args, **kwargs)
        print("═"*20)
    
    return wrapper

@decorate_with_lines
def sum(a, b):
    print(f"{a} + {b} = {a + b}")


sum(2, 3)
```

### Параметри декораторів

Якщо ми хочемо дати можливість налаштовувати поведінку декоратора через параметри, його потрібно обгорнути у ще одну функцію, що прийматиме ці параметри:

```python
# Функція, що приймає параметри
def decorate(symbol="═", width=20):
    # Функція, що прийматиме декоровану функцію
    def decorator(func):
        # Обгортка
        def wrapper(*args, **kwargs):
            print(symbol*width)
            func(*args, **kwargs)
            print(symbol*width)
        return wrapper
    return decorator

@decorate()
def say_hello_1():
    print("Hello 1")

@decorate("*")
def say_hello_2():
    print("Hello 2")

@decorate("#", 30)
def say_hello_3():
    print("Hello 3")


say_hello_1()
say_hello_2()
say_hello_3()
```

В консолі отримаємо:
```
════════════════════
Hello 1
════════════════════

********************
Hello 2
********************

##############################
Hello 3
##############################
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