##下载Derby

	wget http://mirrors.cnnic.cn/apache//db/derby/db-derby-10.12.1.1/db-derby-10.12.1.1-bin.tar.gz
	tar -xvf db-derby-10.12.1.1-bin.tar.gz
	
##设置Derby环境变量

```
export DERBY_HOME=/media/psf/UbuntuWorkspace/JavaDevelop/db-derby-10.12.1.1-bin
export CLASSPATH="${DERBY_HOME}/lib/derby.jar:${DERBY_HOME}/lib/derbytools.jar:${DERBY_HOME}/lib/derbyoptionaltools.jar:${CLASSPATH}"
export PATH=$PATH:$DERBY_HOME/bin	
	
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/db-derby-10.12.1.1-bin$ source /etc/bash.bashrc
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/db-derby-10.12.1.1-bin$ bin
bin/  bind
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/db-derby-10.12.1.1-bin$ bin/setEmbeddedCP
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/db-derby-10.12.1.1-bin$ bin/set
setEmbeddedCP           setEmbeddedCP.bat       setNetworkClientCP      setNetworkClientCP.bat  setNetworkServerCP      setNetworkServerCP.bat
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/db-derby-10.12.1.1-bin$ bin/setNetworkClientCP
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/db-derby-10.12.1.1-bin$ bin/setNetworkServerCP
```

##操作数据库

```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/db-derby-10.12.1.1-bin$ bin/ij
ij version 10.12
ij> connect ‘jdbc:derby:emp;create=true’;
IJ ERROR: Unable to establish connection
ij> IJ ERROR: Unable to establish connection
ij> connect 'jdbc:derby:emp;create=true';
ij> ls
> ;
ERROR 42X01: Syntax error: Encountered "ls" at line 1, column 1.
Issue the 'help' command for general information on IJ command syntax.
Any unrecognized commands are treated as potential SQL commands and executed directly.
Consult your DBMS server reference documentation for details of the SQL syntax supported by your server.
ij> create table name(id int, name varchar(20));
0 rows inserted/updated/deleted
ij> insert into name values(1, "tianyuan");
ERROR 42X04: Column 'tianyuan' is either not in any table in the FROM list or appears within a join specification and is outside the scope of the join specification or appears in a HAVING clause and is not in the GROUP BY list. If this is a CREATE or ALTER TABLE  statement then 'tianyuan' is not a column in the target table.
ij> insert into name values(1, 'tianyuan');
1 row inserted/updated/deleted
ij> select * from nam
> e;
ERROR 42X05: Table/View 'NAM' does not exist.
ij> select * from name;
ID         |NAME
--------------------------------
1          |tianyuan

1 row selected
```


##启动Derby Server

```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/db-derby-10.12.1.1-bin$ bin/startNetworkServer
Mon Aug 29 19:49:58 PDT 2016 : Security manager installed using the Basic server security policy.
Mon Aug 29 19:49:58 PDT 2016 : Apache Derby Network Server - 10.12.1.1 - (1704137) started and ready to accept connections on port 1527
```


##参考资料

<http://db.apache.org/derby/papers/DerbyTut/ns_intro.html>




