# MySQL Enterprise Backup

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
## Login to database using mysql shell
```
mysqlsh root@localhost:3306 --sql
show databases;
\q
```
## Import Sakila database and World_x database
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

