# vrrp协议与keepalived详解

 	keepalived是和heartbeat不同思路的HA开源解决方案，keepalived是基于vrrp协议的，那在介绍keepalived之前，我们先来了解vrrp协议。

**一、Vrrp协议**

**1、VRRP 协议简介**

​	vrrp: Virtual Redundent Routing Protocol 虚拟冗余路由协议

​       在现实的网络环境中两台需要通信的主机大多数情况下并没有直接的物理连接。对于这样的情况它们之间路由怎样选择主机如何选定到达目的主机的下一跳路由

这个问题通常的解决方法有二种：

- 在主机上使用动态路由协议(RIP、OSPF等)
- 在主机上配置静态路由

​       很明显在主机上配置动态路由是非常不切实际的因为管理、维护成本以及是否支持等诸多问题。配置静态路由就变得十分流行但路由器|默认网关|default gateway却经常成为单点故障。

​       VRRP协议的目的就是为了解决静态路由单点故障问题；VRRP通过竞选(election)协议来动态的将路由任务交给LAN中虚拟路由器中的某台VRRP路由器。

**2、**相关术语

**虚拟路由器：**由一个 Master 路由器和多个 Backup 路由器组成。主机将虚拟路由器当作默认网关。

**VRID：**虚拟路由器的标识，有相同 VRID 的一组路由器构成一个虚拟路由器。通常用（0-255）标识

**Master路由器：**虚拟路由器中承担报文转发任务的路由器。

**Backup路由器： **Master路由器出现故障时能够代替 Master路由器工作的路由器。

**虚拟 IP：**地址虚拟路由器的 IP 地址，一个虚拟路由器可以拥有一个或多个IP 地址。

**IP 地址拥有者：**接口IP地址与虚拟IP地址相同的路由器被称为 IP 地址拥有者。

**虚拟 MAC 地址：**一个虚拟路由器拥有一个虚拟 MAC 地址。虚拟 MAC 地址的格式为 00-00-5E-00-01-{VRID}。通常情况下虚拟路由器回应 ARP 请求

使用的是虚拟 MAC 地址只有虚拟路由器做特殊配置的时候才回应接口的真实 MAC 地址。

**priority优先级：**VRRP 根据优先级来确定虚拟路由器中每台路由器的地位。用0-255来表示，数字越小优先级越低。VRRP优先级的取值范围为0到255（数值越大表明优先级越高），可配置的范围是1到254，优先级0为系统保留给路由器放弃Master位置时候使用，255则是系统保留给IP地址拥有者使用。当路由器为IP地址拥有者时，其优先级始终为255。因此，当虚拟路由器内存在IP地址拥有者时，只要其工作正常，则为Master路由器。

**抢占方式：**默认，如果 Backup 路由器工作在抢占方式下，当它收到 VRRP 报文后会将自己的优先级与通告报文中的优先级进行比较。如果自己的优先级比当前的 Master 路由器的优先级高就会主动抢占成为 Master 路由器否则将保持 Backup 状态。

**非抢占方式：**如果 Backup 路由器工作在非抢占方式下则只要 Master 路由器没有出现故障Backup 路由器即使随后被配置了更高的优先级也不会成为Master 路由器。

**3、VRRP 工作机制**

​       在一个VRRP虚拟路由器中有多台物理的VRRP路由器但是这多台的物理的机器并不能同时工作而是由一台称为MASTER的负责路由工作其它的都是BACKUPMASTER并非一成不变VRRP让每个VRRP路由器参与竞选最终获胜的就是MASTER。MASTER拥有虚拟路由器的IP地址我们的主机就是用这个IP地址作为静态路由的MASTER要负责转发发送给网关地址的包和响应ARP请求。

​       VRRP通过竞选协议来实现虚拟路由器的功能所有的协议报文都是通过IP多播(multicast)包形式发送的。虚拟路由器由VRID(范围0-255)和一组IP地址组成对外表现为一个周知的MAC地址。所以在一个虚拟路由 器中不管谁是MASTER对外都是相同的MAC和IP(称之为VIP)。客户端主机并不需要因为MASTER的改变而修改自己的路由配置对客户端来说这种主从的切换是透明的。

