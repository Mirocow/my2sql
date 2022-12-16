# my2sql
Версия go инструмента синтаксического анализа MySQL binlog, анализируя MySQL binlog, 
может генерировать необработанный SQL, откатывать SQL, вставлять SQL, который удаляет первичный ключ, и т.д., 
а также может генерировать статистику DML.Подобными инструментами являются binlog2sql, MyFlash, my2fback и т.д. 
Этот инструмент основан на вторичной разработке инструментов my2fback и binlog_rollback.


# применение
* Быстрый откат данных (flashback)
* Восстановление данных, утерянных новым ведущим устройством после переключения ведущий-ведомый
* Генерировать стандартный SQL из binlog, использовать производные функции
* Генерируйте статистику DML, вы можете узнать, какие таблицы обновляются чаще
* Высокий ввод-вывод и высокий TPS, узнайте, какие таблицы часто обновляются
* Узнайте, есть ли в базе данных большая транзакция или длительная транзакция в определенный момент времени
* Задержка Master-slave, анализ инструкций SQL, выполняемых основной библиотекой
* В дополнение к поддержке обычных типов данных, он также поддерживает типы данных, которые не поддерживаются большинством инструментов, такие как json, blob, text, emoji и другие типы данных. генерация sql


# Сравнение характеристик продукта
binlog2sql в настоящее время является наиболее широко используемым инструментом отката MySQL в отрасли. Давайте сравним производительность my2sql и binlog2sql.

| |my2sql |binlog2sql|
|--- |--- |--- |
| 1.1G binlog генерирует откат SQL | 1 минута 40 секунд | 65 минут |
| 1.1G binlog генерирует исходный SQL | 1 минута 30 секунд | 50 минут |
|1.1G binlog генерирует статистику таблицы DML и статистику транзакций |40 секунд |Не поддерживается |


