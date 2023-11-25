
# PostgreSQL DBA (v16, 15, 14, 13, 12) - 2024 Edition

</br>
*** The book, "PostgreSQL DBA", is published on Amazon, it covers PostgreSQL v12, 13, 14, 15, to the latest v16 ***

</br>
</br>
The "PostgreSQL DBA 16 (2024 Edition)" book's URL:</br>
https://www.amazon.com/PostgreSQL-DBA-v16-v15-Administrators/dp/B0CN5FD1M8/ref=tmm_pap_swatch_0?_encoding=UTF8&qid=1700880466&sr=8-2

</br>
</br>
The "PostgreSQL DBA 16 (2024 Edition)" book's Kindle Edition:</br>
https://www.amazon.com/PostgreSQL-DBA-v16-Administrators-Availablity-ebook/dp/B0CN3HYFZJ/ref=sr_1_2?crid=UVJD8VVHJ85P&keywords=Postgresql&qid=1700880466&sprefix=postgres%2Caps%2C751&sr=8-2

</br>
</br>


Important Note 1: The following commands are FOR INFORMATION ONLY

---
P1.1

SET max_parallel_workers = 4; 

CREATE TABLE orders (order_id serial PRIMARY KEY, customer_id int, order_date date);

CREATE TABLE customers (customer_id serial PRIMARY KEY, customer_name varchar(255),    city varchar(255));

INSERT INTO customers (customer_name, city) VALUES
    ('Customer A', 'New York'),
    ('Customer B', 'Los Angeles'),
    ('Customer C', 'Chicago');

INSERT INTO orders (customer_id, order_date) VALUES
    (1, '2023-01-15'),
    (2, '2023-01-20'),
    (1, '2023-02-10'),
    (3, '2023-02-25');

EXPLAIN SELECT * FROM orders
RIGHT OUTER JOIN customers ON orders.customer_id=customers.customer_id;

---
P1.2

SELECT pg_create_logical_replication_slot('my_replication_slot', 'pgoutput');

CREATE PUBLICATION my_publication FOR TABLE my_table;

CREATE PUBLICATION my_publication FOR ALL TABLES;

CREATE SUBSCRIPTION my_subscription CONNECTION 'dbname=mydb host=standby_host port=5432 user=myuser password=mypassword' PUBLICATION my_publication;

SELECT * FROM pg_stat_subscription;

SELECT * FROM pg_replication_slots

---
P1.3

CREATE PUBLICATION my_publication FOR TABLE my_table;

CREATE SUBSCRIPTION my_subscription 

CONNECTION 'dbname=mydb host=primary_server user=myuser password=mypassword' 
PUBLICATION my_publication WITH (binary=true);

SELECT * FROM pg_stat_subscription;

---
P1.4

SELECT * FROM pg_stat_io;

---
P1.5

SELECT json_object(
    'name' VALUE 'John Doe', 'age' VALUE 30, 'city' VALUE 'New York'
) AS person;

SELECT json_array('apple','banana','cherry'
) AS fruits;

SELECT json_typeof('{"name": "Alice"}'::json) AS type; 

SELECT json_typeof('[1, 2, 3]'::json) AS type; 

SELECT json_typeof('42'::json) AS type;          

SELECT json_typeof('"Hello"'::json) AS type;

SELECT CASE WHEN json_typeof('{"name": "Bob"}'::json) = 'object' THEN 'Valid' ELSE 'Invalid' END AS validation;


---
P1.6

SELECT * FROM pg_stat_progress_vacuum;

VACUUM ANALYZE customers;

SELECT relname, n_tup_ins, n_tup_upd, n_tup_del, n_live_tup, n_dead_tup, idx_scan
FROM pg_stat_all_tables
WHERE relname = 'customers';


---
P1.7

 TYPE  DATABASE        USER            ADDRESS         METHOD
 
local   sameuser        all                             md5

local   all             /^.*helpdesk$                   md5

local   all             @admins                         md5

local   all             +support                        md5


---
P1.8

CREATE TABLE sample_data (
    id serial PRIMARY KEY, category text, value numeric);

INSERT INTO sample_data (category, value) VALUES
    ('A', 10), ('A', 20), ('A', 30),
    ('B', 5),  ('B', 15), ('B', 25);

SELECT
    category, value,
    DENSE_RANK() OVER (ORDER BY value DESC) AS rank
FROM
    (SELECT  category, value
     FROM     sample_data
     GROUP BY category, value) AS subquery;


---
P1.9

INSERT INTO sales (sale_date, amount)
VALUES
    ('2023-01-01', 1000),
    ('2023-01-02', 1500),
    ('2023-01-03', 1200),
    ('2023-01-04', 2000),
    ('2023-01-05', 1800),
    ('2023-01-06', 2200);

SELECT
      sale_date, amount,
      percent_rank() OVER (ORDER BY amount) AS percentile_rank
FROM  sales 
ORDER BY sale_date;


---
P1.10

CREATE TABLE employees (
    employee_id serial PRIMARY KEY,
    employee_name text,
    department text
);

INSERT INTO employees (employee_name, department)
VALUES
    ('Alice', 'HR'),
    ('Bob', 'Engineering'),
    ('Charlie', 'HR'),
    ('David', 'Sales'),
    ('Eva', 'Engineering'),
    ('Frank', 'Sales');

