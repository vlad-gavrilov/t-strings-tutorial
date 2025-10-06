# Работа с t-strings в Python 3.14 🐍

```python {.marimo hide_code="true"}
import marimo as mo

mo.md("# Работа с t-strings в Python 3.14 🐍")
```

Строки `t-strings` являются обобщением классических `f-strings`. Они позволяют перехватывать и изменять значения параметров перед тем, как скомбинировать их в результат. В отличие от `f-strings`, они возвращают не конченую строку, а объект класса `Template`.

`t-strings` были представлены в [PEP 750](https://peps.python.org/pep-0750/) в июле 2024 года, и окончательно приняты 10 апреля 2025 года. Полноценная поддержка `t-strings` пристутствует в Python версии 3.14 и выше.

На текущий день (20.08.25) Python 3.14 находится в стадии `release candidate`, поэтому все дальнейшие примеры будут использовать именно эту версию.

```python {.marimo hide_code="true"}
import sys

print(f"Python version: {sys.version}")
```

## Существовавшие ранее способы работы со строками

- форматирование в стиле `C`
- методы `str.format()` и `str.format_map()`
- класс `strings.Template`
- стандартная функция `format()`
- форматтер `strings.Formatter.parse()`
- f-strings
<!---->
### C-стиль

```python {.marimo}
_name = "Иван"
_surname = "Петров"
_year = 1990

print(
    "Имя: %s, Фамилия: %s, Год рождения: %d, Возраст: %d"
    % (_name, _surname, _year, 2025 - _year)
)
print(
    "Имя: %(name)s, Фамилия: %(surname)s, Год рождения: %(year)d, Возраст: %(age)d"
    % {
        "name": _name,
        "surname": _surname,
        "year": _year,
        "age": 2025 - _year,
    }
)
```

### Методы `str.format()` и `str.format_map()`

```python {.marimo}
_name = "Иван"
_surname = "Петров"
_template = "Привет, {0} {1}!"
print(_template.format(_name, _surname))

_template = "На вашем счете: ${amount:.2f}"
print(_template.format(amount=1234.56789))

_template = "Курсы валют: {usd:.2f}, {eur:.2f}, {gbp:.2f}"
currency_rates = {"usd": 1.0, "eur": 0.898989, "gbp": 0.757473}
print(_template.format_map(currency_rates))
```

### Класс `strings.Template`

```python {.marimo}
from string import Template as _Template

_template = _Template("Привет, $_name $_surname!")
print(_template.substitute(_name="Иван", _surname="Петров"))


class _AtTemplate(_Template):
    delimiter = '@'

_template = _AtTemplate("Привет, @_name @_surname!")
print(_template.substitute(_name="Мария", _surname="Сидорова"))
```

### Стандартная функция `format()`

```python {.marimo}
_number = 1234.56789

print(f"Лево (20 символов):  '{_number:<20.2f}'")
print(f"Право (20 символов): '{_number:>20.2f}'")
print(f"Центр (20 символов): '{_number:^20.2f}'")

print(f"Ширина 15:     '{_number:*^15.2f}'")
print(f"Заполнение 0:  '{_number:0^15.2f}'")
print(f"Точность 1:    '{_number:->15.1f}'")
print(f"Проценты:      '{0.1234:.^15.1%}'")
```

### Форматтер `strings.Formatter.parse()`

```python {.marimo}
from string import Formatter as _Formatter

_template = "У пользователя {user!r} на счету ${amount:.4f}."

for text, field, format_spec, conversion in _Formatter().parse(_template):
    print(f"{text=}, {field=}, {format_spec=}, {conversion=}")
```

### `f-strings`

```python {.marimo}
_name = "Иван"
_surname = "Петров"

print(f"Привет, {_name} {_surname}!")
print(f"На вашем счете: ${1234.56789:.1f}")
```

## Первое знакомство с `t-strings`

```python {.marimo}
_name = "Иван"
_surname = "Петров"

print(t"Привет, {_name} {_surname}!")
```

## Класс `Template`

- `.interpolations` — содержит кортеж объектов класса `Interpolation` для каждого из заменяемых полей
- `.strings` — содержит кортеж из неизменяемых частей (строк) данной `t-string`
- `.values` — содержит кортеж из подставляемых значений

```python {.marimo}
from string.templatelib import Template as _Template

print(dir(_Template)[-3:])
```

```python {.marimo}
_name = "Иван"
_surname = "Петров"
_color = "Зеленый"

_template = t"Привет, {_name} {_surname}! Добро пожаловать в {_color} банк!"

print(_template.strings)
print(_template.interpolations)
print(_template.values)
```

## Класс `Interpolation`

- `.value` — результат вычисления входного выражения до применения к нему форматирования или преобразования
- `.expression` — входное выражение
- `.conversion` — может содержать однин из идентификаторов преобразования `s`, `r`, или `a` (по умолчанию `None`)
- `.format_spec` — может содержать правило для форматирования, наподобие `4.2f` (по умолчанию пустая строка)

```python {.marimo}
from string.templatelib import Interpolation as _Interpolation

print(dir(_Interpolation)[-4:])
```

```python {.marimo}
_amount = 120
_rate = 82.3

print(t"Сумма перевода: {_amount * _rate} рублей")
```

```python {.marimo}
_withdrawal, _banknote = 7500, 500
print(t"Банкомат может выдать сумму указанными купюрами? {_banknote != 0 and _withdrawal % _banknote == 0}")
```

```python {.marimo}
_amount = 1000.00
_template = t"{_amount!r}"

print(_template)

_balance = 1234.56789
_template = t"{_balance:.2f}"

print(_template)

_balance = 1234.56789
_template = t"{_balance!r:>15}"

print(_template)
```

## Обработка t-strings
<!---->
### Итерация по t-string

Так как t-strings являются итерируемыми объектами, по ним можно осуществлять итерацию. Например, в цикле `for`.

```python {.marimo}
_name = "Иван"
_surname = "Петров"
_color = "Зеленый"

_template = t'Привет, {_name} {_surname}! Добро пожаловать в {_color} банк!'

for item in _template:
    print(f"{item}")
    print(type(item))
    print()
```

Для отдельной работы с разными типами элементов, можно использовать `Structural Pattern Matching`.

```python {.marimo}
from string.templatelib import Template as _Template
from string.templatelib import Interpolation as _Interpolation

_name = "Иван"
_surname = "Петров"
_color = "Желтый"
_balance = 1234.5678

_template = (
    t"Привет, {_name} {_surname}! Добро пожаловать в {_color} банк! "
    t"На вашем балансе: {_balance} рублей."
)


def summary(_template):
    if not isinstance(_template, _Template):
        raise TypeError("Переданный объект не является t-string")

    result = []
    for item in _template:
        match item:
            case str(content):
                continue
            case _Interpolation(value, expression, conversion, format_spec):
                if expression == "_name":
                    result.append(f"ИМЯ: {value}")
                elif expression == "_surname":
                    result.append(f"ФАМИЛИЯ: {value}")
                elif expression == "_color":
                    result.append(f"БАНК: {value}")
                else:
                    result.append(f"БАЛАНС: {value}")
        result.append("\n")

    return "".join(result)


print(summary(_template))
```

## Особенности работы с t-strings
<!---->
### Конкатенация t-strings

```python {.marimo}
_name = "Иван"
_surname = "Петров"

print(t"{_name} " + t"{_surname}")
print(t"{_name} " t"{_surname}")

try:      
    print("Hello " + t"{_name}")
except TypeError:
    print("Ошбика конкатенации")
```

### Самодокументируемые t-strings

```python {.marimo}
_amount = 120
_rate = 82.3

print(t"Валюта: {_amount = }, {_rate = }")
```

### Сырые t-strings

```python {.marimo}
_database_path = r"C:\Data\users.db" 
print(rt"Читаем базу данных по адресу {_database_path}")

_field = "phone"
_pattern = r"\+7\d{10}"
print(rt"Проверка поля {_field} по шаблону: {_pattern}")

var = "x"
print(rt"\int_{{{var}}}^{{2{var}}} \frac{{1}}{{{var}^2}} d{var}")
```

### Вложенные t-strings

```python {.marimo}
value = 42
print(t"t-строка, содержащая другую t-строку {t"{value}"}")
```

В отформатированном виде:

```python
Template(
    strings=("t-строка, содержащая другую t-строку ", ""),
    interpolations=(
        Interpolation(
            Template(
                strings=("", ""),
                interpolations=(Interpolation(42, "value", None, ""),),
            ),
            't"{value}"',
            None,
            "",
        ),
    ),
)
```
<!---->
## Подходы к работе с t-strings
<!---->
### Ленивое вычисление

```python {.marimo}
def lazy_currency_conversion(amout, rate):
    return t"${amout} по курсу {rate} равно {amout * rate}"

print(lazy_currency_conversion(120, 82.3))
```

### Преобразование t-string в обычную строку

```python {.marimo}
from string.templatelib import Template as _Template


def to_string(template):
    if not isinstance(template, _Template):
        raise TypeError("Переданный объект не является t-string")

    def convert(value, conversion):
        func = {
            "a": ascii,
            "r": repr,
            "s": str,
        }.get(conversion, lambda x: x)
        return func(value)

    parts = []
    for item in template:
        if isinstance(item, str):
            parts.append(item)
        else:
            parts.append(
                format(convert(item.value, item.conversion), item.format_spec)
            )
    return "".join(parts)


price = 123.456789
print(to_string(t"Цена: ${price:.2f}"))

header = "Заголовок"
print(to_string(t"{header:=^20}"))
```

### Асинхронная работа

```python {.marimo}
from asyncio import sleep as _sleep
from random import random as _random


async def async_currency_conversion(amout):
    return t"${amout} по текущему курсу равно {amout * await get_rate()}"


async def get_rate():
    await _sleep(_random() + 0.5)
    return 82.3


print(await async_currency_conversion(120))
```

## Обработчики t-строк
<!---->
### Защита от SQL-инъекций

```python {.marimo}
from string.templatelib import Template as _Template
from string.templatelib import Interpolation as _Interpolation


def sanitized_sql(template):
    if not isinstance(template, _Template):
        raise TypeError("Переданный объект не является t-string")

    parts = []
    params = []

    for item in template:
        match item:
            case str(content):
                parts.append(content)
            case _Interpolation(value, _, _, _):
                parts.append("?")
                params.append(value)

    query = "".join(parts)
    return query, tuple(params)


_username = "'; DROP TABLE students;--"
_template = t"SELECT * FROM students WHERE name = {_username}"
_query, _params = sanitized_sql(_template)

print("Sanitized SQL Query:", _query)
print("Parameters:", _params)
```

### Экранирование HTML

```python {.marimo}
import html
from string.templatelib import Template as _Template
from string.templatelib import Interpolation as _Interpolation


def generate_safe_html(template):
    if not isinstance(template, _Template):
        raise TypeError("Переданный объект не является t-string")

    parts = []
    for item in template:
        match item:
            case str(content):
                parts.append(content)
            case _Interpolation(value, _, _, _):
                parts.append(html.escape(value))
    return "".join(parts)


_username = "<script>alert('Hacked!')</script>"
_template = t"<p>Hello, {_username}!</p>"

print(generate_safe_html(_template))
```

### Структурированное логирование

```python {.marimo}
import json
import logging
from string.templatelib import Template as _Template
from string.templatelib import Interpolation as _Interpolation

logging.basicConfig(level=logging.INFO, format="%(message)s")


class TemplateMessage:
    def __init__(self, template):
        if not isinstance(template, _Template):
            raise TypeError("Переданный объект не является t-string")
        self.template = template

    @property
    def message(self):
        parts = []
        for item in self.template:
            match item:
                case str(content):
                    parts.append(content)
                case _Interpolation(value, _, _, _):
                    parts.append(str(value))
        return "".join(parts)

    @property
    def values_dict(self):
        values = {}
        for item in self.template:
            if not isinstance(item, str):
                values[item.expression] = item.value
        return values

    def __str__(self):
        return f"{self.message} >>> {json.dumps(self.values_dict)}"


_action, _amount, _item = "refund", 7, "keyboard"
_msg_template = TemplateMessage(t"Process {_action}: {_amount:.2f} {_item}")
logging.warning(_msg_template)
```

## Ресурсы и полезные ссылки

- [PEP 750](https://peps.python.org/pep-0750/)
- [Дискуссия на HN](https://news.ycombinator.com/item?id=43748512)
- [Дискуссия на Reddit](https://www.reddit.com/r/programming/comments/1k4iwkq/pythons_new_tstrings/)
- [Блогпост соавтора PEP 750 Dave Peck](https://davepeck.org/2025/04/11/pythons-new-t-strings/)
- [Блогпост CPython Core Developer Brett Cannon](https://snarky.ca/unravelling-t-strings/)
- [Туториал на RealPython](https://realpython.com/python-t-strings/)
- [Новости на RealPython](https://realpython.com/python-news-may-2025/)
- [Talk Python to Me](https://www.youtube.com/watch?v=WCWNeZ_rE68)
- [PEP 750 examples](https://github.com/t-strings/pep750-examples)
- [t-strings help](https://t-strings.help/)