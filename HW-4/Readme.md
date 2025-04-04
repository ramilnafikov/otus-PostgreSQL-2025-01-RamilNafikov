
# Логический уровень PostgreSQL
* Запускаем WSL
* Создаем новый кластер PostgreSQL 14
  ```
  sudo pg_createcluster -d /var/lib/postgresql/14/main 14 main
  ```
* Проверяем наличие кластера PostgreSQL
  ```
  pg_lsclusters
  ```
  ![image](https://github.com/user-attachments/assets/fd4d1ef2-95b9-4f3f-b870-a6df46b3d9ec)

* Запускаем кластер
  ```
  sudo pg_ctlcluster 14 main start
  ```
  ![image](https://github.com/user-attachments/assets/15ce4ae5-a822-418f-a62d-1fefdb75845d)
* Заходим в наш кластер под пользователем postgres
  ```
  sudo -u postgres psql -p 5432
  ```
  ![image](https://github.com/user-attachments/assets/3a6b70cd-d332-4ac5-8b62-69c4483848fc)
* Проверяем список баз данных
* 
  ![image](https://github.com/user-attachments/assets/50ef9fa1-8dd5-411a-b4a6-46a88a1bf2d6)
* Создаем новую базу данных testdb
  ```
  create database testdb;
  ```
* Подключаемся к созданной базе testdb
  ```
  \c testdb
  ```
  ![image](https://github.com/user-attachments/assets/422c22c7-0b87-49b4-b18b-2142662de940)
* Создаем схему testnm
  ```
  create schema testnm;
  ```
* Создаем новую таблицу t1 с одной колонкой c1 типа integer
  ```
  create table t1 (c1 integer);
  ```
* Вставляем в таблицу t1 строку со значением c1 = 1
  ```
  insert into t1 (c1) values (1);
  ```
* Создаем новую роль readonly
  ```
  create role readonly;
  ```
* Даем новой роли право на подключение к базе данных
  ```
  alter role readonly LOGIN;
  ```
* Даем новой роли право на использование схемы testnm
  ```
  grant usage on schema testnm to readonly;
  ```
* Даем новой роли право на select для всех таблиц схемы testnm
  ```
  grant select on all tables in schema testnm to readonly;
  ```
* Создаем пользователя testread с паролем test123
  ```
  create user testread with password 'test123';
  ```
* Даем роль readonly пользователю testread
  ```
  grant readonly to testread;
  ```
* Заходим в базу testdb под пользователем testread
  ```
  psql -U testread -d testdb -h localhost -p 5432
  ```
  ![image](https://github.com/user-attachments/assets/fa13f262-e041-4c0b-888d-c3515fb1d547)

* Выполним запрос
  ```
  select * from t1;
  ```
* Не получилось. Отказано в доступе
  
  ![image](https://github.com/user-attachments/assets/c9d67ef4-40c9-4f4f-935f-0a86700e300a)

* При создании таблицы t1 не была указана схема и поэтому таблица создалась по умолчанию в схеме public, на которую мы доступ для роли readonly не давали
* Проверим список таблиц
  ```
  \dt
  ```
  ![image](https://github.com/user-attachments/assets/68078f6b-debc-4dfb-b77b-657a987952a7)

* Возвращаемся в базу testdb под пользователем postgres
  ```
  psql -U postgres -d testdb -h localhost -p 5432
  ```
* Удаляем таблицу t1
  ```
  drop table t1;
  ```
* Создаем ее заново с явным указанием имени схемы testnm
  ```
  create table testnm.t1 (c1 integer);
  ```
* Вставляем в таблицу t1 строку со значением c1 = 1
  ```
  insert into testnm.t1 (c1) values (1);
  ```
* Снова заходим в базу testdb под пользователем testread
  ```
  psql -U testread -d testdb -h localhost -p 5432
  ```
* Выполняем следующий запрос
  ```
  select * from testnm.t1;
  ```
* Снова не получилось
  
  ![image](https://github.com/user-attachments/assets/d714c971-3712-4e53-abb4-7d1cf5a2d11a)

* Мы давали доступ роли readonly на запросы только к существующим на тот момент таблицам схемы testnm, поэтому к заново созданной таблице t1 доступа нет (подсмотрено в шпаргалке)
* Снова зайдем в базу testdb под пользователем postgres и выполним следующий код для автоматического назначения прав на все вновь создаваемые таблицы
  ```
  \c testdb postgres

  ALTER default privileges in SCHEMA testnm grant SELECT on TABLES to readonly; 
  ```
* Вернемся в базу testdb под пользователем testread и выполним запрос к таблице t1 схемы testnm
  ```
  \c testdb testread

  select * from testnm.t1;
  ```
* Снова не получилось
  
  ![image](https://github.com/user-attachments/assets/7d917e36-c8eb-4957-8c27-1138b667875f)
  
* Команда ALTER default privileges предоставит доступ на запросы только к новым таблицам схемы testnm (подсмотрено в шпаргалке), поэтому снова зайдем в базу testdb под пользователем postgres и выполним команду
  ```
  grant select on all tables in schema testnm to readonly;
  ```
* Перезайдем в базу testdb под пользователем testread и снова выполним запрос к таблице t1 схемы testnm. Запрос выполнился без ошибок
  ```
  \c testdb testread

  select * from testnm.t1;
  ```
  ![image](https://github.com/user-attachments/assets/8fdec72f-301c-4e97-b61d-d209d85b88ef)

* Теперь попробуем создать таблицу t2 со столбцом c1 типа integer и вставить туда запись со значением 2.
  ```
  create table t2(c1 integer);
  insert into t2 values (2);
  ```
* Код выполнился без ошибок, хотя мы права на создание таблиц и вставку в них роли readonly не давали
  
  ![image](https://github.com/user-attachments/assets/e85579fd-c5dc-490d-aa06-7cb358d29582)

* Так как при создании таблицы t2 мы снова не указали схему, а search_path по умолчанию указывает на схему public, то таблица t2 создалась в схеме public базы testdb.
  А так как право на все действия в этой схеме дается роли public (подсмотрено в шпаргалке) и роль public добавляется всем новым пользователям, то каждый пользователь может создавать объекты в схеме public базы данных 
  testdb
* Запретим роли public создавать объекты в схеме public базы testdb
  ```
  revoke create on schema public from public; 
  ```
* Отменим все привелегии в базе данных testdb для роли public
  ```
  revoke all on database testdb from public;
  ```
* Снова зайти в базу testdb под пользователем testread не получается
  
  ![image](https://github.com/user-attachments/assets/4029e16e-68e1-4a03-8216-ed2e4ccefa09)
  
* Под пользователем postgres следующие команды выполняются без проблем
  ```
  create table t3(c1 integer); insert into t2 values (2);
  ```
  ![image](https://github.com/user-attachments/assets/651ef726-ed53-4a0c-8ed6-b34216a46e0f)

