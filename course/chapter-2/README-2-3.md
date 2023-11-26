# Витрина

![de-datamarts.png](img%2Fde-datamarts.png)

Перед этой главой ты же потренил: расширения и макросы 🤨

Поэтому еще небольшой макрос, чтобы всё по-взрослому было:

```sql
{% macro ref(name) -%}
dds.{{name}}
{% endmacro %}
```

Чтобы не указывать каждый раз схему, ведь источник наших данных уже находится в слое `dds`.

Собираем первую витрину: В разрезе годов, стран, жанров отобразить:
- кол-во фильмов
- средний рейтинг
- средний бюджет
- средние сборы в США
- среднюю окупаемость - `sum(box_office) / sum(buget) - 1`

Пишем SQL код:

```sql
SELECT production_year, 
       country, 
       genre,
       count(1) as cnt,
       avg(imdb) as avg_imdb,
       avg(budget) as avg_budget,
       sum(box_office_usa) / sum(budget) - 1 as profit
FROM dds.raw_kp
GROUP BY production_year, country, genre
ORDER BY production_year
```

Для формирования витрины необходимо:
- добавить ссылку на источник данных для витрины
- добавить конфиг: какой тип материализации использовать

У меня получилось так [dm_countries_and_genres.sql](models%2Fdm%2Fdm_countries_and_genres.sql)

Отрендеренный код [dm_countries_and_genres.sql](compiled%2Fdm_countries_and_genres.sql)

Чтобы предоставить витрину, необходим BI - вот мы и добрались до [Metabase](https://www.metabase.com/), так же как и остальную инфру запускаем в docker - 
[инструкция](https://www.metabase.com/docs/latest/installation-and-operation/running-metabase-on-docker)

Например, такой дашборд может получиться:

![dash-0.png](img%2Fdash-0.png)