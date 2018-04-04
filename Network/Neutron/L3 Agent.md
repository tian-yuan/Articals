## 开始之前

### 环境准备

以下环境读者可以任选其一

- 运行在 RHEL6.5 之上的 IBM Cloud Manager with OpenStack 4.2 或更新的版本
- 开源的 OpenStack Juno 版本或者更新的版本

## Neutron L3 agent 简介

L3 agent 即 neutron-l3-agent 服务。它是 Neutron 的一个 api extension，允许用户创建用于连接 2 层网络的虚机路由(router)。L3 agent 实现了 3 层网络数据转发和 NAT(Network Address Translation)。利用 Linux Network Namespaces，可以创建多个具有重合 IP 地址段的虚拟路由器，这样的话，每个虚拟路由器都具有自己的 namespace，这些 namspace 是基于 router 在 Neutron 中的 UUID 来命名的。

用户可以直接将部署出来的 Nova 实例连接至公共网络，但是这样做的缺点也很多。首先，如果需要使用多个网络，可能会需要多个网桥进行对应。其次网络的控制权，实际上是由公共网络(路由器、防火墙)决定的。

所以一般的使用场景是，创建多个由 Neutron 管理的 tenant network，再通过虚拟路由器连接至公共网络。这样仅需一个网桥连接公共网络，并且 Neutron 可以管理到 tenant network。在这个场景下，作为虚拟路由器的管理者，L3 agent 是必不可少的。

## Neutron L3 HA 的基本概念

在介绍 L3 的 HA 之前，有必要了解一下相关的基本概念。

### Router

L3 agent 中的 router 即虚拟路由器，可以实现多个 tenant network 的连通和 tenant network 至公网的连通。L3 agent 的主要作用就是管理 router，而 router 在实际处理三层网络通信。

### SNAT

SNAT(Source Network Address Translation)允许网络流量从 tenant network 通向 public network。SNAT 发生在 router 里，来自 tenant network 的数据包中的源 IP 会被替换成 public network 的 IP，并发送至公网。在接收到响应后，目的 IP 会被替换成 tenant network 的 IP，并发送回 tenant network。Neutron 中实现 SNAT 可以通过添加 floating IP 和设置 gateway 来使能。

### VIP

VIP(Virtual IP)，跟一般的 IP 地址不一样，VIP 不对应某一具体网络设备，可以根据实际需要在不同的网络设备做迁移。这样，不管后台设备如何变化，VIP 提供的地址始终是不变的。

### VRRP

VRRP(Virtual Router Redundancy Protocol)是一个计算机网络协议，它能够自动选择可用的路由路径，从而提高了路由路径的可用性和稳定性。Neutron L3 的 HA 就是基于 VRRP 实现的。

## Neutron L3 HA 的实现过程

Neutron L3 HA 的提出就是为了保证 OpenStack 环境三层网络的高可用性。Neutron 中的 DVR(Distributed Virtual Router) 也能提高 OpenStack 三层网络的高可用性，为了描述简单明了，本文不考虑 DVR，并且假设 DVR 的功能在本文的环境中没有使能。

从前面的描述可以看出，实际处理网络通信的是虚拟路由器 router，而非 L3 agent，所以 Neutron L3 HA 实际上是针对 router 级别的 HA。接下来详细介绍一个 HA router 是如何工作的，先假设环境中有两个 Neutron L3 agent，相应的 HA router 的结构如图 1 所示。

##### 图 1. HA router 结构

QR 和 QG 分别连接内网和外网，这是 router 本来就有的端口；HA 端口是 HA router 才特有的。一个非 HA router，只会存在于一个 Neutron L3 agent 所在的 Network node 中。而一个 HA router 会在多个 Neutron L3 agent 所在的 Network node 里创建 instance，这包括创建相应的 namespace 和端口。每个 HA router instance 里面的 QR 和 QG 都有相同的设备名和 MAC 地址(Media Access Control Address)。清单 1 展示了 Slave router instance 的网络设备信息；清单 2 展示了 Master router instance 的网络设备信息。

##### 清单 1. Slave router instance 的网络设备信息

