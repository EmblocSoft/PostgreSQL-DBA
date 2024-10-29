
# PostgreSQL DBA (v17, 16, 15, 14, 13) - 2025 Edition

</br>
*** The book, "PostgreSQL DBA", is published on Amazon, it covers PostgreSQL v13, 14, 15, 16 to the latest v17 ***

</br>
</br>
The "PostgreSQL DBA 17 (2025 Edition)" book's URL:</br>
https://www.amazon.com/PostgreSQL-DBA-v16-v15-Administrators/dp/B0CN5FD1M8/ref=tmm_pap_swatch_0?_encoding=UTF8&qid=1700880466&sr=8-2

</br>
</br>
The "PostgreSQL DBA 16 (2024 Edition)" book's Kindle Edition:</br>
https://www.amazon.com/PostgreSQL-DBA-v16-Administrators-Availablity-ebook/dp/B0CN3HYFZJ/ref=sr_1_2?crid=UVJD8VVHJ85P&keywords=Postgresql&qid=1700880466&sprefix=postgres%2Caps%2C751&sr=8-2

</br>
</br>


Important Note 1: The following commands are FOR INFORMATION ONLY

---
5.10.1

CREATE TABLE demo_table (
    id serial PRIMARY KEY, name VARCHAR(50), children INT);

BEGIN;

INSERT INTO demo_table (name) VALUES ('John');
INSERT INTO demo_table (name) VALUES ('Alice');

SELECT * FROM demo_table;

---
5.10.2
-- Go back to Session 1 of PSQL:

BEGIN;

INSERT INTO demo_table (name, children) VALUES ('Alex', 2);

INSERT INTO demo_table (name, children) VALUES ('Sam', 'NO CHILD');

SELECT * FROM demo_table;

ROLLBACK;
SELECT * FROM demo_table;


---
6.1
BEGIN; 

CREATE TABLE employee(
    id INT,
    first_name VARCHAR (50),
    last_name VARCHAR (50),
    salary numeric(10, 2));

EXPLAIN ANALYZE INSERT INTO 
employee (first_name, last_name, salary)
VALUES
    ('John', 'Doe', 50000.00),
    ('Jane', 'Smith', 60000.00),
    ('Bob', 'Johnson', 55000.00);

EXPLAIN ANALYZE SELECT * FROM employee;

ROLLBACK;

---
6.2

nano postgresql.conf  

log_statement = 'all'

log_duration = on

shared_preload_libraries = 'pg_stat_statements'

log_min_duration_statement = 1000    # 1000 milliseconds


CREATE EXTENSION pg_stat_statements;

SELECT query, total_exec_time, calls
FROM pg_stat_statements
WHERE total_exec_time > 1000000   
ORDER BY total_exec_time DESC;

---
6.3

DROP TABLE customers;

DROP TABLE orders;

CREATE TABLE customers 
    customer_id serial PRIMARY KEY
    customer_name varchar(255);

CREATE TABLE orders (
    order_id serial PRIMARY KEY,
    customer_id integer,
    order_date date,
    total_amount numeric);

INSERT INTO customers (customer_name)

SELECT md5(random()::text) FROM generate_series(1, 1000000);

INSERT INTO orders (customer_id, order_date, total_amount)
SELECT
    floor(random() * 100000) + 1,
    current_date - floor(random() * 365)::integer,
    CAST(random() * 1000 AS numeric(10, 2))
FROM generate_series(1, 10000000);

---
Q1

EXPLAIN ANALYZE 
SELECT customer_name
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
    WHERE total_amount > (
        SELECT AVG(total_amount)
        FROM orders
    ));

---
Q2

EXPLAIN ANALYZE 
SELECT customer_id
FROM orders
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM orders);

---
Q3

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

---
Q4

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

CREATE INDEX idx_total_amount ON orders(total_amount);
CREATE INDEX idx_customer_id ON orders(customer_id);

---
6.4

EXPLAIN ANALYZE
SELECT COUNT(1)FROM customers
JOIN orders ON customers.customer_id = orders.customer_id;

EXPLAIN ANALYZE
SELECT COUNT(1)FROM customers
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
6.5

SELECT relname AS table_name, indexrelname AS index_name, idx_scan, 
idx_tup_read, ROUND(CAST(idx_tup_read AS numeric) / NULLIF(idx_scan, 0), 2) AS avg_tup_read_per_scan  
FROM pg_stat_user_indexes 
WHERE idx_scan > 0 
ORDER BY avg_tup_read_per_scan ASC;

---
6.6

SELECT schemaname,  attname, n_distinct, most_common_vals, most_common_freqs 
FROM pg_stats 
WHERE 
schemaname NOT LIKE  'pg_%' AND schemaname <> 'information_schema'  
AND n_distinct <= 10 
AND most_common_vals IS NOT NULL;

---
6.7

SELECT * FROM pg_stat_activity 
WHERE state = 'active' AND 
now() - pg_stat_activity.query_start > interval '60 seconds';

---
6.8

SELECT query,  state, backend_start,  now() - query_start AS duration 
FROM pg_stat_activity 
WHERE state = 'active'  AND now() - query_start > interval '2 minutes' 
ORDER BY now() - query_start DESC;

---
6.9

SELECT mode, relation::regclass, pid, granted,  waitstart 
FROM pg_locks 
WHERE granted = false 
ORDER BY granted, relation::regclass;

---
7.1
---R1 

EXPLAIN ANALYZE 
SELECT * FROM orders
ORDER BY total_amount;

---R2

EXPLAIN ANALYZE 
SELECT *
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

---R3

EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

---
7.2

CREATE INDEX CONCURRENTLY idx_customer_id ON customers(customer_id);

CREATE INDEX CONCURRENTLY idx_order_id ON orders(order_id);

CREATE INDEX CONCURRENTLY idx_total_amount_gt_100 ON orders(total_amount) WHERE total_amount >= 100;

---R4

EXPLAIN ANALYZE 
SELECT * FROM orders
ORDER BY total_amount;

---R5

EXPLAIN ANALYZE 
SELECT *
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

---R6

EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

DROP INDEX idx_total_amount_gt_100;

---R7

EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

REINDEX INDEX CONCURRENTLY idx_customer_id;