SELECT department, string_agg(employee_name, ', ') AS employee_list
FROM employees
GROUP BY department;


---
P1.11

CREATE TABLE orders (
    sale_date date,
    amount numeric
) PARTITION BY RANGE (sale_date);

CREATE TABLE orders_2019 PARTITION OF orders
    FOR VALUES FROM (MINVALUE) TO ('2020-01-01');

CREATE TABLE orders_2020 PARTITION OF orders
    FOR VALUES FROM ('2020-01-01') TO (MAXVALUE);

INSERT INTO orders (sale_date, amount)
VALUES
    ('2019-12-15', 1000),
    ('2020-02-20', 1500),
    ('2020-03-10', 800),
    ('2019-11-05', 1200),
    ('2020-01-05', 900);

EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE sale_date BETWEEN '2019-12-01' AND '2020-01-31';

SELECT *
FROM orders
WHERE sale_date BETWEEN '2019-12-01' AND '2020-01-31'

---
P1.12

VACUUM (BUFFER_USAGE_LIMIT '128MB') orders;

ANALYSE (BUFFER_USAGE_LIMIT '128MB') orders;


---
P1.13

SELECT * FROM pg_stat_all_tables;


---
P1.14

ps axu | grep postgres


---
P1.15

\d pg_prepared_statements


---
P1.16
SELECT * FROM pg_stat_get_backend_idset();

SELECT pg_stat_get_backend_pid(backendid) AS pid,
       pg_stat_get_backend_activity(backendid) AS query
FROM pg_stat_get_backend_idset() AS backendid;


---
P1.17

SELECT pid, wait_event_type, wait_event FROM pg_stat_activity WHERE wait_event is NOT NULL;


---
P.18

SELECT * FROM pg_statio_user_sequences;


---
P1.19
SELECT * FROM pg_stat_wal;


---
P1.20

SELECT * FROM pg_stat_bgwriter;


---
P1.21

SELECT * FROM pg_statio_user_indexes;


---
P1.22

SELECT * FROM pg_statio_user_tables;


---
P1.23

SELECT * FROM pg_stat_database_conflicts;


---
P1.24

SELECT * FROM pg_stat_xact_all_tables


---
P2.1  and P3.1

CREATE DATABASE my_db; 

---
P4
sudo yum install libicu 

sudo yum install lz4 

sudo yum install libxslt 

sudo locale-gen en_US.UTF-8 

cd ~/Downloads

sudo rpm -ivh postgresql16-libs-16.0-1PGDG.rhel9.x86_64.rpm

sudo rpm -ivh postgresql16-16.0-1PGDG.rhel9.x86_64.rpm

sudo rpm -ivh postgresql16-server-16.0-1PGDG.rhel9.x86_64.rpm

sudo rpm -ivh postgresql16-contrib-16.0-1PGDG.rhel9.x86_64.rpm

sudo /usr/pgsql-16/bin/initdb --pgdata=/appl/pgsql/16/data --localeprovider=icu --icu-locale=en_US.UTF-8  


/usr/pgsql-16/bin/pg_ctl -D /appl/pgsql/16/data -l /appl/logs/pgsql/pg_logfile start 
 
/usr/pgsql-16/bin/pg_ctl -D /appl/pgsql/16/data -l /appl/logs/pgsql/pg_logfile status 
 
/usr/pgsql-16/bin/pg_ctl -D /appl/pgsql/16/data -l /appl/logs/pgsql/pg_logfile stop 

/usr/pgsql-16/bin/pg_ctl -D /appl/pgsql/16/data -l /appl/logs/pgsql/pg_logfile restart

CREATE DATABASE my_db; 

/usr/pgsql-16/bin/pg_ctl -D /appl/pgsql/16/data -l /appl/logs/pgsql/pg_logfile reload 



---
P4.1

CREATE USER my_user; 

ALTER ROLE my_user WITH ENCRYPTED PASSWORD 'my_pass'; 

CREATE DATABASE my_db; 

GRANT ALL ON DATABASE my_db TO my_user; 

ALTER DATABASE my_db OWNER TO my_user; 



---
P4.2

(Intended to be blank)


---
P4.3

CREATE TABLE books( 
book_id SERIAL PRIMARY KEY,  
title VARCHAR(100) NOT NULL,
author VARCHAR(100) NOT NULL,     
publication_year INTEGER,   
genre VARCHAR(50),     
qty INTEGER, 
unit_price DECIMAL);


---
P4.4
(Intended to be blank)


---
P4.5

/usr/pgsql-16/bin/psql -h localhost -d my_db -U my_user 


---
P4.6

psql -h localhost -p 5432 -U my_user -d my_db -sslmode require -sslcert /appl/keystore/client_postgresql_cert.pem -sslkey /appl/keystore/client.key 


---
P4.7

CREATE TABLESPACE tbs_encrypt_zone LOCATION '/appl/pgsql/16/data/tbs_encrypt_zone'; 



---
P5.1

SELECT setting FROM pg_settings WHERE name = 'data_directory';


---
P5.2

SELECT setting FROM pg_settings WHERE name = 'config_file';

SELECT setting FROM pg_settings WHERE name = 'hba_file';

SHOW log_directory;

SHOW log_filename;



---
P5.3

SHOW shared_buffers;

SHOW max_connections;

SHOW work_mem;

SHOW maintenance_work_mem;