`[root@xhh62 ~]# ip netns exec qrouter-8c0ab668-bde8-4331-9acd-702a444cfa96 ip addr``11: ha-ab50abda-d8:``link/ether fa:16:3e:d9:48:b5 brd ff:ff:ff:ff:ff:ff``inet 169.254.192.1/18 brd 169.254.255.255 scope global ha-ab50abda-d8``valid_lft forever preferred_lft forever``inet6 fe80::f816:3eff:fed9:48b5/64 scope link ``valid_lft forever preferred_lft forever``12: qg-6d65c2a1-9e:``link/ether fa:16:3e:41:40:e0 brd ff:ff:ff:ff:ff:ff``inet6 fe80::f816:3eff:fe41:40e0/64 scope link ``valid_lft forever preferred_lft forever``13: qr-fe5a9b5f-e0:``link/ether fa:16:3e:69:2d:c8 brd ff:ff:ff:ff:ff:ff``inet6 fe80::f816:3eff:fe69:2dc8/64 scope link ``valid_lft forever preferred_lft forever`

##### 清单 2. Master router instance 的网络设备信息

`[root@xhh33 ~]# ip netns exec qrouter-8c0ab668-bde8-4331-9acd-702a444cfa96 \`` ``ip addr ``12: ha-503d299f-4b:``link/ether fa:16:3e:43:75:66 brd ff:ff:ff:ff:ff:ff``inet 169.254.192.2/18 brd 169.254.255.255 scope global ha-503d299f-4b``valid_lft forever preferred_lft forever``inet6 fe80::f816:3eff:fe43:7566/64 scope link ``valid_lft forever preferred_lft forever``13: qg-6d65c2a1-9e:``link/ether fa:16:3e:41:40:e0 brd ff:ff:ff:ff:ff:ff``inet 10.6.2.101/22 scope global qg-6d65c2a1-9e``valid_lft forever preferred_lft forever``inet6 fe80::f816:3eff:fe41:40e0/64 scope link tentative dadfailed ``valid_lft forever preferred_lft forever``14: qr-fe5a9b5f-e0:``link/ether fa:16:3e:69:2d:c8 brd ff:ff:ff:ff:ff:ff``inet 192.168.1.1/24 scope global qr-fe5a9b5f-e0``valid_lft forever preferred_lft forever``inet6 fe80::f816:3eff:fe69:2dc8/64 scope link ``valid_lft forever preferred_lft forever`

从清单 1 和清单 2 可以看出，Slave 和 Master router instance 的区别在于 Master router instance 的网络端口上是有 IP 地址的。这些 IP 实际上是 VIP，它们只会存在于 Master router instance 上。

从清单 1 和清单 2 还可以看出，每个 router instance 上都有一个 HA 端口，这些端口都有不同的 IP。这些端口是用来进行 VRRP 广播通信的。从图 1 中可以看出，每个 router instance 里面都运行着一个 keepalived 进程。Keepalived 是一个实现了 VRRP 的软件工具，Neutron 利用 keepalived 实现的 L3 HA，有关 keepalived 的介绍，见参考资源。Master router instance 中的 keepalived 进程会通过 HA 端口广播信息。清单 3 展示了这些广播信息。

##### 清单 3. keepalived 的广播信息

`[root@xhh33 ~]# ip netns exec qrouter-8c0ab668-bde8-4331-9acd-702a444cfa96 \`` ``tcpdump -ni ha-503d299f-4b``06:58:47.157080 IP 169.254.192.2 > 224.0.0.18: VRRPv2, Advertisement,`` ``vrid 1, prio 50, authtype none, intvl 2s, length 20`

从清单 3 可以看出，HA 端口向广播地址 224.0.0.18 发送信息。这是由 VRRP 协议规定的。如果 Master router instance 出现故障，不能发出广播信息，导致 Slave router instance 未在一定的时间内收到广播信息。剩下的 Slave router instance 会选取出一个来作为新的 Master router instance。这个 instance 会获取 VIP，从而为 OpenStack 提供三层网络。虽然提供服务的 router instance 变了，但是 IP 没有改变，所以从使用者的角度来看，网络服务没有发生改变。

清单 3 中的广播信息里，vrid 是 HA router 对应的 id，每个 tenant 下面，不同的 router 对应不同的 vrid。需要注意的是，由于 VRRP 协议的限制，vrid 是一个 8 bit 数据。因此每个 tenant 下面，最多只能创建 255 个 HA router。prio 是 instance 的优先级，这个目前是不可配置的，每个 instance 的优先级一样。authtype 是 instance 之间通信的认证方式，默认是不需要认证。intvl 是广播之间的间隔时间，默认是 2 秒，这个可以配置，在后面会说明。

前面已经讲过，每个 HA router 一旦被创建，Neutron L3 agent 会为每个 router instance 创建一个 keepalived 进程，在这之前会先生成一个 keepalived 的配置文件供 keepalived 的进程使用。清单 4 展示了一个这样的配置文件。

