**Load Balancer(负载均衡器)：**
**Load Balancer**是整个集群系统的前端，负责把客户请求转发到Real Server上。Load Balancer通过Ldirectord监测各Real Server的健康状况。在Real Server不可用时把它从群中剔除，恢复时重新加入。
**Backup**是备份Load Balancer，当Load Balancer不可用时接替它，成为实际的Load Balancer。
**Server Array(服务器群)：**
**Server Array**是一组运行实际应用服务的机器，比如WEB, Mail, FTP, DNS, Media等等。在实际应用中，Load Balancer和Backup也可以兼任Real Server的角色。以下的测试就是一台服务器既担任了LVSserver,同时也是realserver节点.
**Shared Storage(共享存储)：**
**Shared Storage**为所有Real Server提供共享存储空间和一致的数据内容。
**Director**: 前端负载均衡器即运行LVS服务可以针对web、ftp、cache、mms甚至mysql等服务做load balances。
**RealServer**: 后端需要负载均衡的服务器，可以为各类系统，Linux、Solaris、Aix、BSD、Windows都可，甚至Director本身也可以作为 RealServer使用.
**LVS( Linux Virtual Server)**,Linux下的负载均衡器，支持**LVS-NAT、 LVS-DR、LVS-TUNL**三种不同的方式

　　**nat**用的不是很多，主要用的是DR、TUNL方式。

　　**DR**方式适合所有的RealServer同一网段下，即接在同一个交换机上.

　　**TUNL**方式就对于RealServer的位置可以任意了，完全可以跨地域、空间，只要系统支持Tunnel就可以,方便以后扩充的话直接Tunl方式即可

## 基础知识介绍

**1、LVS基础及介绍**

LVS是Linux Virtual Server的简写，意即Linux虚拟服务器，是一个虚拟的服务器集群系统。本项目在1998年5月由**章文嵩（目前就职于阿里）**博士成立，是中国国内最早出现的自由软件项目之一。
目前有三种IP负载均衡技术（VS/NAT、VS/TUN和VS/DR）；

十种调度算法（rrr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq）
【参考资料:】
1)官方中文参考资料: [http://www.linuxvirtualserver.org/zh/index.html](http://www.linuxvirtualserver.org/zh/index.html?spm=a2c4e.11153940.blogcont327438.9.5c1a683bxzHzSO)
2)LinuxTone 相关LVS技术档汇总: <http://bbs.linuxtone.org/thread-1191-1-1.html>
**2、 LVS 三种IP负载均衡技术对比**

三种IP负载均衡技术的优缺点归纳在下表中：

|                | VS/NAT        | VS/TUN     | VS/DR          |
| -------------- | ------------- | ---------- | -------------- |
| Server         | any           | Tunneling  | Non-arp device |
| server network | private       | LAN/WAN    | LAN            |
| server number  | low (10~20)   | High (100) | High (100)     |
| server gateway | load balancer | own router | Own router     |

【注】 以上三种方法所能支持最大服务器数目的估计是假设调度器使用100M网卡，调度器的硬件配置与后端服务器的硬件配置相同，而且是对一般Web服务。使用更 高的硬件配置（如千兆网卡和更快的处理器）作为调度器，调度器所能调度的服务器数量会相应增加。当应用不同时，服务器的数目也会相应地改变。所以，以上数 据估计主要是为三种方法的伸缩性进行量化比较。
**3、LVS目前实现的几种调度算法**

IPVS在内核中的负载均衡调度是以连接为粒度的。在HTTP协议（非持久）中，每个对象从WEB服务器上获取都需要建立一个TCP连接，同一用户的不同请求会被调度到不同的服务器上，所以这种细粒度的调度在一定程度上可以避免单个用户访问的突发性引起服务器间的负载不平衡。
在内核中的连接调度算法上，IPVS已实现了以下十种调度算法：
\* 轮叫调度（Round-Robin Scheduling）
* **加权轮叫调度（Weighted Round-Robin Scheduling）**
\* 最小连接调度（Least-Connection Scheduling）
\* 加权最小连接调度（Weighted Least-Connection Scheduling）
\* 基于局部性的最少链接（Locality-Based Least Connections Scheduling）
\* 带复制的基于局部性最少链接（Locality-Based Least Connections with Replication Scheduling）
\* 目标地址散列调度（Destination Hashing Scheduling）
\* 源地址散列调度（Source Hashing Scheduling）
\* 最短预期延时调度（Shortest Expected Delay Scheduling）
\* 不排队调度（Never Queue Scheduling）
对应: rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq,
Ldirecotrd配置选项及ipvsadm使用参数.

