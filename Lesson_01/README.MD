**1. [Исследовательское] Сколько узлов одновременно можно потерять без потери данных в кластере из 10 узлов? Из 100 узлов?**
При стандартном факторе репликации - 3: для 10 узлов можно потерять только 2 узла, для 100 узлов можно потерять также только 2 узла.
Есть вероятность что копии файла находятся именно на вышедших из строя узлах. Вероятность такого на 100 узлах очень мала, но она все же есть. 

**2. Опробовать консольные утилиты для работы с кластером**

**2.1 Создать/скопировать/удалить папку**

`hdfs dfs -mkdir /student5_4`.

`hdfs dfs -cp /student5_4 /student5_4_1`.

`hdfs dfs -rm -R -skipTrash /student5_4_1`.

**2.2 Положить в HDFS любой файл**

`hdfs dfs -copyFromLocal /home/student5_4/.ssh/id_rsa.pub /student5_4/`

**2.3 Скопировать/удалить этот файл**

`hdfs dfs -cp /student5_4/.ssh/id_rsa.pub /student5_4//id_rsa.pub`. 

`hdfs dfs -rm /student5_4//id_rsa.pub`.

**2.4 Просмотреть размер любой папки**

`hdfs dfs -du /student5_4/`.

**2.5 Посмотреть как файл хранится на файловой системе (см. команду fsck)**

`hdfs fsck /student5_4/`.

**2.6 Установить нестандартный фактор репликации (см. команду setrep)**

`hdfs dfs -setrep 1 /student5_4/`.

**3.1 Опробовать rest-доступ для работы с кластером, используя утилиту CURL.**

`curl -X GET 'http://node2.novalocal:14000/webhdfs/v1/data?user.name=student5_4&op=LISTSTATUS'`.

**3.2 Опробовать rest-доступ для работы с кластером, используя python3 и библиотеку requests.**
```
import requests
from pprint import pprint
response = requests.get('http://node2.novalocal:14000/webhdfs/v1/data?user.name=student5_4&op=LISTSTATUS')
pprint(response)
```
Почему то выдает ошибку: `Failed to establish a new connection: [Errno 11001] getaddrinfo failed`. Никак не смог понять в чем проблема.

**4. [Для любителей программирования] Достучаться до файловой системы используя python и библиотеку libhdfs3**
```
import os.path
import sys
sys.path.append(os.path.join(os.path.dirname(__file__), '..'))
sys.path.append('C:\\Users\\Mell\\AppData\\Local\\Programs\\Python\\Python38-32\\Lib\\site-packages\\hdfs3')
from hdfs3 import HDFileSystem
hdfs=HDFileSystem(host='node2.novalocal',port=14000)
hdfs.ls('/opt')
```
Тоже ошибка: `Could not find module 'libhdfs3.so'. Try using the full path with constructor syntax.`
Как только не пробовал - не смог обойти. 
