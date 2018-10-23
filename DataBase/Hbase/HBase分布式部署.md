#### HBase 分布式部署

##### HDFS 分布式部署

参见 《HDFS分布式部署.md》

由于 HDFS 的 namenode 是单点的，一般需要加 HA 以保证高可用；

##### HBase 部署

* 修改 hbase-site.xml

```
<configuration>
<property>
  <name>hbase.tmp.dir</name>
  <value>/root/tools/hbase-2.1.0/daba/hbase</value>
</property>
<property>
  <name>hbase.rootdir</name>
  <value>hdfs://node-master:9000/hbase</value>
</property>
<property>
  <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>
<property>
  <name>hbase.zookeeper.quorum</name>
  <value>node-master:2181,node1:2181,node2:2181</value>
</property>
<property>
  <name>hbase.zookeeper.property.clientPort</name>
  <value>2181</value>
</property>
<property>
  <name>hbase.zookeeper.property.dataDir</name>
  <value>/root/tools/hbase-2.1.0/daba/zookeeper</value>
  <description>property from zoo.cfg,the directory where the snapshot is stored</description>
</property>
</configuration>

```

* 修改 regionservers

```
node-master
node1
node2
```

> 注意，每个节点都需要修改

* 调用 hbase_start.sh

#### 问题

* Master is initializing

```
create 'tsdb-tree',
  {NAME => 't', VERSIONS => 1, COMPRESSION => 'NONE', BLOOMFILTER => 'ROW'}

ERROR: org.apache.hadoop.hbase.PleaseHoldException: Master is initializing
	at org.apache.hadoop.hbase.master.HMaster.checkInitialized(HMaster.java:2846)
	at org.apache.hadoop.hbase.master.HMaster.createTable(HMaster.java:1817)
	at org.apache.hadoop.hbase.master.MasterRpcServices.createTable(MasterRpcServices.java:609)
	at org.apache.hadoop.hbase.shaded.protobuf.generated.MasterProtos$MasterService$2.callBlockingMethod(MasterProtos.java)
	at org.apache.hadoop.hbase.ipc.RpcServer.call(RpcServer.java:409)
	at org.apache.hadoop.hbase.ipc.CallRunner.run(CallRunner.java:130)
	at org.apache.hadoop.hbase.ipc.RpcExecutor$Handler.run(RpcExecutor.java:324)
	at org.apache.hadoop.hbase.ipc.RpcExecutor$Handler.run(RpcExecutor.java:304)
```

重启一下 hbase 恢复，具体原因？