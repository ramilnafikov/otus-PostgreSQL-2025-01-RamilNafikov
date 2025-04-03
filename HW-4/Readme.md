
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
* 
  ![image](https://github.com/user-attachments/assets/d714c971-3712-4e53-abb4-7d1cf5a2d11a)