---
P5.4
SHOW log_connections;


---
P5.5

SHOW max_connections;


---
P5.6

SHOW default_statistics_target;


---
P5.7

SHOW effective_cache_size;


---
P5.8

SHOW statement_timeout;



---
P5.9

CREATE TABLE demo_table (
    id serial PRIMARY KEY,
    name VARCHAR(50),
    children INT
);

BEGIN;
INSERT INTO demo_table (name) VALUES ('John');

INSERT INTO demo_table (name) VALUES ('Alice');

SELECT * FROM demo_table;

COMMIT;

SELECT * FROM demo_table;


---
P5.10

BEGIN;

INSERT INTO demo_table (name, children) VALUES ('Alex', 2);

INSERT INTO demo_table (name, children) VALUES ('Sam', 'NO CHILD');

SELECT * FROM demo_table;

INSERT INTO demo_table (name) VALUES ('Sam', 'NO CHILD');

SELECT * FROM demo_table;

ROLLBACK;

SELECT * FROM demo_table;


---
P5.11 and P5.12 (Please refe to the book)


---
P5.13

pg_basebackup -U your_username -D /path/to/backup_directory -Ft -Xs -P -v -h localhost


---
P6.1
BEGIN; 

CREATE TABLE employee(
    id INT,
    first_name VARCHAR (50),
    last_name VARCHAR (50),
    salary numeric(10, 2)
);

EXPLAIN ANALYZE INSERT INTO employee (first_name, last_name, salary)
VALUES
    ('John', 'Doe', 50000.00),
    ('Jane', 'Smith', 60000.00),
    ('Bob', 'Johnson', 55000.00);

EXPLAIN ANALYZE SELECT * FROM employee;

ROLLBACK;

EXPLAIN ANALYZE SELECT * FROM employee;


---
P6.2.1 (Please refer to the book)


---
P6.2.2

\c my_db

CREATE EXTENSION pg_stat_statements;

SELECT query, total_exec_time, calls
FROM pg_stat_statements
WHERE total_exec_time > 1000000
ORDER BY total_exec_time DESC;


---
P6.3

DROP TABLE customers;

DROP TABLE orders;

CREATE TABLE customers (
    customer_id serial PRIMARY KEY,
    customer_name varchar(255)
);

CREATE TABLE orders (
    order_id serial PRIMARY KEY,
    customer_id integer,
    order_date date,
    total_amount numeric);

INSERT INTO customers (customer_name)
SELECT md5(random()::text)
FROM generate_series(1, 1000000);

INSERT INTO orders (customer_id, order_date, total_amount)
SELECT
    floor(random() * 100000) + 1,
    current_date - floor(random() * 365)::integer,
    CAST(random() * 1000 AS numeric(10, 2))
FROM generate_series(1, 10000000);

EXPLAIN ANALYZE 
SELECT customer_name
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
    WHERE total_amount > (
        SELECT AVG(total_amount)
        FROM orders
    )
);

EXPLAIN ANALYZE 
SELECT customer_id
FROM orders
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM orders
);

EXPLAIN ANALYZE 
SELECT c.customer_name
FROM customers c
JOIN (
    SELECT customer_id
    FROM orders
    WHERE total_amount > (
        SELECT AVG(total_amount)
        FROM orders
    )
) o ON c.customer_id = o.customer_id;

EXPLAIN ANALYZE 
WITH avg_total_amount AS (
    SELECT AVG(total_amount) AS avg_amount
    FROM orders
)
SELECT c.customer_name
FROM customers c
JOIN (
    SELECT customer_id
    FROM orders
    WHERE total_amount > (SELECT avg_amount FROM avg_total_amount)
) o ON c.customer_id = o.customer_id;

EXPLAIN ANALYZE 
SELECT customer_name
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
    WHERE total_amount > (
        SELECT AVG(total_amount)
        FROM orders
    )
);

EXPLAIN ANALYZE 
SELECT customer_id
FROM orders
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM orders
);

EXPLAIN ANALYZE 
SELECT c.customer_name
FROM customers c
JOIN (
    SELECT customer_id
    FROM orders
    WHERE total_amount > (
        SELECT AVG(total_amount)
        FROM orders
    )
) o ON c.customer_id = o.customer_id;


EXPLAIN ANALYZE
WITH avg_total_amount AS (
    SELECT AVG(total_amount) AS avg_amount
    FROM orders
)
SELECT c.customer_name
FROM customers c
JOIN (
    SELECT customer_id
    FROM orders
    WHERE total_amount > (SELECT avg_amount FROM avg_total_amount)
) o ON c.customer_id = o.customer_id;


---
P6.4

EXPLAIN ANALYZE
SELECT COUNT(1)
FROM customers
JOIN orders ON customers.customer_id = orders.customer_id;

EXPLAIN ANALYZE
SELECT COUNT(1)
FROM customers
JOIN orders ON customers.customer_id = orders.customer_id
WHERE customers.customer_id = 1;

EXPLAIN ANALYZE
SELECT COUNT(1)
FROM (SELECT * FROM customers ORDER BY customer_id) AS c
JOIN (SELECT * FROM orders ORDER BY customer_id) AS o
ON c.customer_id = o.customer_id;

EXPLAIN ANALYZE
SELECT COUNT(1)
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id;


---
P6.5

