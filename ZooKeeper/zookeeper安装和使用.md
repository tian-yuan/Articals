#单机版本的ZooKeeper的安装

*	wget http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.8/zookeeper-3.4.8.tar.gz (此链接地址根据下载版本不同会不同)
*	tar -xvf zookeeper-3.4.8.tar.gz
*	cd zookeeper-3.4.8
*	修改配置文件 vim ./conf/zoo.cfg

```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/var/lib/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.1=localhost:2888:3888
```

*	增加 /var/lib/zookeeper/myid 文件，里面只包含 1
*	zookeeper 开启和停止

```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8$ sudo ./bin/zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8$ sudo ./bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

*	检查服务运行状态

```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8$ ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../conf/zoo.cfg
Mode: standalone
```
*	客户端链接服务器

```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8$ ./bin/zkCli.sh -server 127.0.0.1:2181
Connecting to 127.0.0.1:2181
2016-08-25 23:17:05,780 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.8--1, built on 02/06/2016 03:18 GMT
2016-08-25 23:17:05,785 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=ubuntu
2016-08-25 23:17:05,786 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.7.0_111
2016-08-25 23:17:05,788 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2016-08-25 23:17:05,788 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/lib/jvm/java-7-openjdk-amd64/jre
2016-08-25 23:17:05,789 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../build/classes:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../build/lib/*.jar:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../lib/slf4j-log4j12-1.6.1.jar:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../lib/slf4j-api-1.6.1.jar:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../lib/netty-3.7.0.Final.jar:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../lib/log4j-1.2.16.jar:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../lib/jline-0.9.94.jar:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../zookeeper-3.4.8.jar:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../src/java/lib/*.jar:/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8/bin/../conf:
2016-08-25 23:17:05,789 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=:/usr/local/lib:/usr/local/lib:/usr/java/packages/lib/amd64:/usr/lib/x86_64-linux-gnu/jni:/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu:/usr/lib/jni:/lib:/usr/lib
2016-08-25 23:17:05,789 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2016-08-25 23:17:05,789 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2016-08-25 23:17:05,789 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2016-08-25 23:17:05,790 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2016-08-25 23:17:05,790 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=3.13.0-34-generic
2016-08-25 23:17:05,790 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=parallels
2016-08-25 23:17:05,790 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/home/parallels
2016-08-25 23:17:05,791 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/media/psf/UbuntuWorkspace/JavaDevelop/zookeeper-3.4.8
2016-08-25 23:17:05,793 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=127.0.0.1:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@6405b42e
Welcome to ZooKeeper!
2016-08-25 23:17:05,847 [myid:] - INFO  [main-SendThread(127.0.0.1:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server 127.0.0.1/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
2016-08-25 23:17:05,855 [myid:] - INFO  [main-SendThread(127.0.0.1:2181):ClientCnxn$SendThread@876] - Socket connection established to 127.0.0.1/127.0.0.1:2181, initiating session
JLine support is enabled
2016-08-25 23:17:05,925 [myid:] - INFO  [main-SendThread(127.0.0.1:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server 127.0.0.1/127.0.0.1:2181, sessionid = 0x156c57a7e9c0000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: 127.0.0.1:2181(CONNECTED) 0]
```

>注意，有的文档里面使用 ./bin/zkCli.sh server 进行连接，这个是错误的，会出现如下错误:

```
2016-08-25 23:19:14,184 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
       	close
       	ls2 path [watch]
       	history
       	listquota path
       	setAcl path acl
       	getAcl path
       	sync path
       	redo cmdno
       	addauth scheme auth
       	delete path [version]
       	setquota -n|-b val path
```

*	开发 Java 客户端代码

```
import org.apache.zookeeper.*;

import java.io.IOException;

/**
 * Created by liuzebo on 16/8/26.
 */
public class ZKDemo {
    public static void main(String[] args) throws IOException, KeeperException, InterruptedException{
        ZooKeeper zk = new ZooKeeper("10.211.55.3:2181", 6000, new Watcher() {
            public void process(WatchedEvent watchedEvent) {
                System.out.println("EVENT : " + watchedEvent.getType());
            }
        });

        ///< 查看根节点
        System.out.println("ls / ==> " + zk.getChildren("/", true));

        ///< 创建一个节点
        if (zk.exists("/node", true) == null) {
            zk.create("/node", "conan".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            System.out.println("Create /node conan");
            System.out.println("get /node ==> " + new String(zk.getData("/node", false, null)));
            System.out.println("ls / ==> " + zk.getChildren("/", true));
        }

        ///< 关闭连接
        zk.close();
    }
}

```

*	配置 maven 配置

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com</groupId>
    <artifactId>rabbit.zookeeper.demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.6</version>
            <exclusions>
                <exclusion>
                    <groupId>javax.jms</groupId>
                    <artifactId>jms</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>com.sun.jdmk</groupId>
                    <artifactId>jmxtools</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>com.sun.jmx</groupId>
                    <artifactId>jmxri</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

</project>
```

*	运行

```
EVENT : None
ls / ==> [zookeeper]
EVENT : NodeCreated
EVENT : NodeChildrenChanged
Create /node conan
get /node ==> conan
ls / ==> [node, zookeeper]

Process finished with exit code 0

```


