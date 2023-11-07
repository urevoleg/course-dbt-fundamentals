# course-dbt-fundamentals

## Who is Analytic Engineer by dbt

![dbt_anal_engineers.png](img%2Fdbt_anal_engineers.png)

## Started 🚴‍♂️

[Course link](https://courses.getdbt.com/courses/take/fundamentals/)

### 1. Hello, dbt!

Вводные видео о dbt, его месте, парадигмах ETL\ELT

### 2. Analytic Engineer

Кто такой этот Analytic Engineer -> откуда вырос, какими компетенциями обладает и тд

Каждый блок заканчивается квизом, нужно набрать 100% всех ответов, с некоторыми ответами есть несогласие, 
тк даже в самих видео гооврится немного о другом (хотя может я плохо понимаю англ =))

### 3. Setup dbtCloud

Здесь необходимо создать аккаунт в dbtCloud, переходим по [ссылке](https://docs.getdbt.com/), жмахаем `Create free account`

![create_free_account.png](img%2Fcreate_free_account.png)

#### 3.1 Создание первого проекта и подключение источника данных

Ради удобства выбрал BigQuery [link](https://docs.getdbt.com/quickstarts/bigquery?step=1). Тут предстоит пройти небольшой tutorial
, как это всё подключить и настроить, всё расписано по шагам:

![bigquery_guide_steps.png](img%2Fbigquery_guide_steps.png)

ps: тут можно разветится: тк в курсе dbt-fundamentals идут описательные видео про графический интерфейс dbtCloud IDE, а в tutorial по BigQuery
уже начинается небольшое программирование, чтобы было чуть проще можно отвлечься на знакомство с IDE =)

#### Немного о dbt

По-умолчанию, все модели имеют тип `materilazed view`

Models -> находятся в папке `models` проекта
-> 1:1 отображаются в таблицы/вьюхи в вашем DWH

dbt -> build DDL\DML commands


