## Postgres
```bash
apt search postgresql-11
apt-get install postgresql-11
```
#### start/restart/status version cluster
```bash
pg_ctlcluster
pg_ctlcluster 11 main status
```
#### creating a new cluster
```bash
pg_createcluster 11 spos
pg_ctlcluster 11 spos start
pg_dropcluster 11 spos
```
#### switching to the root
```bash
su - postgres
psql
psql -p 9997
```
#### basic commands
```sql
\x on  # '\G' in MySQL
\x off

\q     # quit
\l     # show databases
\dt    # show tables
\c database # connect to a database
\du         # show the list of user
\dn         # list schemas
\d table01  # list content of table01
```
#### creating users (roles)
```sql
CREATE USER db01 WITH PASSWORD 'password';
CREATE USER db02 WITH PASSWORD 'password';
CREATE USER root WITH PASSWORD 'password';
```
```sql
GRANT db02 to "db01";
\du
```
#### creating a database
```sql
CREATE DATABASE db01 WITH OWNER=db01;
```
#### creating a table
```sql
CREATE TABLE table01(
   id SERIAL PRIMARY KEY,
   firstname VARCHAR(30) NOT NULL,
   lastname VARCHAR(30) NOT NULL,
   email VARCHAR(50),
   reg_date TIMESTAMP
);

CREATE TABLE table01(
   id SERIAL PRIMARY KEY,
   name VARCHAR(30) NOT NULL,
   description TEXT NOT NULL,
   created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL,
   price FLOAT NOT NULL
);
```
#### granting  privileges
```sql
GRANT ALL PRIVILEGES ON DATABASE db01 to db01;
GRANT CONNECT ON DATABASE database_name TO username;
GRANT USAGE ON SCHEMA public TO username;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO username;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO username;
GRANT SELECT ON TABLE table01 TO db02;
```
#### all users other than postgres connecting from 'localhost' must use  their passwords
```bash
vim /etc/postgresql/11/main/pg_hba.conf
```
```bash
local   all	postgres	peer
local	  all	all		md5

host    spos_test       spos_ro         10.10.2.103/32          md5
```
#### changing the port and ip addresses
```bash
vim /etc/postgresql/11/main/postgresql.conf
```
```bash
port = 5432
listen_addresses = '127.0.0.1,147.228.67.45'
```
```bash
/etc/init.d/postgresql restart
pg_ctlcluster 11 main restart
systemctl restart postgresql@11-main.service
```
```bash
netstat -tupln | grep post
```
#### remote connection
```
psql -U spos_ro -h 10.10.2.124 -p 5432 spos_test
psql -U spos_test
```
#### inserting data
```bash
for i in `seq 1 50`; do 
   echo "INSERT INTO table01 (firstname, lastname, email, reg_date) VALUES ('$(pwgen 5 1)','$(pwgen 10 1)','$(pwgen 5 1)@spos-silhavyj.spos',now())" | psql db01
done
```
#### creating a user file
```bash
vim ~/.pgpass
chmod 0600 ~/.pgpass
localhost:5432:db01:root:password
psql -d db01
```
#### backup
```bash
su - postgres

pg_dump db01
pg_dump -c db01
pg_dump -c db01 -t table01

pg_dump -c db01 | gzip > db01.sql.gz
DROP DATABASE "db01";
CREATE DATABASE db01 WITH OWNER=db01;
gunzip -c db01.sql.gz | psql db01
```
#### connecting to the database through apache using php
```bash
apt-get install php-pgsql
```
```bash
vim /var/www/ssl-www.silhavyj.spos/db.php
```
```sql
<?php
// Connecting, selecting database
$dbconn = pg_connect("host=localhost port=9998 dbname=db01 user=root password=pasword")
    or die('Could not connect: ' . pg_last_error());

// Performing SQL query
$query = 'SELECT * FROM table01 ORDER BY firstname';
$result = pg_query($query) or die('Query failed: ' . pg_last_error());

// Printing results in HTML
echo "<table>\n";
while ($line = pg_fetch_array($result, null, PGSQL_ASSOC)) {
    echo "\t<tr>\n";
    foreach ($line as $col_value) {
        echo "\t\t<td>$col_value</td>\n";
    }
    echo "\t</tr>\n";
}
echo "</table>\n";

// Free resultset
pg_free_result($result);

// Closing connection
pg_close($dbconn);
?>
```