##### 清单 4. keepalived 的配置文件

`vrrp_sync_group VG_1 {``group {``VR_1``}``notify_master "/var/lib/neutron/ha_confs/8c0ab668-bde8-4331-9acd-702a444cfa96/notify_master.sh"``notify_backup "/var/lib/neutron/ha_confs/8c0ab668-bde8-4331-9acd-702a444cfa96/notify_backup.sh"``notify_fault "/var/lib/neutron/ha_confs/8c0ab668-bde8-4331-9acd-702a444cfa96/notify_fault.sh"``}``vrrp_instance VR_1 {``state BACKUP``interface ha-503d299f-4b``virtual_router_id 1``priority 50``nopreempt``advert_int 2``track_interface {``ha-503d299f-4b``}``virtual_ipaddress {``10.6.2.101/22 dev qg-6d65c2a1-9e``}``virtual_ipaddress_excluded {``192.168.1.1/24 dev qr-fe5a9b5f-e0``}``virtual_routes {``0.0.0.0/0 via 10.6.0.1 dev qg-6d65c2a1-9e``}``}`

从清单 4 可以看出，配置文件中记录了 HA router 所需要的全部信息。因此，就算是因为突发情况导致 Master router instance 出现故障而不能使用，Slave router instance 也能恢复所有的信息。

总的来说，要使用 HA router，首先必须有多个 Neutron L3 agent 在不同的 Network node 上运行。一旦创建了一个 HA router，Neutron 会通过多个 L3 agent 创建相应的 namespace 和端口，这样就有了这个 HA router 的多个 instance。同时，L3 agent 还会创建 keepalived 进程，每个 keepalived 进程都有 router 的全部信息。这样，每个 Network node 上都有 HA router 的一个 instance 和管理这个 router instance 的 keepalived 进程。

当一切就绪之后，这些 router instance 会进行一次选举，胜出的作为 Master instance，剩下的作为 Slave instance。由于所有的 router instance 的优先级都是一样的，选举的结果是随机的。也就是说，如果创建多个 HA router，这些 router 的 Master instance 可能分布在多个 Network node 上。

选举结束后，Master router instance 负责提供 L3 网络服务，并同时向其他的 Slave router instance 发广播报告自身的状况。一旦由于各种原因，广播中断，Slave router instance 会重新选举出新的 Master router instance，继续提供 L3 网络服务，并同时发送广播报告自身的状况。

## 功能验证

本章节主要说明如何配置 Neutron L3 HA，并手动触发一次故障，以验证 router 的 HA。

### 配置

Neutron L3 HA 的配置不多，分布在/etc/neutron/neutron.conf 和/etc/neutron/l3_agent.ini 里面。如清单 5 所示。

##### 清单 5. Neutron L3 HA 的配置项

`l3_ha = True``max_l3_agents_per_router = 3``min_l3_agents_per_router = 2``l3_ha_net_cidr = 169.254.192.0/18``ha_confs_path = $state_path/ha_confs``ha_vrrp_auth_type = PASS``ha_vrrp_auth_password =``ha_vrrp_advert_int = 2`

下面分别对这些配置项加以说明。

- l3_ha， 如果设置成 True，创建的 router 默认是 HA router。
- max_l3_agents_per_router，HA router 最多在几个 L3 agent 上创建 instance，如果这个数值大于或等于现有的 L3 agent 数目，则 HA router 会在所有的 L3 agent 上创建 instance。将这个值设置成 0 有同样的效果。
- min_l3_agents_per_router，限制了创建 HA router 时，必须要求环境中具备至少这么多个 L3 agent。如果环境中的 L3 agent 数目不够，则 HA router 的创建会有问题。
- l3_ha_net_cidr，前面说过的 HA 端口的 IP 所在的网段。
- ha_confs_path，Neutron L3 agent 生成的与 HA 相关的配置文件，例如清单 4 所展示的 keepalived 的配置文件，所在的路径。
- ha_vrrp_auth_type，ha_vrrp_auth_password，前面说过的广播通信的加密方式。
- ha_vrrp_advert_int，前面说过的广播通信的间隔时间，单位是秒。

在这些配置项里面，除了 l3_ha 和 max_l3_agents_per_router，其他的用默认值就能工作。修改完配置项，重启 neutron-server 和 neutron-l3-agent 服务即可。

### 验证结果

