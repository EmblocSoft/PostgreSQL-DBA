
# The book, "PostgreSQL DBA", is published on Amazon, it covers PostgreSQL v11, 12, 13, 14, to the latest v15

The book's introduction video:
https://www.amazon.com/live/video/0365e749066645548e17ee87fd53daaf

The book's URL:
https://www.amazon.com/PostgreSQL-DBA-v15-Administrators-Availablity/dp/B0C2SVRNJ3/ref=tmm_pap_swatch_0?_encoding=UTF8&qid=1682643597&sr=8-3





# PostgreSQL-DBA
PostgreSQL DBA
</br>
Github link to download installers of PostgreSQL (Windows, Mac OSX, and Linux) </br>
https://github.com/EmblocSoft/PostgreSQL



Kindle Edition:
https://www.amazon.com/PostgreSQL-DBA-v15-Administrators-Availablity-ebook/dp/B0C2WGKHQN/ref=sr_1_1?crid=2YLTOOFRR9DF7&keywords=PostgreSQL+DBA&qid=1682397797&s=books&sprefix=postgresql+db%2Cstripbooks-intl-ship%2C403&sr=1-1

More about the guide book, watch this video:

https://www.amazon.com/live/video/0a3530a003cc4e6d9d0166d2d4f9ee22


Important Note 1: The following commands are FOR INFORMATION ONLY, you would need to refer to the guide book "PostgreSQL DBA):

---

Chapter 1 and 2:

1. WINDOWS: sample command to calculate the checksum:

cd C:\Users\YOUR_NAME\Downloads\
certutil -hashfile PostgreSQL_Windows_64-v15.1.0.1.msi sha256 > mynew.Checksum

2. MAC OSX: sample command to calculate the checksum:

cd ~/Downloads
shasum -a 256  PostgreSQL_OSX_64-v15.1.0.1_installer.pkg


3. LINUX: sample command to initialize a database:

mkdir -p /appl/pgsql/15/ 
cd /appl/pgsql/15
sudo /usr/pgsql-15/bin/initdb --pgdata=/appl/pgsql/15/data --localeprovider=icu --icu-locale=en_US.UTF-8  


To start PostgreSQL:

/usr/pgsql-15/bin/pg_ctl -D /appl/pgsql/15/data -l /appl/logs/pgsql/pg_logfile start 

To stop PostgreSQL:

/usr/pgsql-15/bin/pg_ctl -D /appl/pgsql/15/data -l /appl/logs/pgsql/pg_logfile stop

To restart PostgreSQL:

/usr/pgsql-15/bin/pg_ctl -D /appl/pgsql/15/data -l /appl/logs/pgsql/pg_logfile restart 

To reload the PostgreSQL configuration without restart:
 
/usr/pgsql-15/bin/pg_ctl -D /appl/pgsql/15/data -l /appl/logs/pgsql/pg_logfile reload



4. To create, to list, to connect to a new database in PostgreSQL (Windows, or Mac, or Linux)

postgres=# CREATE DATABASE my_db; 

postgres=# \l 

postgres=# \c my_db    

---

Chapter 3:

postgres=# CREATE USER my_user; 

postgres=# ALTER ROLE my_user WITH ENCRYPTED PASSWORD 'my_pass'; 

postgres=# CREATE DATABASE my_db; 

postgres=# GRANT ALL ON DATABASE my_db TO my_user; 

postgres=# ALTER DATABASE my_db OWNER TO my_user; 

postgres=# \q


sample pg_hba.conf entries that enable SSL

hostssl  all    all    192.168.0.100/32   md5  

hostssl  all    all    192.168.0.0/24     md5 

hostssl  my_db  all    192.168.0.0/24     md5 

hostssl  all    userA  192.168.0.0/24     md5

hostssl  my_db  userA  192.168.0.0/24     md5 



CREATE TABLE books  ( 
book_id SERIAL PRIMARY KEY,  
title VARCHAR(100) NOT NULL,     
author VARCHAR(100) NOT NULL,     
publication_year INTEGER,   
genre VARCHAR(50),     
qty INTEGER, 
unit_price DECIMAL );

INSERT INTO books  
(title, author, publication_year, genre)  
VALUES ('To Kill a Mockingbird', 'Harper Lee', 1960, 'Fiction'); 


keytool -genkey -alias mykey -keyalg RSA -keysize 2048 -keystore mykeystore.jks -storetype JKS -storepass mykeystorepassword -keypass mykeypassword -validity 3650 -keysize 256 -dname "CN=mydomain.com, OU=MyOrg, O=MyOrg, L=MyCity, ST=MyState, C=MyCountry" -ext san=dns:mydomain.com 

cd /appl/pgsql/15/data 
nano postgresql.conf   (note: you can use other editor) 

keystore_location = /appl/keystore/mykeystore.jks 
tablespace_encryption_algorithm = 'AES256' 

(save and exit) 

/usr/pgsql-15/bin/psql 
postgres=# SELECT pgx_set_master_ket('mykeypassword'); 

/usr/pgsql-15/bin/pg_ctl -D /appl/pgsql/15/data -l /appl/logs/pgsql/pg_logfile --keystorepassphrase restart 

postgres=# CREATE TABLESPACE tbs_encrypt_zone 
LOCATION '/appl/pgsql/15/data/tbs_encrypt_zone'; 

postgres=# \c my_db

CREATE TABLE my_enpted_table (   
id SERIAL PRIMARY KEY,   
name TEXT ) 
TABLESPACE tbs_encrypt_zone; 

---

Chapter 4

Manually set a checkpoint
pg_ctl checkpoint -D /appl/pgsql/15/data 

or

/usr/pgsql-15/bin/psql 

postgres=#  CHECKPOINT; 

---

Chapter 5

EXPLAIN SELECT * FROM books WHERE genre = 'Fiction';

