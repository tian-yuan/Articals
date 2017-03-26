#Hadoop安装与使用

##下载Hadoop

	http://apache.fayea.com/hadoop/common/stable2/hadoop-2.7.3.tar.gz
	tar -xvf hadoop-2.7.3.tar.gz
	
##设置JAVA_HOME

```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ which java
/usr/bin/java

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ export JAVA_HOME=/usr
```

##单机运行MapeReduce任务
```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ mkdir input

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ cp ./etc/hadoop/*.xml ./input/

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'
Not a valid JAR: /media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ ls ./share/hadoop/mapreduce/hadoop-mapreduce-
hadoop-mapreduce-client-app-2.7.3.jar              hadoop-mapreduce-client-hs-2.7.3.jar               hadoop-mapreduce-client-jobclient-2.7.3-tests.jar
hadoop-mapreduce-client-common-2.7.3.jar           hadoop-mapreduce-client-hs-plugins-2.7.3.jar       hadoop-mapreduce-client-shuffle-2.7.3.jar
hadoop-mapreduce-client-core-2.7.3.jar             hadoop-mapreduce-client-jobclient-2.7.3.jar        hadoop-mapreduce-examples-2.7.3.jar

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ ls ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar
./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep input output 'dfs[a-z.]+'
16/08/29 00:40:13 INFO Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
16/08/29 00:40:13 INFO jvm.JvmMetrics: Initializing JVM Metrics with processName=JobTracker, sessionId=
..............

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ ls ./output/
part-r-00000  _SUCCESS
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ cat ./output/*
1      	dfsadmin

```

##设置无密码ssh登录localhost

```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ ssh localhost
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is f6:8a:3e:a1:d4:e1:fe:ab:7e:e5:08:07:8e:f0:72:01.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.

parallels@localhost's password:
Welcome to Ubuntu 14.04.4 LTS (GNU/Linux 3.13.0-34-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '16.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Sun Aug 28 09:49:34 2016 from 10.211.55.2

parallels@ubuntu:~$ logout
Connection to localhost closed.

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
Generating public/private dsa key pair.
Your identification has been saved in /home/parallels/.ssh/id_dsa.
Your public key has been saved in /home/parallels/.ssh/id_dsa.pub.
The key fingerprint is:
82:8d:82:19:54:14:7c:6f:21:08:9b:f2:12:d2:c9:27 parallels@ubuntu
The key's randomart image is:
+--[ DSA 1024]----+
| o=+o            |
|.oo+ o .         |
|=oE o o .        |
|o* o + o         |
|+ o o + S        |
| . .   .         |
|                 |
|                 |
|                 |
+-----------------+

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ chmod 0600 ~/.ssh/authorized_keys

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ ssh
usage: ssh [-1246AaCfgKkMNnqsTtVvXxYy] [-b bind_address] [-c cipher_spec]
           [-D [bind_address:]port] [-E log_file] [-e escape_char]
           [-F configfile] [-I pkcs11] [-i identity_file]
           [-L [bind_address:]port:host:hostport] [-l login_name] [-m mac_spec]
           [-O ctl_cmd] [-o option] [-p port]
           [-Q cipher | cipher-auth | mac | kex | key]
           [-R [bind_address:]port:host:hostport] [-S ctl_path] [-W host:port]
           [-w local_tun[:remote_tun]] [user@]hostname [command]

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ ssh localhost
Welcome to Ubuntu 14.04.4 LTS (GNU/Linux 3.13.0-34-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

Last login: Mon Aug 29 15:50:33 2016 from localhost

parallels@ubuntu:~$ logout
Connection to localhost closed.

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$
```

##创建伪分布式系统

*	edit etc/hadoop/core-site.xml:

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

*	etc/hadoop/hdfs-site.xml:

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

##Run a MapReduce job locally

*	Format the filesystem:

```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ ./bin/hdfs namenode -format
16/08/29 01:18:49 INFO namenode.NameNode: STARTUP_MSG:
/************************************************************
STARTUP_MSG: Starting NameNode
.............
```

*	Start NameNode daemon and DataNode daemon:

```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ ./sbin/start-dfs.sh
Starting namenodes on [localhost]
localhost: Error: JAVA_HOME is not set and could not be found.
localhost: Error: JAVA_HOME is not set and could not be found.
Starting secondary namenodes [0.0.0.0]
The authenticity of host '0.0.0.0 (0.0.0.0)' can't be established.
ECDSA key fingerprint is f6:8a:3e:a1:d4:e1:fe:ab:7e:e5:08:07:8e:f0:72:01.
Are you sure you want to continue connecting (yes/no)? no
0.0.0.0: Host key verification failed.
```

>注意，上面的错误表示JAVA_HOME环境变量没有设置，但是 export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/ 设置后依然有问题，google 后，StackOverFlow 上有人说是 Hadoop 的 bug，需要把 /etc/hadoop/hadoop-env.sh 里面的 export JAVA_HOME=${JAVA_HOME} 修改为 export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/，然后重新运行 ./sbin/start-dfs.sh

```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ ./sbin/start-dfs.sh
Starting namenodes on [localhost]
localhost: starting namenode, logging to /media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3/logs/hadoop-parallels-namenode-ubuntu.out
localhost: starting datanode, logging to /media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3/logs/hadoop-parallels-datanode-ubuntu.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3/logs/hadoop-parallels-secondarynamenode-ubuntu.out
```

*	Browse the web interface for the NameNode; by default it is available at:

```
NameNode - http://localhost:50070/
```

*	Excute MapeReduce Jop


```
1. bin/hdfs dfs -mkdir /input
	
2. bin/hdfs dfs -mkdir /output
	
2. bin/hdfs dfs -put etc/hadoop /input
	
3. bin/hdfs dfs -ls /input 
	
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ bin/hdfs dfs -ls /input
Found 1 items
drwxr-xr-x   - parallels supergroup          0 2016-08-29 02:19 /input/hadoop	

4. bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep /input/hadoop /output 'dfs[a-z.]+'
	
5. bin/hdfs dfs -get /output output
	
6. cat output/*

parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/hadoop-2.7.3$ cat ./output/output1/*
6      	dfs.audit.logger
4      	dfs.class
3      	dfs.server.namenode.
2      	dfs.period
2      	dfs.audit.log.maxfilesize
2      	dfs.audit.log.maxbackupindex
1      	dfsmetrics.log
1      	dfsadmin
1      	dfs.servers
1      	dfs.replication
1      	dfs.file
```	
	