ALTER INDEX idx_customer_id  SET (fillfactor = 80);

DROP INDEX idx_customer_id ;

---
7.3

EXPLAIN ANALYZE SELECT customer_name FROM customers
WHERE 
(customer_name >= '10000' AND customer_name <= '100000') OR (customer_name >= '200000' AND customer_name <= '300000');

CREATE INDEX CONCURRENTLY idx_customer_name 
ON customers(customer_name);

EXPLAIN ANALYZE 
SELECT customer_name FROM customers WHERE 
(customer_name >= '10000' AND customer_name <= '100000') OR      (customer_name >= '200000' AND customer_name <= '300000');

---
7.4

DROP INDEX IF EXISTS idx_customer_name;

DROP INDEX IF EXISTS customers_customer_name_idx;

EXPLAIN ANALYZE 
SELECT *
FROM customers
ORDER BY customer_name;

--Create index
CREATE INDEX CONCURRENTLY idx_customer_name ON customers(customer_name);

EXPLAIN ANALYZE 
SELECT *
FROM customers
ORDER BY customer_name;

---
7.5.2

SELECT 
datname, pg_size_pretty(pg_database_size(datname)) AS size 
FROM pg_database  
ORDER BY pg_database_size(datname) DESC; 

---
7.5.3

SELECT relname, pg_size_pretty(pg_total_relation_size(relid)) 
FROM pg_catalog.pg_statio_user_tables 
ORDER BY pg_total_relation_size(relid) DESC; 

---
7.7.1

CREATE INDEX IF NOT EXISTS idx_total_amount ON orders(total_amount);

SET enable_seqscan = off;

EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders WHERE order_date >= '2023-01-01' 
AND total_amount >= 100 ORDER BY total_amount;

SET enable_seqscan = on; 

EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders 
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

---
7.7.2

DROP INDEX idx_total_amount;

CREATE INDEX IF NOT EXISTS idx_total_amount ON orders(total_amount);

EXPLAIN ANALYZE 
SELECT customer_id
FROM orders
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM orders);

SET enable_bitmapscan = on;

EXPLAIN ANALYZE 
SELECT customer_id
FROM orders
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM orders);

---
7.7.3

SET enable_indexscan = off;
EXPLAIN ANALYZE 
SELECT customer_id, total_amount
ROM orders
WHERE total_amount >= 100000 AND total_amount <= 1000000;

SET enable_indexscan = on;

EXPLAIN ANALYZE 
SELECT customer_id, total_amount
FROM orders
WHERE total_amount >= 100000 AND total_amount <= 1000000;


---
7.9

BEGIN;

SELECT * FROM orders WHERE total_amount >= 1000 FOR UPDATE;

UPDATE orders SET total_amount = total_amount * 1.10 
WHERE total_amount >= 1000;

COMMIT;

---
7.10

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
    
    LOOP
    
        FETCH NEXT FROM my_cursor INTO customer_id, 
        
                        order_date, total_amount;
        EXIT WHEN NOT FOUND;

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
7.10.2

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

---
7.11.1

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_customer_id 
ON orders(customer_id);

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_order_date 
ON orders(order_date);

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_customers_customer_name 
ON customers(customer_name);

---
7.11.2

EXPLAIN ANALYZE 
SELECT *
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount > 1000;

---
7.11.3

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
7.11.4

EXPLAIN ANALYZE 
SELECT * 
FROM customers 
WHERE EXISTS ( 
      SELECT 1
      FROM orders
      WHERE customers.customer_id = orders.customer_id);

---
7.11.5

---C1

EXPLAIN ANALYZE SELECT * FROM customers
JOIN orders ON customers.customer_id = orders.customer_id; 

---C2

EXPLAIN ANALYZE SELECT * FROM customers 
JOIN orders ON orders.customer_id = customers.customer_id;

---C3

EXPLAIN ANALYZE SELECT * FROM orders 
JOIN customers ON orders.customer_id = customers.customer_id;

---C4

EXPLAIN ANALYZE SELECT * FROM orders 
JOIN customers ON customers.customer_id = orders.customer_id ; 

---C5

EXPLAIN ANALYZE  SELECT * FROM customers 
LEFT JOIN orders ON customers.customer_id = orders.customer_id;

---C6

EXPLAIN ANALYZE SELECT * FROM customers 
LEFT JOIN orders ON orders.customer_id = customers.customer_id;

---C7

EXPLAIN ANALYZE SELECT * FROM orders
LEFT JOIN customers ON orders.customer_id = customers.customer_id;

---C8

EXPLAIN ANALYZE SELECT * FROM orders
LEFT JOIN customers ON customers.customer_id = orders.customer_id ;

---C9

EXPLAIN ANALYZE SELECT * FROM customers 
RIGHT JOIN orders ON customers.customer_id = orders.customer_id;

---C10

EXPLAIN ANALYZE SELECT * FROM customers 
RIGHT JOIN orders ON orders.customer_id = customers.customer_id;

---C11

EXPLAIN ANALYZE SELECT * FROM orders
RIGHT JOIN customers ON orders.customer_id = customers.customer_id;

--C12

EXPLAIN ANALYZE SELECT * FROM orders
RIGHT JOIN customers ON customers.customer_id = orders.customer_id ;

---C13

EXPLAIN ANALYZE SELECT * FROM customers 
FULL JOIN orders ON customers.customer_id = orders.customer_id;

---C14

EXPLAIN ANALYZE SELECT * FROM customers 
FULL JOIN orders ON orders.customer_id = customers.customer_id;

---C15
EXPLAIN ANALYZE SELECT * FROM orders
FULL JOIN customers ON orders.customer_id = customers.customer_id;

---C16

EXPLAIN ANALYZE SELECT * FROM orders
FULL JOIN customers ON customers.customer_id = orders.customer_id ;

---C17

EXPLAIN ANALYZE SELECT * FROM customers 
FULL OUTER JOIN orders ON customers.customer_id = orders.customer_id;

---C18

EXPLAIN ANALYZE SELECT * FROM customers 
FULL OUTER JOIN orders ON orders.customer_id = customers.customer_id;

---C19

EXPLAIN ANALYZE SELECT * FROM orders
FULL OUTER JOIN customers ON orders.customer_id = customers.customer_id;

