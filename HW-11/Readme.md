# Хранимые функции и процедуры часть 3 (Триггеры)
* Удаляем схему pract_functions, если она существует, и создаем ее заново
  ```
  DROP SCHEMA IF EXISTS pract_functions CASCADE;
  CREATE SCHEMA pract_functions;
  ```
* Устанавливаем порядок просмотра схем в поисках объектов БД
  ```
  SET search_path = pract_functions, public
  ```
* Создаем таблицу с товарами с полями "Наименование товара" (good_name) и "Стоимость" (good_price)
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
  VALUES (1, 'Спички хозяйственные', 0.50),
         (2, 'Автомобиль Ferrari FXX K', 185000000);
  ```
* Создаем таблицу "Продажи" с полями "ИД товара" со ссылкой на таблицу "Товары" (good_id), "Время продажи" (sales_time) и "Количество" (sales_qty)
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
  ![image](https://github.com/user-attachments/assets/74219071-85a4-464c-a734-1fe1efc48d1b)
  
* Пишем запрос для генерации отчета – сумма продаж по каждому товару
  ```
  SELECT G.good_name, sum(G.good_price * S.sales_qty)
  FROM goods G
  INNER JOIN sales S ON S.good_id = G.goods_id
  GROUP BY G.good_name
  ```
  ![image](https://github.com/user-attachments/assets/f1f9b5d2-96cd-4f7f-86e3-97358a5c47fc)

* Так как с увеличением объем продаж, запрос может выполняться медленно, принято решение денормализовать базу данных.
  * Создаем таблицу, в которой будут храниться наименование товара и сумма продаж
    ```
    CREATE TABLE good_sum_mart
    (
      good_name varchar(63) NOT NULL,
      sum_sale	numeric(16,2) NOT NULL
    );
    ```
  * Создаем триггерную функцию tf_for_sales, которая будет вызываться при добавлении, изменении и удалении записей в таблице продаж sales
    ```
    CREATE OR REPLACE FUNCTION tf_for_sales()
    RETURNS trigger
    AS
    $TRIG_FUNC$
    declare
      v_good_name varchar(63);
      v_good_price numeric(12,2);
      v_delta_sum numeric(16,2);
    BEGIN
      --Получаем наименование товара и его стоимость для id товара, по которому добавляются, редактируются или удаляются данные по продажам в таблице sales
      SELECT g.good_name, g.good_price INTO v_good_name, v_good_price FROM goods as g WHERE g.goods_id = coalesce(NEW.good_id, OLD.good_id);

      --Вычисляем изменение суммы продаж проданного товара
      IF (TG_OP = 'DELETE') THEN
        v_delta_sum = -1 * (OLD.sales_qty * v_good_price);
      ELSIF (TG_OP = 'UPDATE') THEN
        v_delta_sum = (NEW.sales_qty * v_good_price) - (OLD.sales_qty * v_good_price);
      ELSIF (TG_OP = 'INSERT') THEN
        v_delta_sum = NEW.sales_qty * v_good_price;
      END IF;

      --Если нет записей по продажам указаного товара, то мы добавляем запись в таблице для отчета good_sum_mart, иначе корректируем сумму продаж с учетом изменения количества проданных товаров
      if not exists (select 1 from good_sum_mart where good_name = v_good_name) then
       insert into good_sum_mart (good_name, sum_sale)
        values (v_good_name, v_delta_sum);
      else
        update good_sum_mart
        set
          sum_sale = sum_sale + v_delta_sum
        where good_name = v_good_name;
      end if;

      RETURN NULL;
    END;
    $TRIG_FUNC$
    LANGUAGE plpgsql
    ```
  * Создаем триггер для таблицы продаж sales, который при вставке, редактировании или удалении записей будет вызывать нашу триггерную функцию tf_for_sales
    ```
    CREATE TRIGGER tr_sales
    AFTER INSERT OR UPDATE OR DELETE ON sales
    FOR EACH ROW EXECUTE PROCEDURE tf_for_sales();
    ```
  * Удаляем все записи в таблице продаж, сбрасываем текущее значение последовательности до 1 и добавляем заново несколько записей
    ```
    truncate table sales;

    ALTER SEQUENCE sales_sales_id_seq RESTART WITH 1;

    INSERT INTO sales (good_id, sales_qty)
    VALUES (1, 10), (1, 1), (1, 120), (2, 1);
    ``` 
  * Выполняем запрос к таблице good_sum_mart
    ```
    select * from good_sum_mart;
    ```
    ![image](https://github.com/user-attachments/assets/c8a6d270-16e2-4b73-bc6d-5763e5caa751)

    Как видим, триггер сработал и в поле "Сумма продаж" сидит значение, аналогичное результату первоначального запроса для отчета
  * Теперь добавим, отредактируем или удалим какую-нибудь запись в таблице продаж
    ```
    INSERT INTO sales (good_id, sales_qty)
    values (1, 10);
    ```
    Результат:
    
    ![image](https://github.com/user-attachments/assets/4d8ec142-b66a-4b1b-a806-f14cab900f8a)

    ```
    update sales set sales_qty = 100 where sales_id = 3
    ```
    Результат:

    ![image](https://github.com/user-attachments/assets/fc0b4136-eb87-4e6e-bf28-0577d7380766)

    ```
    delete from sales where sales_id = 3;
    ```
    Результат:

    ![image](https://github.com/user-attachments/assets/b6a80762-61c2-463c-ab9b-5411998c460b)


    
