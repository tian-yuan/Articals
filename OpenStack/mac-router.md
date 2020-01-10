- 查看所有路由表

- - netstat -rn
  - 路由表字段解析看[netstat使用1](https://computing.llnl.gov/tutorials/performance_tools/man/netstat.txt)、[netstat使用2](http://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.cmds4/netstat.htm)

-  查看默认网关

- - route -n get default
  - route -n get [www.yahoo.com](http://www.yahoo.com/)



> 使用下面的 route 命令可以查看 Linux 内核路由表。netstat -nr
>
> ```
> # route
> Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
> 192.168.0.0     *               255.255.255.0   U     0      0        0 eth0
> 169.254.0.0     *               255.255.0.0     U     0      0        0 eth0
> default         192.168.0.1     0.0.0.0         UG    0      0        0 eth0
> ```
>
> route 命令的输出项说明
>
> 输出项 说明
>
> |             |                                                            |
> | ----------- | ---------------------------------------------------------- |
> | Destination | 目标网段或者主机                                           |
> | Gateway     | 网关地址，”*” 表示目标是本主机所属的网络，不需要路由       |
> | Genmask     | 网络掩码                                                   |
> | Flags       | 标记。一些可能的标记如下：                                 |
> |             | U — 路由是活动的（UP）                                     |
> |             | H — 目标是一个主机（HOST）                                 |
> |             | G — 路由指向网关（Gateway）                                |
> |             | R — 恢复动态路由产生的表项（Resume）                       |
> |             | D — 由路由的后台程序动态地安装（Dynamic）                  |
> |             | M — 由路由的后台程序修改（Modify）                         |
> |             | ! — 拒绝路由                                               |
> | Metric      | 路由距离，到达指定网络所需的中转数（linux 内核中没有使用） |
> | Ref         | 路由项引用次数（linux 内核中没有使用）                     |
> | Use         | 此路由项被路由软件查找的次数                               |
> | Iface       | 该路由表项对应的输出接口                                   |
>
>  
>
> #### 主机路由
>
> 主机路由是路由选择表中指向单个IP地址或主机名的路由记录。主机路由的Flags字段为H。例如，在下面的示例中，本地主机通过IP地址192.168.1.1的路由器到达IP地址为10.0.0.10的主机。
>
> ```
> Destination    Gateway       Genmask        Flags     Metric    Ref    Use    Iface
> -----------    -------     -------            -----     ------    ---    ---    -----
> 10.0.0.10     192.168.1.1    255.255.255.255   UH       0    0      0    eth0
> ```
>
> #### 网络路由
>
> 网络路由是代表主机可以到达的网络。网络路由的Flags字段为N。例如，在下面的示例中，本地主机将发送到网络192.19.12的数据包转发到IP地址为192.168.1.1的路由器。
>
> ```
> Destination    Gateway       Genmask      Flags    Metric    Ref     Use    Iface
> -----------    -------     -------         -----    -----   ---    ---    -----
> 192.19.12     192.168.1.1    255.255.255.0      UN      0       0     0    eth0
> ```
>
> #### 默认路由
>
> 当主机不能在路由表中查找到目标主机的IP地址或网络路由时，数据包就被发送到默认路由（默认网关）上。默认路由的Flags字段为G。例如，在下面的示例中，默认路由是IP地址为192.168.1.1的路由器。
>
> ```
> Destination    Gateway       Genmask    Flags     Metric    Ref    Use    Iface
> -----------    -------     ------- -----      ------    ---    ---    -----
> default       192.168.1.1     0.0.0.0    UG       0        0     0    eth0
> ```
>
> **关于路由表的说明**
>
> 对于一个给定的路由器，可以打印出五种不同的标志（ f l a g）：
> U 该路由可以使用。
> G 该路由是到一个网关（路由器）。如果没有设置该标志，说明目的地是直接相连的。
> H 该路由是到一个主机，也就是说，目的地址是一个完整的主机地址。如果没有设置该
> 标志，说明该路由是到一个网络，而目的地址是一个网络地址：一个网络号，或者网
> 络号与子网号的组合。
> D 该路由是由重定向报文创建的（ 9 . 5节）。
> M 该路由已被重定向报文修改（ 9 . 5节）。


路由表的搜索

> 1) 搜索匹配的主机地址；
> 2) 搜索匹配的网络地址；
> 3) 搜索默认表项（默认表项一般在路由表中被指定为一个网络表项，其网络号为0）。
> 匹配主机地址步骤始终发生在匹配网络地址步骤之前。