---C20

EXPLAIN ANALYZE SELECT * FROM orders
FULL OUTER JOIN customers ON customers.customer_id = orders.customer_id ;


---
7.11.6

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
7.12

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
SELECT year, total_sales FROM yearly_sales_total ORDER BY year;

EXPLAIN ANALYZE 
SELECT
    DATE_TRUNC('year', order_date) AS year,
    SUM(total_amount) AS total_sales
FROM orders GROUP BY year;

8.1.1

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_customers_customer_name 
ON customers(customer_name);

REINDEX INDEX CONCURRENTLY idx_customers_customer_name;

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_date_amount 
ON orders(order_date, total_amount);

---
8.1.2

SELECT * FROM pg_stat_activity 
WHERE query ILIKE '%CREATE INDEX%';

SELECT pg_cancel_backend(8044);

---
8.2.1

CREATE TEMP SEQUENCE temp_sales_id_seq START 1;

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
8.2.2

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
8.3

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
8.4

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_sales_new_in_365d 
ON sales(sale_date, sales_id, amount) 
WHERE EXTRACT(YEAR FROM sale_date) >= (current_year() - 1);

CREATE INDEX CONCURRENTLY IF NOT idx_sales_all 
ON sales(sale_date, sales_id, amount) ;

---
8.5

CREATE TABLE articles (   
id SERIAL PRIMARY KEY,   
title TEXT NOT NULL, 
body TEXT NOT NULL );

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

ALTER TEXT SEARCH CONFIGURATION english_fts;  
ALTER MAPPING FOR asciiword WITH english_stem;

CREATE INDEX articles_fts_idx 
ON articles 
USING gin(to_tsvector('english_fts', title || ' ' || body)); 

SELECT * FROM articles
WHERE to_tsvector('english_fts', title || ' ' || body) @@ to_tsquery('english_fts', 'PostgreSQL');

---
8.6

CREATE INDEX CONCURRENTLY IF NOT idx_sales_all 
ON sales(sale_date, sales_id, amount) ;

---
8.7

CREATE TABLE books  ( 
book_id SERIAL PRIMARY KEY,  
title VARCHAR(100) NOT NULL, 
author VARCHAR(100) NOT NULL,
publication_year INTEGER, 
genre VARCHAR(50),
qty INTEGER, 
unit_price DECIMAL );

CREATE INDEX CONCURRENTLY idx_books_genre_price ON books 
USING BTREE(genre, unit_price);

---
8.8

CREATE TABLE my_long_text (     
id SERIAL PRIMARY KEY,     
text_column TEXT );

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

CREATE INDEX idx_long_text_gist ON my_long_text 
USING GIST (text_column gist_trgm_ops);

EXPLAIN ANALYZE 
SELECT * FROM my_long_text 
WHERE text_column @@ to_tsquery('suitable') AND text_column % '%suitable%';

---
8.9

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
8.10

CREATE INDEX idx_brin_books 
ON books 
USING BRIN (publication_year); 

EXPLAIN SELECT * 
FROM books 
WHERE publication_year >= 1970;

---
8.11

CREATE EXTENSION IF NOT EXISTS earthdistance CASCADE;

CREATE TABLE my_network (   
id SERIAL PRIMARY KEY, address cidr,   
country_code varchar(2), location_point point, created_at timestamp default current_timestamp ); 

NSERT INTO my_network (address, country_code, location_point) 
VALUES  
  ('192.168.0.0/24', 'US', point(40.7128, -74.0060)), 
  ('192.168.1.0/24', 'GB', point(51.5074, -0.1278)), 
  ('192.168.2.0/24', 'DE', point(52.5200, 13.4050)), 
  ('192.168.3.0/24', 'FR', point(48.8566, 2.3522)), 
  ('192.168.4.0/24', 'JP', point(35.6762, 139.6503));

CREATE INDEX idx_spgist_my_network ON my_network 
USING spgist(location_point);

SELECT * FROM my_network 
WHERE earth_box(ll_to_earth(40.7128,-74.0060), 10000) @> ll_to_earth(location_point[0], location_point[1]); 

EXPLAIN  SELECT * FROM my_network 
WHERE earth_box(ll_to_earth(40.7128,-74.0060), 10000) @> ll_to_earth(location_point[0], location_point[1]);

---
8.12

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
9.1

CREATE DATABASE test_db;

CREATE ROLE myuser LOGIN ENCRYPTED PASSWORD 'mypassword' VALID UNTIL '2024-12-31';
GRANT ALL PRIVILEGES ON DATABASE test_db TO myuser;

ALTER ROLE myuser CONNECTION LIMIT 2;

---
9.2

CREATE SCHEMA marketing;
GRANT ALL PRIVILEGES ON SCHEMA marketing TO myuser;

---
9.3

REVOKE ALL PRIVILEGES ON SCHEMA marketing FROM myuser;

---
9.4

ALTER DEFAULT PRIVILEGES IN SCHEMA marketing
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO myuser;

---
9.5

GRANT SELECT, INSERT, UPDATE, DELETE, TRUNCATE, TRIGGER ON TABLE books TO myuser;

---
9.6

REVOKE TRUNCATE ON TABLE books FROM myuser;

---
9.7

GRANT SELECT (book_id, title, author) ON books TO myuser;

---
9.8

REVOKE SELECT (author) ON books FROM myuser;

---
9.9

CREATE ROLE books_admin_group WITH LOGIN;

GRANT SELECT, INSERT, UPDATE, DELETE 
ON books TO books_admin_group;

CREATE POLICY insert_update_delete_books_policy ON books
FOR ALL TO books_admin_group USING (true) WITH CHECK (true);

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

-- Create USER alice and assign to admin group
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

SELECT * FROM books WHERE genre <> 'Literary Fiction';

SET ROLE susan;

SELECT * FROM books;

RESET ROLE;

---
9.10

CREATE USER alex 
WITH PASSWORD 'alex_password' 
CREATEDB 
INHERIT VALID UNTIL '2024-12-31';

---
9.11

CREATE USER candice 
WITH ENCRYPTED PASSWORD 'candice_password' 
CREATEDB 
INHERIT VALID UNTIL '2024-12-31';

---
9.12