SELECT 
relname AS table_name, 
indexrelname AS index_name, 
idx_scan, 
idx_tup_read, 
ROUND(CAST(idx_tup_read AS numeric) /
NULLIF(idx_scan, 0), 2) AS avg_tup_read_per_scan  
FROM pg_stat_user_indexes 
WHERE idx_scan > 0 
ORDER BY avg_tup_read_per_scan ASC; 

SELECT 
schemaname, 
attname, 
n_distinct, 
most_common_vals, 
most_common_freqs 
FROM pg_stats 
WHERE schemaname NOT LIKE 
'pg_%' AND schemaname <> 'information_schema'  
AND n_distinct <= 10 
AND most_common_vals IS NOT NULL; 

SELECT 
query, 
state, 
backend_start, 
now() - query_start AS duration 
FROM pg_stat_activity 
WHERE state = 'active' 
AND now() - query_start > interval '2 minutes' 
ORDER BY now() - query_start DESC; 

SELECT 
mode, 
relation::regclass, 
pid, 
granted,  
waitstart 
FROM pg_locks 
WHERE granted = false 
ORDER BY granted, relation::regclass; 

EXPLAIN ANALYZE 
SELECT * FROM books WHERE genre = 'Fiction'; 

CREATE TABLE employees (   
id SERIAL PRIMARY KEY,   
name TEXT,   
age INTEGER,
salary NUMERIC(10, 2) ); 

CREATE INDEX idx_age ON employees(age); 

EXPLAIN 
SELECT * FROM employees WHERE age = 30; 

SELECT 
datname, 
pg_size_pretty(pg_database_size(datname)) AS size 
FROM pg_database 
ORDER BY pg_database_size(datname) DESC; 

SELECT 
relname, 
pg_size_pretty(pg_total_relation_size(relid)) 
FROM pg_catalog.pg_statio_user_tables 
ORDER BY pg_total_relation_size(relid) DESC;

GRANT ALL PRIVILEGES 
ON SCHEMA my_schema TO my_user; 

REVOKE ALL PRIVILEGES 
ON SCHEMA my_schema FROM my_user; 

ALTER DEFAULT PRIVILEGES 
IN SCHEMA my_schema GRANT 
SELECT, 
INSERT, 
UPDATE, 
DELETE 
ON TABLES TO my_role; 

GRANT  SELECT  ON books TO peter; 

REVOKE SELECT ON books FROM peter; 

GRANT SELECT (book_id, title, author) ON books TO john; 

REVOKE SELECT (author) ON employees FROM john;

CREATE USER Fiction; 
 
CREATE ROLE fiction_group; 

ALTER ROLE Fiction IN GROUP fiction_group; 

CREATE POLICY fiction_policy ON books  
FOR ALL  TO  PUBLIC  USING (genre=current_user); 

ALTER TABLE books 
ENABLE ROW LEVEL SECURITY; 

GRANT  SELECT,  UPDATE  ON books TO fiction_group; 

GRANT SELECT  ON books TO regular_users; 

CREATE USER peter 
WITH ENCRYPTED  PASSWORD 'peter_password' CREATEDB 
INHERIT VALID UNTIL '2029-01-01'; 

ALTER USER myuser 
WITH ENCRYPTED PASSWORD ‘newpassword’;

CREATE GROUP marketing;  

ALTER GROUP marketing ADD USER peter; 

ALTER GROUP marketing DROP USER peter; 

CREATE ROLE admin; 

GRANT 
SELECT, 
INSERT, 
UPDATE, 
DELETE 
ON mytable TO admin; 

REVOKE 
DELETE 
ON mytable FROM admin; 

SELECT 
usename, 
ssl 
FROM pg_stat_ssl JOIN pg_user 
ON pg_stat_ssl.pid = pg_backend_pid() 
WHERE usename = 'postgres'; 

ALTER ROLE username CONNECTION LIMIT 1; 

ALTER ROLE username CONNECTION LIMIT DEFAULT; 

SELECT 
pid, 
usename, 
application_name, 
client_addr, 
backend_start 
FROM pg_stat_activity 
WHERE usename = 'peter'; 

SELECT pg_terminate_backend(234567); 

ELECT 
d.datname, 
r.rolname 
FROM 
pg_catalog.pg_database d, 
pg_catalog.pg_roles r 
WHERE d.datdba = r.oid; 

SELECT 
grantee AS user, 
CONCAT(table_schema, '.', table_name) AS table,  
CASE 
WHEN COUNT(privilege_type) = 7 
THEN 'ALL' 
ELSE ARRAY_TO_STRING(ARRAY_AGG(privilege_type), ', ')            
END AS grants 
FROM information_schema.role_table_grants 
GROUP BY table_name, table_schema, grantee 
ORDER BY grantee; 

SELECT 
r.rolname, 
r.rolsuper, 
r.rolinherit, 
r.rolcreaterole, 
r.rolcreatedb, 
r.rolcanlogin,  
r.rolconnlimit, 
r.rolvaliduntil, 
ARRAY(SELECT b.rolname 
FROM pg_catalog.pg_auth_members m 
JOIN pg_catalog.pg_roles b 
ON (m.roleid = b.oid) 
WHERE m.member = r.oid) as memberof, 
r.rolreplication, 
r.rolbypassrls 
FROM pg_catalog.pg_roles r 
WHERE r.rolname !~ '^pg_' 
ORDER BY 1; 

---

Chapter 6

CREATE SCHEMA marketing; 

CREATE TABLE marketing.new_books(
book_id SERIAL PRIMARY KEY,     
title VARCHAR(100) NOT NULL,     
author VARCHAR(100) NOT NULL,     
publication_year INTEGER, 
genre VARCHAR(50) 
); 

CREATE INDEX idx_book_title 
ON books (title); 

SELECT 
relname AS table_name, 
indexrelname AS index_name, 
idx_scan, 
idx_tup_read, 
idx_tup_fetch 
FROM pg_stat_user_indexes 
ORDER BY idx_scan ASC; 

