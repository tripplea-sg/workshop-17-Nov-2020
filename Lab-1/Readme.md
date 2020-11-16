# Backup Recovery and Cloning

## Login to server and source the environment
```
. $HOME/config/8022.env
```
## Create and start database 3306
```
cat $HOME/config/3306.cnf
ls $HOME/data/3306
mysqld --defaults-file=$HOME/config/3306.cnf --initialize-insecure
ls $HOME/data/3306
mysqld_safe --defaults-file=$HOME/config/3306.cnf &
```
## Login to database 3306 using mysql shell
```
mysqlsh root@localhost:3306 --sql
show databases;
\q
```
## Import Sakila database and World_x database into database 3306
```
mysqlsh root@localhost:3306 --sql -e "source /home/opc/script/sakila-schema.sql"
mysqlsh root@localhost:3306 --sql -e "source /home/opc/script/sakila-data.sql"
mysqlsh root@localhost:3306 --sql -e "source /home/opc/script/world_x.sql"
```
## The Power of MySQL Shell
```
mysqlsh root@localhost:3306 --sql -e "show databases;"
mysqlsh root@localhost:33060 -i -e "session.getSchemas()"
mysqlsh root@localhost:33060 -i --py -e "session.get_schemas()"
mysqlsh root@localhost:3306 --sql -e "select * from sakila.actor"
mysqlsh root@localhost:33060 -i -e "session.getSchema('sakila').getTable('actor').select()"
mysqlsh root@localhost:33060 --py -i -e "session.get_schema('sakila').get_table('actor').select()"
mysqlsh root@localhost:33060 -i -e "session.getSchema('world_x').getCollection('countryinfo').find();"
mysqlsh root@localhost:33060 -i -e "session.getSchema('world_x').getCollection('countryinfo').find().limit(2)"
mysqlsh root@localhost:33060 --sql --schema=world_x -e "SELECT JSON_OBJECT('Name', doc->>'$.Name', 'Population', doc->>'$.demographics.Population', 'SurfaceArea', doc->>'$.geography.SurfaceArea') AS results FROM countryinfo WHERE doc->>'$.Name' LIKE 'J%' AND doc->>'$.geography.Continent' = 'Asia';"
```
## Full Backup using MEB on 3306
```
mkdir /home/opc/config/backup
time mysqlbackup --user=root --host=127.0.0.1 --port=3306 --backup-dir=/home/opc/config/backup --with-timestamp backup-and-apply-log
ls /home/opc/config/backup
time mysqlbackup --user=root --host=127.0.0.1 --port=3306 --backup-dir=/home/opc/config/backup --read-threads=6 --process-threads=6 --write-threads=6 --limit-memory=300 --skip-unused-pages --with-timestamp backup-and-apply-log
ls /home/opc/config/backup
mysqlbackup --user=root --host=127.0.0.1 --port=3306 --backup-dir=/home/opc/config/backup backup-and-apply-log
mysqlsh root@localhost:3306 --sql -e "select * from mysql.backup_history \G;"
mysqlsh root@localhost:3306 --sql -e "select * from mysql.backup_progress \G"
```
## Incremental Backup on 3306
```
mkdir /home/opc/config/backup/incremental
mysqlsh root@localhost:3306 --sql -e "create database dummy; create table dummy.backup_trace (i varchar(100)); insert into dummy.backup_trace values ('before incremental')"
mysqlsh root@localhost:3306 --sql -e "select * from dummy.backup_trace;"
mysqlbackup --user=root --host=127.0.0.1 --port=3306 --incremental --incremental-backup-dir=/home/opc/config/backup/incremental --with-timestamp --incremental-base=history:last_backup backup
mysqlsh root@localhost:3306 --sql -e "select * from mysql.backup_history \G;"
```
## Differential Backup on 3306
```
mkdir /home/opc/config/backup/differential
mysqlsh root@localhost:3306 --sql -e "insert into dummy.backup_trace values ('before differential')"
mysqlsh root@localhost:3306 --sql -e "select * from dummy.backup_trace;"
mysqlbackup --user=root --host=127.0.0.1 --port=3306 --incremental --incremental-backup-dir=/home/opc/config/backup/differential --with-timestamp --incremental-base=history:last_full_backup backup
mysqlsh root@localhost:3306 --sql -e "select * from mysql.backup_history \G;"
mysqlsh root@localhost:3306 --sql -e "insert into dummy.backup_trace values ('after differential')"
mysqlsh root@localhost:3306 --sql -e "select * from dummy.backup_trace;"
```
## Restore from full backup to 3307
```
mysqlbackup --datadir=/home/opc/data/3307 --backup-dir=/home/opc/config/backup/<sub-directory-full-backup> copy-back-and-apply-log
mysqld_safe --defaults-file=$HOME/config/3307.cnf &
mysqlsh root@localhost:3307 --sql -e "show databases"
```
## Restore from Differential backup to 3307
```
mysqladmin -uroot -h127.0.0.1 -P3307 shutdown
mysqlbackup --datadir=/home/opc/data/3307 --incremental-backup-dir=/home/opc/config/backup/differential/<sub-directory-incremental-backup> --incremental copy-back-and-apply-log
mysqld_safe --defaults-file=$HOME/config/3307.cnf &
mysqlsh root@localhost:3307 --sql -e "show databases"
mysqlsh root@localhost:3307 --sql -e "select * from dummy.backup_trace;"
```
## Point in time recovery to 3307
```
cat $HOME/config/backup/differential/<sub-directory-incremental-backup>/meta/backup_variables.txt
mysqlbinlog /home/opc/data/3306/bin.000002 --start-position=1712477 | mysql -uroot -h127.0.0.1 -P3307
mysqlsh root@localhost:3307 --sql -e "select * from dummy.backup_trace;"
```



