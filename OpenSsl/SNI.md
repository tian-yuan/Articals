# 一、什么是SNI？

​    SNI是Server Name Indication的缩写，是为了解决一个服务器使用多个域名和证书的SSL/TLS扩展。它允许客户端在发起SSL握手请求时（客户端发出ClientHello消息中）提交请求的HostName信息，使得服务器能够切换到正确的域并返回相应的证书。

​    在SNI出现之前，HostName信息只存在于HTTP请求中，但SSL/TLS层无法获知这一信息。通过将HostName的信息加入到SNI扩展中，SSL/TLS允许服务器使用一个IP为不同的域名提供不同的证书，从而能够与使用同一个IP的多个“虚拟主机”更方便地建立安全连接。

# 二、RFC中SNI的定义

​    RFC 6066《Transport Layer Security (TLS) Extensions: Extension Definitions》对SNI扩展做了详细的定义。要点如下：

1)    Client需要在ClientHello中包含一个名为”server_name”的扩展，这个扩展的”extension_data”域中需要包含”ServerNameList”；

2)    ServerNameList中包含了多个HostName及其类型（name_type），但所有HostName的name_type不能相同（早期的RFC规范允许一个name_type多个HostName，但  实际上当前的client实现只发送一个HostName，而且client不一定知道server选择了哪个HostName，因此禁止一个name_type多个HostName）；

3)    HostName中包含的是server的完全合格的DNS主机名，且HostName中不允许包含IPv4或IPv6地址；

4)    如果server收到的ClientHello中带有”server_name”扩展，它也应该在ServerHello中包含一个”server_name”扩展，其中的”extension_data”域应为空；

5)    当执行会话恢复时，clinet应该在ClientHello中包含与上次会话相同的”server_name”扩展。如果扩展中包含的name与上次的不同，server必须拒绝恢复会话。恢复会话时server必须不能在ServerHello中包含”server_name”扩展；

6)    如果一个应用程序使用应用层协议协商了一个server name然后升级到TLS，并且发送了”server_name”扩展，这个扩展中必须包含与在应用层协议中所协商的相同的server name。如果这个server name成功应用在了TLS会话中，client不应该在应用层尝试请求一个不同的server name。



https://blog.csdn.net/u011130578/article/details/77979325