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

**Для части по потоковой обработке (Flume)
1. Создать Flume-агент с именем, соответствующим имени своего пользователя (например Flume4_20)
2. Создать любой Flume поток используя Flume сервис соответствующего номера.
• Тип источника источник – exeс
• Тип канала – memory
• Тип слива – hdfs
3. Создать поверх данных в hdfs таблицу через которую можно просмотреть полученные данные.
4. [ОБЯЗАТЕЛЬНО] Убедиться что данные поступают в слив.**
