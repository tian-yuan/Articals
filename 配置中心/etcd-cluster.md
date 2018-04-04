## etcd集群搭建

2017年5月8日

[admin](http://www.361way.com/author/admin)

[发表评论](http://www.361way.com/etcd-cluster/5468.html#respond)

[阅读评论](http://www.361way.com/etcd-cluster/5468.html#comments)

### 一、etcd简介与应用场景

etcd 是一个分布式一致性k-v存储系统，可用于服务注册发现与共享配置，具有以下优点：1、简单 ： 相比于晦涩难懂的paxos算法，etcd基于相对简单且易实现的raft算法实现一致性，并通过gRPC提供接口调用；2、安全：支持TLS通信，并可以针对不同的用户进行对key的读写控制；3、高性能：10,000 /秒的写性能。其主要应用于服务注册发现以及共享配置。

#### 1、 服务注册与发现

![etcd-reg](http://www.361way.com/wp-content/uploads/2017/05/etcd-reg.png)

- 服务启动后向etcd注册，并上报自己的监听的端口以及当前的权重因子等信息，且对该信息设置ttl值。
- 服务在ttl的时间内周期性上报权重因子等信息。
- client端调用服务时向etcd获取信息，进行调用，同时监听该服务是否变化(通过watch方法实现)。
- 当新增服务时watch方法监听到变化，将服务加入代用列表，当服务挂掉时ttl失效，client端检测到变化，将服务踢出调用列表，从而实现服务的动态扩展。
- 另一方面，client端通过每次变化获取到的权重因子来进行client端的加权调用策略，从而保证后端服务的负载均衡。

#### **2、共享配置**

一般服务启动时需要加载一些配置信息，如数据库访问地址，连接配置，这些配置信息每个服务都差不多，如果通过读取配置文件进行配置会存在要写多份配置文件，且每次更改时这些配置文件都要更改，且更改配置后，需要重启服务后才能生效，这些无疑让配置极不灵活，如果将配置信息放入到etcd中，程序启动时进行加载并运行，同时监听配置文件的更改，当配置文件发生更改时，自动将旧值替换新值，这样无疑简化程序配置，更方便于服务部署。

### 二、单机模式运行

默认在centos7的yum源里已集成了etcd包，可以通过yum进行安装。也可以去github上下载二进制包：<https://github.com/coreos/etcd/tags>，这里我选择的yum直接安装的。启用etcd服务命令如下：

```
# systemctl start etcd 
```

进行如下测试etcd的可用性：

```
[root@etcd1 ~]# etcdctl set site www.361way.comwww.361way.com[root@etcd1 ~]# etcdctl get sitewww.361way.com
```

从上面可以看到可以进行k/v值的设置和获取。不过单机模式一般很少使用。

### 三、集群模式说明

这里的集群模式是指完全集群模式，当然也可以在单机上通过不同的端口，部署伪集群模式，只是那样做只适合测试环境，生产环境考虑到可用性的话需要将etcd实例分布到不同的主机上，这里集群搭建有三种方式，分布是静态配置，etcd发现，dns发现。默认配置运行etcd，监听本地的2379端口，用于与client端交互，监听2380用于etcd内部交互。etcd启动时，集群模式下会用到的参数如下：

```
--nameetcd集群中的节点名，这里可以随意，可区分且不重复就行--listen-peer-urls监听的用于节点之间通信的url，可监听多个，集群内部将通过这些url进行数据交互(如选举，数据同步等)--initial-advertise-peer-urls建议用于节点之间通信的url，节点间将以该值进行通信。--listen-client-urls监听的用于客户端通信的url,同样可以监听多个。--advertise-client-urls建议使用的客户端通信url,该值用于etcd代理或etcd成员与etcd节点通信。--initial-cluster-token etcd-cluster-1节点的token值，设置该值后集群将生成唯一id,并为每个节点也生成唯一id,当使用相同配置文件再启动一个集群时，只要该token值不一样，etcd集群就不会相互影响。--initial-cluster也就是集群中所有的initial-advertise-peer-urls 的合集--initial-cluster-state new新建集群的标志，初始化状态使用 new，建立之后改此值为 existing
```

主机规划如下：

| name   | IP             |
| ------ | -------------- |
| etcd01 | 192.168.122.51 |
| etcd02 | 192.168.122.52 |
| etcd03 | 192.168.122.53 |

注意，这里的name是etcd集群里使用的名字，不是主机名，当然和主机名一致也是没关系的。

### 四、静态模式

如果只是测试，这里建议使用二进制包进行测试。因为源码包编译的，使用etcd命令执行时加上的参数会被配置文件/etc/etcd/etcd.conf覆盖。直接二进制包的不会，如果是现网使用yum包就比较推荐了。分别在三个节点上使用如下命令启动：

```
#节点1：./etcd --name etcd01 --initial-advertise-peer-urls http://192.168.122.51:2380 \  --listen-peer-urls http://0.0.0.0:2380 \  --listen-client-urls http://0.0.0.0:2379 \  --advertise-client-urls http://192.168.122.51:2379 \  --initial-cluster-token etcd-cluster-1 \  --initial-cluster etcd01=http://192.168.122.51:2380,etcd02=http://192.168.122.52:2380,etcd03=http://192.168.122.53:2380 \  --initial-cluster-state new#节点2./etcd --name etcd02 --initial-advertise-peer-urls http://192.168.122.52:2380 \--listen-peer-urls http://0.0.0.0:2380 \--listen-client-urls http://0.0.0.0:2379 \--advertise-client-urls http://192.168.122.52:2379 \--initial-cluster-token etcd-cluster-1 \--initial-cluster etcd01=http://192.168.122.51:2380,etcd02=http://192.168.122.52:2380,etcd03=http://192.168.122.53:2380 \--initial-cluster-state new#节点3./etcd --name etcd03 --initial-advertise-peer-urls http://192.168.122.53:2380 \--listen-peer-urls http://0.0.0.0:2380 \--listen-client-urls http://0.0.0.0:2379 \--advertise-client-urls http://192.168.122.53:2379 \--initial-cluster-token etcd-cluster-1 \--initial-cluster etcd01=http://192.168.122.51:2380,etcd02=http://192.168.122.52:2380,etcd03=http://192.168.122.53:2380 \--initial-cluster-state new
```

启动成功后，可以使用如下命令查看（启动集群后，将会进入集群选举状态。看到isLeader为true为就是选举出的leader节点）：

```
[root@etcd1 ~]# etcdctl member list692a38059558446: name=etcd03 peerURLs=http://192.168.122.53:2380 clientURLs=http://192.168.122.53:2379 isLeader=true1b1ca7d3774cbbb9: name=etcd02 peerURLs=http://192.168.122.52:2380 clientURLs=http://192.168.122.52:2379 isLeader=falsecb75efb850ec8c2a: name=etcd01 peerURLs=http://192.168.122.51:2380 clientURLs=http://192.168.122.51:2379 isLeader=false[root@etcd1 ~]# etcdctl cluster-healthmember 692a38059558446 is healthy: got healthy result from http://192.168.122.53:2379member 1b1ca7d3774cbbb9 is healthy: got healthy result from http://192.168.122.52:2379member cb75efb850ec8c2a is healthy: got healthy result from http://192.168.122.51:2379
```

如果使用yum包安装的，需要修改etcd.conf配置文件如下：

```
[root@etcd1 ~]# grep -v ^# /etc/etcd/etcd.confETCD_NAME=etcd01ETCD_DATA_DIR="/var/lib/etcd/default.etcd"ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.122.51:2380"ETCD_INITIAL_CLUSTER="etcd01=http://192.168.122.51:2380,etcd02=http://192.168.122.52:2380,etcd03=http://192.168.122.53:2380"ETCD_INITIAL_CLUSTER_STATE="existing"ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster2"ETCD_ADVERTISE_CLIENT_URLS="http://192.168.122.51:2379"
```

修改完成后，还需要修改/usr/lib/systemd/system/etcd.service文件内容如下：

```
[Unit]Description=Etcd ServerAfter=network.targetAfter=network-online.targetWants=network-online.target[Service]Type=notifyWorkingDirectory=/var/lib/etcd/EnvironmentFile=-/etc/etcd/etcd.confUser=etcd# set GOMAXPROCS to number of processorsExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/bin/etcd \--name=\"${ETCD_NAME}\" \--data-dir=\"${ETCD_DATA_DIR}\" \--listen-peer-urls=\"${ETCD_LISTEN_PEER_URLS}\" \--advertise-client-urls=\"${ETCD_ADVERTISE_CLIENT_URLS}\" \--initial-cluster-token=\"${ETCD_INITIAL_CLUSTER_TOKEN}\" \--initial-cluster=\"${ETCD_INITIAL_CLUSTER}\"  \--initial-cluster-state=\"${ETCD_INITIAL_CLUSTER_STATE}\" \--listen-client-urls=\"${ETCD_LISTEN_CLIENT_URLS}\""Restart=on-failureLimitNOFILE=65536[Install]WantedBy=multi-user.target
```

配置过程中可能遇到的问题有两个。

#### 1、本地连接报错

内容如下：

```
Error: client: etcd cluster is unavailable or misconfigurederror #0: dial tcp 127.0.0.1:2379: getsockopt: connection refusederror #1: dial tcp 127.0.0.1:4001: getsockopt: connection refused
```

如果出现如上的错误，是因为ETCD_LISTEN_CLIENT_URLS参数没有配置http://127.0.0.1:2379而导致的，所以这里我使用了0.0.0.0代表了监控所有地址。

#### 2、context deadline exceeded

集群启动后，在通过etcdctl member list查看结果时，一直返回这个。后来发现和我使用的环境有关，因为测试不能上外网，这里配置了一个squid代理，在环境变量里有如下配置：

```
http_proxy=10.212.52.25:3128https_proxy=10.212.52.25:3128ftp_proxy=10.212.52.25:3128export http_proxy https_proxy ftp_proxy 
```

发现将这部分结果注释，重新启动etcd服务就好了。

### 五、动态配置

静态配置前提是在搭建集群之前已经提前知道各节点的信息，而实际应用中可能存在预先并不知道各节点ip的情况，这时可通过已经搭建的etcd来辅助搭建新的etcd集群。首先需要在已经搭建的etcd中创建用于发现的url，命令如下：

```
[root@etcd1 ~]# curl -X PUT http://192.168.122.51:2379/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83/_config/size -d value=3{"action":"set","node":{"key":"/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83/_config/size","value":"3","modifiedIndex":19,"createdIndex":19}}
```

如果没有搭建好的etcd集群用于注册和发现，可使用etcd公有服务来进行服务注册发现。公有etcd服务上创建用于发现的url为：

```
[root@etcd1 ~]# curl https://discovery.etcd.io/new?size=3https://discovery.etcd.io/a6a292c79fff6d25ef09243e3dfd2043
```

在三个节点上启动的命令分别如下：

```
#节点1./etcd --name infra0 --initial-advertise-peer-urls http://192.168.122.51:2380 \  --listen-peer-urls http://0.0.0.0:2380 \  --listen-client-urls http://0.0.0.0:2379 \  --advertise-client-urls http://192.168.122.51:2379 \  --discovery http://192.168.122.51:2379/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83#节点2./etcd --name infra1 --initial-advertise-peer-urls http://10.0.1.109:2380 \  --listen-peer-urls http://0.0.0.0:2380 \  --listen-client-urls http://0.0.0.0:2379 \  --advertise-client-urls http://10.0.1.109:2379 \ --discovery http://192.168.122.51:2379/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83#节点3./etcd --name infra2 --initial-advertise-peer-urls http://10.0.1.110:2380 \  --listen-peer-urls http://0.0.0.0:2380 \  --listen-client-urls http://0.0.0.0:2379 \  --advertise-client-urls http://10.0.1.110:2379 \  --discovery http://192.168.122.51:2379/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83
```

同样，如果使用yum 包安装的，需要修改etcd.conf文件和etcd.service文件。

六、DNS发现

这个配置方法我没测试过，这里直接拿官方给的样例列出下。

dns 发现主要通过dns服务来记录集群中各节点的域名信息，各节点到dns服务中获取相互的地址信息，从而建立集群。etcd各节点通过--discovery-serv配置来获取域名信息,节点间将获取以下域名下的各节点域名，

- _etcd-server-ssl._tcp.example.com
- _etcd-server._tcp.example.com

如果_etcd-server-ssl._tcp.example.com下有值表示节点间将使用ssl协议，相反则使用非ssl。

- _etcd-client._tcp.example.com
- _etcd-client-ssl._tcp.example.com

另一方面，client端将获取以上域名下的节点域名，用于client端与etcd通信，ssl与非ssl的使用取决于以上那个域名下有值。

创建dns记录

```
$ dig +noall +answer SRV _etcd-server._tcp.example.com_etcd-server._tcp.example.com. 300 IN  SRV  0 0 2380 infra0.example.com._etcd-server._tcp.example.com. 300 IN  SRV  0 0 2380 infra1.example.com._etcd-server._tcp.example.com. 300 IN  SRV  0 0 2380 infra2.example.com.$ dig +noall +answer SRV _etcd-client._tcp.example.com_etcd-client._tcp.example.com. 300 IN SRV 0 0 2379 infra0.example.com._etcd-client._tcp.example.com. 300 IN SRV 0 0 2379 infra1.example.com._etcd-client._tcp.example.com. 300 IN SRV 0 0 2379 infra2.example.com.$ dig +noall +answer infra0.example.com infra1.example.com infra2.example.cominfra0.example.com.  300  IN  A  10.0.1.111infra1.example.com.  300  IN  A  10.0.1.109infra2.example.com.  300  IN  A  10.0.1.110
```

然后启动各个节点

```
$ etcd --name infra0 \--discovery-srv example.com \--initial-advertise-peer-urls http://infra0.example.com:2380 \--initial-cluster-token etcd-cluster-1 \--initial-cluster-state new \--advertise-client-urls http://infra0.example.com:2379 \--listen-client-urls http://infra0.example.com:2379 \--listen-peer-urls http://infra0.example.com:2380$ etcd --name infra1 \--discovery-srv example.com \--initial-advertise-peer-urls http://infra1.example.com:2380 \--initial-cluster-token etcd-cluster-1 \--initial-cluster-state new \--advertise-client-urls http://infra1.example.com:2379 \--listen-client-urls http://infra1.example.com:2379 \--listen-peer-urls http://infra1.example.com:2380$ etcd --name infra2 \--discovery-srv example.com \--initial-advertise-peer-urls http://infra2.example.com:2380 \--initial-cluster-token etcd-cluster-1 \--initial-cluster-state new \--advertise-client-urls http://infra2.example.com:2379 \--listen-client-urls http://infra2.example.com:2379 \--listen-peer-urls http://infra2.example.com:2380
```

参考页面：

[github etcd集群配置页](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/clustering.md)