# Deeper Jinja

![deeper.jpeg](img%2Fdeeper.jpeg)

В этой части тебя ждут макросы и расширения.

## Macros

Macros (макроопределения) - кусочки кода, которые можно вызывать, чтобы избежать однотипных повторяемых вызовов, так сказать будешь следовать 
принципу [DRY](https://ru.wikipedia.org/wiki/Don’t_repeat_yourself) (ты сможешь гордо сказать об этом на собесе =))

![dry.png](img%2Fdry.png)


Базовый синтаксис:

```sql
{% macro macro_name(*args, **kwargs)%}

{% endmacro %}
```

Ключевое слово `macro` +  набор аргументов, прямо как в Python - всё как ты любишь.

Использовать в других шаблона так же как и любой параметр `{{macro_name()}}`

Используем наши навыки: напишем макрос, который выводит текущую дату, в качестве параметра зададим имя поля с датой

```sql
{% macro current_datetime(column_name)%}
SELECT now() as {{column_name}};
{% endmacro %}

{{ current_datetime('dt') }}
```

Результат:

```sql
SELECT now() as dt;
```

Как создавать макросы и вызывать их ты умеешь, теперь нужно усилить свою мощь: писать каждый раз макрос ради его написания - плохая практика и противоречит DRY.
Далее познакомимся с тем как расширять модели макросами и решим задачу:

**Задача:**

Чтобы не писать каждый раз обработку нашего запроса (помнишь, там много предобработки, типы, строки и прочее) необходимо:
- составить запрос с предобработкой (базово выделить основную страну, основной жанр, привести год к целому числу, выделить рейтинг IMDb и и привести его к десятичному числу)
- сделать из запроса макрос-источник для последующих моделей

## Extends

Extends или наследование шаблонов - очень удобный инструмент для повышения гибкости ваших шаблонов. Например, тебе лень писать каждый раз код создания таблицы 
или вьюхи из запроса, тогда тебе нужны расширения. 

Extends - отдельный тип шаблонов, внутри которых указано, что заменить или подставить можно некоторых блок, пример синтаксиса:

```sql
SELECT *,
       now() as load_datetime
FROM (
         {% block query %}{% endblock %}
     )
```

Здесь шаблон добавляет дату загрузки данных (или просто текущую на момент запроса) + есть новый блок `{% block query %}{% endblock %}` - именованный блок, который
как раз и позволяет расширятся. В любой другой модели достаточно описать этот блок (с указанием имени) - то есть указать тот SQL код, который нам нужен.
Предыдущий код сохрани в [add_load_datetime.sql](templates%2Fadd_load_datetime.sql)
Пробуешь:

```sql
{% extends 'add_load_datetime.sql' %}
{% block query -%}
SELECT film_name
       country ,
       genre ,
       director ,
       budget ,
       box_office_usa ,
       rating_imbd 
FROM stg.kinopoisk 
{%- endblock %}
```

Строка `{% extends 'add_load_datetime.sql' %}` - как раз и осуществляет наследование, и здесь же мы описываем наш блок `query`, результат рендеринга:

```sql
SELECT *,
       now() as load_datetime
FROM (
         SELECT film_name
       country ,
       genre ,
       director ,
       budget ,
       box_office_usa ,
       rating_imbd
FROM stg.kinopoisk
     ) sq
```

## Соединяя несоединимое

Тебе предстоит путь воина: соединить все кусочка пазла вместе:
- расширения
- макросы
- и тебе понадобится импортирование - здесь вообще всё как в Python

Чтобы импортировать созданный макрос используй синтаксис `{% import 'add_load_datetime.sql' as m %}`, для вызова непосредственно макроса используй точечную нотацию
`{{ m.current_datetime() }}`

Важно: перемещаем макрос в папку макросов `macros/add_load_datetime.sql`, теперь, чтобы все шаблоны загружались пропиши список папок:

```python
stmt = get_rendered_template(["../templates", "../macros", "../models"],
                                 "extends-example.sql")

print(stmt)
```

Попробуешь сам написать код.... (также необходимо немного поправить макрос, тк возвращать запрос нам нужно)

Моё решение найдешь здесь [model-1.sql](models%2Fmodel-1.sql), чур не подглядывать 😎

## Твой первый источник данных

Помнишь нашу задачу:
Чтобы не писать каждый раз обработку нашего запроса (помнишь, там много предобработки, типы, строки и прочее) необходимо:
- составить запрос с предобработкой (базово выделить основную страну, основной жанр, привести год к целому числу, выделить рейтинг IMDb и и привести его к десятичному числу)
- добавить дату загрузки
- добавить имя источника

Теперь для этого у тебя всё есть. Один из вариантов может быть таким

```sql
{% extends 'add_load_datetime_source.sql' %}
{% block qdt -%}
SELECT film_name,
       lower(split_part(country, '_', 1)) as country,
       lower(split_part(genre, '_', 1)) as genre ,
       lower(split_part(director, '_', 1)) as director ,
       regexp_replace(substring(budget from '[\d+\s+]+'), '\s+', '', 'g')::int8 as budget,
       regexp_replace(substring(box_office_usa from '[\d+\s+]+'), '\s+', '', 'g')::int8 as box_office_usa,
       regexp_replace(substring(rating_imbd from '\d.\d+'), '\s+', '', 'g')::float imdb
FROM stg.kinopoisk
WHERE film_name IS NOT NULL
{%- endblock qdt %}
```

Файл [add_load_datetime_source.sql](macros%2Fadd_load_datetime_source.sql) добавляет технические поля, запрос пишем ручками.


Модель источника готова (по крайней мере её зачаточное представление). Чтобы сделать её полноценной необходимо указать материализацию (таблица или вьюха),
для это пишешь еще небольшой макрос:

```sql
{%- macro materialization(type, schema, name) -%}
DROP {% if type=='table' %} TABLE {% else %} VIEW {% endif -%} IF EXISTS {{schema}}.{{name}};
CREATE {% if type=='table' %} TABLE {% else %} VIEW {% endif -%} {{schema}}.{{name}} AS
{%- endmacro -%}
```

Макрос в зависимости от параметра сформирует код для таблицы или вьюхи - действуем по принципу: _каждый раз пересоздаем обьект_

Добавляется он в текст модели простым добавлением `include` - просто весь текст прибавиться к имеющемуся. Итого полный текст нашего источника:

```sql
{% from 'config.sql' import materialization %}
{{materialization(type='table', schema='dds', name='raw_kp')}}
{% extends 'add_load_datetime_source.sql' %}
{% block qdt -%}
SELECT production_year,
       film_name,
       lower(split_part(country, '_', 1)) as country,
       lower(split_part(genre, '_', 1)) as genre ,
       lower(split_part(director, '_', 1)) as director ,
       regexp_replace(substring(budget from '[\d+\s+]+'), '\s+', '', 'g')::int8 as budget,
       regexp_replace(substring(box_office_usa from '[\d+\s+]+'), '\s+', '', 'g')::int8 as box_office_usa,
       regexp_replace(substring(rating_imbd from '\d.\d+'), '\s+', '', 'g')::float imdb
FROM stg.kinopoisk
WHERE film_name IS NOT NULL
AND production_year != ''
{%- endblock qdt %}
```

Генерим исходный код, а давай чтобы не копировать, создадим папку `compiled` и будем туда сохранять отрендеренный шаблон, готовый для исполнения, погнали:

```python
def save_compiled_model(file:str, folder:str='compiled'):
    with open(os.path.join(folder, file), 'w') as f:
        f.write(stmt)
```

Запускаем `main.py` и получаем готовый код для выполнения, исполняем в DBeaver и чекаем результат:

![dds-raw.png](img%2Fdds-raw.png)

👉 [Deep and deeper, go to chapter 2-3](https://github.com/urevoleg/course-dbt-fundamentals/blob/main/course/chapter-2/README-2-3.md)
