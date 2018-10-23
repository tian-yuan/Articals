# VRRP原理和分析

![96](https://upload.jianshu.io/users/upload_avatars/252139/df17f19e9494.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96)

 

[lemonTreeTop](https://www.jianshu.com/u/31568b13d808)

 

**关注

2017.06.27 18:48* 字数 1059 阅读 247评论 0喜欢 2

在配置路由器和防火墙的实验中都出现过需要配置VRRP，那什么VRRP，VRRP有什么用呢，VRRP是如何实现的？

#### VRRP基本概念

VRRP（Virtual Router Redundancy Protocol，虚拟路由器冗余协议）将可以承担网关功能的一组路由器加入到备份组中，形成一台虚拟路由器，这样主机的网关设置成虚拟网关，就能够实现冗余。

#### VRRP原理

VRRP将局域网内的一组路由器划分在一起，称为一个备份组。备份组由一个Master路由器和多个Backup路由器组成，功能上相当于一台虚拟路由器。
VRRP备份组具有以下特点：

• 虚拟路由器具有IP地址，称为虚拟IP地址。局域网内的主机仅需要知道这个虚拟路由器的IP地址，并将其设置为缺省路由的下一跳地址。

• 网络内的主机通过这个虚拟路由器与外部网络进行通信。

• 备份组内的路由器根据优先级，选举出Master路由器，承担网关功能。其他路由器作为Backup路由器，当Master路由器发生故障时，取代Master继续履行网关职责，从而保证网络内的主机不间断地与外部网络进行通信。

#### 典型组网分析：主备方式

![img](https://upload-images.jianshu.io/upload_images/252139-784504007d72d12b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/487)

sec_eudemon_ag_ha_0006_fig01.png

NGFW_A与 NGFW_B组成一组路由器，虚拟成一个虚拟路由器 。但是为什么会有两组VRRP备份组呢？备份组由什么构成的？

虚拟路由器（Virtual Router）：又称 VRRP 备份组，由一个 Master 设备和多个
Backup 设备组成，被当作一个共享局域网内主机的缺省网关

图中VRRP备份组1由NGFW_A与 NGFW_B的下面接口组成，NGFW_A与 NGFW_B的下面接口都有IP，但是内网PC网关是10.1.1.1，而不是这两台防火墙的下面接口其中的一个IP，10.1.1.1这个IP是这两台防火墙组成的备份组所虚拟IP，也就是说这个IP映射了多个防火墙，这样当其中一台防火墙出现故障的时候，另一台防火墙可以继续转发业务流量。备份组2也类似，对外的IP为虚拟IP。这就实际相当于虚拟的防火墙，如图

![img](https://upload-images.jianshu.io/upload_images/252139-8b61cc28bdaa02bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/228)

BaiduShurufa_2017-6-27_14-27-54.png

#### VRRP工作流程分析

![img](https://upload-images.jianshu.io/upload_images/252139-128871a542836f47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

BaiduShurufa_2017-6-27_14-35-2.png

VRRP 备份组中的交换机根据优先级选举出 Master。Master 交换机通过发送免费 ARP
报文，将虚拟 MAC 地址通知给与它连接的设备或者主机，从而承担报文转发任务。

Master 交换机周期性向备份组内所有 Backup 交换机发送 VRRP 通告报文，以公布其配
置信息（优先级等）和工作状况。

如果 Master 交换机出现故障，VRRP 备份组中的 Backup 交换机将根据优先级重新选举
新的 Master。

VRRP 备份组状态切换时，Master 交换机由一台设备切换为另外一台设备，新的
Master 交换机会立即发送携带虚拟路由器的虚拟 MAC 地址和虚拟 IP 地址信息的免费
ARP 报文，刷新与它连接的主机或设备中的 MAC 表项，从而把用户流量引到新的
Master 交换机上来，整个过程对用户完全透明。

原 Master 交换机故障恢复时，若该设备为 IP 地址拥有者（优先级为 255），将直接切
换至 Master 状态。若该设备优先级小于 255，将首先切换至 Backup 状态，且其优先级
恢复为故障前配置的优先级。

Backup 交换机的优先级高于 Master 交换机时，由 Backup 交换机的工作方式（抢占方
式和非抢占方式）决定是否重新选举 Master

参考资料：
VRRP 技术白皮书[http://enterprise.huawei.com/ilink/cnenterprise/download/HW_201057](https://link.jianshu.com/?t=http://enterprise.huawei.com/ilink/cnenterprise/download/HW_201057)