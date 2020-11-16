# MySQL Replication and Replicasets
## Setup Async Replication from 3306 to 3307
```
mysqlsh root@localhost:3306 --sql -e "create user repl@'%' identified with mysql_native_password by 'repl'; grant replication slave on *.* to repl@'%'"
mysqlsh root@localhost:3307 --sql -e "change master to master_user='repl', master_password='repl', master_host='127.0.0.1', master_port=3306, master_auto_position=1 for channel 'channel1'"
mysqlsh root@localhost:3307 --sql -e "set persist log_slave_updates=on"
mysqlsh root@localhost:3307 --sql -e "set persist super_read_only=on"
mysqlsh root@localhost:3307 --sql -e "start replica for channel 'channel1'"
mysqlsh root@localhost:3307 --sql -e "show replica status for channel 'channel1' \G"
mysqlsh root@localhost:3306 --sql -e "create table dummy.test (i int); insert into dummy.test values (1)"
mysqlsh root@localhost:3307 --sql -e "select * from dummy.test"
```