SELECT * 
FROM pg_indexes 
WHERE schemaname = 'public' AND 
tablename = 'books'  AND 
indexname = 'idx_book_title'; 

DROP INDEX idx_book_title; 

CREATE SEQUENCE client_id_seq 
START WITH 1  INCREMENT BY 1 
NO MINVALUE NO MAXVALUE 
CACHE 1; 

CREATE TABLE clients ( 
id INTEGER DEFAULT nextval('client_id_seq'),
name TEXT,
email TEXT, 
PRIMARY KEY (id) 
); 

NSERT INTO clients 
VALUES
(nextval('client_id_seq'), 'ABC LLC', 'info@abc.com'); 

SELECT currval('client_id_seq'); 

CREATE VIEW client_emails  AS SELECT name, email  FROM clients;

SELECT email FROM client_emails; 

ALTER TABLE clients ADD COLUMN phone TEXT; 

CREATE OR REPLACE VIEW client_emails 
AS SELECT name, email, phone FROM clients; 

CREATE MATERIALIZED VIEW mv_clients 
AS SELECT  name, email, phone 
FROM clients ORDER BY name; 

REFRESH MATERIALIZED VIEW mv_clients;

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

INSERT INTO books VALUES
(2, 'SQL', 'Tony', 2023, 'Text book', 1000000, 2000); 
 
SELECT  calculate_order_total(1);


CREATE TABLE books (    
book_id SERIAL PRIMARY KEY, 
title VARCHAR(100) NOT NULL, 
author VARCHAR(100) NOT NULL, 
publication_year INTEGER, 
genre VARCHAR(50),
qty INTEGER, 
unit_price DECIMAL 
); 

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

