# Yet another dbt tutorial
## dbt and his place

Как притча во языцах заветные три буквы: 

![etl_spyder.png](img%2Fetl_spyder.png)

В нужной последовательности каждый расставит сам, но нас будет интересовать буковка Т - это и есть домик для dbt =)

--------------------------

![place_of_dbt.drawio.png](img%2Fplace_of_dbt.drawio.png)

**T** - отвечает за трансформации данных внутри хранилища данных, в целом их можно делать любым удобным способом (
хоть выгружать в Excel, крутить там и возвращать обратно), но всё имеет свою цену.

Будем считать, что в некотором смысле оптимально: трансформации выполнять внутри самой СУБД (при помощи SQL):
- данные не выходят за пределы, как минимум DWH
- нет дополнительных Compute мощностей
- нет дополнительных зависимостей (аля Python и прочее)

Базовым решением видится:
- пишем .sql файлики, храним, версионируем (привет, git)
- даже можно CI\CD заделать

Вот примерно также подумали разработчики и появился **dbt**

![init_dbt.drawio.png](img%2Finit_dbt.drawio.png)

- Что такое [SQL](https://aws.amazon.com/ru/what-is/sql/)
- что такое [jinja2](https://ru.wikipedia.org/wiki/Jinja)
- что такое [python](https://aws.amazon.com/ru/what-is/python/)

----------------------------------------------------

## Init

Попробую разобраться как же работает этот dbt:
- не будет повторения dbt-fundamentals
- скорее компиляция из разных статей


👉 [We are need data, go to chapter 1](https://github.com/urevoleg/course-dbt-fundamentals/tree/main/course/chapter-1)