​       在一个虚拟路由器中只有作为MASTER的VRRP路由器会一直发送VRRP通告信息(VRRPAdvertisement message)BACKUP不会抢占MASTER除非它的优先级(priority)更高。当MASTER不可用时(BACKUP收不到通告信息) 多台BACKUP中优先级最高的这台会被抢占为MASTER。这种抢占是非常快速的(<1s)以保证服务的连续性。由于安全性考虑VRRP包使用了加密协议进行加密。

**4、VRRP 工作流程**

(1).初始化 （还没选举出master时）   

​     路由器启动时如果路由器的优先级是255(最高优先级路由器拥有路由器地址)要发送VRRP通告信息并发送广播ARP信息通告路由器IP地址对应的MAC地址为路由虚拟MAC，设置通告信息定时器，准备定时发送VRRP通告信息，转为MASTER状态，否则进入BACKUP状态设置定时器检查定时检查是否收到MASTER的通告信息。

(2).Master

- 设置定时通告定时器
- 用VRRP虚拟MAC地址响应路由器IP地址的ARP请求
- 转发目的MAC是VRRP虚拟MAC的数据包
- 如果是虚拟路由器IP的拥有者将接受目的地址是虚拟路由器IP的数据包否则丢弃
- 当收到shutdown的事件时，删除定时通告定时器发送优先权级为0的通告包转初始化状态
- 如果定时通告定时器超时时发送VRRP通告信息
- 收到VRRP通告信息时如果优先权为0发送VRRP通告信息否则判断数据的优先级是否高于本机或相等而且实际IP地址大于本地实际IP设置定时通告定时器复位主机超时定时器转BACKUP状态否则的话丢弃该通告包

(3).Backup

- 设置主机超时定时器
- 不能响应针对虚拟路由器IP的ARP请求信息
- 丢弃所有目的MAC地址是虚拟路由器MAC地址的数据包
- 不接受目的是虚拟路由器IP的所有数据包
- 当收到shutdown的事件时删除主机超时定时器转初始化状态
- 主机超时定时器超时的时候发送VRRP通告信息广播ARP地址信息转MASTER状态
- 收到VRRP通告信息时如果优先权为0表示进入MASTER选举，否则判断数据的优先级是否高于本机如果高的话承认MASTER有效，复位主机超时定时器否则的话丢弃该通告包

（4）ARP查询处理

​       当内部主机通过ARP查询虚拟路由器IP地址对应的MAC地址时，MASTER路由器回复的MAC地址为虚拟的VRRP的MAC地址而不是实际网卡的，MAC地址这样在路由器切换时让内网机器觉察不到而在路由器重新启动时不能主动发送本机网卡的实际MAC地址。如果虚拟路由器开启的ARP代理 (proxy_arp)功能代理的ARP回应也回应VRRP虚拟MAC地址。

**5、结论**

​       VRRP实现了对路由器IP地址的冗余功能防止了单点故障造成的网络失效，VRRP本身是热备形式的但可以通过互相主备实现路由器的负载均衡处理。

**6、认证方式与工作模式**

**认证方式**

   简单字符认证

   md5认证

**工作模式**

​    master-backup模式

​    master-master模式

​         双主模式，互为主备模式

**二、Keepalived**

**1、Keepalived 定义**

什么是Keepalived呢?

​       观其名可知保持存活在网络里面就是保持在线了也就是所谓的高可用或热备用来防止单点故障(单点故障是指一旦某一点出现故障就会导致整个系统架构的不可用)

在Linux主机上以daemon守护进程方式实现了vrrp协议并提供了完成配置ipvs规则及实现相应real server状态检测能力。

​     能调用外部脚本，激活简单的外部应用程序

   轻量、灵活但不能解决脑裂问题，

​     适用场景：ipvs, haproxy, nginx(reverse proxy 反向代理)等反向代理或负载均衡器的高可用   

**2、keepalived的工作原理**