SELECT relname AS table_name, indexrelname AS index_name, idx_scan, 
idx_tup_read, ROUND(CAST(idx_tup_read AS numeric) / NULLIF(idx_scan, 0), 2) AS avg_tup_read_per_scan  
FROM pg_stat_user_indexes 
WHERE idx_scan > 0 
ORDER BY avg_tup_read_per_scan ASC;


---
P6.6

SELECT schemaname,  attname, n_distinct, most_common_vals, most_common_freqs 
FROM pg_stats 
WHERE 
schemaname NOT LIKE  'pg_%' AND schemaname <> 'information_schema'  
AND n_distinct <= 10 
AND most_common_vals IS NOT NULL; 


---
P6.7 (Please refet to the book)


---
P6.8

SELECT query,  state, backend_start,  now() - query_start AS duration 
FROM pg_stat_activity 
WHERE state = 'active'  AND now() - query_start > interval '2 minutes' 
ORDER BY now() - query_start DESC;


---
P6.9

SELECT mode, relation::regclass, pid, granted,  waitstart 
FROM pg_locks 
WHERE granted = false 
ORDER BY granted, relation::regclass;


---
P7.1

EXPLAIN ANALYZE 
SELECT * FROM orders
ORDER BY total_amount;

EXPLAIN ANALYZE 
SELECT *
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

DROP INDEX idx_total_amount_gt_100;

EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;


---
P7.2

CREATE INDEX CONCURRENTLY idx_customer_id ON customers(customer_id);

CREATE INDEX CONCURRENTLY idx_order_id ON orders(order_id);

CREATE INDEX CONCURRENTLY idx_total_amount_gt_100 ON orders(total_amount) WHERE total_amount >= 100;

EXPLAIN ANALYZE 
SELECT * FROM orders
ORDER BY total_amount;

EXPLAIN ANALYZE 
SELECT *
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

DROP INDEX idx_total_amount_gt_100;

EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

---
P7.3

DROP INDEX IF EXISTS idx_customer_name;

EXPLAIN ANALYZE 
SELECT customer_name
FROM customers
WHERE (customer_name >= '10000' AND customer_name <= '100000') OR
      (customer_name >= '200000' AND customer_name <= '300000');

CREATE INDEX CONCURRENTLY idx_customer_name ON customers(customer_name);

EXPLAIN ANALYZE 
SELECT customer_name
FROM customers
WHERE (customer_name >= '10000' AND customer_name <= '100000') OR
      (customer_name >= '200000' AND customer_name <= '300000');



---
P7.4

DROP INDEX IF EXISTS idx_customer_name;

DROP INDEX IF EXISTS customers_customer_name_idx;

EXPLAIN ANALYZE 
SELECT *
FROM customers
ORDER BY customer_name;

CREATE INDEX CONCURRENTLY idx_customer_name ON customers(customer_name);

EXPLAIN ANALYZE 
SELECT *
FROM customers
ORDER BY customer_name;


---
P7.5.1

top

df -h


---
P7.5.2

SELECT datname, pg_size_pretty(pg_database_size(datname)) AS size 
FROM pg_database  ORDER BY pg_database_size(datname) DESC; 



---
P7.5.3

SELECT relname, pg_size_pretty(pg_total_relation_size(relid)) 
FROM pg_catalog.pg_statio_user_tables 
ORDER BY pg_total_relation_size(relid) DESC; 


---
P7.6 (Please refet to the book)


---
P7.7.1

CREATE INDEX IF NOT EXISTS idx_total_amount ON orders(total_amount);

SET enable_seqscan = off; 

EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

SET enable_seqscan = on; 

EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;


---
P7.7.2

DROP INDEX idx_total_amount;

CREATE INDEX IF NOT EXISTS idx_total_amount ON orders(total_amount);

SET enable_bitmapscan = off; 

EXPLAIN ANALYZE 
SELECT customer_id
FROM orders
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM orders
);

SET enable_bitmapscan = on; 

EXPLAIN ANALYZE 
SELECT customer_id
FROM orders
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM orders
);


---
P7.7.3

SET enable_indexscan = off; 

EXPLAIN ANALYZE 
SELECT customer_id, total_amount
FROM orders
WHERE total_amount >= 100000 AND total_amount <= 1000000;

SET enable_indexscan = on; 

 
---
P7.8

BEGIN;

SELECT * FROM orders WHERE total_amount >= 1000 FOR UPDATE;

UPDATE orders SET total_amount = total_amount * 1.10 WHERE total_amount >= 1000;

COMMIT;


---
P7.9.1

CREATE OR REPLACE PROCEDURE my_dynamic_cursor(param_value NUMERIC) AS $$
DECLARE
    my_cursor CURSOR FOR
        SELECT customer_id::text, order_date, total_amount::numeric
        FROM orders
        WHERE total_amount > param_value;
    customer_id text;
    order_date DATE;
    total_amount NUMERIC;
BEGIN
    OPEN my_cursor;
    -- Fetch and process rows here
    LOOP
        FETCH NEXT FROM my_cursor INTO customer_id, order_date, total_amount;
        EXIT WHEN NOT FOUND;
        -- Process the row here (you can print or perform any other operation)
        RAISE NOTICE 'Customer ID: %, Order Date: %, Total Amount: %', customer_id, order_date, total_amount;
    END LOOP;

    CLOSE my_cursor;