| ldirectord配置选项 | ipvsadm使用的参数 | ipvsadm -L的输出 | LVS转发方法 |
| -------------- | ------------ | ------------- | ------- |
| gate           | -g           | Route         | LVS-DR  |
| ipip           | -i           | Tunnel        | LVS-TUN |
| masq           | -m           | Masq          | LVS-NAT |

**4、集群架构时我们应该采用什么样的调度算法?**
在一般的网络服务（如HTTP和Mail Service等）调度中，我会使用**加权最小连接调度**wlc或者**加权轮叫调度wrr算法**。
基于局部性的最少链接LBLC和带复制的基于局部性最少链接LBLCR主要适用于Web Cache集群。
目标地址散列调度和源地址散列调度是用静态映射方法，可能主要适合防火墙调度。
最短预期延时调度SED和不排队调度NQ主要是对处理时间相对比较长的网络服务。
其实，它们的适用范围不限于这些。我想最好参考内核中的连接调度算法的实现原理，看看那种调度方法适合你的应用。
**5、LVS的ARP问题**
2.4.x kernels:
Hidden Patch
arptable
iptables
2.6.x kernels: （关闭arp查询响应请求）
net.ipv4.conf.eth0.arp_ignore = 1
net.ipv4.conf.eth0.arp_announce = 2
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
arping tools

## 二、基础知识及一些要点

1、InActConn并不代表错误连接，它是指不活跃连接(Inactive Connections)，
我们将处于TCP ESTABLISH状态以外的连接都称为不活跃连接，例如处于SYN_RECV状态的连接，处于TIME_WAIT状态的连接等。
2、用四个参数来**关闭arp查询响应请求**：
echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
3、ipvsadm -L -n --stats
Prot LocalAddress:Port Conns InPkts OutPkts InBytes OutBytes
连接数 输入包 输出包 输入流量 输出流量
4、注意事项:
1）在LVS方案中，虚拟ip地址与普通网络接口大大不同，这点需要特别注意。
虚拟ip地址的广播地址是它本身，子网掩码是255.255.255.255。 为什么要这样呢？因为有若干机器要使用同一个ip地址，
用本身做广播地址和把子网掩码设成4个255就不会造成ip地址冲突了,否则lvs将不能正常转发访问请求。
2）假如两台VS之间使用的互备关系，那么当一台VS接管LVS服务时，可能会网络不通，这时因为路由器的MAC缓存表里关于vip这个地址的MAC地 址还是被替换的VS的MAC，有两种解决方法，一种是修改新VS的MAC地址，另一种是使用send_arp 命令（piranha软件包里带的一个小工具） 格式如下：
send_arp:
send_arp [-i dev] src_ip_addr src_hw_addr targ_ip_addr tar_hw_addr
这个命令不一定非要在VS上执行，只+要在同一VLAN即可。
/sbin/arping -f -q -c 5 -w 5 -I eth0 -s WEBVIP−UWEBVIP−UGW
5.Virtual Server via Direct Routing（VS/DR）
VS/DR通过改写请求报文的MAC地址，将请求发送到真实服务器，而真实服务器将响应直接返回给客户。同VS/TUN技术一样，VS/DR技术可极大地提高集群系统的伸缩性。这种方法没有IP隧道的开销，对集群中的真实服务器也没有必须支持IP隧道协议的要求，但是要求调度器与真实服务器都有一块网卡连在同一物理网段上。
**6. LVS 经验:**
1). LVS调度的最小单位是“连接”。
2). 当apache的KeepAlive被设置成Off时，“连接”才能被较均衡的调度。
3). 在不指定-p参数时，LVS才真正以“连接”为单位按“权值”调度流量。
4). 在指定了-p参数时，则一个client在一定时间内，将会被调度到同一台RS。
5). 可以通过”ipvsadm ?set tcp tcpfin udp”来调整TCP和UDP的超时，让连接淘汰得快一些。
6). 在NAT模式时，RS的PORT参数才有意义。
7). DR和TUN模式时，InActConn 是没有意义的(Thus the count in the InActConn column for LVS-DR, LVS-Tun is
inferred rather than real.)
/sbin/arping -f -q -c 5 -w 5 -I eth0 -s WEBVIP−UWEBVIP−UGW

## 三、LVS 性能调优

