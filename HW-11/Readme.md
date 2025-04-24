# Хранимые функции и процедуры часть 3 (Триггеры)
* Удаляем схему pract_functions, если она существует. и создаем ее заново
  ```
  DROP SCHEMA IF EXISTS pract_functions CASCADE;
  CREATE SCHEMA pract_functions;
  ```
* Устанавливаем порядок просмотра схем в поисках объектов БД
  ```
  SET search_path = pract_functions, public
  ```
* Создаем таблицу с товарами
  ```
  CREATE TABLE goods
  (
    goods_id   integer PRIMARY KEY,
    good_name  varchar(63) NOT NULL,
    good_price numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
  );
  ```
* Добавляем в таблицу товаров две записи
  ```
  INSERT INTO goods (goods_id, good_name, good_price)
  VALUES 	(1, 'Спички хозяйственные', 0.50),
		      (2, 'Автомобиль Ferrari FXX K', 185000000);
  ```
* Создаем таблицу "Продажи"
  ```
  CREATE TABLE sales
  (
    sales_id   integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    good_id    integer REFERENCES goods (goods_id),
    sales_time timestamp with time zone DEFAULT now(),
    sales_qty  integer CHECK (sales_qty > 0)
  );
  ```
* Заполняем таблицу продаж
  ```
  INSERT INTO sales (good_id, sales_qty)
  VALUES (1, 10), (1, 1), (1, 120), (2, 1);
  ```
* Пишем запрос для генерации отчета – сумма продаж по каждому товару
  ```
  SELECT G.good_name, sum(G.good_price * S.sales_qty)
  FROM goods G
  INNER JOIN sales S ON S.good_id = G.goods_id
  GROUP BY G.good_name
  ```
* Так как с увеличением объем продаж, запрос может выполняться медленно, принято решение денормализовать базу данных.
  * Создаем таблицу, в которой будут храниться наименование товара и сумма продаж
    ```
    CREATE TABLE good_sum_mart
    (
      good_name varchar(63) NOT NULL,
      sum_sale	numeric(16,2) NOT NULL
    );
    ```