ALTER USER alex 
WITH ENCRYPTED PASSWORD 'alex_new_password'; 

---
9.13

DROP USER alex;

---
9.14
CREATE GROUP marketing;

ALTER GROUP marketing  ADD USER peter; 

ALTER GROUP marketing 
DROP USER peter; 

---
9.15

SELECT usename, ssl 
FROM pg_stat_ssl JOIN pg_user 
ON pg_stat_ssl.pid = pg_backend_pid() 
WHERE usename = 'postgres'; 

---
9.16

ALTER ROLE peter 
CONNECTION LIMIT 1; 

ALTER ROLE peter 
CONNECTION LIMIT DEFAULT; 

---
9.18

SELECT  pid,  usename,  application_name, client_addr, backend_start 
FROM pg_stat_activity 
WHERE usename = 'peter';

---
10.1

SELECT d.datname as database, r.rolname as owner
FROM pg_catalog.pg_database d, pg_catalog.pg_roles r 
WHERE d.datdba = r.oid;

SELECT grantee AS user, CONCAT(table_schema, '.', table_name) AS table,  
CASE WHEN COUNT(privilege_type) = 7 
THEN    
    'ALL' 
ELSE  
    ARRAY_TO_STRING(ARRAY_AGG(privilege_type), ', ')
END AS grants 
FROM information_schema.role_table_grants 
GROUP BY table_name, table_schema, grantee 
ORDER BY grantee;

SELECT r.rolname, r.rolsuper, r.rolinherit, r.rolcreaterole, r.rolcreatedb, r.rolcanlogin,  r.rolconnlimit, r.rolvaliduntil, ARRAY(SELECT b.rolname 
FROM pg_catalog.pg_auth_members m 
JOIN pg_catalog.pg_roles b ON (m.roleid = b.oid) 
WHERE m.member = r.oid) 
AS memberof, r.rolreplication, r.rolbypassrls 
FROM pg_catalog.pg_roles r 
WHERE r.rolname !~ '^pg_' 
ORDER BY 1; 

---
10.2

SELECT name, setting, unit, short_desc 
FROM pg_settings 
WHERE name IN ('shared_buffers', 'work_mem', 'maintenance_work_mem', 'effective_cache_size', 'wal_buffers');

---
10.3

SELECT datname, blks_read, blks_hit, temp_files, temp_bytes FROM pg_stat_database;

---
10.4

SELECT pg_size_pretty(pg_database_size('my_db')) 
AS database_size;

---
10.5

SELECT * FROM pg_stat_activity;

SELECT COUNT(*) FROM pg_stat_activity;

SELECT state, COUNT(*) FROM pg_stat_activity GROUP BY state;

SELECT pid, now() - query_start AS duration, query, state 
FROM pg_stat_activity 
WHERE (now() - query_start) > interval '5 minutes';

SELECT a.pid, a.usename, a.query, l.mode, l.granted 
FROM pg_stat_activity a 
JOIN pg_locks l ON a.pid = l.pid 
WHERE NOT l.granted;

---
10.6

SELECT * FROM pg_locks WHERE NOT granted;

SELECT a.pid AS "Blocked PID",
       b.query AS "Blocked Query",
       b.pid AS "Blocking PID",
       b.query AS "Blocking Query"
FROM pg_locks a
JOIN pg_stat_activity b ON a.pid = b.pid
WHERE NOT a.granted;

---
10.7

SELECT 
pg_last_wal_receive_lsn(), 
pg_last_wal_replay_lsn(),       
pg_wal_lsn_diff(pg_last_wal_receive_lsn(), 
pg_last_wal_replay_lsn()) AS lag_in_receive_replay 
FROM pg_replication_slots;


---
11.1

CREATE SCHEMA marketing;

SELECT nspname FROM pg_namespace;

book_id SERIAL PRIMARY KEY,     
title VARCHAR(100) NOT NULL,     
author VARCHAR(100) NOT NULL,     
publication_year INTEGER, 
genre VARCHAR(50) );

INSERT INTO marketing.new_books (title, author, publication_year, genre)
VALUES 
 ('PostgreSQL Essentials', 'Jane Doe', 2021, 'Technical'),
 ('Data-Driven Marketing', 'John Smith', 2020, 'Business'),
 ('The Art of SQL', 'Pat Author', 2022, 'Technical');

SELECT book_id, title, author FROM marketing.new_books;

---
11.2

SELECT  relname AS table_name,  indexrelname AS index_name,  idx_scan, 
idx_tup_read,  idx_tup_fetch  
FROM pg_stat_user_indexes ORDER BY idx_scan ASC;


---
11.3

SELECT * 
FROM pg_indexes 
WHERE schemaname = 'public' AND 
tablename = 'sales'  AND 
indexname = 'idx_sales_new_in_365d';

---
11.4
DROP INDEX idx_sales_new_in_365d;

---
11.5

CREATE SEQUENCE client_id_seq 
START WITH 1  INCREMENT BY 1 
NO MINVALUE NO MAXVALUE 
CACHE 1; 

CREATE TABLE clients ( 
id INTEGER DEFAULT nextval('client_id_seq'),
name TEXT,
email TEXT, 
PRIMARY KEY (id)); 

INSERT INTO clients 
VALUES
(nextval('client_id_seq'), 'ABC LLC', 'info@abc.com');

INSERT INTO clients 
VALUES
(nextval('client_id_seq'), 'ABC LLC', 'info@abc.com');

SELECT currval('client_id_seq');

SELECT * FROM clients;


---
11.6

CREATE VIEW client_emails  AS SELECT name, email  
FROM clients; 

SELECT email FROM client_emails; 

ALTER TABLE clients ADD COLUMN phone TEXT;

CREATE OR REPLACE VIEW client_emails 
AS SELECT name, email, phone FROM clients; 


---
11.7

CREATE MATERIALIZED VIEW mv_clients 
AS SELECT  name, email, phone 
FROM clients ORDER BY name; 

psql -U my_user -d my_db -c "REFRESH MATERIALIZED VIEW mv_clients;"

SELECT * FROM mv_clients; 

REFRESH MATERIALIZED VIEW mv_clients;


---
11.8

CREATE OR REPLACE FUNCTION 

calculate_order_total(this_book_id INTEGER) 

