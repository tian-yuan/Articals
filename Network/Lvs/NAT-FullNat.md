来，我再写篇小水文。

服务器为了提高性能，通常会选择横向扩展，一般有2种做法：

- 前置DNS服务器，同一个域名（Virtual Service）对应不同的真实服务器（Real Server），解析域名的时候，DNS服务器会轮询返回不同的服务器，这样真正提供服务的，就是不同的机器，达到负载分担的目的。
- 前置负载分担（LB）设备。可以是专业的LB设备，也可以是linux服务器开启LVS（4层）或者Nginx（7层）。LVS的使用可以参考前面的一篇文章。

目前的发行版，内核默认会集成LVS module，可以自检下：

```
[root@lvs ~]# lsmod |grep ip_vs
ip_vs_wrr               2275  4 
ip_vs                 161155  8 ip_vs_wrr
ipv6                  323428  60 ip_vs,ip6t_REJECT,nf_conntrack_ipv6,nf_defrag_ipv6

```

社区的linux发行版的LVS提供的报文转发模式有三种：NAT/DR/TUNNEL。阿里云SLB在NAT基础上还支持了FULLNAT模式，该模式在一般的开源版本中不提供，需要打补丁重新编译内核。在了解FULLNAT之前，我们先来看看NAT模式。

![img](http://7xir15.com1.z0.glb.clouddn.com/snat+dnat.png)

如上图。NAT模式对入报文做了DNAT，即将报文的目的地址改为RS的地址，但源地址不变；RS上配置路由策略（如网关），出报文到了LVS设备上后做SNAT，即将报文的源地址改为LVS设备上的地址，目的地址不变。NAT模式的劣势是必须要在RS上配置路由策略（其实还可以解决一个很重要的场景，下文会提到）。

而FULLNAT，顾名思义，就是在出入两个方向均做了SNAT+DNAT。![img](http://7xir15.com1.z0.glb.clouddn.com/fullnat.png)如上图。FULLNAT模式对入报文做了DNAT+SNAT，即将报文的目的地址改为RS的地址，源地址改为LVS设备地址；RS上不需要配置路由策略，出报文到了LVS设备上后做SNAT+DNAT，即将报文的源地址改为LVS设备上的地址，目的地址改为真实的用户地址。

一般来说，我们不需要使用FULLNAT，但是有一种场景，必须使用FULLNAT（或者类似的技术）： 通常LVS是为了解决外部访问集群内部的问题，但是在我们的一个生产环境上，我们遇到了必须在集群内部的server1，向server2/server3（提供sysdb）写log的场景。 server2/server3对外提供了VIP，用户可以从集群外部通过LVS来访问，但是server1访问sysdb的时候，会有路由问题。server1发出的syn报文，经由LVS转发给了server2，而server2应答的syn+ack报文，由于syn报文的源地址是server1，而server1跟server2在同一局域网内，所以server1会直接将该报文转发给server1，而不经过LVS。 所以就会不通。

有了fullnat，syn报文经过LVS的处理以后，源地址改为LVS的LIP(Local IP)、目的地址改为了server2的地址，所以，对于server2来说，该syn请求就是LVS发起的，所以syn+ack报文还是会应答给LVS服务器；而应答报文再经过LVS处理，源地址改为LVS的EIP(External IP)，目的地址改为server1的地址，所以对于server1来说，请求得到了正确的应答，连接可以建立。

fullnat解决了集群内部互相访问的问题。在阿里内部，应该还有更广阔的应用（例如虚机之间通信）。不过，据说青云曾经指出fullnat的一个弊端，即对于Real Server来说，无法审计真实的客户端，只能向LVS（阿里叫做SLB）请求。

下一篇文章讲如何给内核打fullnat补丁。