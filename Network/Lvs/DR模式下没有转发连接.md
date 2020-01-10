#### LVS_DR 安装后无法转发真实服务器，但是配置其他方面都检查的没有问题了。就剩在realserver这边没有在lo口上绑定VIP了



 lvs+keepalived+nginx一共四台机器，两台nginx做后端服务器，lvs+keepalive在nginx两台，后端的服务器服务都开着的，keepalived的服务也是开着的，两台keepalive的iptables也是关了的，selinux也已经关闭了，但是访问虚拟ip的时候，出现 curl :(7) couldn't connet to host，但是在另一台机器是能ping通的。

解决的办法是在realserver的lo口上绑定vip，最后问题得到解决。    也是配置rr算法没有正确的得到应验的解决办法。   解决了lvs不能转发到realserver的问题同时也解决了lvs的rr算法的问题

[root@realserver1 ~]    ifconfig lo:0   192.168.1.111(vip)   broadcast  192.168.1.111(vip)   netmask 255.255.255.255 up

[root@realserver1 ~]    route add -host  192.168.1.111(vip) dev lo:0

[root@realserver2 ~]    ifconfig lo:0    192.168.1.111(vip)    broadcast   192.168.1.111(vip)   netmask 255.255.255.255 up

[root@realserver2 ~]    route add -host   192.168.1.111(vip)   dev lo:0



在 lvs 节点上通过 ipvsadm -Ln 也能看到对应的记录，但是就是没有连接转发

在 dummy0 上加上这个 vip 就正常了，本质上的原因应该是 vrrp 在广播的时候，后端节点会校验这个 vip



