### Canal

​	Canal 的原理是模拟 Slave 向 Master 发送请求， Canal 解析 binglog，单不将解析结构持久化，而是保存在内存中，每次有客户端读取一次消息，就删除该消息。

#### 步骤

**配置Canal**参考https://github.com/alibaba/canal

【mysql配置】
1，配置参数

```
[mysqld]  
log-bin=mysql-bin #添加这一行就ok  
binlog-format=ROW #选择row模式  
server_id=1 #配置mysql replaction需要定义，不能和canal的slaveId重复  

登录数据库，查询现在的数据库（包括主备和slave）的 server_id：
mysql> show variables like 'server_id';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 22604 |
+---------------+-------+
1 row in set (0.00 sec)

Canal 是作为一个 mysql 的 slave 角色，所以 server_id 必须与当前 mysql 集群的 server_id 区分，保持唯一。
```



2，在mysql中 配置canal数据库管理用户，配置相应权限（repication权限）

```
CREATE USER canal IDENTIFIED BY 'canal';      
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON . TO 'canal'@'%';    
-- GRANT ALL PRIVILEGES ON . TO 'canal'@'%' ;    
FLUSH PRIVILEGES;

canal 如果需要监控所有数据库日志文件，那么需要对 sys 下面的表格也要有 select 权限：
show create table `sys`.`host_summary_by_file_io_type`;

比如下面的权限：

mysql> show grants for 'pushtest'@'%';

+-------------------------------------------------------------------------------+

| Grants for pushtest@%                                                         |

+-------------------------------------------------------------------------------+

| GRANT PROCESS, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'pushtest'@'%' |

| GRANT ALL PRIVILEGES ON `push-test`.* TO 'pushtest'@'%'                       |

| GRANT SELECT ON `sys`.* TO 'pushtest'@'%'                                     |

| GRANT SELECT ON `performance_schema`.* TO 'pushtest'@'%'                      |

| GRANT SELECT ON `mysql`.`event` TO 'pushtest'@'%'                             |

| GRANT SELECT ON `mysql`.`help_category` TO 'pushtest'@'%'                     |

| GRANT SELECT ON `mysql`.`slow_log` TO 'pushtest'@'%'                          |

| GRANT SELECT ON `mysql`.`time_zone_leap_second` TO 'pushtest'@'%'             |

| GRANT SELECT ON `mysql`.`proc` TO 'pushtest'@'%'                              |

| GRANT SELECT ON `mysql`.`time_zone_transition` TO 'pushtest'@'%'              |

| GRANT SELECT ON `mysql`.`time_zone_transition_type` TO 'pushtest'@'%'         |

| GRANT SELECT ON `mysql`.`general_log` TO 'pushtest'@'%'                       |

| GRANT SELECT ON `mysql`.`help_relation` TO 'pushtest'@'%'                     |

| GRANT SELECT ON `mysql`.`time_zone_name` TO 'pushtest'@'%'                    |

| GRANT SELECT ON `mysql`.`help_keyword` TO 'pushtest'@'%'                      |

| GRANT SELECT ON `mysql`.`func` TO 'pushtest'@'%'                              |

| GRANT SELECT ON `mysql`.`time_zone` TO 'pushtest'@'%'                         |

+-------------------------------------------------------------------------------+
```



【canal下载和配置】
1，下载canal https://github.com/alibaba/canal/releases  
2，解压

```
mkdir /tmp/canal  
tar zxvf canal.deployer-$version.tar.gz  -C /tmp/canal  
```



3，修改配置文件

```
vi conf/example/instance.properties  

#################################################  
## mysql serverId  
canal.instance.mysql.slaveId = 1234  

# position info，需要改成自己的数据库信息  
canal.instance.master.address = 127.0.0.1:3306   
canal.instance.master.journal.name =   
canal.instance.master.position =   
canal.instance.master.timestamp =   
 
#canal.instance.standby.address =   
#canal.instance.standby.journal.name =  
#canal.instance.standby.position =   
#canal.instance.standby.timestamp =   
 
# username/password，需要改成自己的数据库信息  
canal.instance.dbUsername = canal    
canal.instance.dbPassword = canal  
canal.instance.defaultDatabaseName =  
canal.instance.connectionCharset = UTF-8  
 
# table regex
canal.instance.filter.regex = .\..  
 
#################################################  

```



**注意**

* conf 下面的配置文件目录对应 conf/canal.properties 中的 canal.destinations
* conf 下面的配置会全部加载，如果存在多个配置文件夹，那么会起对个 instance



【canal启动和关闭】

