# dz5
### Домашнее задание
### Настройка autovacuum с учетом особеностей производительности

#### Цель:
#### запустить нагрузочный тест pgbench
#### настроить параметры autovacuum
#### проверить работу autovacuum

Описание/Пошаговая инструкция выполнения домашнего задания:
1. Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
Установить на него PostgreSQL 15 с дефолтными настройками
````
sergt@ubuntu1:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
````
2.  Создать БД для тестов: выполнить pgbench -i postgres test5 <br>
Инсталлируем pgbanch, Создаем базу инициализируем pgbanch
```
sudo apt install postgresql-contrib
postgres=# create database test5;
CREATE DATABASE

sergt@ubuntu1:~$ sudo -u postgres pgbench -i test5
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.08 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 1.06 s (drop tables 0.03 s, create tables 0.13 s, client-side generate 0.47 s, vacuum 0.13 s, primary keys 0.30 s).

```
3. Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
```
sergt@ubuntu1:~$ sudo -u postgres pgbench -c8 -P 6 -T 60  test5
[sudo] password for sergt:
pgbench (15.4 (Ubuntu 15.4-2.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 39.7 tps, lat 193.896 ms stddev 123.811, 0 failed
progress: 12.0 s, 40.3 tps, lat 200.157 ms stddev 108.049, 0 failed
progress: 18.0 s, 38.3 tps, lat 209.512 ms stddev 129.326, 0 failed
progress: 24.0 s, 41.5 tps, lat 193.409 ms stddev 107.830, 0 failed
progress: 30.0 s, 42.0 tps, lat 189.306 ms stddev 103.706, 0 failed
progress: 36.0 s, 39.3 tps, lat 200.433 ms stddev 104.566, 0 failed
progress: 42.0 s, 40.8 tps, lat 199.458 ms stddev 126.010, 0 failed
progress: 48.0 s, 41.3 tps, lat 193.932 ms stddev 116.221, 0 failed
progress: 54.0 s, 41.2 tps, lat 194.110 ms stddev 105.355, 0 failed
progress: 60.0 s, 40.5 tps, lat 197.335 ms stddev 101.185, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 2438
number of failed transactions: 0 (0.000%)
latency average = 197.045 ms
latency stddev = 112.964 ms
initial connection time = 12.606 ms
tps = 40.530185 (without initial connection time)

```
4. Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла <br>
в файл  postgresql.conf в конец добавляем 
```
max_connections = 40
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 512MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 500
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 6553kB
min_wal_size = 4GB
max_wal_size = 16GB
```
Перезагружаем posttgres;
5. Протестировать заново 
```
sergt@ubuntu1:~$ sudo -u postgres pgbench -c8 -P 6 -T 60  test5
pgbench (15.4 (Ubuntu 15.4-2.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 37.5 tps, lat 208.063 ms stddev 125.432, 0 failed
progress: 12.0 s, 39.2 tps, lat 204.007 ms stddev 129.550, 0 failed
progress: 18.0 s, 41.0 tps, lat 193.966 ms stddev 107.886, 0 failed
progress: 24.0 s, 41.2 tps, lat 196.007 ms stddev 106.789, 0 failed
progress: 30.0 s, 40.7 tps, lat 196.587 ms stddev 137.684, 0 failed
progress: 36.0 s, 43.0 tps, lat 185.539 ms stddev 106.950, 0 failed
progress: 42.0 s, 40.8 tps, lat 197.212 ms stddev 109.111, 0 failed
progress: 48.0 s, 42.3 tps, lat 189.091 ms stddev 105.393, 0 failed
progress: 54.0 s, 41.2 tps, lat 193.002 ms stddev 101.198, 0 failed
progress: 60.0 s, 42.5 tps, lat 188.433 ms stddev 99.662, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 2464
number of failed transactions: 0 (0.000%)
latency average = 195.039 ms
latency stddev = 113.479 ms
initial connection time = 14.960 ms
tps = 40.944575 (without initial connection time)

```
Что изменилось и почему? <br>
__было__   number of transactions actually processed: 2438 <br>
__стало__  number of transactions actually processed: 2464 <br>

__было__ latency average = 197.045 ms __стало__	latency average = 195.039 ms <br>
__было__ latency stddev = 112.964 ms	__стало__ latency stddev = 113.479 ms<br>
__было__ initial connection time = 12.606 ms __стало__ initial connection time = 14.960 ms<br>
__было__ tps = 40.530185 (without initial connection time) <br>
__стало__ tps = 40.944575 (without initial connection time)<br>



6. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
```
test5=# CREATE TABLE student( id serial, fio char(100) ) ;
CREATE TABLE
test5=# INSERT INTO   student(fio) SELECT 'noname' FROM generate_series(1,1000000);
INSERT 0 1000000

```
    - Посмотреть размер файла с таблицей 
```
test5=# SELECT pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty
----------------
 135 MB
(1 row)

```
    - 5 раз обновить все строчки и добавить к каждой строчке любой символ
```
test5=# update student set fio = 'name';
UPDATE 1000000
test5=# update student set fio = 'name2';
UPDATE 1000000
test5=# update student set fio = 'name3';
UPDATE 1000000
test5=# update student set fio = 'name4';
UPDATE 1000000
```
    - Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
```
test5=# select c.relname,
current_setting('autovacuum_vacuum_threshold') as av_base_thresh,
current_setting('autovacuum_vacuum_scale_factor') as av_scale_factor,
(current_setting('autovacuum_vacuum_threshold')::int +
(current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples)) as av_thresh,
s.n_dead_tup
from pg_stat_user_tables s join pg_class c ON s.relname = c.relname
where s.n_dead_tup > (current_setting('autovacuum_vacuum_threshold')::int
+ (current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples));
 relname | av_base_thresh | av_scale_factor | av_thresh | n_dead_tup
---------+----------------+-----------------+-----------+------------
 student | 50             | 0.2             |    200050 |    3999824
(1 row)

```
    - Подождать некоторое время, проверяя, пришел ли автовакуум <br>
Пришел
```
test5=# select c.relname,
current_setting('autovacuum_vacuum_threshold') as av_base_thresh,
current_setting('autovacuum_vacuum_scale_factor') as av_scale_factor,
(current_setting('autovacuum_vacuum_threshold')::int +
(current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples)) as av_thresh,
s.n_dead_tup
from pg_stat_user_tables s join pg_class c ON s.relname = c.relname
where s.n_dead_tup > (current_setting('autovacuum_vacuum_threshold')::int
+ (current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples));
 relname | av_base_thresh | av_scale_factor | av_thresh | n_dead_tup
---------+----------------+-----------------+-----------+------------
(0 rows)
```
    - 5 раз обновить все строчки и добавить к каждой строчке любой символ
```
test5=# update student set fio = 'nameA';
UPDATE 1000000
test5=# update student set fio = 'nameB';
UPDATE 1000000
test5=# update student set fio = 'nameC';
UPDATE 1000000
test5=# update student set fio = 'nameD';
UPDATE 1000000
test5=# update student set fio = 'nameE';
UPDATE 1000000

```
      Посмотреть размер файла с таблицей
```
test5=# SELECT pg_size_pretty(pg_total_relation_size('student'));clear
 pg_size_pretty
----------------
 808 MB
(1 row)

```
7. Отключить Автовакуум на конкретной таблице
```
test5=# ALTER TABLE student SET (autovacuum_enabled = false);
ALTER TABLE
```
     - 10 раз обновить все строчки и добавить к каждой строчке любой символ
```
test5=# update student set fio = 'nameA1';
UPDATE 1000000
test5=# update student set fio = 'nameA2';
UPDATE 1000000
test5=# update student set fio = 'nameA3';
UPDATE 1000000
test5=# update student set fio = 'nameA4';
UPDATE 1000000
test5=# update student set fio = 'nameA5';
UPDATE 1000000
test5=# update student set fio = 'nameA6';
UPDATE 1000000
test5=# update student set fio = 'nameA7';
UPDATE 1000000
test5=# update student set fio = 'nameA8';
UPDATE 1000000
test5=# update student set fio = 'nameA9';
UPDATE 1000000
test5=# update student set fio = 'nameA10';
UPDATE 1000000
```
     - Посмотреть размер файла с таблицей
```
test5=# SELECT pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty
----------------
 1482 MB
(1 row)

test5=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 student |    1666567 |    9998059 |    599 | 2023-10-17 22:31:43.165604+00
(1 row)

```
     - Объясните полученный результат
Объяснение такое
т.к. не был включен вакуум записи помеченнные как  удаленные не очищались и при следедущей вставке не могли быть испольованы<br>
таким образом каждый апдейт увеливичевал количесто записей кратно начальному количеству.
     - Не забудьте включить автовакуум)
```
test5=# ALTER TABLE student SET (autovacuum_enabled = true);                                                                                                ALTER TABLE

```
9. Задание со *:
      Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
      Не забыть вывести номер шага цикла.