首先确保有足够的 Neutron L3 agent，创建一个 HA router，并为其添加 gateway 和 interface。此时就能看到与清单 1，清单 2，清单 3 所展示的相似的内容。

将 Master router instance 所在的 Network node 关机，先看看 keepalived 的广播信息变化。

##### 清单 6. 主节点关机后 keepalived 广播信息的变化

`[root@xhh62 ~]#ip netns exec qrouter-8c0ab668-bde8-4331-9acd-702a444cfa96 \ ``tcpdump -ni ha-ab50abda-d8``08:29:40.168626 IP 169.254.192.2 > 224.0.0.18: VRRPv2, Advertisement,`` ``vrid 1, prio 50, authtype none, intvl 2s, length 20``08:29:46.974636 IP 169.254.192.1 > 224.0.0.18: VRRPv2, Advertisement,``vrid 1, prio 50, authtype none, intvl 2s, length 20`

可以看出，关机之后，等待了 3 次都未收到 Master instance 发来的广播信息，原来的 Slave instance 变成了 Master instance，并开始向外发送广播信息。

按照之前的描述，VIP 现在也应该转移到了新的 Master instance 上了。清单 7 展示了相应的网络设备。

##### 清单 7. 新的 Master router instance 的网络设备

`[root@xhh62 ~]# ip netns exec qrouter-8c0ab668-bde8-4331-9acd-702a444cfa96 \``ip addr``11: ha-ab50abda-d8:``link/ether fa:16:3e:d9:48:b5 brd ff:ff:ff:ff:ff:ff``inet 169.254.192.1/18 brd 169.254.255.255 scope global ha-ab50abda-d8``valid_lft forever preferred_lft forever``inet6 fe80::f816:3eff:fed9:48b5/64 scope link ``valid_lft forever preferred_lft forever``12: qg-6d65c2a1-9e:``link/ether fa:16:3e:41:40:e0 brd ff:ff:ff:ff:ff:ff``inet 10.6.2.101/22 scope global qg-6d65c2a1-9e``valid_lft forever preferred_lft forever``inet6 fe80::f816:3eff:fe41:40e0/64 scope link ``valid_lft forever preferred_lft forever``13: qr-fe5a9b5f-e0:``link/ether fa:16:3e:69:2d:c8 brd ff:ff:ff:ff:ff:ff``inet 192.168.1.1/24 scope global qr-fe5a9b5f-e0``valid_lft forever preferred_lft forever``inet6 fe80::f816:3eff:fe69:2dc8/64 scope link ``valid_lft forever preferred_lft forever`

从清单 7 可以看出，VIP 已经完成了迁移。此时，可以发现使用 router 提供的 L3 网络服务正常。

## 总结

HA(High Availability)，高可用性对于提供稳定的服务至关重要。OpenStack Neutron 从 juno 版本开始支持了 L3 的 HA，为 L3 网络的稳定性提供了保障。虽然目前还不是非常完善，但是社区正在努力改进，相信在将来其功能会更加丰富和完整。

#### 相关主题

- [Linux Network Namespaces](http://www.opencloudblog.com/?p=42)：有关 Linux Network Namespaces 的介绍。

- [Basic L3 operations](http://docs.openstack.org/admin-guide-cloud/content/l3_workflow.html)：OpenStack 社区中有关 L3 agent 基本操作的介绍。

- [Neutron L3 HA](https://wiki.openstack.org/wiki/Neutron/L3_High_Availability_VRRP)：OpenStack 社区有关 Neutron L3 HA 的介绍。

- [VRRP](http://datatracker.ietf.org/wg/vrrp/documents/)：VRRP 协议的详细内容。

- [keepalived](http://www.keepalived.org/)：有关 keepalived 的详细介绍。

- developerWorks 云计算站点

   

  提供了有关云计算的更新资源，包括

  - 云计算 [简介](http://www.ibm.com/developerworks/cn/cloud/newto.html)。
  - 更新的 [技术文章和教程，以及网络广播](http://www.ibm.com/developerworks/cn/cloud/resources.html)，让您的开发变得轻松，[专家研讨会和录制会议](http://www.ibm.com/developerworks/cn/cloud/events.html) 帮助您成为高效的云开发人员。
  - 连接转为云计算设计的 [IBM 产品下载和信息](http://www.ibm.com/developerworks/cn/cloud/products.html)。
  - 关于 [社区最新话题](http://www.ibm.com/developerworks/cn/cloud/collaborate.html) 的活动聚合。