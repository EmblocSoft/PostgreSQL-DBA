
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






