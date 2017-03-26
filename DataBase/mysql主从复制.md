# mysql主从复制

## 环境准备

```
10.211.55.3 master；
10.211.55.9 slave;

mysql -h127.0.0.1 -uroot -pxxx
```

> 使用 mysql -uroot -pxxx 连接数据库时，出现如下错误:
> parallels@ubuntu:~$ 
> mysql ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
> 在我的测试环境中这个是由于无法识别 localhost 导致；

## 修改 master 数据库的配置文件

```
sudo vim /etc/mysql/my.cnf

[mysqld]
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
server-id               = 1
log_bin                 = /var/log/mysql/mysql-bin.log
```

## 修改 slave 数据库的配置文件

```
sudo vim /etc/mysql/my.cnf

[mysqld]
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
server-id               = 2
log_bin                 = /var/log/mysql/mysql-bin.log
```

## 重启 master 和 slave 上的数据库

```
/etc/init.d/mysql restart
```

## 在 master 上创建账户并授权 slave

```
parallels@ubuntu:~$ mysql -uroot -p********
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 93
Server version: 5.5.49-0ubuntu0.14.04.1-log (Ubuntu)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> GRANT REPLICATION SLAVE ON *.* to 'mysync'@'%' identified by 'q123456';
Query OK, 0 rows affected (0.00 sec)

mysql> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |    23486 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)

mysql> exit
```

## 配置 slave ，并启动备份功能

```
root@ubuntu:/opt/stack/devstack# mysql -uroot -p*******
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 87
Server version: 5.5.49-0ubuntu0.14.04.1-log (Ubuntu)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> change master to master_host='192.168.145.222',master_user='mysync',master_password='q123456',
    ->
    ->
    -> ;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' at line 1
mysql> change master to master_host='10.211.55.3',master_user='mysync',master_password='q123456',master_log_file='mysql-bin.000001',master_log_pos=23486;
Query OK, 0 rows affected (0.01 sec)

mysql> start slave
    -> ;
Query OK, 0 rows affected (0.01 sec)

mysql> show slave status
    -> ;
```

>注意 master_log_file='mysql-bin.000001' 必须是在 master 上 show master status; 输出的File 对应的文件；

## 验证主从备份功能

master 上的操作：

```
parallels@ubuntu:~$ mysql -h127.0.0.1 -uroot -pliuzebo,./
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 98
Server version: 5.5.49-0ubuntu0.14.04.1-log (Ubuntu)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| cinder             |
| glance             |
| hive               |
| keystone           |
| mysql              |
| neutron            |
| nova               |
| nova_api           |
| performance_schema |
+--------------------+
10 rows in set (0.00 sec)

mysql>
mysql> create database rabbit_test;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| cinder             |
| glance             |
| hive               |
| keystone           |
| mysql              |
| neutron            |
| nova               |
| nova_api           |
| performance_schema |
| rabbit_test        |
+--------------------+
11 rows in set (0.00 sec)

mysql> use rabbit_test;
Database changed
mysql> show tables;
Empty set (0.00 sec)

mysql> create table address_book(id int, name char(32));
Query OK, 0 rows affected (0.01 sec)

mysql> show tables;
+-----------------------+
| Tables_in_rabbit_test |
+-----------------------+
| address_book          |
+-----------------------+
1 row in set (0.00 sec)

mysql> select * from address_book;
Empty set (0.00 sec)

mysql> insert into address_book value(1, "huihui");
Query OK, 1 row affected (0.01 sec)

mysql> select * from address_book;
+------+--------+
| id   | name   |
+------+--------+
|    1 | huihui |
+------+--------+
1 row in set (0.00 sec)

mysql>
```

slave 上的操作：

```
root@ubuntu:/opt/stack/devstack# mysql -uroot -p****** -h127.0.0.1
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 105
Server version: 5.5.49-0ubuntu0.14.04.1-log (Ubuntu)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| cinder             |
| glance             |
| hive               |
| keystone           |
| mysql              |
| neutron            |
| nova               |
| nova_api           |
| performance_schema |
| rabbit_test        |
+--------------------+
11 rows in set (0.00 sec)

mysql> use rabbit_test;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-----------------------+
| Tables_in_rabbit_test |
+-----------------------+
| address_book          |
+-----------------------+
1 row in set (0.00 sec)

mysql> select * from address_book;
+------+--------+
| id   | name   |
+------+--------+
|    1 | bobo   |
|    1 | huihui |
+------+--------+
2 rows in set (0.00 sec)

mysql>
```


> 注意，不要同时操作 master 和 slave，这样也会到只数据不一致；如上就是在 slave 上插入了一条记录 |    1 | bobo   | 导致；