END;
$$ LANGUAGE plpgsql;

UPDATE orders
SET total_amount = floor(random() * (100000 - 1000 + 1) + 1000)
WHERE total_amount = 1210;

CALL my_dynamic_cursor(1000);


---
P7.9.2

PREPARE my_prepared_statement (numeric) AS
SELECT customer_id::text, order_date, total_amount::numeric
FROM orders
WHERE total_amount > $1;

EXECUTE my_prepared_statement(1000);

EXPLAIN ANALYZE EXECUTE my_prepared_statement(1000);

EXPLAIN ANALYZE 
SELECT customer_id::text, order_date, total_amount::numeric
FROM orders
WHERE total_amount > 1000;

DEALLOCATE my_prepared_statement;


---
P7.10.1

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_customer_id ON orders(customer_id);

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_order_date ON orders(order_date);

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_customers_customer_name ON customers(customer_name);



---
P7.10.2

EXPLAIN ANALYZE 
SELECT *
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount > 1000;


---
P7.10.3

EXPLAIN ANALYZE 
WITH total_sales AS (
     SELECT customer_id, SUM(total_amount) AS total
     FROM orders
     GROUP BY customer_id
)
SELECT customers.customer_id, customers.customer_name, total_sales.total 
FROM customers 
JOIN total_sales ON customers.customer_id = total_sales.customer_id;


---
P7.10.4

EXPLAIN ANALYZE 
SELECT * 
FROM customers 
WHERE EXISTS ( 
      SELECT 1
      FROM orders
      WHERE customers.customer_id = orders.customer_id);



---
P7.10.5

EXPLAIN ANALYZE SELECT * FROM customers
JOIN orders ON customers.customer_id = orders.customer_id; 

EXPLAIN ANALYZE SELECT * FROM customers 
JOIN orders ON orders.customer_id = customers.customer_id;

EXPLAIN ANALYZE SELECT * FROM orders 
JOIN customers ON orders.customer_id = customers.customer_id;
 
EXPLAIN ANALYZE SELECT * FROM orders 
JOIN customers ON customers.customer_id = orders.customer_id ;

EXPLAIN ANALYZE  SELECT * FROM customers 
LEFT JOIN orders ON customers.customer_id = orders.customer_id;

EXPLAIN ANALYZE SELECT * FROM customers 
LEFT JOIN orders ON orders.customer_id = customers.customer_id;

EXPLAIN ANALYZE SELECT * FROM orders
LEFT JOIN customers ON orders.customer_id = customers.customer_id;

EXPLAIN ANALYZE SELECT * FROM orders
LEFT JOIN customers ON customers.customer_id = orders.customer_id ;

EXPLAIN ANALYZE SELECT * FROM customers 
RIGHT JOIN orders ON customers.customer_id = orders.customer_id;

EXPLAIN ANALYZE SELECT * FROM customers 
RIGHT JOIN orders ON orders.customer_id = customers.customer_id;

EXPLAIN ANALYZE SELECT * FROM orders
RIGHT JOIN customers ON orders.customer_id = customers.customer_id;

EXPLAIN ANALYZE SELECT * FROM orders
RIGHT JOIN customers ON customers.customer_id = orders.customer_id ;

EXPLAIN ANALYZE SELECT * FROM customers 
FULL JOIN orders ON customers.customer_id = orders.customer_id;

EXPLAIN ANALYZE SELECT * FROM customers 
FULL JOIN orders ON orders.customer_id = customers.customer_id;

EXPLAIN ANALYZE SELECT * FROM orders
FULL JOIN customers ON orders.customer_id = customers.customer_id;

EXPLAIN ANALYZE SELECT * FROM orders
FULL JOIN customers ON customers.customer_id = orders.customer_id ;

EXPLAIN ANALYZE SELECT * FROM customers 
FULL OUTER JOIN orders ON customers.customer_id = orders.customer_id;

EXPLAIN ANALYZE SELECT * FROM customers 
FULL OUTER JOIN orders ON orders.customer_id = customers.customer_id;

EXPLAIN ANALYZE SELECT * FROM orders
FULL OUTER JOIN customers ON orders.customer_id = customers.customer_id;

EXPLAIN ANALYZE SELECT * FROM orders
FULL OUTER JOIN customers ON customers.customer_id = orders.customer_id ;



---
P7.10.6

EXPLAIN ANALYZE 
WITH sales_data AS (
    SELECT
        EXTRACT(YEAR  FROM order_date) AS year,
        EXTRACT(MONTH FROM order_date) AS month,
        total_amount
    FROM orders
    WHERE order_date >= '2023-01-01' AND order_date < '2024-01-01'
)
SELECT year, month, SUM(total_amount) AS total_sales
FROM sales_data
GROUP BY CUBE (year, month);