关键的说明，需要仔细理解，可以多看几遍

> 标志G是非常重要的，因为由它区分了间接路由和直接路由（对于直接路由来说是不设置标志G的）。其区别在于，发往直接路由的分组中不但具有指明目的端的I P地址，还具有其链路层地址（见图3 - 3）。当分组被发往一个间接路由时， I P地址指明的是最终的目的地，但是链路层地址指明的是网关（即下一站路由器）。我们在图3 - 4已看到这样的例子。在这个路由表例子中，有一个间接路由（设置了标志G），因此采用这一项路由的分组其I P地址是最终的目的地（1 4 0 . 2 5 2 . 1 3 . 6 5），但是其链路层地址必须对应于路由器1 4 0 . 2 5 2 . 1 3 . 3 5。
> 理解G和H标志之间的区别是很重要的。G标志区分了直接路由和间接路由，如上所述。但是H标志表明，目的地址（ n e t s t a t命令输出第一行）是一个完整的主机地址。没有设置H标志说明目的地址是一个网络地址（主机号部分为0）。当为某个目的I P地址搜索路由表时，主机地址项必须与目的地址完全匹配，而网络地址项只需要匹配目的地址的网络号和子网号就可以了。另外，大多数版本的n e t s t a t命令首先打印出所有的主机路由表项，然后才是网络路由表项。
>
>  
>
> ## 配置静态路由
>
>  
>
> ### route 命令
>
> 设置和查看路由表都可以用 route 命令，设置内核路由表的命令格式是：
>
> ```
> # route  [add|del] [-net|-host] target [netmask Nm] [gw Gw] [[dev] If]
> ```
>
> 其中：
>
> - add : 添加一条路由规则
> - del : 删除一条路由规则
> - -net : 目的地址是一个网络
> - -host : 目的地址是一个主机
> - target : 目的网络或主机
> - netmask : 目的地址的网络掩码
> - gw : 路由数据包通过的网关
> - dev : 为路由指定的网络接口
>
> ### route 命令使用举例
>
> 添加到主机的路由
>
> ```
> # route add -host 192.168.1.2 dev eth0:0
> # route add -host 10.20.30.148 gw 10.20.30.40
> ```
>
> 添加到网络的路由
>
> ```
> # route add -net 10.20.30.40 netmask 255.255.255.248 eth0
> # route add -net 10.20.30.48 netmask 255.255.255.248 gw 10.20.30.41
> # route add -net 192.168.1.0/24 eth1
> ```
>
> 添加默认路由
>
> ```
> # route add default gw 192.168.1.1
> ```
>
> 删除路由
>
> ```
> # route del -host 192.168.1.2 dev eth0:0
> # route del -host 10.20.30.148 gw 10.20.30.40
> # route del -net 10.20.30.40 netmask 255.255.255.248 eth0
> # route del -net 10.20.30.48 netmask 255.255.255.248 gw 10.20.30.41
> # route del -net 192.168.1.0/24 eth1
> # route del default gw 192.168.1.1
> ```
>
> ### 设置包转发
>
> 在 CentOS 中默认的内核配置已经包含了路由功能，但默认并没有在系统启动时启用此功能。开启 Linux 的路由功能可以通过调整内核的网络参数来实现。要配置和调整内核参数可以使用 sysctl 命令。例如：要开启 Linux 内核的数据包转发功能可以使用如下的命令。
>
> ```
> # sysctl -w net.ipv4.ip_forward=1
> ```
>
> 这样设置之后，当前系统就能实现包转发，但下次启动计算机时将失效。为了使在下次启动计算机时仍然有效，需要将下面的行写入配置文件/etc/sysctl.conf。
>
> ```
> # vi /etc/sysctl.conf
> net.ipv4.ip_forward = 1
> ```
>
> 用户还可以使用如下的命令查看当前系统是否支持包转发。
>
> ```
> # sysctl  net.ipv4.ip_forward
> ```