RETURNS NUMERIC 

AS $$ 

DECLARE 

    total NUMERIC := 0; 
    
    item RECORD; 

BEGIN 

    FOR item IN SELECT qty, unit_price FROM books WHERE book_id = $1 
    
    LOOP 
    
        total := total + (item.qty * item.unit_price); 
    
    END LOOP; 
    
    RETURN total; 

END; 

$$ LANGUAGE plpgsql;

INSERT INTO books 
VALUES(7, 'SQL', 'Tony', 2023, 'Text book', 1000000, 2000);

SELECT  calculate_order_total(7);  


---
11.9

CREATE OR REPLACE FUNCTION insert_book() RETURNS

TRIGGER AS $$ 

BEGIN 

    IF NEW.qty <= 0 THEN 
    
        RAISE EXCEPTION 'Quantity must be greater than 0'; 
    
    END IF; 
    
    RETURN NEW; 

END; 

$$ LANGUAGE plpgsql; 

CREATE TRIGGER insert_book_trigger 

BEFORE INSERT ON books 

FOR EACH ROW 

EXECUTE FUNCTION insert_book();


CREATE OR REPLACE FUNCTION update_book() RETURNS 

TRIGGER AS $$ 

BEGIN 

    IF NEW.qty <= 0 THEN 

        RAISE EXCEPTION 'Quantity must be greater than 0';

    END IF; 

    RETURN NEW; 

END; 

$$ LANGUAGE plpgsql; 

CREATE TRIGGER update_book_trigger 
BEFORE UPDATE ON books 
FOR EACH ROW 
EXECUTE FUNCTION update_book();

CREATE OR REPLACE FUNCTION delete_book() 

RETURNS 

TRIGGER AS $$ 

BEGIN 

    IF OLD.qty <= 0 THEN 

        RAISE EXCEPTION 'Quantity must be greater than 0'; 

    END IF; 

    RETURN OLD; 

END; 

$$ LANGUAGE plpgsql;


CREATE TRIGGER delete_book_trigger 

BEFORE DELETE ON books 

FOR EACH ROW 

EXECUTE FUNCTION delete_book(); 

INSERT INTO books (book_id, title, author, publication_year, genre, qty, unit_price) 
VALUES (11, 'The Catcher in the Rye', 'J.D. Salinger', 1951, 'Fiction', 10, 9.99); 

INSERT INTO books (book_id, title, author, publication_year, genre, qty, unit_price) 
VALUES (12, 'Bad book', 'zero in QTY', 2023, 'Test', 0, 9.99); 

UPDATE books SET qty = 0 WHERE book_id = 11;

DELETE FROM books WHERE book_id = 11; 


---
11.10

CREATE TABLE clients (    
client_id SERIAL PRIMARY KEY,     
first_name VARCHAR(50) NOT NULL, 
last_name VARCHAR(50) NOT NULL ); 

CREATE TABLE orders (     
order_id SERIAL PRIMARY KEY,     
client_id INTEGER NOT NULL,     
order_date DATE NOT NULL, 
total_amount DECIMAL(10,2) NOT NULL, 
FOREIGN KEY (client_id) REFERENCES clients(client_id) );

---
11.11

CREATE USER hugo LOGIN ENCRYPTED PASSWORD 'hugo_password'; 

---
11.12

CREATE ROLE my_user; 

CREATE ROLE my_group; 

GRANT my_group TO my_user; 

GRANT SELECT, INSERT,  UPDATE,  DELETE, TRUNCATE
ON books TO my_user; 
 
REVOKE DELETE, UPDATE 
ON books FROM myuser;

ALTER ROLE my_user WITH LOGIN;

GRANT my_user TO hugo;


---
11.13

CREATE DOMAIN phone_number AS text 
CHECK (VALUE ~'^\d{3}-\d{3}-\d{4}$'); 
 
CREATE TABLE end_users (     
customer_id SERIAL PRIMARY KEY,     
first_name VARCHAR(50) NOT NULL,
last_name VARCHAR(50) NOT NULL, 
phone phone_number, 
email VARCHAR(100) NOT NULL);


---
11.14

CREATE FUNCTION reverse_concat(text, text) RETURNS text 

AS $$ 

    SELECT $2 || $1; 

$$ LANGUAGE SQL IMMUTABLE STRICT;

CREATE OPERATOR ||| ( 
    LEFTARG = text, 
    RIGHTARG = text, 
    PROCEDURE = reverse_concat );

SELECT 'Hello' ||| 'World' AS reversed_concatenation;


---
11.15

SELECT COUNT(1) FROM books; 

SELECT SUM(qty), AVG(qty),MIN(qty), MAX(qty) FROM books;


---
11.16

CREATE EXTENSION pgcrypto; 

SELECT encrypt('Hello, World!', 'my_secret_key'::bytea, 'aes') AS encrypted_data;

SELECT convert_from(decrypt('bytea_encrypted_data', '$1$ItVbOg5B$FgNj7Q7GvtQxdk4wYdWrP.'::bytea, 'aes'), 'UTF8');

SELECT decrypt('\x9982ec13ff9ee4341767a491b0ec38c5'::bytea, 'my_secret_key'::bytea, 'aes') AS decrypted_data;

SELECT convert_from('\x48656c6c6f2c20576f726c6421'::bytea, 'UTF8') AS readable_text;


---

CREATE SERVER my_remote_server 
FOREIGN DATA WRAPPER postgres_fdw 
OPTIONS 
(host 'myremotehost', dbname 'remote_db', port '5432');


---
11.17

CREATE DATABASE my_database 
WITH OWNER = my_user 
ENCODING = 'UTF8' 
LC_COLLATE = 'en_US.utf8' 
LC_CTYPE = 'en_US.utf8' 
TEMPLATE = template0; 

ALTER TABLE my_table 
ALTER COLUMN my_column 
TYPE varchar(255) 
COLLATE "en_US.utf8";


---
11.18

CREATE TABLE large_objects (
id SERIAL PRIMARY KEY, 
data OID); 

INSERT INTO large_objects (data) 
VALUES (lo_import('/appl/my_big_file.zip'));

SELECT * FROM large_objects; 

SELECT lo_export(data, '/tmp/my_big_file.zip') 
FROM large_objects 
WHERE id = 1; 


