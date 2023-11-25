## Deep dive to Jinja and SQL
![deep-dive-jinja-sql.png](..%2F..%2Fimg%2Fdeep-dive-jinja-sql.png)

Чтобы легче вкатиться в мир dbt, начнем знакомство с Jinja поближе и сделаем свой мини-dbt (ну совсем мини 😈)

### First SQL

Подготовим необходимую структуру папок:
- templates - тут сиквел файлы-шаблоны
- scripts - тут живут python скрипты
- macros - на будущее для макросов =)

После выполнения chapter-1 у тебя появилась таблица **stg.kinopoisk**. Для нас она будет источником данных:

```sql
SELECT * FROM stg.kinopoisk; 
```

1. Тренируемся на кошечках

Создаем шаблон: простой SELECT c условиями, попробуем фильтровать по году и стране, например, такой [where-clause.sql](template%2Fwhere-clause.sql)

```sql
SELECT * FROM stg.kinopoisk
WHERE production_year = '{{ year }}'
AND country = '{{ country }}'; 
```

Обрати внимание:
![escape-strings-param.png](..%2F..%2Fimg%2Fescape-strings-param.png)


Используем наработки из chapter-2 для генерации SQL -> [main.py](scripts%2Fmain.py):

```python
if __name__ == "__main__":
    stmt = get_rendered_template("../templates",
                                 "where-clause.sql",
                                 year="2020",
                                 country="США")

    print(stmt)
```

Передаём наши параметры и получаем выходной SQL-скрипт:

```sql
SELECT * FROM stg.kinopoisk
WHERE production_year = '2020'
AND country = 'США'
```

Идем проверим: ошибок нет, получили только 2020 год и США:
![where-clause.png](img%2Fwhere-clause.png)


2. Research data

Давай немного поизучаем данные: посмотрим типы данных и как записаны строковые поля (страна, жанр, режиссер, бюджет и сборы и др).

Типы данных можно проверить нехитрым запросом:

```sql
SELECT column_name,
      data_type
FROM information_schema.COLUMNS
WHERE ![img.png](img.png)table_name = 'kinopoisk';
```

Все поля у нас текстовые, кроме **rating_kp_top** (ему повезло).

Отмечаем:
- жанров может быть несколько - разделены нижним подчеркиванием
- стран может быть несколько - разделены нижним подчеркиванием
- бюджет указан в денежным единицах
- рейтинг IMDb имеет префикс

![research.png](img%2Fresearch.png)

Как минимум, чтобы использовать данные далее необходимо выполнить довольно много преобразований - этим мы займемся, но чуть позже.

А пока рассмотрим еще несколько конструкций jinja, на очереди задание переменных `set` и `for` - у нас задача, для некоторого набора
годов посчитать кол-во выпущенных фильмов (да, это можно сделать при помощи группировки, но ты же идешь в гору, а не в обход, верно😜)

3. Использование переменных

Для выполнения какой-либо конструкции (if/for/set и др) внутри jinja используется синтаксис `{%  %}`, чтобы
задать собственную переменную указываем: `{% set my_var = 'Hello, Jinja2!'%}`

Cоздаем простой шаблон :

```sql
{% set my_var = 'Hello, Jinja2!'%}
SELECT '{{ my_var }}' as external_name,
      now() as dt;
```

Запускаем и вуаля:

```sql
SELECT 'Hello, Jinja2!' as external_name,
      now() as dt;
```