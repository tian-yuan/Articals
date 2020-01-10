# ZooKeeper ACL权限控制

 

[ 2017-05-11 ]

 

[ 回到首页 ]

> 这两天研究了一下 ZooKeeper ACL（Access Control List），总的来说ZooKeeper的权限管理机制还不是很好用。

------

# 权限特性

1. ZooKeeper的权限控制是基于每个znode节点的，需要对每个节点设置权限
2. 每个znode支持设置多种权限控制方案和多个权限
3. 子节点不会继承父节点的权限，客户端无权访问某节点，但可能可以访问它的子节点

------

# 权限类型

| 权限     | ACL简写 | 描述               |
| ------ | ----- | ---------------- |
| CREATE | c     | 可以创建子节点          |
| DELETE | d     | 可以删除子节点（仅下一级节点）  |
| READ   | r     | 可以读取节点数据及显示子节点列表 |
| WRITE  | w     | 可以设置节点数据         |
| ADMIN  | a     | 可以设置节点访问控制列表权限   |

------

# 访问控制列表方案（ACL Schemes）

ZooKeeper内置了一些权限控制方案，可以用以下方案为每个节点设置权限：

| 方案     | 描述                      |
| ------ | ----------------------- |
| world  | 只有一个用户：anyone，代表所有人（默认） |
| ip     | 使用IP地址认证                |
| auth   | 使用已添加认证的用户认证            |
| digest | 使用“用户名:密码”方式认证          |

------

# 权限相关命令

| 命令      | 使用方式                    | 描述      |
| ------- | ----------------------- | ------- |
| getAcl  | getAcl <path>           | 读取ACL权限 |
| setAcl  | setAcl <path> <acl>     | 设置ACL权限 |
| addauth | addauth <scheme> <auth> | 添加认证用户  |

------

# World方案

## 设置方式

```
setAcl <path> world:anyone:<acl>
```

## 客户端示例

```
[zk: localhost:2181(CONNECTED) 0] create /node1 1
Created /node1

[zk: localhost:2181(CONNECTED) 1] getAcl /node1
'world,'anyone  #默认为world方案
: cdrwa #任何人都拥有所有权限

#可以用以下方式设置：
[zk: localhost:2181(CONNECTED) 2] setAcl /node1 world:anyone:cdrwa
cZxid = 0x19000002a1
ctime = Thu May 11 22:00:00 CST 2017
mZxid = 0x19000002a1
mtime = Thu May 11 22:00:00 CST 2017
pZxid = 0x19000002a1
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0
```

------

# IP方案

## 设置方式

```
setAcl <path> ip:<ip>:<acl>
```

<ip>：可以是具体IP也可以是IP/bit格式，即IP转换为二进制，匹配前bit位，如`192.168.0.0/16`匹配`192.168.*.*`

## 客户端示例

```
[zk: localhost:2181(CONNECTED) 0] create /node2 1
Created /node2

[zk: localhost:2181(CONNECTED) 1] setAcl /node2 ip:192.168.100.1:cdrwa #设置IP：192.168.100.1 拥有所有权限
cZxid = 0x1900000239
ctime = Thu May 11 22:00:00 CST 2017
mZxid = 0x1900000239
mtime = Thu May 11 22:00:00 CST 2017
pZxid = 0x1900000239
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0

[zk: localhost:2181(CONNECTED) 2] getAcl /node2
'ip,'192.168.100.1
: cdrwa

#使用IP非 192.168.100.1 的机器
[zk: localhost:2181(CONNECTED) 0] get /node2
Authentication is not valid : /node2 #没有权限

[zk: localhost:2181(CONNECTED) 1] delete /node2 #删除成功（因为设置DELETE权限仅对下一级子节点有效，并不包含此节点）
```

------

# Auth方案

## 设置方式

```
addauth digest <user>:<password> #添加认证用户
setAcl <path> auth:<user>:<acl>
```

## 客户端示例

```
[zk: localhost:2181(CONNECTED) 0] create /node3 1
Created /node3

[zk: localhost:2181(CONNECTED) 1] addauth digest yoonper:123456 #添加认证用户

[zk: localhost:2181(CONNECTED) 2] setAcl /node3 auth:yoonper:cdrwa
cZxid = 0x19000002b8
ctime = Thu May 11 22:00:00 CST 2017
mZxid = 0x19000002b8
mtime = Thu May 11 22:00:00 CST 2017
pZxid = 0x19000002b8
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0

[zk: localhost:2181(CONNECTED) 3] getAcl /node3
'digest,'yoonper:UvJWhBril5yzpEiA2eV7bwwhfLs=
: cdrwa

[zk: localhost:2181(CONNECTED) 4] get /node3
1 #刚才已经添加认证用户，可以直接读取数据，断开会话重连需要重新addauth添加认证用户
cZxid = 0x1900000418
ctime = Thu May 11 22:00:00 CST 2017
mZxid = 0x1900000418
mtime = Thu May 11 22:00:00 CST 2017
pZxid = 0x1900000418
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0
```

------

# Digest方案

## 设置方式

```
setAcl <path> digest:<user>:<password>:<acl>
```

这里的密码是经过SHA1及BASE64处理的密文，在SHELL中可以通过以下命令计算：

```
echo -n <user>:<password> | openssl dgst -binary -sha1 | openssl base64
```

先来算一个密文密码：

```
echo -n yoonper:123456 | openssl dgst -binary -sha1 | openssl base64
UvJWhBril5yzpEiA2eV7bwwhfLs=
```

## 客户端示例

```
[zk: localhost:2181(CONNECTED) 0] create /node4 1
Created /node4

#使用是上面算好的密文密码添加权限：
[zk: localhost:2181(CONNECTED) 1] setAcl /node4 digest:yoonper:UvJWhBril5yzpEiA2eV7bwwhfLs=:cdrwa
cZxid = 0x19000002e3
ctime = Thu May 11 22:00:00 CST 2017
mZxid = 0x19000002e3
mtime = Thu May 11 22:00:00 CST 2017
pZxid = 0x19000002e3
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0

[zk: localhost:2181(CONNECTED) 2] getAcl /node4
'digest,'yoonper:UvJWhBril5yzpEiA2eV7bwwhfLs=
: cdrwa

[zk: localhost:2181(CONNECTED) 3] get /node4
Authentication is not valid : /node4 #没有权限

[zk: localhost:2181(CONNECTED) 4] addauth digest yoonper:123456 #添加认证用户

[zk: localhost:2181(CONNECTED) 5] get /node4
1 #成功读取数据
cZxid = 0x1900000420
ctime = Thu May 11 22:00:00 CST 2017
mZxid = 0x1900000420
mtime = Thu May 11 22:00:00 CST 2017
pZxid = 0x1900000420
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0
```

------

# 参考文档

> [ZooKeeper Programmer's Guide - ZooKeeper access control using ACLs](http://zookeeper.apache.org/doc/r3.4.10/zookeeperProgrammers.html#sc_ZooKeeperAccessControl)