Keepalived是一个基于VRRP协议来实现的LVS服务高可用方案可以利用其来避免单点故障。

​       一个LVS服务会有2台服务器运行Keepalived一台为主服务器MASTER一台为备份服务器BACKUP但是对外表现为一个虚拟IP；主服务器会发送特定的消息给备份服务器，当备份服务器收不到这个消息的时候即主服务器宕机的时候，备份服务器就会接管虚拟IP继续提供服务从而保证了高可用性；Keepalived是VRRP的完美实现。

**3、Keepalived组件**

keepalived是模块化设计，不同模块负责不同的功能：

**core：**keepalived的核心；负责主进程的启动和维护全局配置文件的加载解析等

**check：**负责healthchecker(健康检查)包括了各种健康检查方式以及对应的配置文件的解析

**vrrp:**  VRRPD子进程用来实现VRRP协议

**libipfwc:**  iptables(ipchains)库配置LVS

**libipvs：**配置LVS

[![wKioL1ZfhCjRiwjcAAJ9-ohqITI116.png](http://s3.51cto.com/wyfs02/M02/76/F1/wKioL1ZfhCjRiwjcAAJ9-ohqITI116.png)](http://s3.51cto.com/wyfs02/M02/76/F1/wKioL1ZfhCjRiwjcAAJ9-ohqITI116.png)

components:组件

**3、keepalived进程**

keepalived启动后会有三个进程

父进程内存管理子进程管理等等

**子进程**VRRP子进程

**子进程**healthchecker子进程

两个子进程都被系统WatchDog看管两个子进程各自负责自己的事

healthchecker子进程负责检查各自服务器的健康程度如果healthchecker子进程检查到MASTER上服务不可用了就会通知本机上的兄弟VRRP子进程让他删除通告并且去掉虚拟IP转换为BACKUP状态。

**三、keepalived安装**

**1、安装keepalived**

centos6.4版本及以后yum中自带了keepalived

```
[root@BAIYU_180 html]# yum install keepalived -y
[root@BAIYU_180 html]# rpm -ql keepalived
/etc/keepalived
/etc/keepalived/keepalived.conf
/etc/rc.d/init.d/keepalived
/etc/sysconfig/keepalived
/usr/bin/genhash
/usr/sbin/keepalived
/usr/share/doc/keepalived-1.2.7
```



**四、keepalived配置**

keepalived配置文件/etc/keepalived/keepalived.conf分为三个部分

global_defs: 全局配置

vrrp_instance: vrouter的配置

lvs: ipvs相关配置

**1、查看/etc/keepalived/keepalived.conf**

keepalived的配置文件分为三段：

**global_defs：全局配置**

​       keeplived自带的发送邮件机制是个鸡肋，只有在后端服务器故障时才发送邮件到系统中的用户邮箱（没有实际意义），不能使用外部的服务器邮箱发送邮件到用户邮箱中（原因是会被当成垃圾邮件？）；不用的话相关的配置可以全去掉用脚本实现报警功能

**vrrp_instance：一个vrouter的配置**

**lvs：ipvs相关配置**

```
[root@BAIYU_180 keepalived]# cat keepalived.conf 
! Configuration File for keepalived         #　！和＃号是注释
 
global_defs {      #全局配置
   notification_email {        #报警邮件发送给谁 
后端服务器故障时需要发送email通知以及email发送给哪些邮件地址邮件地址可以多个每行一个，如果是发到本机的用户邮箱格式：已存在的用户@主机名    
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc #通知邮件的发件人邮箱，可以随意自定义
   smtp_server 192.168.200.1     #邮件服务器地址
   smtp_connect_timeout 30      #件服务器连接的超时时间
   router_id LVS_DEVEL        #机器标识，可以不修改，多台机器此id可以相同
}
 
vrrp_instance VI_1 {      #vroute标识
    state MASTER          #当前节点的状态 主节点
    interface eth0        #发送vip通告的接口
    virtual_router_id 51  #虚拟路由的ID号是虚拟路由MAC的最后一位地址
    priority 100           #此节点的优先级主节点的优先级需要比其他节点高
    advert_int 1           #vip通告的时间间隔   
    authentication {       #认证配置
       auth_type PASS     #认证机制默认是明文
      auth_pass 1111     #随机字符当密码，要和虚拟路由器中其它路由器保持一致
    }
    virtual_ipaddress {   #vip
        192.168.100.41
         
    }
     
    track_interface {       #监控物理接口，如果出现故障会被标记为失败
    eth0
    eth1
    }
}
###########################################只要以上的配置把下面的都注释就可以实现
简单高可用此时只能实现主机故障|网络故障|keepalived进程停止时vip转移或者通过脚本实现其它服务的切换
 
virtual_server 192.168.100.41 80 {  #虚拟服务器所使用的VIP和端口和上面使用vip对应
    delay_loop 6          #健康状态检测间隔时间
    lb_algo rr             #lvs调度算法rr|wrr|lc|wlc|lblc|sh|dh
    lb_kind NAT            #lvs负载均衡模式NAT|DR|RUN
    nat_mask 255.255.255.0
    persistence_timeout 50  #持久连接时间
    protocol TCP            #使用的协议
 
    real_server 192.168.100.41 80 {    #节点服务器使用的IP及端口
        weight 1        
        SSL_GET {     #健康状态检测方法  
            url {
              path /    #检测的页面
              #digest ff20ad2481f97b1754ef3e12ecd3a9cc  #检查的摘要信息
              status_code 200 #检查的返回状态码
            }
            url {
              path /mrtg/
              digest 9b3a0c85a887a256d6939da88aabd8cd
            }
            connect_timeout 3           #连接超时时间
            nb_get_retry 3              #重连次数
            delay_before_retry 3        #重连间隔
        }
    }
}
 
virtual_server 10.10.10.2 1358 {
    delay_loop 6
    lb_algo rr 
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
 
    sorry_server 192.168.200.200 1358
 
    real_server 192.168.200.2 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
 
    real_server 192.168.200.3 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
 
virtual_server 10.10.10.3 1358 {
    delay_loop 3
    lb_algo rr 
    lb_kind NAT
    nat_mask 255.255.255.0
    persistence_timeout 50
    protocol TCP
 
    real_server 192.168.200.4 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
 
    real_server 192.168.200.5 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```



**2、配置文件中指令详解**

（1）vip的配置

virtual_ipaddress {

​         <IPADDR>/<MASK>  brd <IPADDR>  dev <STRING>   scope <SCOPE>   label <LABEL>

​                                                   广播地址                 设备                 作用域                    别名

​               192.168.200.17/24 dev eth1

​               172.16.11.91/24 dev eth2 label eth2:1

​           }

如果有其它资源可用于做为主备节点角色判断的标准可以通过脚本实现：

用脚本执行是否成功做条件，如果成功增加优先级，不成功就降低优先级      

```
vrrp_script NAME {      #先定义一个脚本
        
        }
 
vrrp_instance NAME{
        track_script {    #再调用
         
                    }
        }
```



示例：

```
vrrp_script chk_mt_down {           
    script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"  #检测到down文件以错误码1退出，检测不到down文件以0退出，这样实现了，有down脚本执行失误，没down执行成功。
    interval 1  #监控间隔时间 
    weight -15  #当健康检查脚本失败后主机权重将会-15
    fall 2 #失败次数  
    rise 1 #成功数次  
}
 
vrrp_instance NAME{
        track_script {
                chk_mt_down
                }
            }
```



这个脚本实现了手动将节点在MASTER和BACKUP之间切换而不用关闭节点|关闭节点的网络|节点上的keepalived。

（2）健康状态检测方法

HTTP_GET|SSL_GET|TCP_CHECK|SMTP_CHECK|MISC_CHECK

HTTP方式：

```
HTTP_GET {
         url {
             path /index.html
             status_code 200
            }
          connect timeout 3
          nb_get_retry 3         #总共重试次数
          delay_before_retry 3   #每次重试延迟时间
        ｝
```



TCP方式

```
TCP_CHECK {
connect_ip 192.168.100.173  #要检测的后端主机ip
connect_port 80             
#bindto 192.168.100.179         #检测发送报文用的ip
connect_timeout 4
}
```



 SMTP方式这个可以用来给邮件服务器做集群

```
SMTP_CHECK
host {
connect_ip <IP ADDRESS>
connect_port <PORT>                                     #默认检查25端口
14 KEEPALIVED
bindto <IP ADDRESS>
}
connect_timeout <INTEGER>
retry <INTEGER>
delay_before_retry <INTEGER>
# "smtp HELO"|·-ê§à"
helo_name <STRING>|<QUOTED-STRING>
} #SMTP_CHECK
MISC方式
这个
```



可以用来检查很多服务器只需要自己会些脚本即可

```
MISC_CHECK
{
misc_path <STRING>|<QUOTED-STRING> #外部程序或脚本
misc_timeout <INT>                                    #脚本或程序执行超时时间
```



misc_dynamic                    #这个就很好用了可以非常精确的来调整权重是后端每天服务器的压力都能均衡调配这个主要是通过执行的程序或脚本返回的状态代码来动态调整weight值使权重根据真实的后端压力来适当调整不过这需要有过硬的脚本功夫才行哦
\#返回0健康检查没问题不修改权重
\#返回1健康检查失败权重设置为0
\#返回2-255健康检查没问题但是权重却要根据返回代码修改为返回码-2例如如果程序或脚本执行后返回的代码为200#那么权重这回被修改为 200-2
}

（3）nopreempt：设置不抢占，这里只能设置在state为backup的节点上，而且这个节点的优先级必须别另外的高
​          preempt delay：抢占延迟

**3、keepalived的服务脚本配置文件详解**

**开启keepalived的日志：**

编辑/etc/sysconfig/keepalived：

**KEEPALIVED_OPTIONS="-D -S 0"**

编辑/etc/rsyslog.conf：

添加此行

local0.*                                        ``/var/log/keepalived``.log

重启rsyslog：

按上面配置后，keepalived会把日志记录到/var/log/keepalived.log，默认是记录在/var/log/message



**四、双主模式配置示例**

定义2个虚拟路由互为主备

适用于前端调度器LB)的高可用2个公网ip在DNS做2条A记录还实现了一定程度上的负载均衡任何一个调度器挂了也不影响客户访问

