# keepalived高可用调度器配置详解

一、VRRP概述

 1.VRRP协议

​        虚拟路由冗余协议(Virtual Router Redundancy Protocol，简称VRRP)是由IETF提出的解决局域网中配置静态网关出现单点失效现象的路由协议，1998年已推出正式的RFC2338协议标准。VRRP广泛应用在边缘网络中，它的设计目标是支持特定情况下IP数据流量失败转移不会引起混乱，允许主机使用单路由器，以及及时在实际第一跳路由器使用失败的情形下仍能够维护路由器间的连通性。

 

  2.vrrp术语

| 参考：H3C VRRP技术白皮书 |                                          |
| ---------------- | ---------------------------------------- |
| 虚拟路由器            | 由一个Master路由器和多个Backup路由器组成。主机将虚拟路由器当作默认网关。 |
| VRID             | 虚拟路由器的标识。有相同VRID的一组路由器构成一个虚拟路由器          |
| Master路由器        | 虚拟路由器中承担报文转发任务的路由器，主节点(仅能有一个)            |
| Backup路由器        | Master路由器出现故障时，能够代替Master路由器工作的路由器，备用节点(可以有多个) |
| 虚拟IP地址(VIP)      | 虚拟路由器的IP地址。已改为虚拟路由器可以拥有一个或多个IP地址         |
| IP地址拥有者          | 接口IP地址与虚拟IP地址相同的路由器被称为IP地址拥有者            |
| 虚拟MAC地址(VMAC)    | 一个虚拟路由器拥有一个虚拟MAC地址。虚拟路由器回应APR请求使用的是细腻MAC地址，只有虚拟路由器做特护配置的时候，才会回应接口的真实MAC地址 |
| 优先级              | VRRP根据优先级来确定虚拟路由器中每台路由器的地位               |
| 非抢占方式            | 若Backup路由器工作在非抢占方式下，只要Master路由器每出现故障，Backup即使随后被配置更高的优先级也不会成为Master路由器 |
| 抢占方式             | 若Backup路由器工作在抢占方式下，当收到VRRP报文后，会将自己的优先级与通告报文中的优先级进行比较。若当前的Master路由器的优先级高，就会主动抢占成为Master路由器；否则保持Backup状态 |

​       

  3.工作过程

​      (1) 虚拟路由器中的路由器根据优先级选举出Master。Master路由器通过发送ARP报文，将自己的虚拟MAC地址通知给与它连接的设备或者主机，从而承担报文转发任务。

​      (2) Master路由器周期性发送VRRP报文，以公布其配置信息(优先级等)和工作状况

​      (3) 若Master路由器出现故障，虚拟路由器中的Backup路由器将根据优先级重新选举新的Master

​      (4) 虚拟路由器状态切换时，Master路由器由一台设备切换为另一台设备，新的Master路由器知识简单地发送一个携带虚拟路由器的MAC地址和虚拟IP地址信息的ARP此昂管信息。网络中的主机感知不到Master路由器已经切换为另外一台设备

​      (5) Backup路由器的优先级高于Master路由器时，由Backup路由器的工作方式(抢占和非抢占)决定是否重新选举Master

 

 4.VRRP功能

​      (1)Master路由器选举

​      (2)Master路由器状态通告

​      (3)VRRP认证功能

 

 5.VRRP高可用工作模型

​    (1)主备备份

​              业务仅由Master路由器承担。当Master路由器出现故障时，才会由选举出来的Backup路由器接替它工作

