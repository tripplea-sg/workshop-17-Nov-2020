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
mysqlsh root@localhost:3306 --sql -e "select * from sakila.actor"
mysqlsh root@localhost:33060 -e "session.getSchema('world_x').getCollection('countryinfo').find();"
mysqlsh root@localhost:33060 --sql -e "SELECT JSON_OBJECT("Name", doc->>'$.Name', "Population", doc->>'$.demographics.Population', "SurfaceArea", doc->>'$.geography.SurfaceArea') AS results FROM countryinfo WHERE doc->>'$.Name' LIKE "J%" AND doc->>'$.geography.Continent' = 'Asia';"
```
