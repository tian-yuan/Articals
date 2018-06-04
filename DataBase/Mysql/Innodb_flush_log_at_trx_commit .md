#### Innodb_flush_log_at_trx_commit 参数意义



1. 当`innodb_flush_log_at_trx_commit=0`时, log buffer将每秒一次地写入log file, 并且log file的flush(刷新到disk)操作同时进行. 此时, 事务提交是不会主动触发写入磁盘的操作.

2. 当`innodb_flush_log_at_trx_commit=1`时(默认), 每次事务提交时, MySQL会把log buffer的数据写入log file, 并且将log file flush(刷新到disk)中去.

3. 当`innodb_flush_log_at_trx_commit=2`时, 每次事务提交时, MySQL会把log buffer的数据写入log file, 但不会主动触发flush(刷新到disk)操作同时进行. 然而, MySQL会每秒执行一次flush(刷新到disk)操作.

   > 然而, 每秒flush并不能确保100%每秒发生, 因为os调度问题.

默认的1可以获得更好地数据安全, 但性能会打折扣. 不过非1时, 在遇到crash可能会丢失1秒的事务; 设置为0时, 任何mysqld进程crash会丢失上1秒的事务; 设置为2时, 任何os crash或者机器掉电会丢失上1秒的事务; InnoDB的crash recovery运行时会忽略这些数据.