[![wKiom1bLK0GTS9zuAADpvBWgHoo138.png](http://s5.51cto.com/wyfs02/M02/7B/52/wKiom1bLK0GTS9zuAADpvBWgHoo138.png)](http://s5.51cto.com/wyfs02/M02/7B/52/wKiom1bLK0GTS9zuAADpvBWgHoo138.png)

​    (2)主主备份

​            在路由器的一个接口上可创建多个虚拟路与其，使得该路由器可以在一个虚拟路由器中作为Master路与其，同时在其他的虚拟路由器中做为Backup路由器。主主备份方式能实现负载分担的功能

[![wKioL1bLK7qxkbLTAAEFmYOX17k545.png](http://s5.51cto.com/wyfs02/M02/7B/52/wKioL1bLK7qxkbLTAAEFmYOX17k545.png)](http://s5.51cto.com/wyfs02/M02/7B/52/wKioL1bLK7qxkbLTAAEFmYOX17k545.png)

 

 

 

二、keepalived高可用调度器

 1.keepalived功能

​         keepalived程序是vrrp协议在Linux主机上以守护进程方式的实现。能够根据配置文件生成ipvs规则，并对各Real Server的健康做检测，以及LoadBalance主机和BackUP主机之间failover的实现

​         keepalived在CentOS 6.4+收录到发行版光盘内，佛在需要编译安装或者第三方RPM包    

​                                                                                                                                                                                                                                                                                                                                                                                               

  2.程序组件

​          核心程序、IO复用器、内存管理、配置文件分析器

[![wKiom1bLK2ihNvzeAAL6DM_rc1w417.png](http://s1.51cto.com/wyfs02/M00/7B/52/wKiom1bLK2ihNvzeAAL6DM_rc1w417.png)](http://s1.51cto.com/wyfs02/M00/7B/52/wKiom1bLK2ihNvzeAAL6DM_rc1w417.png)

| 核心组件具体如下：           |                                  |
| ------------------- | -------------------------------- |
| Watch  dog          | 高可用监视器                           |
| checkers            | 健康状态检测器(TCP、HTTP、SSL、MISC…  ...) |
| smtp                | 支持发送邮件通告机制                       |
| System  call        | 支持系统调用机制，作出管理操作                  |
| vrrp stack          | VRRP栈的实现，实现VRRP协议调用              |
| Netlink  Reflectior | VRRP借助于Netlink监控网络，实现网络功能配置      |
| ipvs wrapper        | ipvs控制                           |

 

 3.keepalived高可用集群配置前提  

​         (1)各节点时间要同步；必须不能超过一秒，一般使用网络偶时间服务器(ntp server)

​         (2)确保iptables及selinux不会成为障碍；

​         (3)各节点之间可通过主机名互相通信；节点的名称设定与hosts文件中解析的主机名都要保持一致；

​     #uname -n 获得的主机，与解析的主机名要相同；

​         (4)各节点之间基于密钥认证的方式通过ssh互信通信；

   说明：第三条和第四条非必须 

三、keepalived程序环境配置

  1.主配置文件：/etc/keepalived/keepalived.conf

​     (1)GLOBALCONFIGURATION：全局配置

global_defs          # Block id

{

...

}

​    常用配置：

| notification_email  {  …  } | 收件人邮箱地址                |
| --------------------------- | ---------------------- |
| notification_email_from     | 发件人邮箱地址                |
| smtp_server                 | 邮件发送服务器IP；             |
| smtp_connect_timeout        | 邮件服务器建立连接的超时时长；默认30秒   |
| router_id  LVS_DEVEL        | 物理节点的标识符；建立使用主机名；      |
| vrrp_mcast_group4           | IPV4多播地址，默认224.0.0.18； |

​     (2)VRRPDCONFIGURATION：配置vrrp实例

vrrp instance：虚拟路由器

vrrp_instance NAME {

...

}

vrrp synchronization group

vrrp_sync_group NAME  {

...

}

​     1)常用配置：

| state    MASTER\|BACKUP | 在当前VRRP实例中此节点的初始状态；             |
| ----------------------- | ------------------------------- |
| interface   IFACE_NAME  | vrrp用于绑定vip的接口；                 |
| virtual_router_id  #    | 当前VRRP实例的VRID，可用范围为0-255，默认为51； |
| priority #              | 当前节点的优先级，可用范围0-255；             |
| advert_int 1            | 通告时间间隔；默认1秒                     |

​    2)认证方式配置：

authentication{     # Authentication block

\#PASS||AH

\#PASS - Simple Passwd (suggested)

\#AH - IPSEC (not recommended))

auth_typePASS

\#Password for accessing vrrpd.

\#should be the same for all machines.

\#Only the first eight (8) characters are used.

auth_pass1234

}

​    3)虚拟IP配置：

virtual_ipaddress{

<IPADDR>/<MASK>brd <IPADDR> dev <STRING> scope <SCOPE> label <LABEL>

}

​       (3)LVSCONFIGURATION：ipvs的相关配置集群服务，服务内的RS；

 

  2.双主备份VRRP配置实例

​          第一步：node1节点配置

**vrrp_instanceVI_1 {**

**stateMASTER**

**interfaceeno16777736**

**virtual_router_id101**

**priority100**

**advert_int1**

**authentication{**

**auth_typePASS**

**auth_passZPNnTQ6F**

**}**

**virtual_ipaddress{**

**172.16.100.9/16**

**}**

**}**

**vrrp_instanceVI_2 {**

**stateBACKUP**

**interfaceeno16777736**

**virtual_router_id102**

**priority99**

**advert_int1**

**authentication{**

**auth_typePASS**

**auth_passIWyijM5Q**

**}**

**virtual_ipaddress{**

**172.16.100.10/16**

**}**

}                                

​          第二步：node2节点配置

**vrrp_instanceVI_1 {**

**stateBACKUP**

**interfaceeno16777736**

**virtual_router_id101**

**priority99**

**advert_int1**

**authentication{**

**auth_typePASS**

**auth_passZPNnTQ6F**

**}**

**virtual_ipaddress{**

**172.16.100.9/16**

**}**

**}**

**vrrp_instanceVI_2 {**

**stateMASTER**

**interfaceeno16777736**

**virtual_router_id102**

**priority100**

**advert_int1**

**authentication{**

**auth_typePASS**

**auth_passIWyijM5Q**

**}**

**virtual_ipaddress{**

**172.16.100.10/16**

**}**

**}  **

****

****

四、keepalived通知脚本检测维护服务

 1.VRRP配置段定义邮件通知

​       在vrrp_instance中notify_master、notify_backup、notify_fault、notify定义出处理脚路径和传递参数。一般notify_master、notify_backup、notify_fault为一组联合使用，或者notify单独定义使用。

vrrp_instance{

...

notify_master  <STRING>|<QUOTED-STRING>

notify_backup  <STRING>|<QUOTED-STRING>

notify_fault  <STRING>|<QUOTED-STRING>

notify  <STRING>|<QUOTED-STRING>

}        

示例脚本：

\#!/bin/bash

\# Description: An example of notify script

\#

contact='root@localhost'

 

notify() {

mailsubject="$(hostname) to be $1: vipfloating"

mailbody="$(date +'%F %H:%M:%S'): vrrptransition, $(hostname) changed to be $1"

echo $mailbody | mail -s "$mailsubject"$contact

}

case $1 in

master)

notify master

exit 0

;;

backup)

notify backup

exit 0

;;

fault)

