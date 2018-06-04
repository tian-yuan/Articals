# LVS负载均衡之工作原理说明（原理篇）

原创

清白之年

2016-02-26 00:09:33

评论(2)

6511人阅读

   说起lvs，不得不说说关于lvs的工作原理，通常来说lvs的工作方式有三种：nat模式（LVS/NAT),直接路由模式（ LVS/DR），ip隧道模式(LVS/TUN),不过据说还有第四种模式（FULL NAT），下面我们来介绍介绍关于lvs常用的三种工作模式说明

一. NAT模式（LVS/NAT）

**LVS负载均衡模式---NAT****模式原理**

 LVS-NAT模式:NAT用法本来是因为网络IP地址不足而把内部保留IP地址通过映射转换成公网地址的一种上网方式(原地址NAT)如果把NAT的过程稍微变化,就可以 成为负载均衡的一种方式原理其实就是把从客户端发来的IP包的IP头目的地址在DR上换成其中一台REALSERVER的IP地址并发至此 REALSERVER,而REALSERVER则在处理完成后把数据经过DR主机发回给客户端,DR在这个时候再把数据包的原IP地址改为DR接口上的 IP地址即可期间,无论是进来的流量,还是出去的流量,都必须经过DR

如下为lvs nat模式的示意说明图：

