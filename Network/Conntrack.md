## 解决恶心的 Nf_conntrack: Table Full 问题

相信有一定 Linux 服务器运维经验的人肯定见过这个问题：`dmesg` 或者 `/var/log/messages` 里大片地输出类似这样的日志：

`Dec  8 11:22:29 product08 kernel: nf_conntrack: table full, dropping packet.Dec  8 11:22:29 product08 kernel: nf_conntrack: table full, dropping packet.`

同时服务器上的各种网络服务耗时大幅上升，各种 timed out，各种丢包，完全无法正常提供服务。

随着我们流量的提升，最近又开始被这个问题虐了。几个月前第一次遇到此问题的时候，我们使用了提高 nf_conntrack table size 的方法解决的，事实证明这是个治标不治本的方法，流量上来了，改得再大最后还是会爆掉。况且 table 太大，占用的内存也会很多。

今天再次研究了一下这个问题，换了另一种更好的方案，尝试解决掉它。简单地说，方案就是在 iptables raw 表里增加规则，将无需跟踪状态的包标记为 `NOTRACK`，这样就无需耗费 nf_conntrack table entry 来记录其状态了。

## 1. nf_conntrack 是干嘛的

`nf_conntrack`（在老版本的 Linux 内核中叫 `ip_conntrack`）是一个内核模块，用于跟踪一个连接的状态的。连接状态跟踪可以供其他模块使用，最常见的两个使用场景是 iptables 的 `nat` 的 `state` 模块。

iptables 的 `nat` 通过规则来修改目的/源地址，但光修改地址不行，我们还需要能让回来的包能路由到最初的来源主机。这就需要借助 `nf_conntrack` 来找到原来那个连接的记录才行。

而 `state` 模块则是直接使用 `nf_conntrack` 里记录的连接的状态来匹配用户定义的相关规则。例如下面这条 INPUT 规则用于放行 80 端口上的状态为 NEW 的连接上的包。

`iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT`

## 2. 能否完全禁用 nf_conntrack

能否禁用 `nf_conntrack` 要看服务器干啥了，如果没有使用 `nat` 模块和 `state` 模块是可以禁用掉它的，这样做一劳永逸，再也不用被这个恶心的问题折磨了。

但是我发现 RHEL 默认的 iptables 规则里都是用到了 `state` 模块的：

`iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPTiptables -A INPUT -p icmp -j ACCEPTiptables -A INPUT -i lo -j ACCEPTiptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPTiptables -A INPUT -j REJECT --reject-with icmp-host-prohibited`

按理说打开端口的规则完全不需要使用 `-m state --state NEW`，只要匹配了目标端口直接放行就好。我理解这样做是为了优化性能。注意看第一条规则，只要连接状态是 ESTABLISHED 或者 RELATED 直接放行。这样做，只有连接初始化时的包需要经过下面的重重规则去一条条匹配，一旦建立上连接了（代表被放行了），后续的包直接通过第一条规则就放行了，省去了一条条匹配后续规则的麻烦（O(1)复杂度？）。

反之，如果我们不这么写，那连接的每个包都需要经过前面的层层规则过滤之后才能被放行，对性能有影响（O(n)复杂度？）。当然只有 iptables 规则很多的时候，这个问题才能突显，且到底能带来多大的影响，我手头没有测试数据，还不是特别清楚。

保险起见，我们就没有全部禁用掉 `nf_conntrack`，而是采用了 `raw` 表来部分地禁用连接跟踪。

## 3. 哪些连接可以不必跟踪状态

我们先看看怎么通过 `raw` 表针对部分连接禁用 conntrack。

`# 针对进入本机的包iptables -t raw -A PREROUTING -p tcp -m tcp --dport 8080 -j NOTRACK# 针对从本机出去的包iptables -t raw -A OUTPUT -p tcp -m tcp --dport 8080 -j NOTRACK`

重点就是 `-t raw` 和 `-j NOTRACK`，前者指定 table，后者指定 action 为 `NOTRACK`——不跟踪状态。

那么问题来了，要针对哪些连接应用此规则呢？需要 NAT 的肯定不行，各种 redirect 的连接本质上也是 NAT，也一样不行。除此之外的大部分的普通连接都是可以的。

比如所有 lo 接口上的连接：