180节点

```
[root@BAIYU_180 keepalived]# cat keepalived.conf
! Configuration File for keepalived
 
global_defs {
   notification_email {
       xxj@192.168.100.180
   }
   notification_email_from xiexiaojun
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
vrrp_script chk_mt_down {
script "[ -f /etc/keepalived/down ] && exit 1 || exit 0"
interval 1
weight -15   
}
 
vrrp_instance VI_1 {   
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass vi111
    }
    virtual_ipaddress {
        192.168.100.41
    }
track_script {
chk_mt_down
}
}
 
vrrp_instance VI_2 {
    state BACKUP
    interface eth0
    virtual_router_id 52
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass vi222
    }
    virtual_ipaddress {
        192.168.100.51
    }
track_script {
chk_mt_down
}
}
```



179节点

```
[root@BAIYU_179 keepalived]# cat keepalived.conf
! Configuration File for keepalived
 
global_defs {
   notification_email {
       xxj@192.168.100.180
   }
   notification_email_from xiexiaojun
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
vrrp_script chk_mt_down {
script "[ -f /etc/keepalived/down ] && exit 1 || exit 0"
interval 1
weight -15
}
 
 
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass vi111
    }
    virtual_ipaddress {
        192.168.100.41
    }
track_script {
chk_mt_down
}
}
 
vrrp_instance VI_2 {
    state MASTER
    interface eth0
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass vi222
    }
    virtual_ipaddress {
        192.168.100.51
    }
track_script {
chk_mt_down
}
}
```



