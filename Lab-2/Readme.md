# MySQL Replication and Replicasets
## Setup Async Replication from 3306 to 3307
```
mysqlsh root@localhost:3306 --sql -e "create user repl@'%' identified with mysql_native_password by 'repl'; grant replication slave on *.* to repl@'%'"
mysqlsh root@localhost:3306 --sql -e "set persist gtid_mode='OFF_PERMISSIVE'; set persist gtid_mode='ON_PERMISSIVE'; set persist enforce_gtid_consistency=on; set persist gtid_mode=on"
mysqlsh root@localhost:3307 --sql -e "set persist gtid_mode='OFF_PERMISSIVE'; set persist gtid_mode='ON_PERMISSIVE'; set persist enforce_gtid_consistency=on; set persist gtid_mode=on"
mysqlsh root@localhost:3307 --sql -e "set persist server_id=100"
mysqlsh root@localhost:3307 --sql -e "change master to master_user='repl', master_password='repl', master_host='127.0.0.1', master_port=3306, master_auto_position=1 for channel 'channel1'"
mysqlsh root@localhost:3307 --sql -e "set persist super_read_only=on"
mysqlsh root@localhost:3307 --sql -e "start replica for channel 'channel1'"
mysqlsh root@localhost:3307 --sql -e "show replica status for channel 'channel1' \G"
mysqlsh root@localhost:3306 --sql -e "create table dummy.test (i int); insert into dummy.test values (1)"
mysqlsh root@localhost:3307 --sql -e "select * from dummy.test"
```
## Setup Async Replication from 3306 to 3308
```
mysqlsh root@localhost:3308 --sql -e "set persist gtid_mode='OFF_PERMISSIVE'; set persist gtid_mode='ON_PERMISSIVE'; set persist enforce_gtid_consistency=on; set persist gtid_mode=on"
mysqlsh root@localhost:3308 --sql -e "set persist server_id=1000"
mysqlsh root@localhost:3308 --sql -e "change master to master_user='repl', master_password='repl', master_host='127.0.0.1', master_port=3306, master_auto_position=1 for channel 'channel1'"
mysqlsh root@localhost:3308 --sql -e "set persist super_read_only=on"
mysqlsh root@localhost:3308 --sql -e "start replica for channel 'channel1'"
mysqlsh root@localhost:3308 --sql -e "show replica status for channel 'channel1' \G"
mysqlsh root@localhost:3306 --sql -e "insert into dummy.test values (2)"
mysqlsh root@localhost:3307 --sql -e "select * from dummy.test"
mysqlsh root@localhost:3308 --sql -e "select * from dummy.test"
```
## Auto Failover Replication (New Feature 8.0.22)
```
mysqlsh root@localhost:3306 --sql -e "create user gradmin@'%' identified with mysql_native_password by 'grpass'; grant replication slave on *.* to gradmin@'%'"
mysqlsh -e "dba.deploySandboxInstance(3311)"
mysqlsh root@localhost:3311 --sql -e "install plugin clone soname 'mysql_clone.so'; set global clone_valid_donor_list='127.0.0.1:3306'"
mysqlsh root@localhost:3311 --sql -e "clone instance from clone@'127.0.0.1':3306 identified by 'clone'"
mysqlsh root@localhost:3311 --sql -e "show databases; select * from dummy.test"
mysqlsh root@localhost:3311 --sql -e "change master to master_user='gradmin', master_host='127.0.0.1', master_port=3308, master_auto_position=1, master_password='grpass', source_connection_auto_failover=1, master_connect_retry=3, master_retry_count=3 for channel 'channel1';" 
mysqlsh root@localhost:3311 --sql -e "select asynchronous_connection_failover_add_source('channel1', '127.0.0.1', 3308, '', 100)"
mysqlsh root@localhost:3311 --sql -e "select asynchronous_connection_failover_add_source('channel1', '127.0.0.1', 3307, '', 90)"
mysqlsh root@localhost:3311 --sql -e "select asynchronous_connection_failover_add_source('channel1', '127.0.0.1', 3306, '', 80)"
mysqlsh root@localhost:3311 --sql -e "select * from mysql.replication_asynchronous_connection_failover"
mysqlsh root@localhost:3311 --sql -e "start replica for channel 'channel1'"
mysqlsh root@localhost:3311 --sql -e "show replica status for channel 'channel1' \G"
mysqlsh root@localhost:3306 --sql -e "insert into dummy.test values (3)"
mysqlsh root@localhost:3311 --sql -e "select * from dummy.test"
mysqladmin -uroot -h127.0.0.1 -P3308 shutdown
mysqlsh root@localhost:3306 --sql -e "insert into dummy.test values (4)"
mysqlsh root@localhost:3311 --sql -e "select * from dummy.test"
mysqlsh root@localhost:3311 --sql -e "show replica status for channel 'channel1' \G"
mysqladmin -uroot -h127.0.0.1 -P3307 shutdown
mysqlsh root@localhost:3306 --sql -e "insert into dummy.test values (5)"
mysqlsh root@localhost:3311 --sql -e "select * from dummy.test"
mysqlsh root@localhost:3311 --sql -e "show replica status for channel 'channel1' \G"
mysqlsh root@localhost:3311 --sql -e "stop replica for channel 'channel1'"
```
## Replicasets
```
mysqld_safe --defaults-file=$HOME/config/3307.cnf &
mysqld_safe --defaults-file=$HOME/config/3308.cnf &
mysqlsh root@localhost:3306 --sql -e "drop user gradmin@'%'"
mysqlsh root@localhost:3307 --sql -e "set persist super_read_only=off; drop user gradmin@'%'"
mysqlsh root@localhost:3308 --sql -e "set persist super_read_only=off; drop user gradmin@'%'"
mysqlsh -- dba configure-instance { --user=root --host=127.0.0.1 --port=3306 } --clusterAdmin=gradmin --clusterAdminPassword=grpass --interactive=false --restart=true
mysqlsh -- dba configure-instance { --user=root --host=127.0.0.1 --port=3307 } --clusterAdmin=gradmin --clusterAdminPassword=grpass --interactive=false --restart=true
mysqlsh -- dba configure-instance { --user=root --host=127.0.0.1 --port=3308 } --clusterAdmin=gradmin --clusterAdminPassword=grpass --interactive=false --restart=true


