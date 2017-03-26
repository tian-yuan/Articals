##下载Hive可执行程序

	wget http://apache.fayea.com/hive/stable-2/apache-hive-2.1.0-bin.tar.gz
	tar -xvf apache-hive-2.1.0-bin.tar.gz
	
##配置信息设置

	cp ./conf/hive-default.xml.template ./conf/hive-site.xml

```	
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://localhost:3306/hive?createData baseIfNotExist=true</value>
  <description>JDBC connect string for a JDBC metastore</description>
</property>
<property>
<name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
  <description>Driver class name for a JDBC metastore</description>
</property>
<property>
<name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
   <description>username to use against metastore database</description>
</property>
<property>
<name>javax.jdo.option.ConnectionPassword</name>
   <value>liuzebo,./</value>
   <description>password to use against metastore database</description>
</property>
<property>
<name>datanucleus.schema.autoCreateTables</name>
<value>true</value>
<description>auto create tables</description>
</property>
</configuration>
```

上面配置的mysql数据库的连接信息

##运行metastore
	
	./bin/hive --service metastore
	
##运行./bin/hive	
	
```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/apache-hive-2.1.0-bin$ bin/hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/media/psf/UbuntuWorkspace/JavaDevelop/apache-hive-2.1.0-bin/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/media/psf/UbuntuWorkspace/JavaDevelop/apache-hive-2.1.0-bin/lib/hive-common-2.1.0.jar!/hive-log4j2.properties Async: true
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. tez, spark) or using Hive 1.X releases.
hive>
```	



##错误处理

*	启动问题

```
org.apache.thrift.transport.TTransportException: Could not create ServerSocket on address 0.0.0.0/0.0.0.0:9083.
       	at org.apache.thrift.transport.TServerSocket.<init>(TServerSocket.java:109)
       	at org.apache.thrift.transport.TServerSocket.<init>(TServerSocket.java:91)
       	at org.apache.thrift.transport.TServerSocket.<init>(TServerSocket.java:83)
       	
使用 netstat -nap | grep 9083 发现 9083 端口被占用，这个被占用有两种情况，一种是其他应用程序占用了这个端口，另外一个原因是 metastore 已经运行了，此时需要 kill 掉，然后重新启动：./bin/hive --service metastore       	
```


*	配置错误

```
Logging initialized using configuration in jar:file:/media/psf/UbuntuWorkspace/JavaDevelop/apache-hive-2.1.0-bin/lib/hive-common-2.1.0.jar!/hive-log4j2.properties Async: true
Exception in thread "main" java.lang.IllegalArgumentException: java.net.URISyntaxException: Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D

./conf/hive-site.xml 中有如下配置：
<property>
    <name>hive.exec.local.scratchdir</name>
    <value>${system:java.io.tmpdir}/${system:user.name}</value>
    <description>Local scratch space for Hive jobs</description>
  </property>
  <property>
    <name>hive.downloaded.resources.dir</name>
    <value>${system:java.io.tmpdir}/${hive.session.id}_resources</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>
  
需要修改成：
<property>
    <name>hive.exec.local.scratchdir</name>
    <value>/tmp/hive/local</value>
    <description>Local scratch space for Hive jobs</description>
  </property>
  <property>
    <name>hive.downloaded.resources.dir</name>
    <value>/tmp/hive/resources</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>
  
```

*	缺少 JDBC jar 包

```
Attempt to invoke the "BONECP" plugin to create a ConnectionPool gave an error : The specified datastore driver ("com.mysql.jdbc.Driver") was not found in the CLASSPATH. Please check your CLASSPATH specification, and the name of the driver.

需要把 mysql-connector-java-5.1.34.jar 包复制到 $HIVE_HOME/lib 目录下

然后把 $HIVE_HOME/lib 放到 CLASSPATH 中
```

*	服务没有运行起来

```
Exception in thread "main" java.lang.RuntimeException: org.apache.hadoop.hive.ql.metadata.HiveException: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient

./bin/hive --service metastore
启动 metastore
```

##参考资料

<http://www.tutorialspoint.com/hive/hive_installation.htm>
	