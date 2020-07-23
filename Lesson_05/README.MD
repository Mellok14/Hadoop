**Для части по SQOOP
Провести импорт таблицы из вашего сервера БД в Hadoop с использованием SQOOP в любых двух вариантах из перечисленных ниже.
a. в Hive-таблицу (--hive-import)
b. в HDFS в формате avro (--as-avrodatafile)
c. в HDFS в формате sequencefile (--as-sequencefile)
Если у вас нет своего сервера то можно использовать тот Postgres, который я показал на лекции.**

Импорт в Hive таблицу:

```
[student5_4@manager ~]$ ssh node3.novalocal
[student5_4@node3 ~]$ sqoop import --connect jdbc:postgresql://node3.novalocal/pg_db --username exporter -P --table character --hive-import --hive-database student5_4 --hive-table character
```
Убедимся, что все отработало корректно:

```
SELECT * FROM character LIMIT 5;
```

Рузультат: 

| character.charid | character.charname | character.abbrev  | character.description | character.speechcount |
|------------------|--------------------|-------------------|-----------------------|-----------------------|
| 1apparition-mac  | First Apparition   | First Apparition  | null                  | 1                     |
| 1citizen         | First Citizen      | First Citizen     | null                  | 3                     |
| 1conspirator     | First Conspirator  | First Conspirator | null                  | 3                     |
| 1gentleman-oth   | First Gentleman    | First Gentleman   | null                  | 1                     |
| 1goth            | First Goth         | First Goth        | null                  | 4                     |

Импорт в HDFS в формате AVRO:

```
[student5_4@node3 ~]$ sqoop import --connect jdbc:postgresql://node3.novalocal/pg_db --table work --target-dir /student5_4/tables/work --as-avrodatafile --username exporter -P
```

Результат:

```
[student5_4@node3 ~]$ hdfs dfs -ls /student5_4/tables/work
Found 6 items
-rw-r--r--   3 student5_4 supergroup          0 2020-07-21 17:52 /student5_4/tables/work/_SUCCESS
-rw-r--r--   3 student5_4 supergroup       1325 2020-07-21 17:52 /student5_4/tables/work/part-m-00000.avro
-rw-r--r--   3 student5_4 supergroup       3570 2020-07-21 17:52 /student5_4/tables/work/part-m-00001.avro
-rw-r--r--   3 student5_4 supergroup        999 2020-07-21 17:52 /student5_4/tables/work/part-m-00002.avro
-rw-r--r--   3 student5_4 supergroup       3592 2020-07-21 17:52 /student5_4/tables/work/part-m-00003.avro
-rw-r--r--   3 student5_4 supergroup       1089 2020-07-21 17:52 /student5_4/tables/work/part-m-00004.avro
```

**Для части по потоковой обработке (Flume)**
**1. Создать Flume-агент с именем, соответствующим имени своего пользователя (например Flume4_20)**
**2. Создать любой Flume поток используя Flume сервис соответствующего номера.**
**• Тип источника источник – exeс**
**• Тип канала – memory**
**• Тип слива – hdfs**
**3. Создать поверх данных в hdfs таблицу через которую можно просмотреть полученные данные.**
**4. [ОБЯЗАТЕЛЬНО] Убедиться что данные поступают в слив.**

Созданим новый Flume и заполним конфигурационный файл:
```
# Naming the components on the current agent
Flume5_4.sources  = Execsource
Flume5_4.channels = MemChannel
Flume5_4.sinks    = HDFSsink

# Describing/Configuring the source
Flume5_4.sources.Execsource.type = exec
Flume5_4.sources.Execsource.command = tail -F /var/log/messages
Flume5_4.sources.Execsource.interceptors = TimestampInterceptor
Flume5_4.sources.Execsource.interceptors.TimestampInterceptor.type = timestamp

# Describing/Configuring the HDFS sink
Flume5_4.sinks.HDFSsink.type = hdfs
Flume5_4.sinks.HDFSsink.hdfs.path = /flume/flume_5_4/exec_hdfs/%y-%m-%d/
Flume5_4.sinks.HDFSsink.hdfs.filePrefix = events

# Describing/Configuring the channel
Flume5_4.channels.MemChannel.type = memory
Flume5_4.channels.MemChannel.capacity = 10000
Flume5_4.channels.MemChannel.transactionCapacity = 100

# Bind the source and sink to the channel
Flume5_4.sources.Execsource.channels = MemChannel
Flume5_4.sinks.HDFSsink.channel = MemChannel
```
Убедимся что все работает корректно и файлы создаются:
```
[student5_4@node3 log]$ hdfs dfs -ls -R /flume/flume_5_4/exec_hdfs/
drwxr-xr-x   - flume flume          0 2020-07-23 14:34 /flume/flume_5_4/exec_hdfs/20-07-23
-rw-r--r--   3 flume flume        988 2020-07-23 14:34 /flume/flume_5_4/exec_hdfs/20-07-23/events.1595514820139
```
Поверх дынных в HDFS создадим таблицу в Hive:
```
CREATE EXTERNAL TABLE student5_4.flume_logs (
    text string
)
PARTITIONED BY (`date` string)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/flume/flume_5_4/exec_hdfs/';

#set hive.msck.path.validation=ignore;
msck repair table flume_logs;

select * from flume_logs;
```
