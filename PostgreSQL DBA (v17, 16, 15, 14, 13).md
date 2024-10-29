
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
-- Create a sample table
CREATE TABLE demo_table (
    id serial PRIMARY KEY, name VARCHAR(50), children INT);

-- Start a transaction explicitly
BEGIN;

-- Insert data into the table
INSERT INTO demo_table (name) VALUES ('John');
INSERT INTO demo_table (name) VALUES ('Alice');

-- Check if data exists 
SELECT * FROM demo_table;

---
5.10.2
-- Go back to Session 1 of PSQL:

BEGIN;
-- Insert a row with GOOD data into the table
INSERT INTO demo_table (name, children) VALUES ('Alex', 2);
-- Insert a row with BAD data into the table
INSERT INTO demo_table (name, children) VALUES ('Sam', 'NO CHILD');
-- Check if data exists 
SELECT * FROM demo_table;

ROLLBACK;
-- Check if data exists 
SELECT * FROM demo_table;

---
5.20.1
# Add or modify the following lines in postgresql.conf 

cd /appl/pgsql/17/data

nano postgresql.conf 
###
log_destination = 'stderr'
logging_collector = on
log_directory = '/appl/logs/pgsql/'
log_filename = 'postgresql-%d.log'
log_rotation_age = 1d
log_rotation_size = 0
log_truncate_on_rotation = on
log_line_prefix  = '%m [%p] [%a] [%d] [%h] [%r] [%i] [%e] [%x]'
log_statement = 'ddl'
###

---
6.1
BEGIN; -- Start a transaction

-- Create a new table
CREATE TABLE employee(
    id INT,
    first_name VARCHAR (50),
    last_name VARCHAR (50),
    salary numeric(10, 2));

-- Insert data into the table
EXPLAIN ANALYZE INSERT INTO 
employee (first_name, last_name, salary)
VALUES
    ('John', 'Doe', 50000.00),
    ('Jane', 'Smith', 60000.00),
    ('Bob', 'Johnson', 55000.00);

-- Query data from the table
EXPLAIN ANALYZE SELECT * FROM employee;

-- Rollback the transaction to cancel changes
ROLLBACK;

---
6.2
nano postgresql.conf    # You can use another text editor
###
log_statement = 'all'
log_duration = on
shared_preload_libraries = 'pg_stat_statements'
log_min_duration_statement = 1000    # 1000 milliseconds
###

-- Enable pg_stat_statement EXTENSION to your database
CREATE EXTENSION pg_stat_statements;

-- Run this SQL to find slow queries
SELECT query, total_exec_time, calls
FROM pg_stat_statements
WHERE total_exec_time > 1000000   -- 1 second in microseconds
ORDER BY total_exec_time DESC;

---
6.3
-- Drop old sample tables
DROP TABLE customers;
DROP TABLE orders;

-- Create table: customer
CREATE TABLE customers 
    customer_id serial PRIMARY KEY
    customer_name varchar(255);

-- Create table: orders
CREATE TABLE orders (
    order_id serial PRIMARY KEY,
    customer_id integer,
    order_date date,
    total_amount numeric);

-- Insert sample data to customers table
INSERT INTO customers (customer_name)

SELECT md5(random()::text)
FROM generate_series(1, 1000000);

-- Generate and insert 10,000,000 sample rows to order
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
    )
);

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

--
-- Create indexes 
CREATE INDEX idx_total_amount ON orders(total_amount);
CREATE INDEX idx_customer_id ON orders(customer_id);

---
6.4
EXPLAIN ANALYZE
SELECT COUNT(1)FROM customers
JOIN orders ON customers.customer_id = orders.customer_id;

---
EXPLAIN ANALYZE
SELECT COUNT(1)FROM customers
JOIN orders ON customers.customer_id = orders.customer_id
WHERE customers.customer_id = 1;

---
EXPLAIN ANALYZE
SELECT COUNT(1)
FROM (SELECT * FROM customers ORDER BY customer_id) AS c
JOIN (SELECT * FROM orders ORDER BY customer_id) AS o
ON c.customer_id = o.customer_id;

---
EXPLAIN ANALYZE
SELECT COUNT(1)
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id;

---
6.5
ELECT relname AS table_name, indexrelname AS index_name, idx_scan, 
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
ELECT query,  state, backend_start,  now() - query_start AS duration 
FROM pg_stat_activity 
WHERE state = 'active'  AND now() - query_start > interval '2 minutes' 
ORDER BY now() - query_start DESC;

---
6.9
ELECT mode, relation::regclass, pid, granted,  waitstart 
FROM pg_locks 
WHERE granted = false 
ORDER BY granted, relation::regclass;

---
7.1
-- R1 
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
-- Create an index on “customer_id” in “customers” table
CREATE INDEX CONCURRENTLY idx_customer_id ON customers(customer_id);

-- Create an index on “order_id” in “orders” table
CREATE INDEX CONCURRENTLY idx_order_id ON orders(order_id);

