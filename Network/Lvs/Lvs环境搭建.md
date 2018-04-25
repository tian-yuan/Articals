### 环境部署

* 创建一个虚拟机
* 开启多个docker容器作为后端服务



| 角色               | IP地址           |                        |
| ---------------- | -------------- | ---------------------- |
| Directory Server | 192.168.99.100 | Mac virtual machine    |
| Real Server      | 172.17.0.2     | httpd docker container |
| Test Machine     | 192.168.3.102  | Mac                    |



Mac 笔记本上面安装了一个 ubuntu 虚拟机，虚拟机上安装 LVS，部署 NAT 模式，虚拟机上再安装 docker-ce，启动两个 docker httpd 容器，当做 RealServer；

> 注意，虚拟机上可以直接访问 docker 容器里面的服务，此次测试的 docker 容器，里面的 httpd 在 80 端口监听，

```
虚拟机能访问 docker 容器是因为虚拟机里面存在一条路由记录 ： 
172.17.0.0/16 dev docker0  proto kernel  scope link  src 172.17.0.1

虚拟机 docker0 网桥：
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:3f:8a:86:5c brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:3fff:fe8a:865c/64 scope link
       valid_lft forever preferred_lft forever
```



```
tianyuan@tianyuan-VirtualBox:~$ docker ps
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                   NAMES
fe33015ea250        httpd               "httpd-foreground"   11 hours ago        Up 11 hours         0.0.0.0:18080->80/tcp   affectionate_thompson
tianyuan@tianyuan-VirtualBox:~$
```





```
tianyuan@tianyuan-VirtualBox:~$ sudo ipvsadm -L -n
[sudo] password for tianyuan:
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.99.100:80 rr
  -> 172.17.0.2:80                Masq    1      0          0
  -> 192.168.3.102:18280          Masq    1      0          0
tianyuan@tianyuan-VirtualBox:~$
```

```
在 Test Machine 上 telnet 192.168.99.100 80，然后查询 Test Machine 上的连接情况如下：
➜  ~ netstat -na | grep tcp | grep 80 | grep 192.168.99.100
tcp4       0      0  192.168.99.1.63606     192.168.99.100.80      ESTABLISHED
➜  ~

在 httpd docker 容器里面的连接情况如下
root@fe33015ea250:/usr/local/apache2# netstat -na | grep tcp | grep 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
tcp        0      0 172.17.0.2:80           192.168.99.1:63606      SYN_RECV

注意，这里的端口是不变的，都是 63606，LVS 只是做了一个数据转发，把 DIP 改成了目的主机的 IP
```

> NAT 模式下，Real Server 必须能把数据包返回给 LVS，一般情况下，Real Server 都是把 LVS 当做默认网关，LVS 和 Real Server 组成一个局域网

