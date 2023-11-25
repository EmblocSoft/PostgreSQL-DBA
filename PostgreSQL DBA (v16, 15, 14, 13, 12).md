
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


# PostgreSQL-DBA 16 (2w024 Edition)
PostgreSQL DBA 16 (2024 Edition)
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
PUBLICATION my_publication
WITH (binary=true);

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