diff /appl/my_big_file.zip /tmp/my_big_file.zip 


---
11.19

CREATE RULE protect_books_delete 
AS ON DELETE TO books DO INSTEAD NOTHING;

SELECT * FROM books; 

DELETE FROM books; 

SELECT * FROM books;


---
11.20

CREATE TYPE gender 
AS ENUM ('Male', 'Female', 'Other'); 

CREATE TABLE users (   
id SERIAL PRIMARY KEY, 
name VARCHAR(50) NOT NULL,   
gender gender NOT NULL );


---
12.1
CREATE TABLE my_image (     
id SERIAL PRIMARY KEY,     
name VARCHAR(100),     
data BYTEA);

pip3 install psycopg2

nano /appl/load_image.py

import psycopg2 from psycopg2 

from psycopg2 import sql

conn = psycopg2.connect(     

   host="localhost",     
   
   database="my_db",     
   
   user="my_user",     
   
   password="my_password" ) 

with open("/appl/image.png", "rb") as f:     

   image_data = f.read() 

with conn.cursor() as cur:     

   query = sql.SQL("INSERT INTO my_image (name, data) VALUES (%s, %s)")   
 
   cur.execute(query, ("/appl/image.png", psycopg2.Binary(image_data)))     
   
   conn.commit() 

conn.close()

python3 load_image.py


---
12.2

DROP TABLE my_json;

CREATE TABLE my_json (     
id SERIAL PRIMARY KEY, 
data_json JSON, 
data_jsonb JSONB );

INSERT INTO my_json (data_json, data_jsonb) 
VALUES ('{"name": "Eve", "age": 25}', '{"name": "Eve", "age": 25}');
 
INSERT INTO my_json (data_json, data_jsonb) 
VALUES ('{"name": "Gina", "age": 41, "address": {"street": "123 Main St", "city": 
"Anytown", "state": "CA"}}', '{"name": "Gina", "age": 41, "address": {"street": "123 Main St", "city": "Anytown", "state": "CA"}}');

INSERT INTO my_json (data_json, data_jsonb) 
VALUES ('{"name": "Frank", "age": 33, "hobbies": ["gaming", "reading"]}', 
'{"name": "Frank", "age": 33, "hobbies": ["gaming", "reading"]}');

INSERT INTO my_json (data_json, data_jsonb) 
VALUES ('{"name": "Ivy", "age": 52, "address": {"street": "789 Elm St", "city": 
"Othertown", "state": "TX"}}', '{"name": "Ivy", "age": 52, "address": {"street": "789 Elm St", "city": "Othertown", "state": "TX"}}'); 

INSERT INTO my_json (data_json, data_jsonb) 
VALUES ('{"name": "John", "age": 30, "hobbies": ["photography"], "address": {"street": "1010 Pine St", "city": "AnotherTown", "state": "CA"}}', '{"name": "John", "age": 30, "hobbies": ["photography"], "address": {"street": "1010 Pine St", "city": "AnotherTown", "state": "CA"}}');

CREATE INDEX idx_gin_data_json 
ON my_json 
USING GIN ((data_json->>'name') gin_trgm_ops);

SELECT * 
FROM my_json 
WHERE data_json->>'name' = 'John'; 

SELECT  data_json->>'name' AS name,  data_json->>'age' AS age 
FROM my_json 
WHERE CAST(data_json->>'age' AS INTEGER) > 20; 

CREATE INDEX idx_gin_data_jsonb 
ON my_json 
USING GIN ((data_jsonb->>'name') gin_trgm_ops);

SELECT * 
FROM my_json 
WHERE data_jsonb ->> 'name' ='John';

SELECT 
data_jsonb->>'name' AS name, 
data_jsonb->>'hobbies' AS hobbies 
FROM my_json 
WHERE data_jsonb->>'hobbies'  
LIKE  '%reading%'


---
12.3

CREATE EXTENSION hstore;

CREATE TABLE sku (   
product_id SERIAL PRIMARY KEY,   
name VARCHAR(50) NOT NULL, 
attributes HSTORE );

INSERT INTO sku (name, attributes) 
VALUES 
('Product 1', 'color => "red", weight => "10 lbs"'), 
('Product 2', 'color => "green", length=> "12 inches"'), 
('Product 3', 'gender => "male", height=> "180 cm"'), 
('Product 4', 'gender => "female", height=> "170 cm"');

CREATE INDEX idx_sku_attributes 
ON sku  USING GIN(attributes);


---
12.4

CREATE TABLE mytable (   
id SERIAL PRIMARY KEY, content TEXT );

CREATE TEXT SEARCH CONFIGURATION english (COPY= simple );

CREATE TEXT SEARCH DICTIONARY my_english_st (
TEMPLATE = pg_catalog.simple, 
STOPWORDS = english );

CREATE EXTENSION pg_trgm;

CREATE INDEX mytable_content_trgm_idx 
ON mytable USING gin (content gin_trgm_ops);

INSERT INTO mytable (content) 
VALUES  
('This is a sample text for full-text search indexing in PostgreSQL.'), 
('PostgreSQL is a powerful relational database management system.'), ('Full-text search in PostgreSQL allows for advanced searching of unstructured data.');

SELECT * 
FROM mytable 
WHERE to_tsvector('english', content) @@ to_tsquery('english', 'database'); 



---
12.5

CREATE TABLE movies (     
movie_id SERIAL PRIMARY KEY,   
title VARCHAR(100) NOT NULL,   
author VARCHAR(100) NOT NULL,   
publication_year INTEGER,     
genre VARCHAR(50), 
movie_xml XML);

INSERT INTO movies (title, author, publication_year, genre, movie_xml)
VALUES
('The Great Gatsby', 'F. Scott Fitzgerald', 1925, 'Literary Fiction', XMLPARSE(DOCUMENT '<movie><episode>Episode 1</episode><episode>Episode 2</episode></movie>'));

SELECT xpath('/movie/episode/text()', movie_xml) 
FROM movies  
WHERE title = 'The Great Gatsby';



---
12.6

CREATE TYPE business_type 
AS ENUM  ('Retail', 'Finance', 'Healthcare', 'Manufacturing');

CREATE TYPE ownership_type 
AS ENUM  ('Public', 'Private', 'Nonprofit');

