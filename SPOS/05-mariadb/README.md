
  
## MySQL
```bash
apt-get install mariadb-server 
apt-get install pwgen # generating passwords
```
#### basic SQL commands
```sql
show databases;
use database;
show tables;
desc table;
show create table table01\G # shows the SQL commmand used to create the table
```
#### creating users
```sql
CREATE USER 'db01'@'localhost' IDENTIFIED BY 'password';
CREATE USER 'db01'@'%' IDENTIFIED BY 'password'; # db01 can connect from any ip
CREATE USER 'db01'@'147.228.%.%' IDENTIFIED BY 'password';
```
#### deleting users
```sql
DELETE FROM mysql.user WHERE `user` = "root";
```
#### showing all users
```sql
select * from mysql.user\G
select host,user from mysql.user\G
desc mysql.user;
```
#### creating a database
```sql
CREATE DATABASE db01 DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
```
#### granting privileges on the database to the users
```sql
GRANT ALL PRIVILEGES ON db01.* to 'db01'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT,UPDATE ON db01.* to 'db01'@'10.10.2.121' IDENTIFIED BY 'password';
```
```sql
FLUSH PRIVILEGES;
```
```sql
SHOW GRANTS FOR 'spos_ro'@'172.17.0.3';
```
```sql
REVOKE SELECT ON spos_test.* FROM 'spos_ro'@'172.17.0.3';
```
#### creating a table
```sql
CREATE TABLE table01(
   id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
   firstname VARCHAR(30) NOT NULL,
   lastname VARCHAR(30) NOT NULL,
   email VARCHAR(50),
   reg_date TIMESTAMP
)
```
```sql
CREATE TABLE table01(
   id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
   name VARCHAR(30) NOT NULL,
   description TEXT NOT NULL,
   created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL,
   price FLOAT NOT NULL
)
```
```sql
CREATE TABLE players (
	id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
	firstname VARCHAR(30) NOT NULL,
	lastname VARCHAR(30) NOT NULL,
	team VARCHAR(10) NOT NULL,
	goals INT UNSIGNED NOT NULL,
	score FLOAT NOT NULL
)
```
#### logging to the database
```bash
mysql -u root
mysql -h 127.0.0.1 -u db01 -p
mysql -h 127.0.0.1 -u db01 -p db01
mysql -h 127.0.0.1 -u db01 -p db01 --port 9999
mysql -u spos_ro -p -h 10.10.2.124 --port 9999 spos_test
```
```bash
vim ~/.my.cnf
```
```bash
[client]
user=spos_ro
password=spos_ro
port=9999
host=172.17.0.2
```
#### inserting random data into the database
```bash
for i in `seq 1 50`; do 
   echo "INSERT INTO table01 (firstname, lastname, email, reg_date) VALUES ('$(pwgen 5 1)','$(pwgen 10 1)','$(pwgen 5 1)@spos-silhavyj.spos',now())" | mysql db01
done
```
```bash
for i in `seq 1 50`; do 
   echo "INSERT INTO table01 (name, description, created_at, price) VALUES ('$(pwgen 5 1)','$(pwgen 10 1)',now(),'$(seq 0 .01 1 | shuf | head -n1)')" | mysql spos_test
done
```
```bash
for i in `seq 1 50`; do
	echo "INSERT INTO players (firstname, lastname, team, goals, score) VALUES (
		'$(pwgen -0A 6 1)',
		'$(pwgen -0A 8 1)',
		'$(pwgen -0c 5 1 | tr a-z A-Z)',
		'$(seq 1 20 | shuf | head -n1)',
		'$(seq 0 .01 1 | shuf | head -n1)'
	)" | mysql sport
done
```
##### random floats
```bash
'$(seq 0 .01 1 | shuf | head -n1)'
```
#### configuration
```bash
vim /etc/mysql/mariadb.conf.d/50-server.cnf
port = 9999
bind-address = 127.0.0.1
```
```bash
/etc/init.d/mysql restart
netstat -tupln | grep 9999
```
#### creating a user file
```bash
vim ~/.my.cnf
```
```bash
[client]
user=db01
password=password
database=db01
port=9999
```
#### backup
```bash
mysqldump db01
mysqldump db01 | gzip > db01.sql.gz
```
```sql
drop database db01;
create database db01;
```
```bash
zcat db01.sql.gz | mysql
```
#### connecting to the database through apache using php
```bash
apt-get install php-mysql # driver for php
/etc/init.d/apache2 restart
```
```bash
vim /var/www/ssl-www.silhavyj.spos/db.php
```
```php
<?php
$servername = "localhost";
$username = "spos_test";
$password = "password";
$dbname = "spos_test";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);
// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

$sort = $_GET['sort'];
if ($sort == 1) {
    $sql = "SELECT id, name, description, created_at, price FROM table01 ORDER BY price";
} else {
    $sql = "SELECT id, name, description, created_at, price FROM table01";
}
$result = $conn->query($sql);

if ($result->num_rows > 0) {
    // output data of each row
    while($row = $result->fetch_assoc()) {
        echo "id: " . $row["id"]. " - Name: " . $row["name"]. " " . $row["description"]. " " . $row["created_at"] . " " . $row["price"]. "<br>";
    }
} else {
    echo "0 results";
}
$conn->close();
?>
```
```php
$sql = "SELECT id, firstname, lastname, team, score FROM players WHERE goals > 16 ORDER BY score DESC";
```