测试

180:

```
[root@BAIYU_180 keepalived]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:f9:cb:14 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.180/24 brd 192.168.100.255 scope global eth0
    inet 192.168.100.41/32 scope global eth0
    inet6 fe80::20c:29ff:fef9:cb14/64 scope link 
       valid_lft forever preferred_lft forever
[root@BAIYU_180 keepalived]# ls
keepalived.conf  keepalived.conf.orig
```



179:

```
[root@BAIYU_179 keepalived]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:53:f6:29 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.179/24 brd 192.168.100.255 scope global eth0
    inet 192.168.100.51/32 scope global eth0
    inet6 fe80::20c:29ff:fe53:f629/64 scope link 
       valid_lft forever preferred_lft forever
[root@BAIYU_179 keepalived]# ls
keepalived.conf  keepalived.conf.orig
```



在/etc/keepalived目录下创建down文件再查看

```
[root@BAIYU_179 keepalived]# touch down
[root@BAIYU_179 keepalived]# ls
down  keepalived.conf  keepalived.conf.orig
[root@BAIYU_179 keepalived]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:53:f6:29 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.179/24 brd 192.168.100.255 scope global eth0
    inet6 fe80::20c:29ff:fe53:f629/64 scope link 
       valid_lft forever preferred_lft forever
[root@BAIYU_179 keepalived]# ssh 192.168.100.180 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:f9:cb:14 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.180/24 brd 192.168.100.255 scope global eth0
    inet 192.168.100.41/32 scope global eth0
    inet 192.168.100.51/32 scope global eth0
    inet6 fe80::20c:29ff:fef9:cb14/64 scope link 
       valid_lft forever preferred_lft forever
```



