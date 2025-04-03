
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
