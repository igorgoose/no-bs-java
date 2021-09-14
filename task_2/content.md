# RDBMS(SQL)
DBMS = СУБД = Система управления базами данных
RDBMS = Система управления реляционными базами данных

## Зачем?

Приложениям часто нужно сохранять данные, которые должны быть 
доступны всегда. Т.е. после перезапуска приложения/компьютера мы хотим,
чтобы мы могли получить наши данные(например, мы хотим, чтобы данные о 
пользователях приложения не стирались после перезапуска приложения). 
В этом случае первое, что приходит в голову - сохранять данные в 
файлах в файловой системе. Такое решение
может подойти для каких-то простых случаев. Однако при решении более 
сложных задач с большим количеством данных возникает множество трудностей:

1. Как решить вопрос с одновременным изменением одних данных
2. Необходимость криво извлекать данные из файлов 
   (нужно самому придумывать механизмы поиска по критериям и т.д.)
3. Скорость чтения/записи, в особенности частичных данных или с сложными критериями
   (обычные файлы не оптимизированы для таких операций)
   
Поэтому умные люди придумали СУБД, которые решают все эти проблемы. 
В частности, СУРБД предоставляют удобный язык запросов SQL, с помощью которого
можно выполнять всевозможные операции создания, обновления, чтения, удаления данных.

## Реляционные базы данных 
Вкратце про что это такое можно почитать [здесь](https://younglinux.info/sqlite/db)
(на 3 минуты). 

## SQL
[Видос для начала](https://www.youtube.com/watch?v=7S_tz1z_5bA&ab_channel=ProgrammingwithMosh) (который на 3 часа)

Установи любую субд(Postgresql, MySql, Oracle(не люблю её)).

[Задания по sql](https://github.com/mjc-school/MJC-School/tree/master/stage%20%232/module%20%232%20SQL) 
(там есть ссылки на краткие инструкции по запросам с примерами). Нужно сделать 1-12 задания.
В корне проекта создай папку sql, в неё закидывай файлы №_задания.sql со скриптами, которые получились.