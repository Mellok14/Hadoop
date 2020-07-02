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

**6. Сделать любой отчет по загруженным данным используя JOIN.**

```
select s.state, sum(bc.value), bc.measure as sum_value
from student5_4.border_crossing_v1 bc
left join student5_4.state s
on bc.state = s.id
WHERE bc.measure = 'Trucks'
group by s.state, bc.measure
order by sum_value limit 1;
```
Реpультат выполнения запроса:

| s.state | _c1      | sum_value |
|---------|----------|-----------|
| Alaska  | 235497.0 | Trucks    |

**7.[Продвинутый вариант] Сделать все вышеперечисленное с использованием JSON SerDe. Подсказка: см в сторону команды «ADD JAR»**

Пытался сделать: создал таблицу, загрузил в нее данные без ошибок. Но при выполнении любого select запроса - очень длинный лог ошибки.
Применял get_json_object - не помогло.

```
ADD JAR /opt/cloudera/parcels/CDH-5.16.2-1.cdh5.16.2.p0.8/lib/hive-hcatalog/share/hcatalog/hive-hcatalog-core.jar;

drop table student5_4.border_crossing_json;

create external table student5_4.border_crossing_json
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
    'org.apache.hive.hcatalog.data.JsonSerDe'
TBLPROPERTIES (
    'serialization.null.format' = '',
    'skip.header.line.count' = '1')
;

LOAD DATA INPATH '/user/student5_4/border_crossing.json' INTO TABLE student5_4.border_crossing_json;

select * from student5_4.border_crossing_json;
'''

После select:

'''
Bad status for request TFetchResultsReq(fetchType=0, operationHandle=TOperationHandle(hasResultSet=True, modifiedRowCount=None, operationType=0, operationId=THandleIdentifier(secret='WK\xc45\x03]D\x00\x9e<\xc0\xe8\xc0g,\xe9', guid='\xe9\x0f\x81\xb5~\xd9L$\xb9\xfd<\xbcd\x84^\xec')), orientation=4, maxRows=100): TFetchResultsResp(status=TStatus(errorCode=0, errorMessage='java.io.IOException: org.apache.hadoop.hive.serde2.SerDeException: org.codehaus.jackson.JsonParseException: Current token (VALUE_STRING) not numeric, can not use numeric value accessors\n at [Source: java.io.ByteArrayInputStream@3f204368; line: 1, column: 35]', sqlState=None, infoMessages=['*org.apache.hive.service.cli.HiveSQLException:java.io.IOException: org.apache.hadoop.hive.serde2.SerDeException: org.codehaus.jackson.JsonParseException: Current token (VALUE_STRING) not numeric, can not use numeric value accessors\n at [Source: java.io.ByteArrayInputStream@3f204368; line: 1, column: 35]:25:24', 'org.apache.hive.service.cli.operation.SQLOperation:getNextRowSet:SQLOperation.java:463', 'org.apache.hive.service.cli.operation.OperationManager:getOperationNextRowSet:OperationManager.java:294', 'org.apache.hive.service.cli.session.HiveSessionImpl:fetchResults:HiveSessionImpl.java:769', 'sun.reflect.GeneratedMethodAccessor29:invoke::-1', 'sun.reflect.DelegatingMethodAccessorImpl:invoke:DelegatingMethodAccessorImpl.java:43', 'java.lang.reflect.Method:invoke:Method.java:498', 'org.apache.hive.service.cli.session.HiveSessionProxy:invoke:HiveSessionProxy.java:78', 'org.apache.hive.service.cli.session.HiveSessionProxy:access$000:HiveSessionProxy.java:36', 'org.apache.hive.service.cli.session.HiveSessionProxy$1:run:HiveSessionProxy.java:63', 'java.security.AccessController:doPrivileged:AccessController.java:-2', 'javax.security.auth.Subject:doAs:Subject.java:422', 'org.apache.hadoop.security.UserGroupInformation:doAs:UserGroupInformation.java:1924', 'org.apache.hive.service.cli.session.HiveSessionProxy:invoke:HiveSessionProxy.java:59', 'com.sun.proxy.$Proxy21:fetchResults::-1', 'org.apache.hive.service.cli.CLIService:fetchResults:CLIService.java:462', 'org.apache.hive.service.cli.thrift.ThriftCLIService:FetchResults:ThriftCLIService.java:696', 'org.apache.hive.service.cli.thrift.TCLIService$Processor$FetchResults:getResult:TCLIService.java:1553', 'org.apache.hive.service.cli.thrift.TCLIService$Processor$FetchResults:getResult:TCLIService.java:1538', 'org.apache.thrift.ProcessFunction:process:ProcessFunction.java:39', 'org.apache.thrift.TBaseProcessor:process:TBaseProcessor.java:39', 'org.apache.hive.service.auth.TSetIpAddressProcessor:process:TSetIpAddressProcessor.java:56', 'org.apache.thrift.server.TThreadPoolServer$WorkerProcess:run:TThreadPoolServer.java:286', 'java.util.concurrent.ThreadPoolExecutor:runWorker:ThreadPoolExecutor.java:1149', 'java.util.concurrent.ThreadPoolExecutor$Worker:run:ThreadPoolExecutor.java:624', 'java.lang.Thread:run:Thread.java:748', '*java.io.IOException:org.apache.hadoop.hive.serde2.SerDeException: org.codehaus.jackson.JsonParseException: Current token (VALUE_STRING) not numeric, can not use numeric value accessors\n at [Source: java.io.ByteArrayInputStream@3f204368; line: 1, column: 35]:29:4', 'org.apache.hadoop.hive.ql.exec.FetchOperator:getNextRow:FetchOperator.java:508', 'org.apache.hadoop.hive.ql.exec.FetchOperator:pushRow:FetchOperator.java:415', 'org.apache.hadoop.hive.ql.exec.FetchTask:fetch:FetchTask.java:140', 'org.apache.hadoop.hive.ql.Driver:getResults:Driver.java:2071', 'org.apache.hive.service.cli.operation.SQLOperation:getNextRowSet:SQLOperation.java:458', '*org.apache.hadoop.hive.serde2.SerDeException:org.codehaus.jackson.JsonParseException: Current token (VALUE_STRING) not numeric, can not use numeric value accessors\n at [Source: java.io.ByteArrayInputStream@3f204368; line: 1, column: 35]:30:1', 'org.apache.hive.hcatalog.data.JsonSerDe:deserialize:JsonSerDe.java:174', 'org.apache.hadoop.hive.ql.exec.FetchOperator:getNextRow:FetchOperator.java:489', '*org.codehaus.jackson.JsonParseException:Current token (VALUE_STRING) not numeric, can not use numeric value accessors\n at [Source: java.io.ByteArrayInputStream@3f204368; line: 1, column: 35]:36:6', 'org.codehaus.jackson.JsonParser:_constructError:JsonParser.java:1292', 'org.codehaus.jackson.impl.JsonParserMinimalBase:_reportError:JsonParserMinimalBase.java:385', 'org.codehaus.jackson.impl.JsonNumericParserBase:_parseNumericValue:JsonNumericParserBase.java:399', 'org.codehaus.jackson.impl.JsonNumericParserBase:getIntValue:JsonNumericParserBase.java:254', 'org.apache.hive.hcatalog.data.JsonSerDe:extractCurrentField:JsonSerDe.java:274', 'org.apache.hive.hcatalog.data.JsonSerDe:populateRecord:JsonSerDe.java:213', 'org.apache.hive.hcatalog.data.JsonSerDe:deserialize:JsonSerDe.java:169'], statusCode=3), results=None, hasMoreRows=None)
'''
