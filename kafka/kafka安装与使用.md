#kafka使用简介

##kafka下载

	wget http://download.nus.edu.sg/mirror/apache/kafka/0.10.0.1/kafka_2.11-0.10.0.1.tgz
	tar -xvf kafka_2.11-0.10.0.1.tgz
	
	parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/kafka_2.11-0.10.0.1$ ls
	bin  config  libs  LICENSE  NOTICE  site-docs
	
	目录结构如上。

##启动kafka
	
	cat ./config/server.properties
	里面的配置信息不需要修改，注意里面的 zookeeper.connect=localhost:2181 ，表示 zookeeper 需要开启来，当前可以创建单机版的 zookeeper；
	
	./bin/kafka-server-start.sh config/server.properties
	启动 kafka
	
##Create a topic

```
Let's create a topic named "test" with a single partition and only one replica:

> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

We can now see that topic if we run the list topic command:

> bin/kafka-topics.sh --list --zookeeper localhost:2181
test

Alternatively, instead of manually creating topics you can also configure your brokers to auto-create topics when a non-existent topic is published to.
```		

##Send some messages

```
Let's create a topic named "test" with a single partition and only one replica:

> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

We can now see that topic if we run the list topic command:

> bin/kafka-topics.sh --list --zookeeper localhost:2181
test

Alternatively, instead of manually creating topics you can also configure your brokers to auto-create topics when a non-existent topic is published to.
```

```
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/kafka_2.11-0.10.0.1$ bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
Created topic "test".
parallels@ubuntu:/media/psf/UbuntuWorkspace/JavaDevelop/kafka_2.11-0.10.0.1$ bin/kafka-topics.sh --list --zookeeper localhost:2181
test
```

##Start a consumer


```
Kafka also has a command line consumer that will dump out messages to standard output.

> bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
This is a message
This is another message

If you have each of the above commands running in a different terminal then you should now be able to type messages into the producer terminal and see them appear in the consumer terminal.

All of the command line tools have additional options; running the command with no arguments will display usage information documenting them in more detail.
```

>如果创建多个 consumer，会接收到一样的消息！

##创建Java生产者客户端

```
import java.util.Date;
import java.util.Properties;

import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;

/**
 * 详细可以参考：https://cwiki.apache.org/confluence/display/KAFKA/0.8.0+Producer+Example
 * @author Fung
 *
 */
public class ProducerDemo {
    public static void main(String[] args) {
        int events=100;

        // 设置配置属性
        Properties props = new Properties();
        props.put("metadata.broker.list","10.211.55.3:9092");
        props.put("serializer.class", "kafka.serializer.StringEncoder");
        // key.serializer.class默认为serializer.class
        props.put("key.serializer.class", "kafka.serializer.StringEncoder");
        // 可选配置，如果不配置，则使用默认的partitioner
        props.put("partitioner.class", "com.rabbit.kafka.demo.PartitionerDemo");
        // 触发acknowledgement机制，否则是fire and forget，可能会引起数据丢失
        // 值为0,1,-1,可以参考
        // http://kafka.apache.org/08/configuration.html
        props.put("request.required.acks", "-1");
        ProducerConfig config = new ProducerConfig(props);

        // 创建producer
        Producer<String, String> producer = new Producer<String, String>(config);
        // 产生并发送消息
        long start=System.currentTimeMillis();
        for (long i = 0; i < events; i++) {
            long runtime = new Date().getTime();
            String ip = "192.168.2." + i;
            String msg = runtime + ",www.example.com," + ip;
            //如果topic不存在，则会自动创建，默认replication-factor为1，partitions为0
            KeyedMessage<String, String> data = new KeyedMessage<String, String>(
                    "page_visits", ip, msg);
            producer.send(data);
        }
        System.out.println("耗时:" + (System.currentTimeMillis() - start));
        // 关闭producer
        producer.close();
    }
}

////////////////////////////////////////////////

package com.rabbit.kafka.demo;

import kafka.producer.Partitioner;
import kafka.utils.VerifiableProperties;

public class PartitionerDemo implements Partitioner {
    public PartitionerDemo(VerifiableProperties props) {

    }

    public int partition(Object obj, int numPartitions) {
        int partition = 0;
        if (obj instanceof String) {
            String key=(String)obj;
            int offset = key.lastIndexOf('.');
            if (offset > 0) {
                partition = Integer.parseInt(key.substring(offset + 1)) % numPartitions;
            }
        }else{
            partition = obj.toString().length() % numPartitions;
        }

        return partition;
    }

}

///////////////////////////////////////////////

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com</groupId>
    <artifactId>rabbit.kafka.demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.10</artifactId>
            <version>0.8.2.1</version>
            <exclusions>
                <exclusion>
                    <artifactId>jmxri</artifactId>
                    <groupId>com.sun.jmx</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>jms</artifactId>
                    <groupId>javax.jms</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>jmxtools</artifactId>
                    <groupId>com.sun.jdmk</groupId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

</project>

```

>注意，运行时可能出现：Exception in thread "main" kafka.common.FailedToSendMessageException: Failed to send messages after 3 tries. 的错误，这个在我测试的环境下是由于 #advertised.listeners=PLAINTEXT://your.host.name:9092 没有设置，设置成 advertised.listeners=PLAINTEXT://10.211.55.3:9092 就可以了。


		