CREATE TABLE company (     
company_id SERIAL PRIMARY KEY,
name VARCHAR(100) NOT NULL,
industries business_type[],
ownership ownership_type[]); 

INSERT INTO company (name, industries, ownership)  
VALUES  
('ABC Company', '{Retail, Healthcare}', '{Private, Nonprofit}'),
('XYZ Inc.', '{Finance, Manufacturing}', '{Public, Private}'); 

CREATE INDEX idx_company_industries  ON company
USING gin(industries);

SELECT *  FROM company 
WHERE 'Retail' = ANY(industries);



---
13.1

nano postgresql.conf
standby_mode = on
hot_standby = on
wal_level = replica

primary_conninfo = 'host=primary_host port=5432 user=replication_user password=password'
trigger_file = '/tmp/pgsql.trigger' 


nano pg_hba.conf
hostssl  all  all   192.168.0.0/24   md5 

touch /path/to/standby/data/standby.signal

nano postgresql.conf
hot_standby = on
wal_level = replica
primary_conninfo = 'host=primary_host_IP port=5432 user=replication_user password=your_password'


---
13.2

sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-$(rpm -E %{rhel})-x86_64/pgdg-redhat-repo-latest.noarch.rpm

nano postgresql.conf
wal_level = replica

nano pg_hba.conf
hostssl  all  all   192.168.0.0/24   md5 


nano /etc/repmgr.conf 

node_id=1

node_name=node1

conninfo='host=node1_host user=repmgr_user dbname=repmgr_db password=repmgr_password connect_timeout=2'

data_directory='/appl/pgsql/17/data/'

use_replication_slots=yes

ssh_options='-q -o BatchMode=yes -o ConnectTimeout=10'

reconnect_attempts=3

reconnect_interval=5

failover=automatic

promote_command='repmgr standby promote -f /etc/repmgr.conf --log-to-file'

follow_command='repmgr standby follow -f /etc/repmgr.conf --log-to-file --upstream-node-id=%n'

log_file='/var/log/repmgr/repmgr.log'

log_level=INFO

log_status_interval=300


nano postgresql.conf

wal_level = replica

touch /appl/pgsql/17/data/standby.signal

nano pg_hba.conf

hostssl  all  all   192.168.0.0/24   md5 


nano /etc/repmgr.conf 

node_id=2

node_name=node2

conninfo='host=node1_host user=repmgr_user dbname=repmgr_db password=repmgr_password connect_timeout=2'

data_directory='/appl/pgsql/17/data/'

use_replication_slots=yes

ssh_options='-q -o BatchMode=yes -o ConnectTimeout=10'

reconnect_attempts=3

reconnect_interval=5

failover=automatic

promote_command='repmgr standby promote -f /etc/repmgr.conf --log-to-file'

follow_command='repmgr standby follow -f /etc/repmgr.conf --log-to-file --upstream-node-id=%n'

log_file='/var/log/repmgr/repmgr.log'

log_level=INFO

log_status_interval=300


---
13.3

nano postgresql.conf

wal_level = logical

max_replication_slots = 3

max_wal_senders = 3


/usr/pgsql-17/bin/pg_ctl -D /appl/pgsql/17/data -l /appl/logs/pgsql/pg_logfile restart

CREATE PUBLICATION my_publication FOR TABLE books;

CREATE SUBSCRIPTION my_subscription 

CONNECTION 'host=publisher_host port=5432 user=replication_user password=replication_password dbname=postgres' 

PUBLICATION my_publication;

ALTER SUBSCRIPTION my_subscription ENABLED;

INSERT INTO books 
VALUES(13, 'SQL', 'Peter', 2023, 'Test Book', 1000, 2000);

SELECT * from books WHERE book_id = 13;



---
13.4

nano postgresql.conf

wal_level = replica  # or 'logical' 

archive_mode = on

archive_command = 'cp %p /path_to_wal_archive/%f'  

pg_basebackup -h localhost -D /appl/pgsql_pitr/17/data -U replication_user -P -Fp -Xs -R 


nano /appl/pgsql_pitr/17/data/recovery.conf

restore_command = 'cp /path_to_wal_archive/%f %p'  

recovery_target_time = 'YYYY-MM-DD HH:MM:SS'  

recovery_target_action = 'pause'  


---
14.1

SELECT 
nspname || '.' || relname AS tablename,   
pg_size_pretty(pg_total_relation_size(c.oid)) AS total_size,   pg_size_pretty(pg_relation_size(c.oid)) AS on_disk_size, 
pg_size_pretty(pg_total_relation_size(c.oid) - pg_relation_size(c.oid)) 
AS bloat_size, 
(pg_total_relation_size(c.oid) - pg_relation_size(c.oid)) /
 NULLIF(pg_relation_size(c.oid), 0) AS bloat_ratio 
FROM 
pg_catalog.pg_class c 
INNER JOIN pg_catalog.pg_namespace n 
ON c.relnamespace = n.oid 
WHERE 
relkind = 'r' AND 
nspname NOT LIKE 'pg_%' AND 
nspname <> 'information_schema' AND 
pg_total_relation_size(c.oid) > 20 * 1024 * 1024
ORDER BY  bloat_ratio DESC; 


---
14.2

SELECT pid, now() - pg_stat_activity.query_start AS duration, query, state 
FROM pg_stat_activity 
WHERE now() - pg_stat_activity.query_start > interval '120 seconds' 
AND state = 'active' 
ORDER BY duration DESC;

---
14.3

SELECT  
a.datname AS database_name, 
a.pid, a.usename AS username, 
a.client_addr, a.application_name, 
a.wait_event_type, a.wait_event, 
l.relation::regclass AS locked_relation, 
l.mode AS lock_mode, l.granted, a.query 
FROM  
pg_stat_activity a  
JOIN pg_locks l ON a.pid = l.pid 
WHERE  
a.state = 'active'  
AND l.granted = false; 


---
14.4

SELECT * FROM pg_stat_replication; 

SELECT * FROM pg_replication_slots;

SELECT * FROM pg_stat_activity; 


---
15.1

SELECT  pid,  usename,  datname, client_addr, application_name, backend_start, state_change,  state, (now() - state_change) AS idle_time 
FROM pg_stat_activity 
WHERE  state = 'idle' AND datname = 'your_database_name'; 

