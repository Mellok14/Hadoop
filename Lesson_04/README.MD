Дана большая таблица
```
create external table hive_db.citation_data
(
oci string,
citing string,
cited string,
creation string,
timespan string,
journal_sc string,
author_sc string
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
location '/test_datasets/citation'
```
Её размер вот такой:
```
hdfs dfs -du -h -s /test_datasets/citation
97.2 G 291.5 G /test_datasets/citation
```
Что вам нужно сделать:

**1. Создать две таблицы в форматах PARQUET/ORC/AVRO c компрессией и без оной.**
**a. Первая без компрессии в произвольном формате (выберите один из форматов хранения)**
**b. Вторая с компрессией в том-же формате. (выберите произвольный способ сжатия из поддерживаемых)**

Возьмем формат PARQUET. Первый вариант без компрессии:

```
-hdfs dfs -mkdir -p /user/student5_4/parquet_data

create external table student5_4.parquet_data (
    oci string,
    citing string,
    cited string,
    creation string,
    timespan string,
    journal_sc string,
    author_sc string
)
STORED AS PARQUET
LOCATION '/user/student5_4/parquet_data';
```

**2. Заполнить данными из большой таблицы hive_db.citation_data**

```
insert into student5_4.parquet_data
select * from hive_db.citation_data;
```
Запрос начал выполнятся и в YARN появилось приложение:
![Скриншот CLOUDERA MANAGER](https://github.com/Mellok14/Hadoop/blob/master/Lesson_04/App2.png)
