**Для части по потоковой обработке (Flume) + HBASE**
**1. Модифицировать свой Flume-фгент, созданный в предыдущем ДЗ таким образом, чтобы данные попадали в HDFS и Hbase одновременно**
Создадим HBase базу:
```
[student5_4@manager ~]$ hbase shell
hbase(main):008:0> create 'Student5_4', 'log_messages'
0 row(s) in 4.5400 seconds

=> Hbase::Table - Student5_4
```
База успешно создана:
```
hbase(main):001:0> describe 'Student5_4'
Table Student5_4 is ENABLED
Student5_4
COLUMN FAMILIES DESCRIPTION
{NAME => 'log_messages', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NO
NE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
1 row(s) in 0.3800 seconds
```
Теперь изменим наш Flume, который раньше скидывал данные только в HDFS, изменив конфигурационный файл:
```
# Naming the components on the current agent
Flume5_4.sources  = Execsource
Flume5_4.channels = HDFSChannel HbaseChannel
Flume5_4.sinks    = HDFSsink HbaseSink

# Describing/Configuring the source
Flume5_4.sources.Execsource.type = exec
Flume5_4.sources.Execsource.command = tail -F /var/log/messages
Flume5_4.sources.Execsource.interceptors = TimestampInterceptor
Flume5_4.sources.Execsource.interceptors.TimestampInterceptor.type = timestamp

# Describing/Configuring the HDFS sink
Flume5_4.sinks.HDFSsink.type = hdfs
Flume5_4.sinks.HDFSsink.hdfs.path = /flume/flume_5_4/exec_hdfs_v2/date=%y-%m-%d/
Flume5_4.sinks.HDFSsink.hdfs.filePrefix = events

# Describing/Configuring the Hbase sink
Flume5_4.sinks.HbaseSink.type = hbase
Flume5_4.sinks.HbaseSink.table = Student5_4
Flume5_4.sinks.HbaseSink.columnFamily = log_messages

# Describing/Configuring the channel for HDFS
Flume5_4.channels.HDFSChannel.type = memory
Flume5_4.channels.HDFSChannel.capacity = 10000
Flume5_4.channels.HDFSChannel.transactionCapacity = 100

# Describing/Configuring the channel for Hbase
Flume5_4.channels.HbaseChannel.type = memory
Flume5_4.channels.HbaseChannel.capacity = 10000
Flume5_4.channels.HbaseChannel.transactionCapacity = 100

# Bind the source and sink to the channel
Flume5_4.sources.Execsource.channels = HDFSChannel HbaseChannel
Flume5_4.sinks.HDFSsink.channel = HDFSChannel
Flume5_4.sinks.HbaseSink.channel = HbaseChannel
```
Перелогинимся и проверим поступление данных:
```
hbase(main):001:0> scan 'Student5_4'
ROW                                         COLUMN+CELL
 default23c2bf62-9ee8-401d-a0f1-6cf7996bae3 column=log_messages:pCol, timestamp=1596128199649, value=Jul 30 16:01:01 node3 systemd: Starting Session 755 of user root.
 8
 default26265b73-3b19-499f-bf85-f42eda25bff column=log_messages:pCol, timestamp=1596128199649, value=Jul 30 16:35:03 node3 systemd: Starting User Slice of student5_4.
 f
 default2c645ee8-871b-48f9-9f2c-2b80e1d86b5 column=log_messages:pCol, timestamp=1596128199649, value=Jul 30 15:01:01 node3 systemd: Starting Session 754 of user root.
 8
 default42b2bcb8-54fb-4a51-b194-5910f49d318 column=log_messages:pCol, timestamp=1596128199649, value=Jul 30 16:35:03 node3 systemd: Started Session 756 of user student5_4.
 9
 default43295ba5-adbf-4a8d-96d5-f4ac16a5616 column=log_messages:pCol, timestamp=1596128199649, value=Jul 30 16:35:03 node3 systemd-logind: New session 756 of user student5
 f                                          _4.
 default5dba92c7-f206-42a3-ade9-372da2b4a54 column=log_messages:pCol, timestamp=1596128199649, value=Jul 30 16:01:01 node3 systemd: Started Session 755 of user root.
 e
 default93420c89-9b98-4a0f-b901-25a400e7fbe column=log_messages:pCol, timestamp=1596128199649, value=Jul 30 16:35:03 node3 systemd: Starting Session 756 of user student5_4
 3                                          .
 defaulta5d7995e-61a1-491b-b3d6-b297f95d00f column=log_messages:pCol, timestamp=1596128199649, value=Jul 30 14:01:01 node3 systemd: Starting Session 753 of user root.
 6
 defaultb67ebc97-4017-4643-b393-176c4e10e58 column=log_messages:pCol, timestamp=1596128199649, value=Jul 30 15:01:01 node3 systemd: Started Session 754 of user root.
 c
 defaultba28888a-c9c9-4c36-b91c-04d8666aae9 column=log_messages:pCol, timestamp=1596128199649, value=Jul 30 16:35:03 node3 systemd: Created slice User Slice of student5_4.
 c
 incRow                                     column=log_messages:iCol, timestamp=1596128199777, value=\x00\x00\x00\x00\x00\x00\x00\x0A
11 row(s) in 0.3080 seconds
```
Flume отработал корректно. 