SELECT 
pg_terminate_backend(pid) 
FROM pg_stat_activity 
WHERE state = 'idle' 
AND datname = 'your_database_name'; 



---
18.1

CREATE EXTENSION oracle_fdw;

CREATE SERVER oracle_server 
FOREIGN DATA WRAPPER oracle_fdw 
OPTIONS (
dbname 'ORCL', 
host 'localhost', 
port '1521'); 

CREATE USER MAPPING 
FOR current_user 
SERVER oracle_server 
OPTIONS (
user 'my_oracle_user', 
password 'my_oracle_password'
);

CREATE FOREIGN 
TABLE my_oracle_table (  
id INTEGER,   
name VARCHAR(50), 
age INTEGER 
) 
SERVER oracle_server 
OPTIONS (
schema 'my_oracle_schema', 
table 'my_oracle_table');

CREATE FOREIGN 
TABLE my_oracle_table 
( 
id INTEGER,
name VARCHAR(50), 
age INTEGER 
) 
SERVER oracle_server 
OPTIONS (
schema 'my_schema', 
table 'my_table'); 

-- Query the foreign table in PostgreSQL 
SELECT id, name, age 
FROM my_oracle_table 
WHERE age > 30;

---
18.2

CREATE EXTENSION postgres_fdw; 

CREATE SERVER my_server FOREIGN DATA WRAPPER postgres_fdw 
OPTIONS 
(
host 'remote_host', 
dbname 'remote_dbname', 
port 'remote_port'
);   

CREATE SERVER my_server FOREIGN DATA WRAPPER postgres_fdw 
OPTIONS 
(
host 'remote_host', 
dbname 'remote_dbname', 
port 'remote_port'
);   

CREATE USER MAPPING 
FOR my_user SERVER my_server 
OPTIONS (user 'remote_user', password 'remote_password'); 

CREATE FOREIGN TABLE my_foreign_table_pgsql ( 
id INTEGER, name VARCHAR(50), age INTEGER 
) SERVER my_server OPTIONS (
schema 'remote_schema', 
table 'foreign_table');  

SELECT id, name, age 
FROM my_foreign_table_pgsql; 



---
18.3

CREATE EXTENSION parquet_s3_fdw;

CREATE SERVER parquet_s3_server 
FOREIGN DATA WRAPPER parquet_s3_fdw 
OPTIONS (use_minio 'true');

CREATE USER MAPPING 
FOR parquet_s3_user 
SERVER parquet_s3_server 
OPTIONS ( 
'username' 's3_user', 
'password' 's3_password' );   

CREATE FOREIGN 
TABLE parquet_s3_table (     
id int, first_name text, last_name text)
SERVER parquet_s3_server 
OPTIONS (filename 's3://bucket/dir/userdata1.parquet' ); 

SELECT * FROM parquet_s3_table;

IMPORT FOREIGN 
SCHEMA "/path/to/directory" 
FROM SERVER parquet_s3_srv 
INTO public;



---
19.1

DROP TABLE sales;
CREATE TABLE sales ( 
sale_id SERIAL,   
sale_date DATE NOT NULL, 
sale_amount NUMERIC(10,2) NOT NULL, 
CONSTRAINT sale_pk PRIMARY KEY (sale_date, sale_id) ) 
PARTITION BY RANGE (sale_date);

CREATE TABLE sales_2023 
PARTITION OF sales     
FOR VALUES FROM ('2023-01-01') TO ('2023-12-31');

CREATE TABLE sales_2024 
PARTITION OF sales     
FOR VALUES FROM ('2024-01-01') TO ('2024-12-31');

INSERT INTO sales (sale_date, sale_amount) VALUES 
('2023-01-01', 100.00), ('2023-01-02', 150.00), 
('2023-01-03', 200.00), ('2024-01-01', 300.00), 
('2024-01-02', 250.00), ('2024-01-03', 350.00);

SELECT * FROM sales; 

SELECT * FROM sales_2023; 

SELECT * FROM sales_2024; 


---
19.2

CREATE TABLE invoice (     
order_id SERIAL PRIMARY KEY, 
product_id INTEGER REFERENCES products(product_id), 
quantity INTEGER, 
price DECIMAL(10, 2));


WITH RECURSIVE cte_products AS ( 

    SELECT product_id, name, price 

    FROM products 

    WHERE product_id NOT IN (

        SELECT DISTINCT product_id 

        FROM invoice 

    ) 

    UNION ALL 

    SELECT products.product_id, products.name, products.price 

    FROM products 

    INNER JOIN cte_products ON cte_products.product_id::TEXT = 

ANY(products.tags) -- Cast to text 

    WHERE products.tags != '{}' -- Ignore empty tags array 

) 

SELECT cte_products.product_id, cte_products.name,

SUM(orders.sales_amount) 

AS sales_amount 

FROM cte_products 

JOIN orders ON orders.name = cte_products.name 

GROUP BY cte_products.product_id, cte_products.name 

ORDER BY cte_products.product_id;


---
20.1

ora2pg -t SHOW_SCHEMA -o schema.sql 

ora2pg -t ANALYZE -o analysis_report.txt 

ora2pg -t ANALYZE -o analysis_report.txt 



---
20.1

CREATE EXTENSION amcheck;

pg_amcheck -d <your_database_name> --index=PATTERN -table=PATTERN

pg_amcheck -d my_db -t books 

REINDEX INDEX CONCURRENTLY idx_book_titles; 

REINDEX INDEX CONCURRENTLY books;


---
21.1

CREATE EXTENSION IF NOT EXISTS anon CASCADE; 

SELECT anon.start_dynamic_masking();

CREATE ROLE test01 LOGIN; 

SECURITY LABEL FOR anon ON ROLE test01 IS 'MASKED'; 

CREATE TABLE info (
name varchar(30), 
important_id varchar(15));

SECURITY LABEL FOR anon  ON COLUMN info.important_id IS  'MASKED WITH FUNCTION anon.partial(important_id,5,$$***$$,0)';

SECURITY LABEL FOR anon ON ROLE test01 IS NULL;

SELECT anon.stop_dynamic_masking();

--- END ---








