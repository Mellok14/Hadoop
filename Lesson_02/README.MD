**1.[Задание на 5++] Повторить вот этот пример https://www.michael-noll.com/tutorials/writing-an-hadoop-mapreduce-program-in-python/ .**

Сначала найдём jar-файл:

`find /opt -name "hadoop-streaming.jar"`.

Зададим для этого файла переменную:

`export YARN_EXAMPLES=/opt/cloudera/parcels/CDH-5.16.2-1.cdh5.16.2.p0.8/lib/hadoop-mapreduce`.

Создадим и заполним два файла mapper.py и reducer.py в директории `/students/student5_4/python_mapreduce`:
```
cat mapper.py

#!/usr/bin/env python

import sys

for line in sys.stdin:
    line = line.strip()
    words = line.split()
    for word in words:
        print '%s\t%s' % (word, 1)
```
```
cat reducer.py

#!/usr/bin/env python

from operator import itemgetter
import sys

current_word = None
current_count = 0
word = None

for line in sys.stdin:
    line = line.strip()

    word, count = line.split('\t', 1)

    try:
        count = int(count)
    except ValueError:
        continue

    if current_word == word:
        current_count += count
    else:
        if current_word:
            print '%s\t%s' % (current_word, current_count)
        current_count = count
        current_word = word

if current_word == word:
    print '%s\t%s' % (current_word, current_count)
```    
    
Изменим права для этих вайлов, во избежание ошибок:
```
chmod +x ./mapper.py
chmod +x ./reducer.py
```
Проверим работоспособность файлов на тестовых дыннх:

```
echo 'foo foo quux labs foo bar quux' | ./mapper.py

foo     1
foo     1
quux    1
labs    1
foo     1
bar     1
quux    1
```
```
echo "foo foo quux labs foo bar quux" | ./mapper.py | sort -k1,1 | ./reducer.py

bar     1
foo     3
labs    1
quux    2
```
Все работает, теперь можно приступать к загрузке тестовых данных. Сначала загрузим данные на локальную машину (для примера возьмем одну книгу):

`wget http://www.gutenberg.org/files/4300/4300-0.txt`.

Загрузим текстовый файл на HDFS Hadoop:

`hdfs dfs  -copyFromLocal /students/student5_4/test_file/* /student5_4/test_file`.

Запустим jar-файл:
```
hadoop jar $YARN_EXAMPLES/hadoop-streaming.jar \
 -file /students/student5_4/python_mapreduce/mapper.py \
 -mapper /students/student5_4/python_mapreduce/mapper.py \
 -file /students/student5_4/python_mapreduce/reducer.py \
 -reducer /students/student5_4/python_mapreduce/reducer.py \
 -input /student5_4/test_file/4300-0.txt \
 -output /student5_4/test_file/output
```
В ходе выполнения команды появилось приложение `application_1592839005008_0020`:
![Скриншот CLOUDERA MANAGER](https://github.com/Mellok14/Hadoop/raw/master/Lesson_02/application_1592839005008_0020.png)

Проверим результат работы:
```
hdfs dfs -ls /student5_4/test_file/output

Found 7 items
-rw-r--r--   3 student5_4 supergroup          0 2020-06-25 18:31 /student5_4/test_file/output/_SUCCESS
-rw-r--r--   3 student5_4 supergroup      87964 2020-06-25 18:31 /student5_4/test_file/output/part-00000
-rw-r--r--   3 student5_4 supergroup      89331 2020-06-25 18:31 /student5_4/test_file/output/part-00001
-rw-r--r--   3 student5_4 supergroup      88506 2020-06-25 18:31 /student5_4/test_file/output/part-00002
-rw-r--r--   3 student5_4 supergroup      89230 2020-06-25 18:31 /student5_4/test_file/output/part-00003
-rw-r--r--   3 student5_4 supergroup      88674 2020-06-25 18:31 /student5_4/test_file/output/part-00004
-rw-r--r--   3 student5_4 supergroup      86978 2020-06-25 18:31 /student5_4/test_file/output/part-00005
```
```
hdfs dfs -cat /student5_4/test_file/output/part-00000

#фрагмент ответа
—U.     1
—Up     1
—Wait   4
—Wait.  2
—Was    8
—We’ll  5
—Which  4
—Whom   1
```
Программа отработала корректно.

