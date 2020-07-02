**1. Скачать любой датасет из списка ниже.**

https://www.kaggle.com/shuyangli94/food-com-recipes-and-user-interactions
https://www.kaggle.com/datasnaek/youtube-new
https://www.kaggle.com/akhilv11/border-crossing-entry-data
https://www.kaggle.com/tristan581/17k-apple-app-store-strategy-games
https://www.kaggle.com/gustavomodelli/forest-fires-in-brazil

Возьмем датасет: `border-crossing-entry-data`

**2. Загрузить этот датасет в HDFS в свою домашнюю папку.**

Выделим из датасета отдельную таблицу `State` для логичного выполнения JOIN запроса.
Для этого на локальной машине выполним SQL запрос, после чего сохраним файлы в csv и json: 

```
create table state (
	id INT auto_increment primary key,
	state VARCHAR(255)
) as select distinct state from border_crossing_entry_data;

select * from state;

update  border_crossing_entry_data bc
inner join state s
on s.state = bc.state
set bc.state = s.id;

select * from border_crossing_entry_data;
```
Файлы загружены в HDFS:

```
[student5_4@manager ~]$ hdfs dfs -ls /user/student5_4
Found 5 items
drwx------   - student5_4 student5_4          0 2020-06-25 18:31 /user/student5_4/.staging
-rw-r--r--   3 student5_4 student5_4   37079639 2020-07-02 16:57 /user/student5_4/border_crossing.csv
-rw-r--r--   3 student5_4 student5_4   66898598 2020-07-02 16:57 /user/student5_4/border_crossing.json
-rw-r--r--   3 student5_4 student5_4        223 2020-07-02 16:57 /user/student5_4/state.csv
-rw-r--r--   3 student5_4 student5_4        478 2020-07-02 16:57 /user/student5_4/state.json
```

**3. Создать собственную базу данных в HIVE.**

`create database student5_4;`

**4. Создать EXTERNAL таблицы внутри базы данных с использованием всех загруженных файлов. Один файл – одна таблица.**

```
create external table student5_4.border_crossing_v1
(
    port_name string,
    state int,
    port_code int,
    border string,
    `date` date,
    measure int,
    value int,
    `location` string
)
ROW FORMAT SERDE
    'org.apache.hadoop.hive.serde2.OpenCSVSerde'
TBLPROPERTIES (
    'serialization.null.format' = '',
    'skip.header.line.count' = '1')
;

LOAD DATA INPATH '/user/student5_4/border_crossing.csv' INTO TABLE student5_4.border_crossing_v1;

select * from student5_4.border_crossing_v1;

create external table student5_4.state
(
    id int,
    state string
)
ROW FORMAT SERDE
    'org.apache.hadoop.hive.serde2.OpenCSVSerde'
TBLPROPERTIES (
    'serialization.null.format' = '',
    'skip.header.line.count' = '1')
;

LOAD DATA INPATH '/user/student5_4/state.csv' INTO TABLE student5_4.state;

select * from  student5_4.state;
```
**5. Сделать любой отчет по загруженным данным используя груповые и агрегатные функции.**

```
select measure, sum(value)
from student5_4.border_crossing_v1
group by measure;
```
Реpультат выполнения запроса:

| measure                     | _c1          |
|-----------------------------|--------------|
| Bus Passengers              | 142330871.0  |
| Buses                       | 8543756.0    |
| Pedestrians                 | 1044218114.0 |
| Personal Vehicle Passengers | 5457391275.0 |
| Personal Vehicles           | 2559691192.0 |
| Rail Containers Empty       | 21139444.0   |
| Rail Containers Full        | 38288393.0   |
| Train Passengers            | 6197450.0    |
| Trains                      | 903864.0     |
| Truck Containers Empty      | 64046035.0   |
| Truck Containers Full       | 177190288.0  |
| Trucks                      | 253654160.0  |