CREATE OR REPLACE FUNCTION update_book() 
RETURNS 
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
VALUES (11, 'The Catcher in the Rye', 'J.D. Salinger', 1951, 'Fiction', 10, 

UPDATE books SET qty = 0 WHERE book_id = 11; 

DELETE FROM books WHERE book_id = 11; 
CREATE TABLE clients (    
client_id SERIAL PRIMARY KEY,     
first_name VARCHAR(50) NOT NULL, 
last_name VARCHAR(50) NOT NULL 
); 

CREATE TABLE orders (     
order_id SERIAL PRIMARY KEY,     
client_id INTEGER NOT NULL,     
order_date DATE NOT NULL, 
total_amount DECIMAL(10,2) NOT NULL, 
FOREIGN KEY (client_id) REFERENCES clients(client_id) 
); 

CREATE ROLE my_user LOGIN PASSWORD 'mypassword'; 

Create a group role: 
CREATE ROLE my_group; 

GRANT my_group TO my_user; 
Grant privileges to a role: 
GRANT 
SELECT, 
INSERT, 
UPDATE, 
DELETE 
ON books TO my_user; 

REVOKE 
DELETE, 
UPDATE 
ON mytable FROM myuser; 

CREATE TABLE clients (    
client_id SERIAL PRIMARY KEY,     
first_name VARCHAR(50) NOT NULL, 
last_name VARCHAR(50) NOT NULL 
); 

CREATE TABLE orders (     
order_id SERIAL PRIMARY KEY,     
client_id INTEGER NOT NULL,     
order_date DATE NOT NULL, 
total_amount DECIMAL(10,2) NOT NULL, 
FOREIGN KEY (client_id) REFERENCES clients(client_id) 
); 

CREATE TYPE book_type AS (     
book_id INT,     
pages INTEGER, 
publisher VARCHAR(150) 
); 

CREATE TABLE pre_orders ( 
book book_type,     
client_id INTEGER, 
order_qty INTEGER 
); 

CREATE DOMAIN phone_number AS text 
CHECK (VALUE ~'^\d{3}-\d{3}-\d{4}$'); 
 
CREATE TABLE customers (     
customer_id SERIAL PRIMARY KEY,     
first_name VARCHAR(50) NOT NULL,
last_name VARCHAR(50) NOT NULL, 
phone phone_number, 
email VARCHAR(100) NOT NULL 
); 

CREATE FUNCTION reverse_concat(text, text) 
RETURNS text 
AS $$ 
    SELECT $2 || $1; 
$$ LANGUAGE SQL; 

CREATE OPERATOR ||| ( 
    LEFTARG = text, 
    RIGHTARG = text, 
    PROCEDURE = reverse_concat ); 

CREATE FUNCTION median_final (state anyarray) 
RETURNS float8 
AS $$ 
   SELECT percentile_cont(0.5) 
   WITHIN GROUP (ORDER BY unnest($1)) 
$$ LANGUAGE SQL IMMUTABLE; 

SELECT median(qty) FROM books; 

CREATE SERVER my_remote_server 
FOREIGN DATA WRAPPER postgres_fdw 
OPTIONS 
(
host 'myremotehost', 
dbname 'remote_db', 
port '5432'
); 

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

CREATE TABLE large_objects (
id SERIAL PRIMARY KEY, 
data OID); 

INSERT INTO large_objects (data) 
VALUES (lo_import('/appl/my_big_file.zip'));

SELECT * FROM large_objects; 
SELECT lo_export(data, '/tmp/my_big_file.zip') 
FROM large_objects 
WHERE id = 1; 

CREATE TABLE books (     
book_id SERIAL PRIMARY KEY,
title VARCHAR(100) NOT NULL,
author VARCHAR(100) NOT NULL,
publication_year INTEGER,     
genre VARCHAR(50),     
qty INTEGER, 
unit_price DECIMAL 
); 

CREATE RULE protect_books_delete 
AS ON DELETE TO books DO INSTEAD NOTHING; 

CREATE TYPE gender 
AS ENUM ('Male', 'Female', 'Other'); 

CREATE TABLE users (   
id SERIAL PRIMARY KEY, 
name VARCHAR(50) NOT NULL,   
gender gender NOT NULL 
);

CREATE EXTENSION IF NOT EXISTS pg_trgm; 

CREATE TABLE documents (   
id SERIAL PRIMARY KEY,   
content TEXT,   
search_vector TSVECTOR, 
search_query TSQUERY 
); 

CREATE INDEX documents_search_index 
ON documents 
USING GIN(search_vector); 

INSERT INTO documents (content, search_vector) 
VALUES 
('PostgreSQL is a powerful open source object-relational database system.',  to_tsvector('English', 'PostgreSQL is a powerful open source object-relational database system.')); 

SELECT * 
FROM documents 
WHERE search_vector @@ 
to_tsquery('English', 'PostgreSQL & database'); 

---

Chapter 7

CREATE ROLE replication_user 
REPLICATION 
LOGIN PASSWORD 'your_password'; 
 
CREATE SUBSCRIPTION sub_name 
CONNECTION 
'host=master_server_ip_address 
port=5432 
dbname=db_name 
user=replication_user 
password=your_password' 
PUBLICATION pub_name; 

ALTER SUBSCRIPTION
sub_name ENABLED;

---

Chapter 8

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

SELECT 
pid, 
now() - pg_stat_activity.query_start AS duration, 
query, 
state 
FROM pg_stat_activity 
WHERE 
now() - pg_stat_activity.query_start > interval '120 seconds' 
AND state = 'active' 
ORDER BY duration DESC; 

SELECT  
a.datname AS database_name, 
a.pid, 
a.usename AS username, 
a.client_addr, 
a.application_name, 
a.wait_event_type, 
a.wait_event, 
l.relation::regclass AS locked_relation, 
l.mode AS lock_mode, 
l.granted, 
a.query 
FROM  
pg_stat_activity a  
JOIN pg_locks l ON a.pid = l.pid 
WHERE  
a.state = 'active'  
AND l.granted = false; 

SELECT * FROM pg_stat_replication;

SELECT * FROM pg_replication_slots; 

SELECT * FROM pg_stat_activity; 

SELECT  
pid,    
usename,    
datname,    
client_addr,    
application_name,    
backend_start,  
state_change,  
state, 
(now() - state_change) AS idle_time 
FROM pg_stat_activity 
WHERE  state = 'idle'  
AND datname = 'your_database_name'; 

SELECT 
pg_terminate_backend(pid) 
FROM pg_stat_activity 
WHERE state = 'idle' 
AND datname = 'your_database_name'; 

Sample recovery.conf file: 

restore_command = 'cp /mnt/wal_archive/%f "%p"' 
recovery_target_time = '2023-04-03 12:00:00' 

---

Chapter 10

CREATE INDEX 
CONCURRENTLY idx_books_author
ON books(author, book_id); 

SELECT * 
FROM pg_stat_activity 
WHERE state 
LIKE '% %concurrently%%'; 

CREATE OR REPLACE FUNCTION current_year() 
RETURNS integer AS 
$$  
   SELECT EXTRACT(YEAR FROM CURRENT_DATE); 
$$ LANGUAGE SQL IMMUTABLE; 

CREATE INDEX  
CONCURRENTLY  idx_books_new_in_365d 
ON books (publication_year, author, book_id) 
WHERE publication_year >= current_year() - 1; 

REINDEX INDEX 
CONCURRENTLY idx_books_new_in_365d; 

CREATE INDEX idx_author_genre 
ON books ((genre || ' ' || author));

CREATE INDEX idx_books_pubyear_author_genre_qty  
ON books (publication_year, author, genre, qty); 

CREATE TABLE articles (   
id SERIAL PRIMARY KEY,   
title TEXT NOT NULL, 
body TEXT NOT NULL 
); 

INSERT INTO articles (title, body) 
VALUES 
  ('PostgreSQL Full Text Search', 'PostgreSQL has a powerful Full Text Search functionality.'), 
  ('How to use FTS in PostgreSQL', 'In order to use FTS in PostgreSQL, you need to create a text search configuration and define a full text index.'), 
  ('Why use FTS in PostgreSQL', 'FTS in PostgreSQL is great for searching large amounts of textual data quickly and efficiently.'), 
  ('Limitations of FTS in PostgreSQL', 'FTS in PostgreSQL is not as feature-rich as some other search engines, but it is still very powerful.'); 

CREATE TEXT 
SEARCH CONFIGURATION english_fts
 (COPY = english); 

ALTER TEXT 
SEARCH CONFIGURATION english_fts  
ALTER MAPPING FOR asciiword WITH english_stem; 

CREATE INDEX articles_fts_idx 
ON articles USING 
gin(to_tsvector('english_fts', title || ' ' || body)); 

CREATE INDEX articles_search_vector_idx 
ON articles 
USING gin(search_vector); 

SELECT * 
FROM articles 
WHERE 
search_vector @@ to_tsquery('my_english', 'PostgreSQL'); 

CREATE INDEX idx_publication_title 
ON books (publication_year, title);

CREATE INDEX 
CONCURRENTLY idx_books_year_genre 
ON  books 
USING BTREE  (publication_year, genre); 

CREATE TABLE my_long_text (     
id SERIAL PRIMARY KEY,     
text_column TEXT 
); 

INSERT INTO my_long_text (text_column) 
VALUES ('GiST stands for Generalized Search Tree and it is an index structure in PostgreSQL that is suitable for handling complex data types and complex queries. It is an alternative index type to the B-tree index and can be used when the B-tree index is not suitable for the data type or the query. 
B-tree index is not suitable for some data types such as text and array data types. 
These data types require more complex and specialized indexing methods, such as GIN and GiST indexes. B-tree index is designed for range queries, and its performance degrades when handling large amounts of data or complex queries. 
'); 

INSERT INTO my_long_text (text_column) 
VALUES ('B-tree index is not suitable for some data types such as TEXT or array of text. These data types require more complex and specialized indexing methods, such as Gist indexes since B-tree index is designed for range queries and B-tree's performance degrades when handling large amounts of data or complex queries.'); 

INSERT INTO my_long_text (text_column) 
VALUES ('The GiST index can handle many different types of data such as full-text search, geometric data, and even custom data types that you define. It supports operations like equality, inequality, and distance-based searches.'); 

INSERT INTO my_long_text (text_column) 
VALUES ('A functional index in PostgreSQL, also called Expression index,  is an index built on an expression rather than on a simple column or columns. A functional index is useful when you want to index the result of a function or expression instead of just the value in a column.'); 

CREATE INDEX idx_long_text_gist 
ON my_long_text 
USING GIST (text_column gist_trgm_ops); 

EXPLAIN SELECT * 
FROM my_long_text 
WHERE text_column LIKE  '%suitable%'; 

CREATE TABLE products (     
product_id SERIAL PRIMARY KEY,     
name TEXT NOT NULL,     
description TEXT,    
price NUMERIC(10,2),     
tags TEXT[] 
); 

INSERT INTO products (name, description, price, tags) 
VALUES 
('Product 1', 'This is the first product', 10.00, '{"tag1", "tag2"}'), 
('Product 2', 'This is the second product', 20.00, '{"tag2", "tag3"}'), 
('Product 3', 'This is the third product', 30.00, '{"tag1", "tag3"}'), 
('Product 4', 'This is the fourth product', 40.00, '{"tag2", "tag4"}'),  
('Product 5', 'This is the fifth product', 50.00, '{"tag4", "tag5"}'); 

CREATE INDEX idx_products_tags_gist 
ON products 
USING GIN (tags); 

SELECT * 
FROM products 
WHERE tags @> '{"tag2"}';

SELECT lo_export(data, '/tmp/my_big_file.zip') 
FROM large_objects 
WHERE id = 1; 


CREATE TABLE my_json (     
id SERIAL PRIMARY KEY,
data JSON 
); 

INSERT INTO my_json (data) VALUES 
    ('{"name": "John", "age": 30, "city": "New York"}'), 
    ('{"name": "Jane", "age": 25, "city": "San Francisco"}'), 
    ('{"name": "Bob", "age": 40, "city": "Chicago"}'), 
    ('{"name": "Alice", "age": 35, "city": "Seattle"}'), 
    ('{"name": "Mike", "age": 45, "city": "Boston"}'); 

CREATE INDEX idx_gin_json 
ON my_json 
USING GIN ((data>>'name') gin_trgm_ops); 

SELECT * 
FROM my_json 
WHERE data->>'name' = 'Alice'; 

EXPLAIN SELECT * 
FROM my_json 
WHERE data->>'name' = 'Alice'; 

CREATE INDEX idx_gist_json 
ON my_json
USING GIST ((data->>'name') gist_trgm_ops); 

EXPLAIN SELECT * 
FROM my_json 
WHERE data->>'name' = 'Alice'; 

CREATE INDEX idx_brin_books 
ON books 
USING BRIN (publication_year); 

EXPLAIN SELECT * 
FROM books 
WHERE publication_year < 1970; 

CREATE EXTENSION 
IF NOT EXISTS earthdistance 
CASCADE; 

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

INSERT INTO 
books (title, author, publication_year, genre, qty, unit_price) 
VALUES  
('The Great Gatsby', 'F. Scott Fitzgerald', 1925, 'Literary Fiction', 10, 9.99), 
('To Kill a Mockingbird', 'Harper Lee', 1960, 'Literary Fiction', 15, 12.99), 
('1984', 'George Orwell', 1949, 'Dystopian Fiction', 5, 7.99), 
('Pride and Prejudice', 'Jane Austen', 1813, 'Romantic Fiction', 8, 6.99), 
('The Catcher in the Rye', 'J.D. Salinger', 1951, 'Literary Fiction', 12, 10.99); 
 
CREATE INDEX idx_hash_books 
ON books 
USING HASH (author); 

SELECT * 
FROM books 
WHERE author IN ('F. Scott Fitzgerald','J.D. Salinger'); 

EXPLAIN SELECT * 
FROM books 
WHERE author IN ('F. Scott Fitzgerald','J.D. Salinger'); 

---

Chapter 11

CREATE SERVER oracle_server 
FOREIGN DATA WRAPPER oracle_fdw 
OPTIONS (
dbname 'ORCL', 
host 'localhost', 
port '1521'
); 

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
table 'my_oracle_table'
); 

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
table 'my_table'
); 

SELECT id, name, age 
FROM my_oracle_table 
WHERE age > 30; 

---

CREATE SERVER my_server FOREIGN DATA WRAPPER postgres_fdw 
OPTIONS 
(
host 'remote_host', 
dbname 'remote_dbname', 
port 'remote_port'
); 

CREATE USER MAPPING 
FOR my_user 
SERVER my_server 
OPTIONS (
user 'remote_user', 
password 'remote_password'
); 

CREATE FOREIGN TABLE my_foreign_table_pgsql 
( 
id INTEGER,
name VARCHAR(50),
age INTEGER 
) 
SERVER my_server 
OPTIONS (
schema 'remote_schema', 
table 'mytable'
); 

SELECT id, name, age 
FROM my_foreign_table_pgsql; 

---

CREATE SERVER my_mysql_server  
FOREIGN DATA WRAPPER mysql_fdw  
OPTIONS (
host 'localhost', 
port '3306', 
dbname 'mydb', 
user 'myuser', 
password 'mypassword'
); 

CREATE USER MAPPING 
FOR myuser 
SERVER my_mysql_server 
OPTIONS (
user 'myuser', 
password 'mypassword'
); 

CREATE FOREIGN 
TABLE my_foreign_table_mysql 
( 
id INTEGER,     
name VARCHAR(50), 
age INTEGER 
) 
SERVER my_mysql_server 
OPTIONS (
dbname 'mydb', 
table_name 'mytable'
); 

SELECT * FROM my_foreign_table_mysql; 

---

CREATE SERVER my_db2 
FOREIGN DATA WRAPPER db2_fdw 
OPTIONS (dbserver 'mydb2db'); 

CREATE USER MAPPING 
FOR PUBLIC SERVER my_db2 
OPTIONS (
user 'db2user', 
password 'my_password'
); 

IMPORT FOREIGN 
SCHEMA "my_db2" 
FROM SERVER my_db2 
INTO public; 

SELECT * 
FROM test_able_in_db2;

---

CREATE SERVER informix_server 
FOREIGN DATA WRAPPER informix_fdw 
OPTIONS (
host 'informix.example.com', 
service '1526', 
dbname 'mydb', 
client_library '/opt/informix/lib/esql/'
);

CREATE USER MAPPING 
FOR my_user 
SERVER informix_server 
OPTIONS (
username 'informix_user', 
password 'informix_password'
);

CREATE FOREIGN TABLE my_informix_table 
( 
id INTEGER,     
name VARCHAR(50),     
age INTEGER 
) 
SERVER informix_server 
OPTIONS (table 'mytable');

SELECT * 
FROM my_informix_table; 

---

CREATE SERVER my_sybase_server 
  FOREIGN DATA WRAPPER tds_fdw 
  OPTIONS ( 
    'servername' 'my_sybase_db', 
    'port' 'my_sybase_port', 
    'tds_version' 'my_tds_version', 
    'client_charset' 'my_charset' 
  ); 

CREATE USER MAPPING FOR my_postgres_user 
SERVER my_sybase_server 
OPTIONS ( 
'username' 'my_username',    
'password' 'my_password' 
); 

CREATE FOREIGN 
TABLE my_sybase_table  ( 
 id INT, 
 name VARCHAR(50), 
 age INT 
  ) 
  SERVER my_sybase_server 
  OPTIONS ( 
  'schema_name' 'my_sybase_schema', 
  'table_name' 'my_sybase_table' 
  ); 

SELECT * 
FROM my_sybase_table;

---

CREATE SERVER parquet_s3_server 
FOREIGN DATA WRAPPER parquet_s3_fdw 
OPTIONS ( (use_minio 'true');

CREATE USER MAPPING 
FOR parquet_s3_user 
SERVER parquet_s3_server 
OPTIONS ( 
'username' 's3_user', 
'password' 's3_password' 
 ); 

CREATE FOREIGN 
TABLE parquet_s3_table (     
id int,     
first_name   text, 
last_name    text 
) 
SERVER parquet_s3_server 
OPTIONS ( 
filename 's3://bucket/dir/userdata1.parquet' 
); 

SELECT * 
FROM parquet_s3_table; 

IMPORT FOREIGN 
SCHEMA "/path/to/directory" 
FROM SERVER parquet_s3_srv 
INTO public; 

---

Chapter 12

CREATE TABLE my_image (     
id SERIAL PRIMARY KEY,     
name VARCHAR(100),     
data BYTEA 
); 



import psycopg2 from psycopg2 import sql 


conn = psycopg2.connect(
     host="localhost",
     database="my_db",
     user="my_user",
     password="my_password" ) 

open("/appl/image.png", "rb") as f:
     image_data = f.read() 

conn.cursor() as cur:     
query = sql.SQL("INSERT INTO images (name, data) VALUES (%s, %s)")   

cur.execute(query, ("/appl/image.png", psycopg2.Binary(image_data)))     

conn.commit() 
conn.close()



CREATE TABLE my_json (     
id SERIAL PRIMARY KEY, 
json_col JSON, 
jsonb_col JSONB 
); 

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
VALUES ('{"name": "John", "age": 30, "hobbies": ["photography"], "address": {"street": "1010 Pine St", "city": "AnotherTown", "state": "CA"}}', '{"name": "John", "age": 30, "hobbies": ["photography"], "address": {"street": "1010 Pine St", "city": 
"AnotherTown", "state": "CA"}}'); 

CREATE INDEX idx_gin_data_json 
ON my_json 
USING GIN ((data_json->>'name') gin_trgm_ops); 
my_db=#  SELECT * 
FROM my_json 
WHERE data_json->>'name' = 'John'; 

SELECT 
data_json->>'name' AS name, 
data_json->>'age' AS age 
FROM my_json 
WHERE 
CAST(data_json->>'age' AS INTEGER) > 20; 

CREATE INDEX idx_gin_data_jsonb 
ON my_json 
USING GIN 
((data_jsonb->>'name') gin_trgm_ops);

SELECT * 
FROM my_json 
WHERE data_josnb >> ‘name’ =‘John’:

SELECT 
data_jsonb->>'name' AS name, 
data_jsonb->>'hobbies' AS hobbies 
FROM my_json 
WHERE data_jsonb->>'hobbies'  
LIKE  '%reading%'; 

CREATE TABLE sku (   
product_id SERIAL PRIMARY KEY,   
name VARCHAR(50) NOT NULL, 
attributes HSTORE 
); 

INSERT INTO sku (name, attributes) 
VALUES 
('Product 1', 'color => "red", weight => "10 lbs"'), 
('Product 2', 'color => "green",  length=> "12 inches"'), 
('Product 3', 'gender => "male",  height=> "180 cm"'), 
('Product 4', 'gender => "female", height=> "170 cm"'); 

CREATE INDEX idx_sku_attributes 
ON sku 
USING GIN(attributes); 

SELECT * 
FROM sku 
WHERE attributes @> 'length => "12 inches"';

CREATE TABLE files (   
id SERIAL PRIMARY KEY,   
title TEXT NOT NULL, 
content TEXT NOT NULL 
); 

INSERT INTO files (title, content) 
VALUES  
('Document 1', 'This is the content of document 1.'), 
('Document 2', 'This is the content of document 2.'), 
('Document 3', 'This is the content of document 3.'); 

CREATE INDEX idx_documents_content 
ON files USING gin(to_tsvector('english', content)); 

SELECT * 
FROM files  
WHERE to_tsvector('english', content) @@ to_tsquery('english', 'content');

CREATE TABLE mytable (   
id SERIAL PRIMARY KEY, 
content TEXT 
); 

CREATE TEXT SEARCH CONFIGURATION english (COPY= simple ); 

CREATE TEXT SEARCH DICTIONARY my_english_st ( TEMPLATE = pg_catalog.simple, 
STOPWORDS = english 
); 

CREATE EXTENSION pg_trgm;  

CREATE INDEX mytable_content_trgm_idx 
ON mytable 
USING gin (content gin_trgm_ops); 
my_db=# INSERT INTO mytable (content) 
VALUES  
('This is a sample text for full-text search indexing in PostgreSQL.'), 
('PostgreSQL is a powerful relational database management system.'), ('Full-text search in PostgreSQL allows for advanced searching of unstructured data.'); 

SELECT * 
FROM mytable 
WHERE to_tsvector('english', content) @@ to_tsquery('english', 'database'); 

CREATE TABLE movies (     
movie_id SERIAL PRIMARY KEY,   
title VARCHAR(100) NOT NULL,   
author VARCHAR(100) NOT NULL,   
publication_year INTEGER,     
genre VARCHAR(50), 
movie_xml XML 
); 

INSERT INTO movies (
title, 
author, 
publication_year, 
genre, 
movie_xml)  
VALUES 
('The Great Gatsby', 'F. Scott Fitzgerald', 1925, 'Literary Fiction',   
XML '<movie><episode>Episode 1</episode><episode>Episode 2</ episode></movie>'); 

SELECT xpath('/movie/episode/text()', movie_xml) 
FROM movies  
WHERE title = 'The Great Gatsby'; 

CREATE TYPE business_type 
AS ENUM 
('Retail', 'Finance', 'Healthcare', 'Manufacturing'); 

CREATE TYPE ownership_type 
AS ENUM  ('Public', 'Private', 'Nonprofit');
 
CREATE TABLE company (     
company_id SERIAL PRIMARY KEY,
name VARCHAR(100) NOT NULL,
industries business_type[],|
ownership ownership_type[] 
); 

INSERT INTO company (
name, 
industries, 
ownership)  
VALUES  
('ABC Company', '{Retail, Healthcare}', '{Private, Nonprofit}'), 
('XYZ Inc.', '{Finance, Manufacturing}', '{Public, Private}'); 

CREATE INDEX idx_company_industries 
ON company 
USING gin(industries); 

SELECT * 
FROM company 
WHERE 'Retail' = ANY(industries); 

---

Chapter 13

CREATE TABLE sales ( 
sale_id SERIAL,   
sale_date DATE NOT NULL, 
sale_amount NUMERIC(10,2) NOT NULL, 
CONSTRAINT sale_pk PRIMARY KEY (sale_date, sale_id) 
) 
PARTITION BY RANGE (sale_date); 

CREATE TABLE sales_2023 
PARTITION OF sales     
FOR VALUES 
FROM ('2023-01-01') 
TO ('2023-12-31'); 

CREATE TABLE sales_2024 
PARTITION OF sales     
FOR VALUES 
FROM ('2024-01-01') 
TO ('2024-12-31'); 

INSERT INTO sales (
sale_date, 
sale_amount
) 
VALUES 
('2023-01-01', 100.00), 
('2023-01-02', 150.00), 
('2023-01-03', 200.00),
('2024-01-01', 300.00), 
('2024-01-02', 250.00), 
('2024-01-03', 350.00); 

SELECT * FROM sales; 

SELECT * FROM sales_2023; 

SELECT * FROM sales_2024; 

SELECT ROW_NUMBER() 
OVER (ORDER BY book_id) AS row_num, 
title, 
author 
FROM books; 

SELECT 
title, 
qty, 
RANK() 
OVER (ORDER BY qty DESC) AS rank, 
DENSE_RANK() 
OVER (ORDER BY qty DESC) AS dense_rank 
FROM books; 

INSERT INTO books (title, author, publication_year, genre, qty) 
VALUES ('To Kill a Mockingbird v1', 'Harper Lee', 1960, 'Fiction', 30000); 

INSERT INTO books (title, author, publication_year, genre, qty) 
VALUES ('To Kill a Mockingbird v2', 'Harper Lee', 1961, 'Fiction', 80000); 

INSERT INTO books (title, author, publication_year, genre, qty) 
VALUES ('To Kill a Mockingbird v2', 'Harper Lee', 1961, 'Fiction', 80000); 

SELECT title, publication_year, qty, 
LAG(qty) 
OVER (PARTITION BY title 
ORDER BY publication_year) AS prev_year_qty 
FROM books; 

INSERT INTO books (title, author, publication_year, genre, qty) 
VALUES ('To Kill a Mockingbird v1', 'Harper Lee', 1960, 'Fiction', 30000); 

INSERT INTO books (title, author, publication_year, genre, qty) 
VALUES ('To Kill a Mockingbird v2', 'Harper Lee', 1961, 'Fiction', 80000); 

INSERT INTO books (title, author, publication_year, genre, qty) 
VALUES ('To Kill a Mockingbird v2', 'Harper Lee', 1961, 'Fiction', 80000); 

SELECT title, publication_year, qty, 
LAG(qty) 
OVER (PARTITION BY title 
ORDER BY publication_year) AS prev_year_qty 
FROM books; 

SELECT 
title,  
SUM(qty) AS total_sales, 
FIRST_VALUE(publication_year) 
OVER (PARTITION BY title 
ORDER BY publication_year) AS first_sale_date, 
LAST_VALUE(publication_year) 
OVER (PARTITION BY title 
ORDER BY publication_year 
ROWS BETWEEN UNBOUNDED PRECEDING AND 
UNBOUNDED FOLLOWING) AS last_sale_date 
FROM books 
GROUP BY title, publication_year; 

SELECT 
genre, 
title, 
qty, 
NTILE(3) 
OVER (ORDER BY qty DESC) as qty_group 
FROM books; 

SELECT 
publication_year, 
title, 
qty, 
PERCENT_RANK() OVER 
(PARTITION BY publication_year ORDER BY qty) AS pct_rank, 
CUME_DIST() OVER 
(PARTITION BY publication_year ORDER BY qty) AS cum_dist 
FROM books;

CREATE TABLE products (     
product_id SERIAL PRIMARY KEY,     
name TEXT NOT NULL,     
description TEXT,    
price NUMERIC(10,2),     
tags TEXT[] 
); 

CREATE TABLE orders ( 
name TEXT NOT NULL  PRIMARY KEY, 
price NUMERIC(10,2),    
sales_amount NUMERIC(10,2),     
sales_qty NUMERIC(10,2),     
order_date DATE 
); 

INSERT INTO products (name, description, price, tags) 
VALUES  
('pg_01', 'PostgreSQL DBA', 99.99, ARRAY['PostgreSQL', 'DBA', 'Database', 'PGSQL Tuning']), 
('pg_02', 'SQL for Everyone', 79.99, ARRAY['PostgreSQL', 'SQL']),       
('sp_01', 'Iceberg DBA', 89.99, ARRAY['Iceberg', 'Apache']); 


INSERT INTO orders (product_id, price, sales_amount, sales_qty, order_date) 
VALUES  
(1, 99.99, 585941.40, 5860, '2023-04-05'), 
(2, 79.99, 391151.10, 4890, '2023-04-04'), 
(3, 89.99, 340162.20, 3780, '2023-04-03'); 

WITH sales_by_product AS ( 
SELECT name, SUM(sales_amount) AS total_sales 
FROM orders 
GROUP BY name 
) 
SELECT p.description, s.total_sales 
FROM sales_by_product s 
JOIN products p ON p.name = s.name 
WHERE s.total_sales > 500000;

CREATE TABLE invoice (     
order_id SERIAL PRIMARY KEY, 
product_id INTEGER REFERENCES products(product_id), 
quantity INTEGER, 
price DECIMAL(10, 2) 
); 

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
ANY(products.tags)  
    WHERE products.tags != '{}' -- Ignore empty tags array 
) 
SELECT cte_products.product_id, cte_products.name, SUM(orders.sales_amount) 
AS sales_amount 
FROM cte_products 
JOIN orders ON orders.name = cte_products.name 
GROUP BY cte_products.product_id, cte_products.name 
ORDER BY cte_products.product_id; 