-- Create a partial index for orders with a total_amount >= 100
CREATE INDEX CONCURRENTLY idx_total_amount_gt_100 ON orders(total_amount) WHERE total_amount >= 100;

---R4
EXPLAIN ANALYZE 
SELECT * FROM orders
ORDER BY total_amount;

---R5
XPLAIN ANALYZE 
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

---
DROP INDEX idx_total_amount_gt_100;

---R7
EXPLAIN ANALYZE 
SELECT DISTINCT customer_id, total_amount
FROM orders
WHERE order_date >= '2023-01-01' AND total_amount >=100
ORDER BY total_amount;

- Rebuild an index
REINDEX INDEX CONCURRENTLY idx_customer_id;

-- Change the fill factor of an index
ALTER INDEX idx_customer_id  SET (fillfactor = 80);

-- Disable an index
DROP INDEX idx_customer_id ;

---
7.3
EXPLAIN ANALYZE SELECT customer_name FROM customers
WHERE 
(customer_name >= '10000' AND customer_name <= '100000') OR (customer_name >= '200000' AND customer_name <= '300000');

---
CREATE INDEX CONCURRENTLY idx_customer_name 
ON customers(customer_name);

---
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

---
EXPLAIN ANALYZE 
SELECT *
FROM customers
ORDER BY customer_name;

---
7.5.2
SELECT 
datname, 
pg_size_pretty(pg_database_size(datname)) AS size 
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

---
SET enable_seqscan = on; 

---
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

---
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
EGIN;

-- Select and lock rows for update
SELECT * FROM orders WHERE total_amount >= 1000 FOR UPDATE;

-- Perform your mass update operation here
-- Let's assume we are increasing the total_amount by 10%
UPDATE orders SET total_amount = total_amount * 1.10 
WHERE total_amount >= 1000;

-- Commit the transaction to release the locks
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
    -- Fetch and process rows here
    LOOP
        FETCH NEXT FROM my_cursor INTO customer_id, 
                        order_date, total_amount;
        EXIT WHEN NOT FOUND;
        -- Process the row here (you can print or perform any other operation)
        RAISE NOTICE 'Customer ID: %, Order Date: %, Total Amount: %', customer_id, order_date, total_amount;
    END LOOP;

    CLOSE my_cursor;
END;
$$ LANGUAGE plpgsql;

---
UPDATE orders
SET total_amount = floor(random() * (100000 - 1000 + 1) + 1000)
WHERE total_amount = 1210;

---
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

---
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

---
CREATE TABLE monthly_sales_total AS
SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(total_amount) AS total_sales
FROM orders
GROUP BY month;

---
CREATE TABLE yearly_sales_total AS
SELECT
    DATE_TRUNC('year', order_date) AS year,
    SUM(total_amount) AS total_sales
FROM orders
GROUP BY year;

---
SELECT order_date, total_sales 
FROM daily_sales_total
ORDER BY order_date;GROUP BY year;

---
SELECT month, total_sales
FROM monthly_sales_total 
ORDER BY month;

---
SELECT year, total_sales
FROM yearly_sales_total
ORDER BY year;

---
EXPLAIN ANALYZE 
SELECT year, total_sales FROM yearly_sales_total ORDER BY year;

---
EXPLAIN ANALYZE 
SELECT
    DATE_TRUNC('year', order_date) AS year,
    SUM(total_amount) AS total_sales
FROM orders GROUP BY year;

---
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

-- Create Partial Index
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
-- Create a sample table
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
-- Partial Index
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_sales_new_in_365d 
ON sales(sale_date, sales_id, amount) 
WHERE EXTRACT(YEAR FROM sale_date) >= (current_year() - 1);

-- Create Full Index with same index structure for comparison
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

ALTER TEXT SEARCH CONFIGURATION english_fts  
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
\c test_db
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
\c postgres
GRANT SELECT, INSERT, UPDATE, DELETE, TRUNCATE, TRIGGER ON TABLE books TO myuser;

---
9.6
\c postgres
REVOKE TRUNCATE ON TABLE books FROM myuser;

---
9.7
\c postgres
GRANT SELECT (book_id, title, author) ON books TO myuser;

---
9.8
\c postgres
REVOKE SELECT (author) ON books FROM myuser;

---
9.9
CREATE ROLE books_admin_group WITH LOGIN;

GRANT SELECT, INSERT, UPDATE, DELETE 
ON books TO books_admin_group;

CREATE POLICY insert_update_delete_books_policy ON books
FOR ALL TO books_admin_group USING (true) WITH CHECK (true);

---
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
-- Create USER peter and assign to Literary Group
CREATE USER peter WITH ENCRYPTED PASSWORD 'peter_password';
GRANT literary_group TO peter;
-- Create USER john and assign to Dystopian Group
CREATE USER john WITH ENCRYPTED PASSWORD 'john_password';
GRANT dystopian_group TO john;
-- Create USER susan and assign to Romantic Group
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
EELECT d.datname as database, r.rolname as owner
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




