`iptables -t raw -A PREROUTING -i lo -j NOTRACKiptables -t raw -A OUTPUT -o lo -j NOTRACK`

在应用和 Nginx 都跑在一台服务器上的情况下，这能禁用掉很多连接的状态跟踪，因为在并发高的时候，Nginx 和 Upstream 的连接也不是个小数目。

再比如针对流量很大的特定端口：

`iptables -t raw -A PREROUTING -p tcp -m tcp --dport 8080 -j NOTRACKiptables -t raw -A OUTPUT -p tcp -m tcp --sport 8080 -j NOTRACK`

需要特别特别注意的是，被 `NOTRACK` 的包无法匹配上依赖特定状态的其他规则。比如 INPUT 链上假如有这么一条规则：

`iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT`

那么在增加了上面的 `NOTRACK` 规则后，你会发现这个端口的包都被丢掉了。因为没有了连接状态跟踪，那个 `--state NEW` 不可能匹配得上。上午我就因为犯了这个错误导致某关键服务宕机了几分钟……



## 解决恶心的 Nf_conntrack: Table Full 问题

相信有一定 Linux 服务器运维经验的人肯定见过这个问题：`dmesg` 或者 `/var/log/messages` 里大片地输出类似这样的日志：

`Dec  8 11:22:29 product08 kernel: nf_conntrack: table full, dropping packet.Dec  8 11:22:29 product08 kernel: nf_conntrack: table full, dropping packet.`

同时服务器上的各种网络服务耗时大幅上升，各种 timed out，各种丢包，完全无法正常提供服务。

随着我们流量的提升，最近又开始被这个问题虐了。几个月前第一次遇到此问题的时候，我们使用了提高 nf_conntrack table size 的方法解决的，事实证明这是个治标不治本的方法，流量上来了，改得再大最后还是会爆掉。况且 table 太大，占用的内存也会很多。

今天再次研究了一下这个问题，换了另一种更好的方案，尝试解决掉它。简单地说，方案就是在 iptables raw 表里增加规则，将无需跟踪状态的包标记为 `NOTRACK`，这样就无需耗费 nf_conntrack table entry 来记录其状态了。

## 1. nf_conntrack 是干嘛的

`nf_conntrack`（在老版本的 Linux 内核中叫 `ip_conntrack`）是一个内核模块，用于跟踪一个连接的状态的。连接状态跟踪可以供其他模块使用，最常见的两个使用场景是 iptables 的 `nat` 的 `state` 模块。

iptables 的 `nat` 通过规则来修改目的/源地址，但光修改地址不行，我们还需要能让回来的包能路由到最初的来源主机。这就需要借助 `nf_conntrack` 来找到原来那个连接的记录才行。

而 `state` 模块则是直接使用 `nf_conntrack` 里记录的连接的状态来匹配用户定义的相关规则。例如下面这条 INPUT 规则用于放行 80 端口上的状态为 NEW 的连接上的包。

`iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT`

## 2. 能否完全禁用 nf_conntrack

能否禁用 `nf_conntrack` 要看服务器干啥了，如果没有使用 `nat` 模块和 `state` 模块是可以禁用掉它的，这样做一劳永逸，再也不用被这个恶心的问题折磨了。

但是我发现 RHEL 默认的 iptables 规则里都是用到了 `state` 模块的：

`iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPTiptables -A INPUT -p icmp -j ACCEPTiptables -A INPUT -i lo -j ACCEPTiptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPTiptables -A INPUT -j REJECT --reject-with icmp-host-prohibited`

按理说打开端口的规则完全不需要使用 `-m state --state NEW`，只要匹配了目标端口直接放行就好。我理解这样做是为了优化性能。注意看第一条规则，只要连接状态是 ESTABLISHED 或者 RELATED 直接放行。这样做，只有连接初始化时的包需要经过下面的重重规则去一条条匹配，一旦建立上连接了（代表被放行了），后续的包直接通过第一条规则就放行了，省去了一条条匹配后续规则的麻烦（O(1)复杂度？）。

反之，如果我们不这么写，那连接的每个包都需要经过前面的层层规则过滤之后才能被放行，对性能有影响（O(n)复杂度？）。当然只有 iptables 规则很多的时候，这个问题才能突显，且到底能带来多大的影响，我手头没有测试数据，还不是特别清楚。

