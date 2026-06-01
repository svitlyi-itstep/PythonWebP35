`Домашня робота №10`

## REST API для симуляції боїв персонажів

Розробити REST API на базі Django REST Framework для керування персонажами та симуляції боїв між ними.

---

## Модель персонажа

| Поле    | Тип        | Обмеження  |
|---------|------------|------------|
| id      | auto       |            |
| name    | string     |            |
| hp      | integer    | > 0        |
| attack  | integer    | > 0        |
| defense | integer    | ≥ 0        |

```json
{ "id": 1, "name": "Knight", "hp": 100, "attack": 20, "defense": 5 }
```

---

## API персонажів

| Метод  | URL                    | Дія                    |
|--------|------------------------|------------------------|
| GET    | `/api/characters/`     | Список персонажів      |
| GET    | `/api/characters/{id}/`| Один персонаж          |
| POST   | `/api/characters/`     | Створити персонажа     |
| PUT    | `/api/characters/{id}/`| Оновити персонажа      |
| DELETE | `/api/characters/{id}/`| Видалити персонажа     |

---

## Симуляція бою

```http
POST /api/battle/
```

**Запит:**
```json
{ "character_1": 1, "character_2": 2 }
```

**Відповідь:**
```json
{
    "winner": "Knight",
    "rounds": 4,
    "battle_log": [
        "Round 1: Knight dealt 15 damage",
        "Round 1: Orc dealt 10 damage",
        "..."
    ]
}
```

### Правила

- Бій покроковий: спочатку атакує перший, потім другий.
- Формула шкоди: `damage = max(attack - defense, 1)`
- Бій триває, доки HP одного з персонажів не стане ≤ 0.

---

## Додатково: Історія боїв

Модель `Battle` зберігає: дату бою, учасників, переможця, кількість раундів.

Реалізувати API для перегляду історії боїв.