从上面我们可以看到之前在179节点上的51ip已经成功转移到了180节点上。

我们通过179节点日志/var/log/message来查看ip转移的详细过程

```
Dec  2 23:50:04 BAIYU_179 Keepalived_vrrp[50770]: VRRP_Script(chk_mt_down) failed
Dec  2 23:50:05 BAIYU_179 Keepalived_vrrp[50770]: VRRP_Instance(VI_2) Entering FAULT STATE
Dec  2 23:50:05 BAIYU_179 Keepalived_vrrp[50770]: VRRP_Instance(VI_2) removing protocol VIPs.
Dec  2 23:50:05 BAIYU_179 Keepalived_vrrp[50770]: VRRP_Instance(VI_2) Now in FAULT state
Dec  2 23:50:05 BAIYU_179 Keepalived_healthcheckers[50769]: Netlink reflector reports IP 192.168.100.51 removed

```



180节点日志/var/log/message来查看ip转移的详细过程

```
Dec  2 23:50:15 BAIYU_180 Keepalived_vrrp[60506]: VRRP_Instance(VI_2) Transition to MASTER STATE
Dec  2 23:50:16 BAIYU_180 Keepalived_vrrp[60506]: VRRP_Instance(VI_2) Entering MASTER STATE
Dec  2 23:50:16 BAIYU_180 Keepalived_vrrp[60506]: VRRP_Instance(VI_2) setting protocol VIPs.
Dec  2 23:50:16 BAIYU_180 Keepalived_vrrp[60506]: VRRP_Instance(VI_2) Sending gratuitous ARPs on eth0 for 192.168.100.51
Dec  2 23:50:16 BAIYU_180 Keepalived_healthcheckers[60505]: Netlink reflector reports IP 192.168.100.51 added
Dec  2 23:50:21 BAIYU_180 Keepalived_vrrp[60506]: VRRP_Instance(VI_2) Sending gratuitous ARPs on eth0 for 192.168.100.51
```



此时再179节点上删除/etc/keepalived/down文件179节点又会夺回51ip

```
[root@BAIYU_179 keepalived]# rm down 
[root@BAIYU_179 keepalived]# ls
keepalived.conf  keepalived.conf.orig
[root@BAIYU_179 keepalived]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:53:f6:29 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.179/24 brd 192.168.100.255 scope global eth0
    inet 192.168.100.51/32 scope global eth0
    inet6 fe80::20c:29ff:fe53:f629/64 scope link 
       valid_lft forever preferred_lft forever
```