保险起见，我们就没有全部禁用掉 `nf_conntrack`，而是采用了 `raw` 表来部分地禁用连接跟踪。

## 3. 哪些连接可以不必跟踪状态

我们先看看怎么通过 `raw` 表针对部分连接禁用 conntrack。

`# 针对进入本机的包iptables -t raw -A PREROUTING -p tcp -m tcp --dport 8080 -j NOTRACK# 针对从本机出去的包iptables -t raw -A OUTPUT -p tcp -m tcp --dport 8080 -j NOTRACK`

重点就是 `-t raw` 和 `-j NOTRACK`，前者指定 table，后者指定 action 为 `NOTRACK`——不跟踪状态。

那么问题来了，要针对哪些连接应用此规则呢？需要 NAT 的肯定不行，各种 redirect 的连接本质上也是 NAT，也一样不行。除此之外的大部分的普通连接都是可以的。

比如所有 lo 接口上的连接：

`iptables -t raw -A PREROUTING -i lo -j NOTRACKiptables -t raw -A OUTPUT -o lo -j NOTRACK`

在应用和 Nginx 都跑在一台服务器上的情况下，这能禁用掉很多连接的状态跟踪，因为在并发高的时候，Nginx 和 Upstream 的连接也不是个小数目。

再比如针对流量很大的特定端口：

`iptables -t raw -A PREROUTING -p tcp -m tcp --dport 8080 -j NOTRACKiptables -t raw -A OUTPUT -p tcp -m tcp --sport 8080 -j NOTRACK`

需要特别特别注意的是，被 `NOTRACK` 的包无法匹配上依赖特定状态的其他规则。比如 INPUT 链上假如有这么一条规则：

`iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT`

那么在增加了上面的 `NOTRACK` 规则后，你会发现这个端口的包都被丢掉了。因为没有了连接状态跟踪，那个 `--state NEW` 不可能匹配得上。上午我就因为犯了这个错误导致某关键服务宕机了几分钟……



### 设置 conntrack table size



Mar 26 21:53:50 Gentoo-2012 kernel: [974749.140915] net_ratelimit: 3606 callbacks suppressed
Mar 26 21:53:50 Gentoo-2012 kernel: [974749.140918] nf_conntrack: table full, dropping packet.
Mar 26 21:53:50 Gentoo-2012 kernel: [974749.144022] nf_conntrack: table full, dropping packet.
Mar 26 21:53:50 Gentoo-2012 kernel: [974749.144031] nf_conntrack: table full, dropping packet.
Mar 26 21:53:50 Gentoo-2012 kernel: [974749.144037] nf_conntrack: table full, dropping packet.
解决办法

修改内核参数
vi /etc/sysctl.conf
net.ipv4.netfilter.ip_conntrack_max = 655350
net.ipv4.netfilter.ip_conntrack_tcp_timeout_established = 1200
\#默认超时时间为5天，作为一个主要提供HTTP服务的服务器来讲，完全可以设置得比较短
sysctl -p # 让刚刚修改过的设置生效

默认值是
net.ipv4.netfilter.ip_conntrack_max = 65536
net.ipv4.netfilter.ip_conntrack_tcp_timeout_established = 432000

 

centos6.1上面如下设置

net.netfilter.nf_conntrack_max = 655360
net.nf_conntrack_max = 655360

查看是否生效

cat /proc/sys/net/netfilter/nf_conntrack_max

655360

 

### 关于nf_conntrack以及和ip_conntrack的区别



网上关于ip_conntrack的介绍并不少。 Linux内核的ip_conntrack模块会记录每一个tcp协议的estiablished connection记录。关且一个默认的timeout值是432000秒（五天时间）。每个ip_conntrack记录约会占用292Bytes的内存，所以系统所能记录的ip_conntrack也是有限的，如果超过了这个限度，就会出现内核级错误“ip_conntrack: table full, dropping packet”，其结果就是无法再有任何的网络连接了。
今天又研究了下iptables, 随在系统里寻找ip_conntrack的设置，却怎么也找不到了。莫明其妙的，竟然模块lsmod里，有一个nf_conntrack。 看样子和ip_conntrack一样，而且cat /proc/net/nf_conntrack 也能看到类似与ip_conntrack的内容。 所以一顿搜索，终于发现。 这2个其实是一个东西啊。 **找不到ip_conntracks的同学们，不必慌张了。新内核的linux系统里，你基本都是用的nf_conntrack了。**