notify fault

exit 0

;;

*)

echo "Usage: $(basename $0){master|backup|fault}"

exit 1

;;

esac

 

  2.定义外部脚本来检测高可用功能依赖到的资源的监控

​       第一步：在global_defs中定义脚本

vrrp_script NAME  {

script

interval  #

weight  #

}

​       第二步：在vrrp_instance实例追踪定义脚本，用于作为实例的当前节点的监控机制

track_script {

NAME

}

实例：当检测到down文件、监测不到nginx服务是转移

vrrp_script  chk_down {

script  "[[ -f /etc/keepalived/down ]]&& exit 1 || exit 0"

interwval 1

weight -2

}

vrrp_script  chk_nginx {

script  "killall -0 nginx "

interwval 2

weight -5

}

track_script {

chk_down

chk_nginx

}

注意：其转移的方式和服务本身配置没有直接关系

  3.监控关注的网络接口

track_interface {

IFACE_NAME

}

| nopreempt           | 使用非抢占模式  |
| ------------------- | -------- |
| preempt_delay  TIME | 使用延迟抢占模式 |

 

 

五、keepalived高可用ipvs服务

  1.定义集群服务方法

方法一：此方法内部要使用 protocol 指定协议

virutal_server vip  port  {

...

}

方法二：

virtual_server fwmark int  {

...

}

​        常用的参数：

| delay_loop  #                            | 每延迟多久探测一次     |
| ---------------------------------------- | ------------- |
| lb_algo   rr\|wrr\|lc\|wlc\|lblc\|sh\|dh | 定义调度算法        |
| lb_kind  NAT\|DR\|TUN                    | 定义集群类型        |
| persistence_timeout  #                   | 持久连接时长        |
| protocol TCP                             | 支持的传输协议       |
| sorry_server  <IPADDR>   <PORT>          | 所有RS错误，返回用户信息 |

 

 2.定义RS的方法(应用上下文virutal_server)

real_server <IPADDR> <PORT>  {

...

}

​      常用的参数：

| weight   #                             | 指明当前RS权重 |
| -------------------------------------- | -------- |
| notify_up  <STRING>\|<QUOTED-STRING>   | 定义up脚本   |
| notify_down  <STRING>\|<QUOTED-STRING> | 定义down脚本 |

 

 

六、keepalived健康状态检测机制

  1.web应用层检测

HTTP_GET|SSL_GET

{

...

}

检测参数：

| url  {path  <STRING>status_code  <INT>digest  <STRING>} | 检测URL路径期望返回状态指纹信息(注意status_code和digest有其一即可) |
| ---------------------------------------- | ---------------------------------------- |
| nb_get_retry  <INT>                      | get请求的重试次数                               |
| delay_before_retry  <INT>                | 两次重试之间的时间间隔；                             |
| connect_timeout  <INTEGER>               | 连接超时时长，默认为5s；                            |
| warmup <INT>                             | 健康状态检测延迟；                                |

  2.传输层检测（tcp协议层）

TCP_CHECK{

...}

检测参数：connect_timeout  <INTEGER>

connect_ip<IP ADDRESS>

connect_port<PORT>

bindto<IP ADDRESS>

bind_port<PORT>

