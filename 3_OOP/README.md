# 3. Об'єктно-орієнтоване програмування в Python

Python, як і майже будь-яка мова програмування високого рівня, підтримує концепції ООП, але, на відміну від C-подібних мов, робить це дуже гнучко. 

Для початку згадаємо основи:

> [**Об’єкт**](https://acode.com.ua/object-oriented-programming-python/) — це будь-яка сутність, яка має атрибути (дані) та поведінку (методи/функції). Наприклад, кошеня — це об’єкт. В нього є:
>   - атрибути — ім’я, вік, колір тощо.
>   - поведінка — бігає, нявкає, спить і т.д.
> 
> ---
> **Клас** — це шаблон об'єкту, який описує його будову. Також можна сказати, що клас — це загальний тип для об'єктів. Саме в класі описується структура майбутніх об'єктів.

![](https://miro.medium.com/v2/1*ppwbbp6i3aFyt20NDk09gQ.jpeg)

![](https://images.squarespace-cdn.com/content/v1/53d60643e4b05dcadd61bb4b/1452659806598-XL6ES7XKG7BM1TVOZ51N/image-asset.jpeg)


Приклад реалізації класу у Python:

```python
class Unit:
    name = ""
    hp = 100

    # Конструктор
    def __init__(self, name, hp=100):
        self.name = name
        self.hp = hp

    # Конвертація у рядок (user-friendly)
    def __str__(self):
        return f"{self.name} with {self.hp}HP"

    # Конвертація у рядок (debug)
    def __repr__(self):
        return f"Unit({self.name=}, {self.hp=})"

    def is_alive(self):
        return self.hp > 0

    def take_damage(self, damage):
        self.hp -= damage
```

Створити об'єкт на основі класу можна наступним чином:
```python
unit = Unit("Unit 1", 75)
print(unit) # Unit 1 with 75HP

unit = Unit("Unit 2", 100)
print(repr(unit)) # Unit(self.name="Unit 2", self.hp=100)
```

### Інкапсуляція у Python

В Python немає повноцінної реалізації специфікаторів доступу, як це зроблено у C-подібних мовах. Але є інструмент, який дозволяє організувати інкапсуляцію та керувати зміною полів класу.

```python
class Unit:
    def __init__(self):
        self._hp = 100

    @property
    def hp(self):
        return self._hp

    @hp.setter
    def hp(self, value):
        if value < 0:
            self._hp = 0
        else:
            self._hp = value
```

В основній частині програми це можна використати так:

```python
unit = Unit()
unit.hp -= 10
```


## Довідкові матеріали:
- Об’єктно-орієнтоване програмування (ООП) в Python — https://acode.com.ua/object-oriented-programming-python/
- ООП в Python — http://ruslan.rv.ua/python-essential/oop/classes.html
- Що означає self у Python — https://www.it-notes.wiki/python/what-does-self-mean-in-python/
- Методи \_\_str\_\_() та \_\_repr\_\_() у Python — https://www.it-notes.wiki/python/methods-str-and-repr/
- Декоратор @property в Python — https://acode.com.ua/property-decorator-python/
- Python + Pygame. Урок 1. — https://devzone.org.ua/post/python-pygame-urok-1
- Pygame Cheat Sheet — https://medium.com/@amit25173/pygame-cheat-sheet-311cfc7b6ce8
- Beginners Python cheatsheet — [pygame_cheatsheet.pdf](/3_OOP/pygame_cheatsheet.pdf)