EXPLAIN ANALYZE 
SELECT 
    EXTRACT(YEAR FROM  order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    SUM(total_amount) AS total_sales
FROM orders  
WHERE order_date >= '2023-01-01' AND order_date < '2024-01-01' 
GROUP BY CUBE (EXTRACT(YEAR FROM order_date), EXTRACT(MONTH  FROM order_date))
ORDER BY year, month;


---
P7.11

CREATE TABLE daily_sales_total AS
SELECT
    order_date,
    SUM(total_amount) AS total_sales
FROM orders
GROUP BY order_date;

CREATE TABLE monthly_sales_total AS
SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(total_amount) AS total_sales
FROM orders
GROUP BY month;

CREATE TABLE yearly_sales_total AS
SELECT
    DATE_TRUNC('year', order_date) AS year,
    SUM(total_amount) AS total_sales
FROM orders
GROUP BY year;

SELECT order_date, total_sales 
FROM daily_sales_total
ORDER BY order_date;GROUP BY year;

SELECT month, total_sales
FROM monthly_sales_total 
ORDER BY month;

SELECT year, total_sales
FROM yearly_sales_total
ORDER BY year;

EXPLAIN ANALYZE 
SELECT year, total_sales
FROM yearly_sales_total
ORDER BY year;

EXPLAIN ANALYZE 
SELECT
    DATE_TRUNC('year', order_date) AS year,
    SUM(total_amount) AS total_sales
FROM orders
GROUP BY year;


---
P8.1.1

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_customers_customer_name ON customers(customer_name);

REINDEX INDEX CONCURRENTLY idx_customers_customer_name;

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_date_amount ON orders(order_date, total_amount);


---
P8.1.2

SELECT * FROM pg_stat_activity 
WHERE query ILIKE '%CREATE INDEX%';

SELECT pg_cancel_backend(8044);



---
P8.2.1

CREATE TEMP SEQUENCE temp_sales_id_seq START 1;

INSERT INTO sales (sale_date, amount)
SELECT
    '2020-01-01'::date + (random() * 365)::integer AS sale_date,
    (random() * 1000)::numeric(10, 2) AS amount
FROM generate_series(1, 100000) AS row_number;

INSERT INTO sales (sale_date, amount)
SELECT
    '2021-01-01'::date + (random() * 365)::integer AS sale_date,
    (random() * 1000)::numeric(10, 2) AS amount
FROM generate_series(1, 100000) AS row_number;

INSERT INTO sales (sale_date, amount)
SELECT
    '2022-01-01'::date + (random() * 365)::integer AS sale_date,
    (random() * 1000)::numeric(10, 2) AS amount
FROM generate_series(1, 100000) AS row_number;

INSERT INTO sales (sale_date, amount)
SELECT
    '2023-01-01'::date + (random() * 365)::integer AS sale_date,
    (random() * 1000)::numeric(10, 2) AS amount
FROM generate_series(1, 100000) AS row_number;

CREATE OR REPLACE FUNCTION current_year() 
RETURNS integer AS 
$$  
    SELECT EXTRACT(YEAR FROM CURRENT_DATE); 
$$ LANGUAGE SQL IMMUTABLE; 

CREATE INDEX idx_sales_new_in_365d 
ON sales(sale_date, sales_id, amount) 
WHERE EXTRACT(YEAR FROM sale_date)>= current_year() - 1; 

CREATE INDEX idx_sales_all 
ON sales(sale_date, sales_id, amount) ; 


---
P8.2.2

SELECT 
 i.relname "Table Name",
 indexrelname "Index Name",
 pg_size_pretty(pg_total_relation_size(relid)) As "Total Size",
 pg_size_pretty(pg_relation_size(relid)) as "Table Size",
 pg_size_pretty(pg_relation_size(indexrelid)) "Index Size",
 reltuples::bigint "Estimated table row count"
FROM pg_stat_all_indexes i JOIN pg_class c ON i.relid=c.oid 
WHERE i.relname='sales';


---
P8.3

CREATE TABLE contacts (
    contact_id serial PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    company VARCHAR(50),
    department VARCHAR(50),
    phone VARCHAR(50),
    Email VARCHAR(50) 
);

CREATE INDEX idx_contact_name 
ON contacts((first_name || ' ' || last_name));



---
P8.4
CREATE TABLE contacts (
    contact_id serial PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    company VARCHAR(50),
    department VARCHAR(50),
    phone VARCHAR(50),
    Email VARCHAR(50) 
);

CREATE INDEX idx_contact_name 
ON contacts((first_name || ' ' || last_name));


---
P8.5

CREATE TABLE articles (   
id SERIAL PRIMARY KEY,   
title TEXT NOT NULL, 
body TEXT NOT NULL ); 
-- Insert 4 test records 
INSERT INTO articles (title, body) 
VALUES 
  ('PostgreSQL Full Text Search', 'PostgreSQL has a powerful Full Text Search functionality.'), 
  ('How to use FTS in PostgreSQL', 'In order to use FTS in PostgreSQL, you need to create a text search configuration and define a full text index.'), 
  ('Why use FTS in PostgreSQL', 'FTS in PostgreSQL is great for searching large amounts of textual data quickly and efficiently.'), 
  ('Limitations of FTS in PostgreSQL', 'FTS in PostgreSQL is not as feature-rich as some other search engines, but it is still very powerful.'); 

INSERT INTO articles (title, body)
VALUES
    ('PostgreSQL Full Text Search', 'PostgreSQL has a powerful Full Text Search functionality.'),
    ('How to use FTS in PostgreSQL', 'In order to use FTS in PostgreSQL, you need to create a text search configuration and define a full text index.'),
    ('Why use FTS in PostgreSQL', 'FTS in PostgreSQL is great for searching large amounts of textual data quickly and efficiently.'),
    ('Limitations of FTS in PostgreSQL', 'FTS in PostgreSQL is not as feature-rich as some other search engines, but it is still very powerful.');

CREATE TEXT SEARCH CONFIGURATION english_fts (COPY = english); 

ALTER TEXT SEARCH CONFIGURATION english_fts  
ALTER MAPPING FOR asciiword WITH english_stem; 
CREATE INDEX articles_fts_idx 
ON articles 
USING gin(to_tsvector('english_fts', title || ' ' || body)); 

SELECT * FROM articles
WHERE to_tsvector('english_fts', title || ' ' || body) @@ to_tsquery('english_fts', 'PostgreSQL');



---
P8.6

CREATE INDEX CONCURRENTLY IF NOT idx_sales_all 
ON sales(sale_date, sales_id, amount) ;



---
P8.7

CREATE TABLE books  ( 
book_id SERIAL PRIMARY KEY,  
title VARCHAR(100) NOT NULL, 
author VARCHAR(100) NOT NULL,
publication_year INTEGER, 
genre VARCHAR(50),
qty INTEGER, 
unit_price DECIMAL );

CREATE INDEX CONCURRENTLY idx_books_genre_price 
ON  books 
USING BTREE (genre, unit_price);



---
P8.8

CREATE TABLE my_long_text (     
id SERIAL PRIMARY KEY,     
text_column TEXT 
); 

INSERT INTO my_long_text (text_column) 
VALUES ('GiST stands for Generalized Search Tree and it is an index structure in PostgreSQL that is suitable for handling complex data types and complex queries. It is an alternative index type to the B-tree index and can be used when the B-tree index is not suitable for the data type or the query. 
B-tree index is not suitable for some data types such as text and array data types. These data types require more complex and specialized indexing methods, such as GIN and GiST indexes. B-tree index is designed for range queries, and its performance degrades when handling large amounts of data or complex queries. '); 

INSERT INTO my_long_text (text_column) 
VALUES ('B-tree index is not suitable for some data types such as TEXT or array of text. These data types require more complex and specialized indexing methods, such as Gist indexes since B-tree index is designed for range queries and B-tree performance degrades when handling large amounts of data or complex queries.'); 

INSERT INTO my_long_text (text_column) 
VALUES ('The GiST index can handle many different types of data such as full-text search, geometric data, and even custom data types that you define. It supports operations like equality, inequality, and distance-based searches.'); 

INSERT INTO my_long_text (text_column) 
VALUES ('A functional index in PostgreSQL, also called Expression index,  is an index built on an expression rather than on a simple column or columns. A functional index is useful when you want to index the result of a function or expression instead of just the value in a column.'); 

CREATE EXTENSION pg_trgm;

CREATE INDEX idx_long_text_gist 
ON my_long_text 
USING GIST (text_column gist_trgm_ops);

EXPLAIN ANALYZE 
SELECT * 
FROM my_long_text 
WHERE text_column @@ to_tsquery('suitable') 
AND text_column % '%suitable%';



---
P8.9

CREATE TABLE my_json (id SERIAL PRIMARY KEY, data JSON );
 
INSERT INTO my_json (data) VALUES 
    ('{"name": "John", "age": 30, "city": "New York"}'), 
    ('{"name": "Jane", "age": 25, "city": "San Francisco"}'), 
    ('{"name": "Bob", "age": 40, "city": "Chicago"}'), 
    ('{"name": "Alice", "age": 35, "city": "Seattle"}'), 
    ('{"name": "Mike", "age": 45, "city": "Boston"}'); 
    
CREATE INDEX idx_gin_json 
ON my_json USING GIN ((data->>'name') gin_trgm_ops); 

SELECT * FROM my_json 
WHERE data->>'name' = 'Alice'; 

EXPLAIN ANALYZE
SELECT * FROM my_json
WHERE data->>'name' = 'Alice';



---
P8.10

CREATE INDEX idx_brin_books 
ON books 
USING BRIN (publication_year); 

EXPLAIN SELECT * 
FROM books 
WHERE publication_year >= 1970;



---
P8.11

CREATE EXTENSION IF NOT EXISTS earthdistance CASCADE; 

CREATE TABLE my_network (   
id SERIAL PRIMARY KEY, 
address cidr,   
country_code varchar(2),   
location_point point, 
created_at timestamp default current_timestamp 
); 

INSERT INTO my_network (address, country_code, location_point) 
VALUES  
  ('192.168.0.0/24', 'US', point(40.7128, -74.0060)), 
  ('192.168.1.0/24', 'GB', point(51.5074, -0.1278)), 
  ('192.168.2.0/24', 'DE', point(52.5200, 13.4050)), 
  ('192.168.3.0/24', 'FR', point(48.8566, 2.3522)), 
  ('192.168.4.0/24', 'JP', point(35.6762, 139.6503)); 
CREATE INDEX idx_spgist_my_network 
ON my_network 
USING spgist(location_point); 

SELECT * 
FROM my_network 
WHERE earth_box(ll_to_earth(40.7128,-74.0060), 10000) @> ll_to_earth(location_point[0], location_point[1]); 


EXPLAIN 
SELECT * 
FROM my_network 
WHERE earth_box(ll_to_earth(40.7128,-74.0060), 10000) @> ll_to_earth(location_point[0], location_point[1]);



---
P8.12

INSERT INTO  books (title, author, publication_year, genre, qty, unit_price) 
VALUES  
('The Great Gatsby', 'F. Scott Fitzgerald', 1925, 'Literary Fiction', 10, 9.99), 
('To Kill a Mockingbird', 'Harper Lee', 1960, 'Literary Fiction', 15, 12.99), 
('1984', 'George Orwell', 1949, 'Dystopian Fiction', 5, 7.99), 
('Pride and Prejudice', 'Jane Austen', 1813, 'Romantic Fiction', 8, 6.99), 
('The Catcher in the Rye', 'J.D. Salinger', 1951, 'Literary Fiction', 12, 10.99); 
 
CREATE INDEX idx_hash_books 
ON books  USING HASH (author); 

SELECT * 
FROM books 
WHERE author IN ('F. Scott Fitzgerald','J.D. Salinger'); 

EXPLAIN 
SELECT * FROM books 
WHERE author IN ('F. Scott Fitzgerald','J.D. Salinger'); 



---
P9.1

CREATE DATABASE test_db;

CREATE ROLE myuser LOGIN ENCRYPTED PASSWORD 'mypassword' VALID UNTIL '2024-12-31';
GRANT ALL PRIVILEGES ON DATABASE test_db TO myuser;

ALTER ROLE myuser CONNECTION LIMIT 2;




---
P9.2

CREATE SCHEMA marketing;

GRANT ALL PRIVILEGES ON SCHEMA marketing TO myuser;


---
P9.3

REVOKE ALL PRIVILEGES ON SCHEMA marketing FROM myuser;


---
P9.4

ALTER DEFAULT PRIVILEGES IN SCHEMA marketing
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO myuser;



---
P9.5

GRANT SELECT, INSERT, UPDATE, DELETE, TRUNCATE, TRIGGER ON TABLE books TO myuser;



---
P9.6

REVOKE TRUNCATE ON TABLE books FROM myuser;


---
P9.7

GRANT SELECT (book_id, title, author) ON books TO myuser;


---
P9.8

REVOKE SELECT (author) ON books FROM myuser;




---
P9.9

CREATE ROLE books_admin_group WITH LOGIN;

GRANT SELECT, INSERT, UPDATE, DELETE ON books TO books_admin_group;

CREATE POLICY insert_update_delete_books_policy ON books
FOR ALL TO books_admin_group
USING (true) 
WITH CHECK (true);

CREATE ROLE literary_group WITH LOGIN;

GRANT SELECT ON books TO literary_group;

CREATE POLICY select_literary_fiction_policy ON books
FOR SELECT TO literary_group
USING (genre = 'Literary Fiction');

ALTER TABLE books ENABLE ROW LEVEL SECURITY;

CREATE ROLE dystopian_group WITH LOGIN;

GRANT SELECT ON books TO dystopian_group;

CREATE POLICY select_dystopian_fiction_policy ON books
FOR SELECT TO dystopian_group 
USING (true)
USING (genre = 'Dystopian Fiction');

CREATE ROLE romantic_group WITH LOGIN;

GRANT SELECT ON books TO romantic_group;

CREATE POLICY select_romantic_fiction_policy ON books
FOR SELECT TO romantic_group 
USING (genre = 'Romantic Fiction');

CREATE USER alice WITH ENCRYPTED PASSWORD 'alice_password'; 

GRANT books_admin_group TO alice;

CREATE USER peter WITH ENCRYPTED PASSWORD 'peter_password';

GRANT literary_group TO peter;

CREATE USER john WITH ENCRYPTED PASSWORD 'john_password';

GRANT dystopian_group TO john;

CREATE USER susan WITH ENCRYPTED PASSWORD 'susan_password';

GRANT romantic_group TO susan;

SET ROLE alice;

INSERT INTO books (book_id, title, author, publication_year, genre, qty, unit_price) 
VALUES (6, 'New Book', 'New Author', 2023, 'New Genre', 100, 9.99);

UPDATE books set author = 'Alice' WHERE book_id =6;

SELECT * FROM books;

RESET ROLE;


SET ROLE peter;

SELECT * FROM books;

RESET ROLE;

SET ROLE susan;

SELECT * FROM books;

RESET ROLE;


---
P9.10

CREATE USER alex 
WITH PASSWORD 'alex_password' 
CREATEDB 
INHERIT VALID UNTIL '2024-12-31';



---
P9.11

CREATE USER candice 
WITH ENCRYPTED   PASSWORD 'candice_password' 
CREATEDB 
INHERIT VALID UNTIL '2024-12-31';


---
P9.12
ALTER USER alex 
WITH ENCRYPTED PASSWORD 'alex_new_password'; 


---
P9.13

DROP USER alex;



---
P9.14

CREATE GROUP marketing;

ALTER GROUP marketing  ADD USER peter; 

ALTER GROUP marketing 
DROP USER peter; 



---
P9.15

SELECT usename, ssl 
FROM pg_stat_ssl JOIN pg_user 
ON pg_stat_ssl.pid = pg_backend_pid() 
WHERE usename = 'postgres'; 




---
P9.16

ALTER ROLE peter CONNECTION LIMIT 1; 

ALTER ROLE peter CONNECTION LIMIT DEFAULT; 



---
P9.17 (Please refer to the book)



---
P9.18

SELECT  pid,  usename,  application_name, client_addr, backend_start 
FROM pg_stat_activity 
WHERE usename = 'peter';