SELECT 
products.name AS product_name,     
orders.order_date, 
SUM(orders.sales_amount) AS total_sales_amount 
FROM orders 
LEFT JOIN products 
ON orders.name = products.name 
GROUP BY 
CUBE(products.name, orders.order_date) 
ORDER BY products.name, orders.order_date; 

SELECT 
products.name AS product_name, orders.order_date, 
SUM(orders.sales_amount) AS total_sales_amount 
FROM orders 
LEFT JOIN products ON orders.name = products.name 
GROUP BY ROLLUP(products.name, orders.order_date) 
ORDER BY products.name, orders.order_date; 

SELECT 
orders.order_date, 
SUM(orders.sales_amount) AS total_sales_amount 
FROM products  
JOIN orders ON products.name = orders.name 
GROUP BY 
GROUPING SETS (
  (products.name, orders.order_date), date()
)
ORDER BY  orders.order_date; 

CREATE OR REPLACE 
FUNCTION avg_price_sfunc(numeric[], numeric) 
RETURNS numeric[] AS $$ 
BEGIN 
  -- Append the value to the array 
  RETURN array_append($1, $2); 
END; 
$$ LANGUAGE plpgsql; 
CREATE OR REPLACE 
FUNCTION avg_price_finalfunc(numeric[]) 
RETURNS numeric AS $$ 
BEGIN 
  RETURN COALESCE( 
    (SELECT AVG(val) FROM unnest($1) val), 
    0 
  ); 
