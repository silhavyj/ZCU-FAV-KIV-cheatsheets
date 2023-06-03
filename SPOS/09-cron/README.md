## Cron
```
vim /etc/crontab
```
```
/etc/cron.d
/etc/cron.daily
/etc/cron.hourly
/etc/cron.monthly
/etc/cron.weekly
```
```
/var/spool/cron/crontabs/
```
```
crontab -l
crontab -e
crontab -l -u pepa
```
```
* * * * * echo "I'm running every minute" >> /tmp/text-cron.txt 
30 * * * * ./mysql-bak.sh
30 * * * * ./postgresql-bak.sh
* 2 * * * rsync -ra ~/A/ ~/B/
```
#### MYSQL backup of all databases using Cron
```bash
vim mysql-bak.sh
```
```bash
#!/bin/bash

DEST=/mnt/backup/mysql

for DB in $(mysql -e 'show databases' -s --skip-column-names); do
    mysqldump $DB --single-transaction | gzip > $DEST/MYSQL-$DB."`date +"%d-%m-%Y-%H-%M"`".sql.gz
done
```
```bash
chmod 744 mysql-bak.sh
```
#### POSTGRES backup of all databases using Cron
```
vim postgresql-bak.sh
```
```bash
#!/bin/bash

for DB in $(su - postgres -c "psql -lt" | cut -d \| -f 1 | awk '{$1=$1};1' | grep "\S"); do
    su - postgres -c "pg_dump $DB | gzip > /mnt/databases-bak/POSTGRES-$DB."`date +"%d-%m-%Y-%H-%M"`".sql.gz"
done
```
```bash
#!/bin/bash

PORT=9998
DEST=/mnt/backup/postgresql

for DB in $(psql -lt -p $PORT | cut -d \| -f 1 | awk '{$1=$1};1' | grep "\S" | grep -v template | grep -v postgres); do
    pg_dump -p $PORT $DB | gzip > $DEST/POSTGRES-$DB."`date +"%d-%m-%Y-%H-%M"`.sql.gz"
done
```
##### changing the group of the target folder so postgres can write to it
```
chown -R root:postgres /mnt/databases-bak/
chmod 775 /mnt/databases-bak/
```