1，启动

```
sh bin/startup.sh  
```



2，查看日志

```
vi logs/canal/canal.log  

2013-02-05 22:45:27.967 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## start the canal server.  
<pre name="user-content-code">2013-02-05 22:45:28.113 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[10.1.29.120:11111]  
2013-02-05 22:45:28.210 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## the canal server is running now ......  

```



具体instance的日志：

```
vi logs/example/example.log  

2013-02-05 22:50:45.636 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]  
2013-02-05 22:50:45.641 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [example/instance.properties]  
2013-02-05 22:50:45.803 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start CannalInstance for 1-example   
2013-02-05 22:50:45.810 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start successful....  

```



3，关闭

```
sh bin/stop.sh  
```



注意：

* 这里只需要配置好参数后，就可以直接运行
* Canal没有解析后的文件，不会持久化，只会保存到内存



**二、创建客户端**
参考https://github.com/alibaba/canal/wiki/ClientExample

其中一个是连接canal并操作的类，一个是redis的工具类，使用maven主要是依赖包的下载很方便。

![img](https://img-blog.csdn.net/20160204170841493?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

pom.xml

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.alibaba.otter</groupId>
  <artifactId>canal.sample</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <dependencies>
    <dependency>  
        <groupId>com.alibaba.otter</groupId>  
        <artifactId>canal.client</artifactId>  
        <version>1.0.12</version>  
    </dependency>  
    
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-test</artifactId>  
        <version>3.1.2.RELEASE</version>  
        <scope>test</scope>  
    </dependency>  
      
    <dependency>  
        <groupId>redis.clients</groupId>  
        <artifactId>jedis</artifactId>  
        <version>2.4.2</version>  
    </dependency>  
    
    </dependencies>
  <build/>
</project>
```



2，ClientSample代码

这里主要做两个工作，一个是循环从Canal上取数据，一个是将数据更新至Redis

```
package canal.sample;

import java.net.InetSocketAddress;  
import java.util.List;  

import com.alibaba.fastjson.JSONObject;
import com.alibaba.otter.canal.client.CanalConnector;  
import com.alibaba.otter.canal.common.utils.AddressUtils;  
import com.alibaba.otter.canal.protocol.Message;  
import com.alibaba.otter.canal.protocol.CanalEntry.Column;  
import com.alibaba.otter.canal.protocol.CanalEntry.Entry;  
import com.alibaba.otter.canal.protocol.CanalEntry.EntryType;  
import com.alibaba.otter.canal.protocol.CanalEntry.EventType;  
import com.alibaba.otter.canal.protocol.CanalEntry.RowChange;  
import com.alibaba.otter.canal.protocol.CanalEntry.RowData;  
import com.alibaba.otter.canal.client.*;  
 
public class ClientSample {  

   public static void main(String args[]) {  
	   
       // 创建链接，当前的 username 和 passward 没有使用到，权限认证需要配置其他系统进行处理
       CanalConnector connector = CanalConnectors.newSingleConnector(new InetSocketAddress(AddressUtils.getHostIp(),  
               11111), "example", "", "");  
       int batchSize = 1000;  
       try {  
           connector.connect();  
           connector.subscribe(".*\\..*");  
           connector.rollback();    
           while (true) {  
               Message message = connector.getWithoutAck(batchSize); // 获取指定数量的数据  
               long batchId = message.getId();  
               int size = message.getEntries().size();  
               if (batchId == -1 || size == 0) {  
                   try {  
                       Thread.sleep(1000);  
                   } catch (InterruptedException e) {  
                       e.printStackTrace();  
                   }  
               } else {  
                   printEntry(message.getEntries());  
               }  
 
               connector.ack(batchId); // 提交确认  
               // connector.rollback(batchId); // 处理失败, 回滚数据  
           }  
 
       } finally {  
           connector.disconnect();  
       }  
   }  
 
   private static void printEntry( List<Entry> entrys) {  
       for (Entry entry : entrys) {  
           if (entry.getEntryType() == EntryType.TRANSACTIONBEGIN || entry.getEntryType() == EntryType.TRANSACTIONEND) {  
               continue;  
           }  
 
           RowChange rowChage = null;  
           try {  
               rowChage = RowChange.parseFrom(entry.getStoreValue());  
           } catch (Exception e) {  
               throw new RuntimeException("ERROR ## parser of eromanga-event has an error , data:" + entry.toString(),  
                       e);  
           }  
 
           EventType eventType = rowChage.getEventType();  
           System.out.println(String.format("================> binlog[%s:%s] , name[%s,%s] , eventType : %s",  
                   entry.getHeader().getLogfileName(), entry.getHeader().getLogfileOffset(),  
                   entry.getHeader().getSchemaName(), entry.getHeader().getTableName(),  
                   eventType));  
 
           for (RowData rowData : rowChage.getRowDatasList()) {  
               if (eventType == EventType.DELETE) {  
            	   redisDelete(rowData.getBeforeColumnsList());  
               } else if (eventType == EventType.INSERT) {  
            	   redisInsert(rowData.getAfterColumnsList());  
               } else {  
                   System.out.println("-------> before");  
                   printColumn(rowData.getBeforeColumnsList());  
                   System.out.println("-------> after");  
                   redisUpdate(rowData.getAfterColumnsList());  
               }  
           }  
       }  
   }  
 
   private static void printColumn( List<Column> columns) {  
       for (Column column : columns) {  
           System.out.println(column.getName() + " : " + column.getValue() + "    update=" + column.getUpdated());  
       }  
   }  
   
	  private static void redisInsert( List<Column> columns){
		  JSONObject json=new JSONObject();
		  for (Column column : columns) {  
			  json.put(column.getName(), column.getValue());  
	       }  
		  if(columns.size()>0){
			  RedisUtil.stringSet("user:"+ columns.get(0).getValue(),json.toJSONString());
		  }
	   }
	  
	  private static  void redisUpdate( List<Column> columns){
		  JSONObject json=new JSONObject();
		  for (Column column : columns) {  
			  json.put(column.getName(), column.getValue());  
	       }  
		  if(columns.size()>0){
			  RedisUtil.stringSet("user:"+ columns.get(0).getValue(),json.toJSONString());
		  }
	  }
  
	   private static  void redisDelete( List<Column> columns){
		   JSONObject json=new JSONObject();
			  for (Column column : columns) {  
				  json.put(column.getName(), column.getValue());  
		       }  
			  if(columns.size()>0){
				  RedisUtil.delKey("user:"+ columns.get(0).getValue());
			  }
	   }

   
}  
```



3，RedisUtil代码

```
package canal.sample;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

public class RedisUtil {

	// Redis服务器IP
	private static String ADDR = "10.1.2.190";

	// Redis的端口号
	private static int PORT = 6379;

	// 访问密码
	private static String AUTH = "admin";

	// 可用连接实例的最大数目，默认值为8；
	// 如果赋值为-1，则表示不限制；如果pool已经分配了maxActive个jedis实例，则此时pool的状态为exhausted(耗尽)。
	private static int MAX_ACTIVE = 1024;

	// 控制一个pool最多有多少个状态为idle(空闲的)的jedis实例，默认值也是8。
	private static int MAX_IDLE = 200;

	// 等待可用连接的最大时间，单位毫秒，默认值为-1，表示永不超时。如果超过等待时间，则直接抛出JedisConnectionException；
	private static int MAX_WAIT = 10000;

	// 过期时间
	protected static int  expireTime = 60 * 60 *24;
	
	// 连接池
	protected static JedisPool pool;

	/**
	 * 静态代码，只在初次调用一次
	 */
	static {
		JedisPoolConfig config = new JedisPoolConfig();
		//最大连接数
		config.setMaxTotal(MAX_ACTIVE);
		//最多空闲实例
		config.setMaxIdle(MAX_IDLE);
		//超时时间
		config.setMaxWaitMillis(MAX_WAIT);
		//
		config.setTestOnBorrow(false);
		pool = new JedisPool(config, ADDR, PORT, 1000);
	}

	/**
	 * 获取jedis实例
	 */
	protected static synchronized Jedis getJedis() {
		Jedis jedis = null;
		try {
			jedis = pool.getResource();
		} catch (Exception e) {
			e.printStackTrace();
			if (jedis != null) {
				pool.returnBrokenResource(jedis);
			}
		}
		return jedis;
	}

	/**
	 * 释放jedis资源
	 * 
	 * @param jedis
	 * @param isBroken
	 */
	protected static void closeResource(Jedis jedis, boolean isBroken) {
		try {
			if (isBroken) {
				pool.returnBrokenResource(jedis);
			} else {
				pool.returnResource(jedis);
			}
		} catch (Exception e) {

		}
	}

	/**
	 *  是否存在key
	 * 
	 * @param key
	 */
	public static boolean existKey(String key) {
		Jedis jedis = null;
		boolean isBroken = false;
		try {
			jedis = getJedis();
			jedis.select(0);
			return jedis.exists(key);
		} catch (Exception e) {
			isBroken = true;
		} finally {
			closeResource(jedis, isBroken);
		}
		return false;
	}

	/**
	 *  删除key
	 * 
	 * @param key
	 */
	public static void delKey(String key) {
		Jedis jedis = null;
		boolean isBroken = false;
		try {
			jedis = getJedis();
			jedis.select(0);
			jedis.del(key);
		} catch (Exception e) {
			isBroken = true;
		} finally {
			closeResource(jedis, isBroken);
		}
	}

	/**
	 *  取得key的值
	 * 
	 * @param key
	 */
	public static String stringGet(String key) {
		Jedis jedis = null;
		boolean isBroken = false;
		String lastVal = null;
		try {
			jedis = getJedis();
			jedis.select(0);
			lastVal = jedis.get(key);
			jedis.expire(key, expireTime);
		} catch (Exception e) {
			isBroken = true;
		} finally {
			closeResource(jedis, isBroken);
		}
		return lastVal;
	}

	/**
	 *  添加string数据
	 * 
	 * @param key
	 * @param value
	 */
	public static String stringSet(String key, String value) {
		Jedis jedis = null;
		boolean isBroken = false;
		String lastVal = null;
		try {
			jedis = getJedis();
			jedis.select(0);
			lastVal = jedis.set(key, value);
			jedis.expire(key, expireTime);
		} catch (Exception e) {
			e.printStackTrace();
			isBroken = true;
		} finally {
			closeResource(jedis, isBroken);
		}
		return lastVal;
	}

	/**
	 *  添加hash数据
	 * 
	 * @param key
	 * @param field
	 * @param value
	 */
	public static void hashSet(String key, String field, String value) {
		boolean isBroken = false;
		Jedis jedis = null;
		try {
			jedis = getJedis();
			if (jedis != null) {
				jedis.select(0);
				jedis.hset(key, field, value);
				jedis.expire(key, expireTime);
			}
		} catch (Exception e) {
			isBroken = true;
		} finally {
			closeResource(jedis, isBroken);
		}
	}

}
```



>  注意：客户端的Jedis连接不同于项目里的Jedis连接需要Spring注解，直接使用静态方法就可以。

**运行**

* 运行canal服务端startup.bat / startup.sh
* 运行客户端程序



**注意**:

* 虽然canal服务端解析binlog后不会把数据持久化，但canal服务端会记录每次客户端消费的位置(客户端每次ack时服务端会记录pos点)。如果数据正在更新时，canal服务端挂掉，客户端也会跟着挂掉，mysql依然在插入数据，而redis则因为客户端的关闭而停止更新，造成mysql和redis的数据不一致。解决办法是，只要重启canal服务端和客户端就可以了，虽然canal服务端因为重启之前解析数据清空，但因为canal服务端记录的是客户端最后一次获取的pos点，canal服务端再从这个pos点开始解析，客户端更新至redis，以达到数据的一致。
* 如果只有一个canal服务端和一个客户端，肯定存在可用性低的问题，一种做法是用程序来监控canal服务端和客户端，如果挂掉，再重启；一种做法是多个canal服务端+zk，将canal服务端的配置文件放在zk，任何一个canal服务端挂掉后，切换到其他canal服务端，读到的配置文件的内容就是一致的（还有记录的消费pos点），保证业务的高可用，客户端可使用相同的做法。



#### 其他配置项

* 网络

```
# network config
canal.instance.network.receiveBufferSize = 1048576
canal.instance.network.sendBufferSize = 1048576
canal.instance.network.soTimeout = 30

netty 相关配置，可以适当设置大点
```



* 数据库 bin log 格式

```
# binlog format/image check
canal.instance.binlog.format = ROW,STATEMENT,MIXED 
canal.instance.binlog.image = FULL,MINIMAL,NOBLOB

mysql> show variables like 'binlog_format';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+
1 row in set (0.00 sec)
```



* 字符编码

```
canal.instance.connectionCharset=UTF-8

mysql> show variables like 'character_set_database';
+------------------------+---------+
| Variable_name          | Value   |
+------------------------+---------+
| character_set_database | utf8mb4 |
+------------------------+---------+
1 row in set (0.00 sec)

```



* 缓存大小

```
## memory store RingBuffer size, should be Math.pow(2,n)
canal.instance.memory.buffer.size = 1048576
## memory store RingBuffer used memory unit size , default 1kb
canal.instance.memory.buffer.memunit = 1024 

此为 canal 用于 bin log 的缓存
size = 1048576 指可以保存 1024 * 1024 条记录
```