END; 
$$ LANGUAGE plpgsql; 

CREATE AGGREGATE avg_price(numeric) 
( 
sfunc = avg_price_sfunc,   
stype = numeric[], 
finalfunc = avg_price_finalfunc,   
initcond = '{}' 
); 

SELECT avg_price(price) AS average_price 
FROM products; 

---

Chapter 14

REINDEX INDEX 
CONCURRENTLY idx_book_titles; 

REINDEX INDEX 
CONCURRENTLY books; 

CREATE EXTENSION IF NOT EXISTS anon CASCADE;  

SELECT anon.start_dynamic_masking(); 

CREATE ROLE test01 LOGIN; 

SECURITY LABEL FOR anon 
ON ROLE test01 IS 'MASKED';  

CREATE TABLE info (
name varchar(30), 
important_id varchar(15)
); 

INSERT INTO info VALUES ('person 1', 'A123456(1)'); 

INSERT INTO info VALUES('person 2', 'B987653(2)'); 

SELECT * FROM info;   

SECURITY LABEL 
FOR anon 
ON COLUMN info.important_id 
IS 
'MASKED WITH FUNCTION 
anon.partial(important_id,5,$$***$$,0)'; 

SELECT * FROM info;

SECURITY LABEL 
FOR anon 
ON ROLE test01 IS NULL; 

SELECT 
anon.stop_dynamic_masking(); 

---

Chapter 15 & 16

ora2pg -t SHOW_SCHEMA -o schema.sql 

ora2pg -t ANALYZE -o analysis_report.txt 

*** END ***





