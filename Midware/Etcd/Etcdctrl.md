#### etcd 使用



```
➜  etcd git:(master) ./bin/etcdctl put foo bar
rm 3                                                                                                  No help topic for 'put'
➜  etcd git:(master) ETCDCTL_API=3 ./bin/etcdctl put foo bar
OK
➜  etcd git:(master) ETCDCTL_API=3 ./bin/etcdctl get foo
foo
bar
➜  etcd git:(master)
```

注意，上面 No help topic for 'put' 错误是由于没有加 ETCDCTL_API=3 的原因



#### 动态变更集群

* 第一步，直接使用 etcd 启动一个进程
* 第二步，动态加入一个节点

```
➜  etcdctl member add my_new_node_name http://localhost:3380
➜  etcd git:(master)./bin/etcd --name my_new_node_name --initial-advertise-peer-urls http://localhost:3380 \
  --listen-peer-urls http://localhost:3380 \
  --listen-client-urls http://127.0.0.1:3379 \
  --advertise-client-urls http://localhost:3379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster my_new_node_name=http://localhost:3380,default=http://localhost:2380 \
  --initial-cluster-state existing
```

* 第三步，检查集群状态

```
etcdctl cluste-health
```