# Быстрый старт
### Выполнить операцию ретроспективного просмотра конкретного операционного процесса
[https://blog.csdn.net/liuhanran/article/details/107426162](https://blog.csdn.net/liuhanran/article/details/107426162)
### Разбор binlog для генерации стандартного SQL
[https://blog.csdn.net/liuhanran/article/details/107427204](https://blog.csdn.net/liuhanran/article/details/107427204)
### Анализ статистики binlog DML, анализ длинных транзакций и больших транзакций
[https://blog.csdn.net/liuhanran/article/details/107427391](https://blog.csdn.net/liuhanran/article/details/107427391)



# Описание важных параметров

-U
```
Приоритет отдается использованию уникального ключа в качестве условия where, значение по умолчанию равно false
```

-mode
```
repl: Сделайте вид, что разбираете файлы binlog из библиотеки, file: разбирайте файлы binlog в автономном режиме, repl по умолчанию
```

-local-binlog-file
```
При указании параметра-mode=file вам необходимо указать относительный или абсолютный путь к файлу binlog-local-binlog-file. Можно непрерывно анализировать несколько файлов binlog. Вам нужно только указать начальное имя файла, и программа автоматически продолжит анализ следующего файла.
```

-add-extraInfo
```
Следует ли размещать database/table/datetime/binlogposition...Перед добавлением информации в каждый сгенерированный sql в качестве комментария значение по умолчанию равно false
```

```
# datetime=2020-07-16_10:44:09 база данных=таблица оркестратора=имя_домена кластера binlog=mysql-bin.011519 startpos=15552 stoppos=15773
UPDATE `orchestrator`.`cluster_domain_name` SET `last_registered`='2020-07-16 10:44:09' WHERE `cluster_name`='192.168.1.1:3306'
```

-big-trx-row-limit n
```
транзакция с затронутыми строками, превышающими или равными этому значению, рассматривается как большая транзакция
Найдите транзакции, удовлетворяющие n sql, значение по умолчанию равно 500
```

-databases 、 -tables
```
Условная фильтрация библиотеки и таблицы, разделенные запятыми
```

-sql
```
Тип анализируемого sql, необязательные параметры insert, update, delete, все анализируется по умолчанию
```

-doNotAddPrifixDb

```
Имя таблицы с префиксом и имя базы данных в sql, например: вставить в insert into db1.tb1 (x1, x1) values (y1, y1)
Вставка в db1 insert into db1.tb1 (x1, x1)  значения (y1, y1) класс sql, вы также можете сгенерировать sql без имени библиотеки
```

-file-per-table
```
Создайте sql-файл для каждой таблицы
```

-full-columns
```
Для обновления sql включите неизмененные столбцы. для обновления и удаления используйте все столбцы для построения условия where.
значение по умолчанию false, это означает, используйте измененные столбцы для построения части набора, используйте первичный / уникальный ключ для построения условия where
Независимо от того, содержит ли сгенерированный sql всю информацию о столбце, значение по умолчанию равно false
```
-ignorePrimaryKeyForInsert
```
Независимо от того, удаляет ли сгенерированный оператор insert первичный ключ, значение по умолчанию равно false
```

-output-dir
```
Сохраните сгенерированные результаты в каталоге разработки
```

-output-toScreen
```
Выведите сгенерированные результаты на экран и запишите в файл по умолчанию
```

-threads
```
Количество потоков, по умолчанию 8
```

-work-type
```
2sql: сгенерировать исходный sql, откат: сгенерировать откат sql, статистика: учитывать только DML и информацию о транзакции
```

# Пример использования
### Разбор стандартного SQL
#### Разбор стандартного SQL на основе момента времени

```
# Притворись, что разбираешь binlog из библиотеки
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306 -mode repl -work-type 2sql  -start-file mysql-bin.011259  -start-datetime "2020-07-16 10:20:00" -stop-datetime "2020-07-16 11:00:00" -output-dir ./tmpdir

# Прямое чтение анализа файла binlog
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306 -mode file -local-binlog-file ./mysql-bin.011259  -work-type 2sql  -start-file mysql-bin.011259  -start-datetime "2020-07-16 10:20:00" -stop-datetime "2020-07-16 11:00:00" -output-dir ./tmpdir
```

#### Разбор стандартного SQL на основе точек pos

```
# Притворись, что разбираешь binlog из библиотеки
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306 -mode repl  -work-type 2sql  -start-file mysql-bin.011259  -start-pos 4 -stop-file mysql-bin.011259 -stop-pos 583918266  -output-dir ./tmpdir

# Прямое чтение анализа файла binlog
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306  -mode file -local-binlog-file ./mysql-bin.011259  -work-type 2sql  -start-file mysql-bin.011259  -start-pos 4 -stop-file mysql-bin.011259 -stop-pos 583918266  -output-dir ./tmpdir
```

### Проанализируйте откат SQL
#### Проанализируйте SQL отката в соответствии с моментом времени

```
# Притворись, что разбираешь binlog из библиотеки
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306 -mode repl -work-type rollback  -start-file mysql-bin.011259  -start-datetime "2020-07-16 10:20:00" -stop-datetime "2020-07-16 11:00:00" -output-dir ./tmpdir

# Прямое чтение анализа файла binlog
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306  -mode file -local-binlog-file ./mysql-bin.011259 -work-type rollback  -start-file mysql-bin.011259  -start-datetime "2020-07-16 10:20:00" -stop-datetime "2020-07-16 11:00:00" -output-dir ./tmpdir
```

#### Проанализируйте SQL отката на основе точки pos

```
# Притворись, что разбираешь binlog из библиотеки
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306 -mode repl -work-type rollback  -start-file mysql-bin.011259  -start-pos 4 -stop-file mysql-bin.011259 -stop-pos 583918266  -output-dir ./tmpdir

# Прямое чтение анализа файла binlog
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306   -mode file -local-binlog-file ./mysql-bin.011259  -work-type rollback  -start-file mysql-bin.011259  -start-pos 4 -stop-file mysql-bin.011259 -stop-pos 583918266  -output-dir ./tmpdir
```

### Статистика DML и крупных событий
#### Подсчитайте количество операций DML в каждой таблице в диапазоне времени и подсчитайте транзакции с транзакцией, превышающей 500, и временем, превышающим 300 секунд

```
# Притворись, что разбираешь binlog из библиотеки
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306  -mode repl -work-type stats  -start-file mysql-bin.011259  -start-datetime "2020-07-16 10:20:00" -stop-datetime "2020-07-16 11:00:00"  -big-trx-row-limit 500 -long-trx-seconds 300   -output-dir ./tmpdir

# Прямое чтение анализа файла binlog
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306 -mode file -local-binlog-file ./mysql-bin.011259   -work-type stats  -start-file mysql-bin.011259  -start-datetime "2020-07-16 10:20:00" -stop-datetime "2020-07-16 11:00:00"  -big-trx-row-limit 500 -long-trx-seconds 300   -output-dir ./tmpdir
```

#### Подсчитайте количество операций DML в каждой таблице в диапазоне точек pos за определенный период времени и подсчитайте транзакцию с транзакцией, превышающей 500, и временем, превышающим 300 секунд.

```
# Притворись, что разбираешь binlog из библиотеки
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306  -mode repl -work-type stats  -start-file mysql-bin.011259  -start-pos 4 -stop-file mysql-bin.011259 -stop-pos 583918266  -big-trx-row-limit 500 -long-trx-seconds 300   -output-dir ./tmpdir

# Прямое чтение анализа файла binlog
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306 -mode file -local-binlog-file ./mysql-bin.011259  -work-type stats  -start-file mysql-bin.011259  -start-pos 4 -stop-file mysql-bin.011259 -stop-pos 583918266  -big-trx-row-limit 500 -long-trx-seconds 300   -output-dir ./tmpdir
```

### Проанализируйте стандартный SQL из определенной точки pos и продолжайте печатать на экране

```
# Притворись, что разбираешь binlog из библиотеки
./my2sql  -user root -password xxxx -host 127.0.0.1   -port 3306 -mode repl  -work-type 2sql  -start-file mysql-bin.011259  -start-pos 4   -output-toScreen 
```

# Скачать двоичную версию

+ Существует скомпилированная версия linux (CentOS release 7.x) [Нажмите, чтобы загрузить версию Linux] (https://github.com/liuhr/my2sql/blob/master/releases/centOS_release_7.x/my2sql )

# Скомпилировать и установить
```
cd $GOPATH/src
git clone https://github.com/liuhr/my2sql.git
cd my2sql/
go build .
go build -o my2sql main.go
```

# предел
* При использовании функции отката/ретроспективного просмотра формат binlog должен быть row, а binlog_row_image=full, статистика DML и анализ больших транзакций не затрагиваются.
* Откатить можно только DML, но не DDL
* При использовании функции отката структура таблицы анализируемого сегмента binlog должна быть согласованной (например: parse mysql-bin.файл 000001, таблица этого файла binlog содержит операции добавления столбца или удаления столбца, затем выполнение отката может вызвать исключение)
* Поддержка указания часового пояса-tl для интерпретации содержимого поля time/datetime в binlog.Время начала-start-datetime и время окончания-stop-datetime также будут использовать этот указанный часовой пояс，
  Однако обратите внимание, что это время начала и окончания для метки времени unix, сохраненной в заголовке события binlog.Дополнительная информация о времени datetime в результате находится в заголовке события binlog.
  отметка времени
* Этот инструмент замаскирован под извлечение binlog из библиотеки. Пользователь, которому необходимо подключиться к базе данных, имеет разрешения SELECT, REPLICATION SLAVE и REPLICATION CLIENT.
* Версия MySQL8.0 должна добавить default_authentication_plugin = mysql_native_password в файл конфигурации. Для разрешения проверки подлинности паролем пользователя должно быть mysql_native_password.

# спасибо
Спасибо [https://github.com/siddontang ](https://github.com/siddontang ) библиотека синтаксического анализа binlog, благодаря библиотеке sqlbuilder от dropbox, благодаря my2fback, binlog_rollback

# ЗАДАЧИ
-[x] Транзакция GTID анализируется в единицах
-[x] Ретроспективный снимок, откат, добавление метки начала/фиксации транзакции