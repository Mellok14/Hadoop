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

Посмотрим на размер данных:

```
[student5_4@manager ~]$ hdfs dfs -du -h -s /user/student5_4/parquet_data
77.3 G  232.3 G  /user/student5_4/parquet_data
```

Теперь создадим таблицу с компрессией:

```
SET hive.exec.compress.output=true;
	SET parquet.compression=SNAPPY;

create external table student5_4.parquet_data_compress (
    oci string,
    citing string,
    cited string,
    creation string,
    timespan string,
    journal_sc string,
    author_sc string
)
STORED AS PARQUET
LOCATION '/user/student5_4/parquet_data_compress';
```
Заполним данными из большой таблицы hive_db.citation_data

```
insert into student5_4.parquet_data_compress
select * from hive_db.citation_data;
```
Запрос начал выполнятся и в YARN появилось приложение:
![Скриншот CLOUDERA MANAGER](https://github.com/Mellok14/Hadoop/blob/master/Lesson_04/App3.png)

**3. Верифицировать способ сжатия при помощи parquet-tools/avro-tools/hive --orcfiledump**

```
[student5_4@manager ~]$ parquet-tools meta hdfs://manager.novalocal:8020/user/student5_4/parquet_data_compress/000000_0
creator:     parquet-mr version 1.5.0-cdh5.16.2 (build ${buildNumber})

file schema: hive_schema
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
oci:         OPTIONAL BINARY O:UTF8 R:0 D:1
citing:      OPTIONAL BINARY O:UTF8 R:0 D:1
cited:       OPTIONAL BINARY O:UTF8 R:0 D:1
creation:    OPTIONAL BINARY O:UTF8 R:0 D:1
timespan:    OPTIONAL BINARY O:UTF8 R:0 D:1
journal_sc:  OPTIONAL BINARY O:UTF8 R:0 D:1
author_sc:   OPTIONAL BINARY O:UTF8 R:0 D:1

row group 1: RC:2628577 TS:345954345
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
oci:          BINARY SNAPPY DO:0 FPO:4 SZ:63395310/231280810/3.65 VC:2628577 ENC:PLAIN,BIT_PACKED,RLE
citing:       BINARY SNAPPY DO:0 FPO:63395314 SZ:2302676/33882008/14.71 VC:2628577 ENC:PLAIN,BIT_PACKED,RLE,PLAIN_DICTIONARY
cited:        BINARY SNAPPY DO:0 FPO:65697990 SZ:41773020/76403880/1.83 VC:2628577 ENC:PLAIN,BIT_PACKED,RLE
creation:     BINARY SNAPPY DO:0 FPO:107471010 SZ:42863/61498/1.43 VC:2628577 ENC:BIT_PACKED,RLE,PLAIN_DICTIONARY
timespan:     BINARY SNAPPY DO:0 FPO:107513873 SZ:3698820/4255628/1.15 VC:2628577 ENC:BIT_PACKED,RLE,PLAIN_DICTIONARY
journal_sc:   BINARY SNAPPY DO:0 FPO:111212693 SZ:43495/61390/1.41 VC:2628577 ENC:BIT_PACKED,RLE,PLAIN_DICTIONARY
author_sc:    BINARY SNAPPY DO:0 FPO:111256188 SZ:7317/9131/1.25 VC:2628577 ENC:BIT_PACKED,RLE,PLAIN_DICTIONARY
```
Как можно увидеть тип сжатия SNAPPY был применен

**4. Посмотреть на получившийся размер данных.**

Посмотрим на размер данных (с компрессией):

```
[student5_4@manager ~]$ hdfs dfs -du -h -s /user/student5_4/parquet_data_compress
22.7 G  68.2 G  /user/student5_4/parquet_data_compress
```
**5. Сделать выводы о эффективности хранения и сжатия.**

Как можно убедиться сжатие позвовило снизить объем занимаемых данных в формате PARQUET с 77.3 до 22.7 гигабайт, т.е. в 3.4 раза. 
К слову исходные данные в формате CSV занимали 97.2 гигабайта.

```
[student5_4@manager ~]$ hdfs dfs -du -h -s /test_datasets/citation
97.2 G  194.4 G  /test_datasets/citation
```