Least services in System or Compile kernel.
Performace Tuning base LVS:
LVS self tuning( ipvsadm Timeout (tcp tcpfin udp)).
ipvsadm -Ln --timeout
Timeout (tcp tcpfin udp): 900 120 300
ipvsadm --set tcp tcpfin udp
Improving TCP/IP performance
net.ipv4.tcp_tw_recyle=1
net.ipv4.tcp_tw_reuse=1
net.ipv4.tcp_max_syn_backlog=8192
net.ipv4.tcp_keepalive_time=1800
net.ipv4.tcp_fin_timeout=30
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 65536 16777216
net.core.netdev_max_backlog=3000

## **1 通过NAT实现虚拟服务器（VS/NAT）**

　　NAT的工作原理是报文头（目标地址、源地址和端口等）被正确改写后，客户相信它们连接一个IP地址，而不同IP地址的服务器组也认为它们是与客户直接相连的。

　　可以用NAT方法将不同IP地址的并行网络服务变成在一个IP地址上的一个虚拟服务。

　　客户通过Virtual IP Address（虚拟服务的IP地址）访问网络服务时，请求报文到达调度器，调度器根据连接调度算法从一组真实服务器中选出一台服务器，将报文的目标地址Virtual IP Address改写成选定服务器的地址，报文的目标端口改写成选定服务器的相应端口，最后将修改后的报文发送给选出的服务器。同时，调度器在连接Hash表中记录这个连接，当这个连接的下一个报文到达时，从连接Hash表中可以得到原选定服务器的地址和端口，进行同样的改写操作，并将报文传给原选定的服务器。当来自真实服务器的响应报文经过调度器时，调度器将报文的源地址和源端口改为Virtual IP Address和相应的端口，再把报文发给用户。

　　在连接上引入一个状态机，不同的报文会使得连接处于不同的状态，不同的状态有不同的超时值。在TCP连接中，根据标准的TCP有限状态机进行状态迁移；在UDP中，我们只设置一个UDP状态。不同状态的超时值是可以设置的，在缺省情况下，SYN状态的超时为1分钟，ESTABLISHED状态的超时为15分钟，FIN状态的超时为1分钟；UDP状态的超时为5分钟。当连接终止或超时，调度器将这个连接从连接Hash表中删除。

## **2 通过IP隧道实现虚拟服务器（VS/TUN）**

　　大多数Internet服务都有这样的特点：请求报文较短而响应报文往往包含大量的数据。如果能将请求和响应分开处理，即在负载调度器中只负责调度请求而响应直接返回给客户，将极大地提高整个集群系统的吞吐量。

　　调度器根据各个服务器的负载情况，动态地选择一台服务器，将请求报文封装在另一个IP报文中，再将封装后的IP报文转发给选出的服务器；服务器收到报文后，先将报文解封获得原来目标地址为VIP的报文，服务器发现VIP地址被配置在本地的IP隧道设备上，所以就处理这个请求，然后根据路由表将响应报文直接返回给客户。

　　响应报文根据服务器的路由表直接返回给客户，而不经过负载调度器，所以负载调度器只处于从客户到服务器的半连接中

## **3 通过直接路由实现虚拟服务器（VS/DR）**

　　VS/DR利用大多数Internet服务的非对称特点，负载调度器中只负责调度请求，而服务器直接将响应返回给客户，可以极大地提高整个集群系统的吞吐量

　　在VS/DR中，调度器根据各个服务器的负载情况，动态地选择一台服务器，不修改也不封装IP报文，而是将数据帧的MAC地址改为选出服务器的MAC地址，再将修改后的数据帧在与服务器组的局域网上发送。因为数据帧的MAC地址是选出的服务器，所以服务器肯定可以收到这个数据帧，从中可以获得该IP报文。当服务器发现报文的目标地址VIP是在本地的网络设备上，服务器处理这个报文，然后根据路由表将响应报文直接返回给客户。

|                | VS/NAT       | VS/TUN     | VS/DR          |
| -------------- | ------------ | ---------- | -------------- |
| Server         | any          | Tuneling   | Non-arp device |
| server number  | low10-20     | high100    | high100        |
| server gateway | loadbalancer | own router | own router     |
| server network | private      | lan/wan    | lan            |

**NAT **

​    当服务器数目变多时，调度器成为瓶颈

**IP Tuneling **

​    术对服务器有要求，即所有的服务器必须支持“IP Tunneling”者“IPEncapsulation ”协议

**DR **

​    跟VS/TUN相比，这种方法没有IP隧道的开销，但是要求负载调度器与实际服务器都有一块网卡连在同一物理网段上，服务器网络设备（或者设备别名）不作ARP响应，或者能将报文重定向（Redirect）到本地的Socket端口上。