[![wKioL1bUEjPy1efOAACrHcSP91E705.png](http://s3.51cto.com/wyfs02/M00/7C/A1/wKioL1bUEjPy1efOAACrHcSP91E705.png)](http://s3.51cto.com/wyfs02/M00/7C/A1/wKioL1bUEjPy1efOAACrHcSP91E705.png)

CIP：192.168.10.13/24
VIP：192.168.10.100/24

DIR：eth0:192.168.1.100/24 Eth1:192.168.10.100/24

Real-server：192.168.1.10/24 和 192.168.1.11/24. 192.168.1.12/24（提供http服务）

整个请求过程示意：

1>client发送request到LVS的VIP上，VIP根据负载算法选择一个Real-server，并记录连接信息到hash表中，然后修改client的request的目的IP地址为Real-server的地址，将请求发给Real-server;

2> Real-server收到request包后，发现目的IP是自己的IP，于是处理请求，然后发送reply给LVS;

3> LVS收到reply包后，修改reply包的的源地址为VIP，发送给client;

4> 从client来的属于本次连接的包，查hash表，然后发给对应的Real-server。

5> 当client发送完毕，此次连接结束或者连接超时，那么LVS自动从hash表中删除此条记录。

以下为数据的封装过程

① client向目标vip发出请求，DiR接收。此时IP包头及数据帧头信息如下：

| src ip        | src port | dst ip         | dst port |
| ------------- | -------- | -------------- | -------- |
| 192.168.10.13 | 10011    | 192.168.10.100 | 80       |

② DIR根据负载均衡算法选择一台active的RS（RIP1），将此RIP1所在网卡的ip地址作为目标ip地址，并记录到hash表中，DIR讲请求包发送到局域网里。此时IP包头及数据帧头信息如下：

| src ip        | src port | dst ip       | dst port |
| ------------- | -------- | ------------ | -------- |
| 192.168.10.13 | 10011    | 192.168.1.10 | 80       |

③ RIP1(192.168.1.10)在局域网中收到这个包，拆开后发现目标IP与本地接口ip匹配，于是处理这个报文。随后重新封装报文，发送到局域网。此时IP包头及数据帧头信息如下：

| src ip       | src port | dst ip        | dst port |
| ------------ | -------- | ------------- | -------- |
| 192.168.1.10 | 80       | 192.168.10.13 | 10011    |

④ LVS(192.168.1.10)收到后端的Rip的reply包，利用SANT重新封装ip包首部将源ip修改为vip，再发送给客户端，此时IP包头信息如下:

| src ip         | src port | dst ip        | dst port |
| -------------- | -------- | ------------- | -------- |
| 192.168.10.100 | 80       | 192.168.10.13 | 10011    |

  LVS-NAT模式主要是将客户单发送过来的包目标地址更改为后端RIP的ip，主要把三层的信息做了更改，其他都不变，此配置下后端RIP不需要做任何配置，但必须保证RIP的网关必须为与RIP相连的DIR接口ip，因为LVS-NAT主要用到DNAT和SNAT原理。

**LVS负载均衡模式---NAT****模式特点**

(1)RS和DIP属于同一IP网络中网卡应该使用私网地址，且RS的网关要指向DIP；

(2)请求和响应报文都要经由director转发；极高负载的场景中，director可能会成为系统瓶颈；

(3)支持端口映射；

(4)RS可以使用任意操作系统（OS）；

(5)DIR需要两块网卡（属于典型的lan/wan）RIP和Director的必须有一块网卡在同一IP网络；

(6)vip需要配置在DIR接受客户端请求网卡上，且直接对外提供服务

优点：实现方便简单，也容易理解；

缺点：Director会称为一个优化的瓶颈，所有的报文都要经过Director，因此，负载后端RS的台数在10-20台左右，服务器性能而定，如果Director坏掉，后果很严重，不支持异地容灾；

二. DR模式（LVS/DR）

**LVS负载均衡模式---DR****模式原理**

  LVS-DR模式：每个Real Server上都有两个IP：VIP和RIP，但是VIP是隐藏的，就是不能提高解析等功能，只是用来做请求回复的源IP的，Director上只需要一个网卡，然后利用别名来配置两个IP：VIP和DIP，在DIR接收到客户端的请求后，DIR根据负载算法选择一台rs sever的网卡mac作为客户端请求包中的目标mac，通过arp转交给后端rs serve处理，后端再通过自己的路由网关回复给客户端

如下为lvs DR模式的示意说

[![wKiom1bTrTXCPxZFAACgh-_xgho912.png](http://s1.51cto.com/wyfs02/M02/7C/9C/wKiom1bTrTXCPxZFAACgh-_xgho912.png)](http://s1.51cto.com/wyfs02/M02/7C/9C/wKiom1bTrTXCPxZFAACgh-_xgho912.png)

CIP：192.168.1.13
VIP：192.168.1.100

DIR: 192.168.1.2

RS ：192.168.1.10; 192.168.1.11; 192.168.1.12（提供http服务）

整个请求过程示意：

这里假设CIP的mac地址为：00-50-56-C0-00-08 ，DIR的Eth0的mac地址为：00-50-56-C0-00-01, RIP1的mac地址为: D0-50-99-18-18-15。CIP在请求之前会发一个arp广播包，即请求“谁是VIP",由于所有的DIR和RIP都在一个物理网络中，而DIR和RIP都有vip地址，为了让请求发送到DIR上，所以必须让RIP不能响应CIP发出的arp请求（这也是为什么RIP上要把vip配置在lo口以及要仰制arp查询和响应）这时客户端就会将请求包发送给DIR，接下来就是DIR的事情了：

① client向目标vip发出请求，DiR接收。此时IP包头及数据帧头信息如下：

| src mac           | dst mac           | src ip       | src port | dst ip        | dst port |
| ----------------- | ----------------- | ------------ | -------- | ------------- | -------- |
| 00-50-56-C0-00-08 | 00-50-56-C0-00-01 | 192.168.1.13 | 10011    | 192.168.1.100 | 80       |

② DIR根据负载均衡算法选择一台active的RS（RIP1），将此RIP1所在网卡的mac地址作为目标mac地址，发送到局域网里。此时IP包头及数据帧头信息如下：

| src mac           | dst mac           | src ip       | src port | dst ip        | dst port |
| ----------------- | ----------------- | ------------ | -------- | ------------- | -------- |
| 00-50-56-C0-00-01 | D0-50-99-18-18-15 | 192.168.1.13 | 10011    | 192.168.1.100 | 80       |

③RIP1(192.168.1.10)在局域网中收到这个帧，拆开后发现目标IP(VIP)与本地匹配，于是处理这个报文。随后重新封装报文，发送到局域网。此时IP包头及数据帧头信息如下：

| src mac           | dst mac           | src ip        | src port | dst ip       | dst port |
| ----------------- | ----------------- | ------------- | -------- | ------------ | -------- |
| D0-50-99-18-18-15 | 00-50-56-C0-00-08 | 192.168.1.100 | 80       | 192.168.1.13 | 10011    |

  如果client与VS同一网段，那么client(192.168.10.13)将收到这个回复报文。如果跨了网段，那么报文通过gateway/路由器经由Internet返回给用户。在实际情况下，可能只有一个公网，其他都是内网，这时VIP绑定地址应该是公网那个ip，或者利用路由器静态NAT映射将公网与内网vip绑定也行。

*关于lvs-dr模式下一些疑问：*

**1. LVS/DR如何****处理请求报文的，会修改IP包内容吗？**

vs/dr本身不会关心IP层以上的信息，即使是端口号也是tcp/ip协议栈去判断是否正确，vs/dr本身主要做这么几个事：

①接收client的请求，根据你设定的负载均衡算法选取一台realserver的ip；

②以选取的这个ip对应的mac地址作为目标mac，然后重新将IP包封装成帧转发给这台RS；

③在hash table中记录连接信息。

vs/dr做的事情很少，也很简单，所以它的效率很高，不比硬件负载均衡设备差多少，数据包、数据帧的大致流向是这样的：client --> VS --> RS --> client

**2. RealServer为什么要在lo接口上配置VIP？在出口网卡上配置VIP可以吗？**

既然要让RS能够处理目标地址为vip的IP包，首先必须要让RS能接收到这个包。在lo上配置vip能够完成接收包并将结果返回client。不可以将VIP设置在出口网卡上,否则会响应客户端的arp request,造成client/gateway arp table紊乱，以至于整个load balance都不能正常工作。

**3. RealServer为什么要抑制arp帧？**

我们知道仰制arp帧需要在server上执行以下命令，如下:

```
echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
```

因为arp对逻辑口没有意义。实际上起作用的只有以下两条:

```
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
```

即对所有的物理网卡设置arp仰制。对仰制所有的物理网卡设置arp仰制是为了让CIP发送的请求顺利转交给DIR以及防止整个LVS环境arp表混乱，不然容易到导致整个lvs不能工作

备注:关于arp_ingnone和arp_announce的相关说明

*arp_announce ： INTEGER  默认为0*

意义：对网络接口上本地IP地址发出的ARP回应作出相应级别的限制（即用来限制是否使用发送端口的ip地址设置arp请求的源地址）

0 - (默认) 使用任意网络接口上（包括逻辑口）的任何本地地址

1 - 代表不使用ip包的源地址来设置arp请求的源地址，如果ip包的源地址和该端口的ip地址在相同的子网，那么使用ip包的源地址作为arp请求中源地址，否者使用“2”

2 - 代表不使用ip包的源地址来设置arp请求的源地址，而是由系统来选择最好的借口来发送

举例:

假设linux主机有A、B两块网卡，其对应的IP地址分别为IP_A、IP_B，对应的MAC地址为MAC_A、MAC_B，假设一个应用程序准备与外 部通信，它的socket绑定了源IP地址为IP_A，但是根据系统路由及相关设置，其通信数据包将会从B网卡发送，在发送数据包前，系统会通过网卡B发 送ARP请求数据包。如果我们将arp_announce的值设定为0，那该ARP请求数据包的发送方IP地址是IP_A，而发送方MAC地址为 MAC_B，这样就会在网络设备或对方主机的ARP地址表上留下IP_A与MAC_B的对应记录，但是实际正确的应该是IP_A对应MAC_A、IP_B 对应MAC_B，所以这可能会引起潜在的网络问题，具体问题和表现与网络的拓扑结构及网络配置有关。而如果我们将arp_announce设置为2，那在 发送ARP请求数据包时，发送方IP地址将不是IP_A，而是IP_B，这样就不会引起刚才所说的问题。

*arp_ignore ： INTEGER  默认为0*

含义:定义对目标地址为本地IP的ARP询问不同的应答模式

0 - (默认值): 回应任何网络接口上对任何本地IP地址的arp查询请求（不管该arp请求包中目标ip地址是不是为接收该arp请求接口上的ip）

举例:比如eth0=192.168.0.1/24,eth1=10.1.1.1/24,那么即使eth0收到来自10.1.1.2这样地址发起的对10.1.1.1 的arp查询也会回应--而原本这个请求该是出现在eth1上，也该有eth1回应的

1 - 只回应arp包中目标IP地址为该arp请求包进入的接口ip的arp查询请求

举例:比如eth0=192.168.0.1/24,eth1=10.1.1.1/24,那么即使eth0收到来自10.1.1.2这样地址发起的对192.168.0.1的查询会回答，而对10.1.1.1 的arp查询不会回应

2 - 只回应arp包中目标IP地址为该arp请求包进入的接口ip的arp查询请求,且arp包中发送者ip必须在该网络接口的子网段内

举例:比如eth0=192.168.0.1/24,eth1=10.1.1.1/24,eth1收到来自10.1.1.2发起的对192.168.0.1的查询不会回答，而对eth0收到192.168.0.2发起的对192.168.0.1的arp查询会回应

**4. LVS/DR load balancer（director）与RS为什么要在同一网段中？**

lvs/dr它是在数据链路层来实现的，即RIP必须能够接受到DIR的arp请求，如果不在同一网段则会隔离arp，这样arp请求就不能转发到指定的RIP上，所以director必须和RS在同一网段里面。

**5. 为什么director上eth0接口除了VIP另外还要配一个ip（即DIP）？**

如果是用了keepalived等工具做HA或者Load Balance,则在健康检查时需要用到DIP。 没有健康检查机制的HA或者Load Balance则没有存在的实际意义。

**6. LVS/DR ip_forward需要开启吗？**

不需要。因为director跟realserver是同一个网段，无需开启转发。

**7. director的vip的netmask一定要是255.255.255.255吗？**

lvs/dr里，director的vip的netmask 没必要设置为255.255.255.255，director的vip本来就是要像正常的ip地址一样对外通告的,不要搞得这么特殊.

**LVS负载均衡模式---DR****模式特点**

(1)各RIP 必须与 DIP 在同一个网络中(相同的广播域),叶居士

(2)RS 的 RIP 可以使用私有地址，也可以使用公网地址，以方便配置

(3)不支持支持端口映射；

(4)RS可以使用必须为uninx操作系统（OS）；且RS需要仰制arp，需要在loopback配置vip

(5)Director 仅负责处理入站请求，响应报文由 Realserver 直接发往客户端

(6)Realserver 不能将网关指向 DIP，而直接使用前端网关响应请求报文

优点：负载均衡器只负责将请求包分发给物理服务器，而物理服务器将应答包直接发给用户。所以，负载均衡器能处理很巨大的请求量，这种方式，一台负载均衡能为 超过100台的物理服务器服务，负载均衡器不再是系统的瓶颈。使用VS-DR方式，如果你的负载均衡器拥有100M的全双工网卡的话，就能使得整个 Virtual Server能达到1G的吞吐量。甚至更高

不足：但是，这种方式需要所有的DIR和RIP都在同一广播域；不支持异地容灾

总结：LVS-DR是三种模式中性能最高的一种模式，比LVS-NAT模式下负载的RS serve更多，通常在100台左右，对网络环境要求更高，也是日常应用的最多的一种工作模式

三. TUN模式（LVS/TUN）

**LVS负载均衡模式---TUN****模式原理**

  LVS-TUN模式：它的连接调度和管理与VS/NAT中的一样，利用ip隧道技术的原理，即在原有的客户端请求包头中再加一层IP Tunnel的包头ip首部信息，不改变原来整个请求包信息，只是新增了一层ip首部信息，再利用路由原理将请求发给RS server，不过要求的是所有的server必须支持"IPTunneling"或者"IP Encapsulation"协议

如下为lvs DR模式的示意说

![wKioL1bVcYCiB01-AACotW484Yw670.png](http://s2.51cto.com/wyfs02/M01/7C/AB/wKioL1bVcYCiB01-AACotW484Yw670.png)CIP：202.10.1.10/24
VIP：202.10.1.101/24

DIR: Eth1:202.10.1.100/24 Eth0:192.168.1.100/24

RS ：RIP1( Eth0:192.168.1.10/24  &&  Eth1:10.10.1.10/24（提供http服务）

整个请求过程示意：

  这里假设CIP的CIP地址为：202.10.1.100 ，DIR的Eth1的ip地址为：202.10.1.100， Eth0的ip地址为:192.168.1.100/24 ,RIP1的Eth0地址为:192.168.1.10/24,Eth1的ip地址为:10.10.10.10/24,下面的就讲讲请求细节：

① client向目标vip发出请求，DIR接收。此时IP包头及数据帧头信息如下：

| src ip      | src port | dst ip       | dst port |
| ----------- | -------- | ------------ | -------- |
| 202.10.1.10 | 10011    | 202.10.1.101 | 80       |

② DIR根据负载均衡算法选择一台active的RS（RIP1），利用ip tunnel技术将此RIP1所在网卡的ip地址作为目标ip地址，将DIP作为源地址重封装一层IP首部，并记录到hash表中，DIR将请求包发送给RIP1。此时IP包头及数据帧头信息如下：

| src ip        | dst ip       | src ip      | src port | dst ip       | dst port |
| ------------- | ------------ | ----------- | -------- | ------------ | -------- |
| 192.168.1.100 | 192.168.1.10 | 202.10.1.10 | 10011    | 202.10.1.101 | 80       |

③RIP1(192.168.1.10)收到DIR发过来的请求后，拆开后发现请求包中里面还有一层ip包头，并且该ip包头的目标IP(VIP)与本地loopback口地址匹配，于是处理这个报文。随后重新封装报文，通过自己的网关将响应报文发送给客户端，此时IP包头及数据帧头信息如下：

| src ip       | src port | dst ip      | dst port |
| ------------ | -------- | ----------- | -------- |
| 202.10.1.101 | 80       | 202.10.1.10 | 10011    |

  LVS/TUN模式就是利用ip tunnel技术原理，在不改变原有的ip包头首部信息的基础上再封装一层ip首部信息，再利用路由的原理将请求转交给后端RS server，所以所有的server都必须支持ip tunnel隧道 ，相比LVS/DR模式，LVS/TUN对网络的消耗比较大，因为要支持ip tunnel的开销，所以这也是为什么DR模式是三种模式中效率最高的一种模式

**LVS负载均衡模式---TUN****模式特点**

(1)各RIP 与 DIP 不需要在同一个网络中，这样一来lvs/tun就可以应用在夸网络跨机房环境集群环境中

(2)RS 的 RIP 可以使用私有地址，也可以使用公网地址，以方便配置

(3)不支持支持端口映射；

(4)RS可以使用必须为uninx操作系统（OS），需要在loopback配置vip，必须支持ip tunnel隧道，即 modprobe tun；modprobe ipip

(5)Director 仅负责处理入站请求，响应报文由 Realserver 直接发往客户端，也是属于半链接状态

(6)Realserver 不能将网关指向 DIP，而直接使用前端网关响应请求报文

优点：负载均衡器只负责将请求包分发给物理服务器，而物理服务器将应答包直接发给用户。所以，负载均衡器能处理很巨大的请求量，这种方式，一台负载均衡能为 超过100台的物理服务器服务，负载均衡器不再是系统的瓶颈。使用VS-TUN方式，如果你的负载均衡器拥有100M的全双工网卡的话，就能使得整个 Virtual Server能达到1G的吞吐量。

不足：但是，这种方式需要所有的服务器支持"IP Tunneling"(IP Encapsulation)协议；

总结:LVS-tun是三种模式中性能仅此于LVS/DR的一种模式，解决了LVS/NAT和LVS/DR这两种模式因调度器和RS因距离太远无法做到异地容灾局限性问题，而LVS/tun模式存在实质就是为了解决这一问题的，因为lvs-tun模式中调度器和各种RS可以不再同一网络，只要路由可达，就可以跨越千山万水，这样一来就可以做到异地灾备了
*关于lvs-dr模式下一些疑问：*

**1. LVS/DR load balancer（director）与RS一定要在同一网段中？**

在LVS/TUN模式中，DIR与RIP不是一定在同一网段，也可以夸网段，因为根据LVS/TUN模式的原理，DIR只是新增加一层ip包头信息，但需要注意的是：若DIR和RIP在同一网络（广播域）还是要做arp仰制，因为同一广播域中会曾在arp欺骗，导致arp表混乱

**2.LVS/TUN ip_forward需要开启吗？**

不需要。因为realserver走的是自己的路由，无需开启转发。

**3 L****VS/TUN rp_filter为什么设置为零（关闭反向过滤）？**

 在 Linux 接收网络数据包时，如果一个包的源地址与其网卡地址不相符，则可通过 rp_filter 选项禁止接受这类包。通过这个来方式 ip 欺骗。需要注意的是，如果你的机器有多个网卡，不同的网卡有不同的IP；或者单个网卡有不同的IP。你的内核可能会禁止正常的包。另外，就算你没有打开rp_filter 选项。“防止广播欺骗”的功能是一直被开启的。不过他只针对内网地址，外网地址并不会被过滤。

四. LVS的三种模式区别对比

方三种负载均衡技术比较总结表：

| 要求      | LVS/NAT         | LVS/TUN              | LVS/DR                      |
| ------- | --------------- | -------------------- | --------------------------- |
| RS OS要求 | 任何系统            | 必须支持ip隧道协议，目前只有Linux | 服务节点需支持vip并能够禁用该设备上的arp响应功能 |
| 网络要求    | 拥有私有ip的局域网环境    | 拥有合法ip的局域网或广域网       | 拥有合法ip的局域网，服务节点需与均衡器在同一广播域  |
| 负载节点数   | 10-20个，视均衡器性能而定 | >100                 | >100                        |
| RS  网关  | 均衡器             | 自有的网关路由              | 自有的网关路由                     |
| 优点      | 地址和端口转换         | Wan/Lan环境加密数据        | 性能最高                        |
| 缺点      | 效率低             | 需要隧道支持               | 不能跨域LAN                     |

关于LVS原理讲解就介绍到这里了，以下为关于LVS相关的知识点学习网址:

<http://www.linuxvirtualserver.org/VS-NAT.html>  
<http://www.linuxvirtualserver.org/VS-IPTunneling.html>  
<http://www.linuxvirtualserver.org/VS-DRouting.html> 