在netfilter 的一个group emails ,有这么一句话：
ip_conntrack_* helper modules have been replaced by nf_conntrack_* when available and moved from net/ipv4/netfilter to net/netfilter.

ip_conntrack 的helper modules 都被替换成了nf_conntrack ， 而且在proc的文件系统里，也换了位置。所以你找不到也就不奇怪了。

**nf_contrack 设置位于新的位置：**

cat /proc/net/nf_conntrack

cat /proc/sys/net/netfilter/nf_* 看所有的配置参数都在这里了。 当然除了动态的修改和优化nf_conntrack外，你也需要在sysctl.conf中进行配置。

==================================
附上网上关于nf_conntrack优化的[一篇文章](http://jaseywang.me/2012/08/16/%E8%A7%A3%E5%86%B3-nf_conntrack-table-full-dropping-packet-%E7%9A%84%E5%87%A0%E7%A7%8D%E6%80%9D%E8%B7%AF/)：

解决 nf_conntrack: table full, dropping packet 的几种思路
Posted on August 16, 2012
nf_conntrack 工作在 3 层，支持 IPv4 和 IPv6，而 ip_conntrack 只支持 IPv4。目前，大多的 ip_conntrack_* 已被 nf_conntrack_* 取代，很多 ip_conntrack_* 仅仅是个 alias，原先的 ip_conntrack 的 /proc/sys/net/ipv4/netfilter/ 依然存在，但是新的 nf_conntrack 在 /proc/sys/net/netfilter/ 中，这个应该是做个向下的兼容：
$ pwd
/proc/sys/net/ipv4/netfilter
$ ls
ip_conntrack_buckets ip_conntrack_tcp_loose ip_conntrack_tcp_timeout_syn_recv
ip_conntrack_checksum ip_conntrack_tcp_max_retrans ip_conntrack_tcp_timeout_syn_sent
ip_conntrack_count ip_conntrack_tcp_timeout_close ip_conntrack_tcp_timeout_syn_sent2
ip_conntrack_generic_timeout ip_conntrack_tcp_timeout_close_wait ip_conntrack_tcp_timeout_time_wait
ip_conntrack_icmp_timeout ip_conntrack_tcp_timeout_established ip_conntrack_udp_timeout
ip_conntrack_log_invalid ip_conntrack_tcp_timeout_fin_wait ip_conntrack_udp_timeout_stream
ip_conntrack_max ip_conntrack_tcp_timeout_last_ack
ip_conntrack_tcp_be_liberal ip_conntrack_tcp_timeout_max_retrans

$ pwd
/proc/sys/net/netfilter
$ ls
nf_conntrack_acct nf_conntrack_tcp_timeout_close
nf_conntrack_buckets nf_conntrack_tcp_timeout_close_wait
nf_conntrack_checksum nf_conntrack_tcp_timeout_established
nf_conntrack_count nf_conntrack_tcp_timeout_fin_wait
nf_conntrack_events nf_conntrack_tcp_timeout_last_ack
nf_conntrack_events_retry_timeout nf_conntrack_tcp_timeout_max_retrans
nf_conntrack_expect_max nf_conntrack_tcp_timeout_syn_recv
nf_conntrack_generic_timeout nf_conntrack_tcp_timeout_syn_sent
nf_conntrack_icmp_timeout nf_conntrack_tcp_timeout_time_wait
nf_conntrack_log_invalid nf_conntrack_tcp_timeout_unacknowledged
nf_conntrack_max nf_conntrack_udp_timeout
nf_conntrack_tcp_be_liberal nf_conntrack_udp_timeout_stream
nf_conntrack_tcp_loose nf_log/
conntrack_tcp_max_retrans

查看当前的连接数：
\# grep ip_conntrack /proc/slabinfo
ip_conntrack 38358 64324 304 13 1 : tunables 54 27 8 : slabdata 4948 4948 216

查出目前 ip_conntrack 的排名：
$ cat /proc/net/ip_conntrack | cut -d ‘ ‘ -f 10 | cut -d ‘=’ -f 2 | sort | uniq -c | sort -nr | head -n 10

nf_conntrack/ip_conntrack 跟 nat 有关，用来跟踪连接条目，它会使用一个哈希表来记录 established 的记录。nf_conntrack 在 2.6.15 被引入，而 ip_conntrack 在 2.6.22 被移除，如果该哈希表满了，就会出现：
nf_conntrack: table full, dropping packet

解决此问题有如下几种思路。

1.不使用 nf_conntrack 模块
首先要移除 state 模块，因为使用该模块需要加载 nf_conntrack。确保 iptables 规则中没有出现类似 state 模块的规则，如果有的话将其移除：
-A INPUT -m state –state RELATED,ESTABLISHED -j ACCEPT

注释 /etc/sysconfig/iptables-config 中的：
IPTABLES_MODULES=”ip_conntrack_netbios_ns”

移除 nf_conntrack 模块：
$ sudo modprobe -r xt_NOTRACK nf_conntrack_netbios_ns nf_conntrack_ipv4 xt_state
$ sudo modprobe -r nf_conntrack

现在 /proc/net/ 下面应该没有 nf_conntrack 了。

2.调整 /proc/ 下面的参数
可以增大 conntrack 的条目(sessions, connection tracking entries) CONNTRACK_MAX 或者增加存储 conntrack 条目哈希表的大小 HASHSIZE
默认情况下，CONNTRACK_MAX 和 HASHSIZE 会根据系统内存大小计算出一个比较合理的值：
对于 CONNTRACK_MAX，其计算公式：
CONNTRACK_MAX = RAMSIZE (in bytes) / 16384 / (ARCH / 32)
比如一个 64 位 48G 的机器可以同时处理 48*1024^3/16384/2 = 1572864 条 netfilter 连接。对于大于 1G 内存的系统，默认的 CONNTRACK_MAX 是 65535。

对于 HASHSIZE，默认的有这样的转换关系：
CONNTRACK_MAX = HASHSIZE * 8
这表示每个链接列表里面平均有 8 个 conntrack 条目。其真正的计算公式如下：
HASHSIZE = CONNTRACK_MAX / 8 = RAMSIZE (in bytes) / 131072 / (ARCH / 32)
比如一个 64 位 48G 的机器可以存储 48*1024^3/131072/2 = 196608 的buckets(连接列表)。对于大于 1G 内存的系统，默认的 HASHSIZE 是 8192。

可以通过 echo 直接修改目前系统 CONNTRACK_MAX 以及 HASHSIZE 的值：
$ sudo su -c “echo 100000 > /proc/sys/net/netfilter/nf_conntrack_max”
$ sudo su -c “echo 50000 > /proc/sys/net/netfilter/nf_conntrack_buckets”

还可以缩短 timeout 的值：
$ sudo su -c “echo 600 > /proc/sys/net/ipv4/netfilter/ip_conntrack_tcp_timeout_established”

3.使用 raw 表，不跟踪连接
iptables 中的 raw 表跟包的跟踪有关，基本就是用来干一件事，通过 NOTRACK 给不需要被连接跟踪的包打标记，也就是说，如果一个连接遇到了 -j NOTRACK，conntrack 就不会跟踪该连接，raw 的优先级大于 mangle, nat, filter，包含 PREROUTING 和 OUTPUT 链。
当执行 -t raw 时，系统会自动加载 iptable_raw 模块(需要该模块存在)。raw 在 2.4 以及 2.6 早期的内核中不存在，除非打了 patch，目前的系统应该都有支持:
$ sudo iptables -A FORWARD -m state –state UNTRACKED -j ACCEPT
$ sudo iptables -t raw -A PREROUTING -p tcp -m multiport –dport 80,81,82 -j NOTRACK
$ sudo iptables -t raw -A PREROUTING -p tcp -m multiport –sport 80,81,82 -j NOTRACK

上面三种方式，最有效的是 1 跟 3，第二种治标不治本。

ref:

http://www.digipedia.pl/usenet/thread/16263/7806/

http://serverfault.com/questions/72366/how-do-i-disable-the-nf-conntrack-kernel-module-in-centos-5-3-without-recompilin

http://wiki.khnet.info/index.php/Conntrack_tuning

此篇文章已被阅读1771 次