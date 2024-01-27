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
WHERE table_name = 'kinopoisk';
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

В качестве переменной можно задать список `{% set my_list = ['once', 'twice', 'qwerty']%}`:

```sql
SELECT '['once', 'twice', 'qwerty']' as external_name,
      now() as dt;
```

4. Использование `for`

Умеем создавать список значений, значит по нему можно пройтись, для этого и необходима конструкция `for`, синтаксис:

```python
{% for elem in elem_list%}

{% endfor %}
```

Внутри необходимо написать то, что мы хотим выполнить для каждого элемента, используем задание перменной и цикл для формирование нескольких селектов,
обращаться к переменной цикла: `{{ elem }}`

```sql
{% set my_list = ['once', 'twice', 'qwerty']%}

{% for elem in my_list%}
SELECT '{{ elem }}' as external_name,
      now() as dt
{% endfor %}
```

Имеем следующий результат:

```sql
SELECT 'once' as external_name,
      now() as dt

SELECT 'twice' as external_name,
      now() as dt

SELECT 'qwerty' as external_name,
      now() as dt
```

Почти то, что хотелось, для объединения запросов необходимо добавить оператор `UNION ALL`, дописывай после запроса и смотри результат:

```sql
SELECT 'once' as external_name,
      now() as dt
UNION ALL

SELECT 'twice' as external_name,
      now() as dt
UNION ALL

SELECT 'qwerty' as external_name,
      now() as dt
UNION ALL
```

5. `if`
Мешает последняя запись `UNION ALL`, которая будет вызывать ошибку при выполнении SQL, и здесь к тебе на помощь приходит еще одна конструкция - `if`:

```sql
{% set my_list = ['once', 'twice', 'qwerty']%}

{% for elem in my_list%}
SELECT '{{ elem }}' as external_name,
      now() as dt
{% if not loop.last %}
UNION ALL
{% endif %}
{% endfor %}
```

Как ты мог заметить, синтаксис ни чем не отличается от других конструкций и чем-то похож на Python. Выше используется проверка специальной переменной `loop`
пока не последний элемент будет добавлятся текст указанный внутри `if`, для последнего элемента добавления не будет, смотрим на результат:

![set-for-if-example.png](img%2Fset-for-if-example.png)


6. Решаем поставленную задачу

```
Для каждого года в списке вывести кол-во фильмов
```

Задачу можно решить несколькими способами, например:
- использовать `CASE\WHEN`
- использовать `WHERE\UNION`
- использовать `WHERE\GROUP BY`
- ....

Я буду использовать первый вариант [set-for-if-example-1.sql](templates%2Fset-for-if-example-1.sql), сгенерированный код и результат запроса:

![set-for-if-example-1.png](img%2Fset-for-if-example-1.png)

### Интересное и важное

Ты мог заметить, что при генерации появляются пустые строки между строками запроса, их наличие или отсутствие определяется еле заметным изменением 
в синтаксисе:

![minus-empty-string.png](img%2Fminus-empty-string.png)

☝️Указание `минуса` в синтаксисе управляет наличием пустой строки перед или после указанного оператора.

👉 [Deep and deeper, go to chapter 2-2](https://github.com/urevoleg/course-dbt-fundamentals/blob/main/course/chapter-2/README-2-2.md#deeper-jinja)