### 构建C1000K的服务器(1) – 基础

著名的 [C10K 问题](http://www.kegel.com/c10k.html)提出的时候, 正是 2001 年, 到如今 12 年后的 2013 年, C10K 已经不是问题了, 任何一个普通的程序员, 都能利用手边的语言和库, 轻松地写出 C10K 的服务器. 这既得益于软件的进步, 也得益于硬件性能的提高.

现在, 该是考虑 [C1000K](http://www.ideawu.net/blog/tag/c100k), 也就是百万连接的问题的时候了. 像 Twitter, weibo, Facebook 这些网站, 它们的同时在线用户有上千万, 同时又希望消息能接近实时地推送给用户, 这就需要服务器能维持和上千万用户的 TCP 网络连接, 虽然可以使用成百上千台服务器来支撑这么多用户, 但如果每台服务器能支持一百万连接(C1000K), 那么只需要十台服务器.

有很多技术声称能解决 [C1000K](http://www.ideawu.net/blog/tag/c100k) 问题, 例如 Erlang, Java NIO 等等, 不过, 我们应该首先弄明白, 什么因素限制了 C1000K 问题的解决. 主要是这几点:

1. [操作系统能否支持百万连接?](http://www.ideawu.net/blog/archives/740.html#q1)
2. [操作系统维持百万连接需要多少内存?](http://www.ideawu.net/blog/archives/740.html#q2)
3. [应用程序维持百万连接需要多少内存?](http://www.ideawu.net/blog/archives/740.html#q3)
4. [百万连接的吞吐量是否超过了网络限制?](http://www.ideawu.net/blog/archives/740.html#q4)

下面来分别对这几个问题进行分析.

### [1. 操作系统能否支持百万连接?](http://www.ideawu.net/blog/archives/740.html#q1)

对于绝大部分 [Linux](http://www.ideawu.net/blog/category/linux) 操作系统, 默认情况下确实不支持 C1000K! 因为操作系统包含最大打开文件数(Max Open Files)限制, 分为系统全局的, 和进程级的限制.

#### 全局限制

在 Linux 下执行:

```
cat /proc/sys/fs/file-nr
```

会打印出类似下面的一行输出:

```
5100	0	101747
```

第三个数字 `101747` 就是当前系统的全局最大打开文件数(Max Open Files), 可以看到, 只有 10 万, 所以, 在这台服务器上无法支持 C1000K. 很多系统的这个数值更小, 为了修改这个数值, 用 root 权限修改 /etc/sysctl.conf 文件:

```
fs.file-max = 1020000
net.ipv4.ip_conntrack_max = 1020000
net.ipv4.netfilter.ip_conntrack_max = 1020000
```

需要重启系统服务生效:

```
# Linux
$ sudo sysctl -p /etc/sysctl.conf 
# BSD
$ sudo /etc/rc.d/sysctl reload
```

### 进程限制

执行:

```
ulimit -n
```

输出:

```
1024
```

说明当前 Linux 系统的每一个进程只能最多打开 1024 个文件. 为了支持 C1000K, 你同样需要修改这个限制.

**临时修改**

```
ulimit -n 1020000
```

不过, 如果你不是 root, 可能不能修改超过 1024, 会报错:

```
-bash: ulimit: open files: cannot modify limit: Operation not permitted
```

**永久修改**

编辑 /etc/security/limits.conf 文件, 加入如下行:

```
# /etc/security/limits.conf
work         hard    nofile      1020000
work         soft    nofile      1020000
```

第一列的 `work` 表示 work 用户, 你可以填 `*`, 或者 `root`. 然后保存退出, 重新登录服务器.

注意: Linux 内核源码中有一个常量(NR_OPEN in /usr/include/linux/fs.h), 限制了最大打开文件数, 如 RHEL 5 是 1048576(2^20), 所以, 要想支持 [C1000K](http://www.ideawu.net/blog/tag/c1000k), 你可能还需要重新编译内核.

### [2. 操作系统维持百万连接需要多少内存?](http://www.ideawu.net/blog/archives/740.html#q2)

解决了操作系统的参数限制, 接下来就要看看内存的占用情况. 首先, 是操作系统本身维护这些连接的内存占用. 对于 Linux 操作系统, socket(fd) 是一个整数, 所以, 猜想操作系统管理一百万个连接所占用的内存应该是 4M/8M, 再包括一些管理信息, 应该会是 100M 左右. 不过, 还有 socket 发送和接收缓冲区所占用的内存没有分析. 为此, 我写了最原始的 C 网络程序来验证:

#### 服务器

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <arpa/inet.h>
#include <netinet/tcp.h>
#include <sys/select.h>

#define MAX_PORTS 10

int main(int argc, char **argv){
    struct sockaddr_in addr;
    const char *ip = "0.0.0.0";
    int opt = 1;
    int bufsize;
    socklen_t optlen;
    int connections = 0;
    int base_port = 7000;
    if(argc > 2){
        base_port = atoi(argv[1]);
    }

    int server_socks[MAX_PORTS];

    for(int i=0; i<MAX_PORTS; i++){
        int port = base_port + i;
        bzero(&addr, sizeof(addr));
        addr.sin_family = AF_INET;
        addr.sin_port = htons((short)port);
        inet_pton(AF_INET, ip, &addr.sin_addr);

        int serv_sock;
        if((serv_sock = socket(AF_INET, SOCK_STREAM, 0)) == -1){
            goto sock_err;
        }
        if(setsockopt(serv_sock, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt)) == -1){
            goto sock_err;
        }
        if(bind(serv_sock, (struct sockaddr *)&addr, sizeof(addr)) == -1){
            goto sock_err;
        }
        if(listen(serv_sock, 1024) == -1){
            goto sock_err;
        }

        server_socks[i] = serv_sock;
        printf("server listen on port: %d\n", port);
    }

    //optlen = sizeof(bufsize);
    //getsockopt(serv_sock, SOL_SOCKET, SO_RCVBUF, &bufsize, &optlen);
    //printf("default send/recv buf size: %d\n", bufsize);

    while(1){
        fd_set readset;
        FD_ZERO(&readset);
        int maxfd = 0;
        for(int i=0; i<MAX_PORTS; i++){
            FD_SET(server_socks[i], &readset);
            if(server_socks[i] > maxfd){
                maxfd = server_socks[i];
            }
        }
        int ret = select(maxfd + 1, &readset, NULL, NULL, NULL);
        if(ret < 0){
            if(errno == EINTR){
                continue;
            }else{
                printf("select error! %s\n", strerror(errno));
                exit(0);
            }
        }

        if(ret > 0){
            for(int i=0; i<MAX_PORTS; i++){
                if(!FD_ISSET(server_socks[i], &readset)){
                    continue;
                }
                socklen_t addrlen = sizeof(addr);
                int sock = accept(server_socks[i], (struct sockaddr *)&addr, &addrlen);
                if(sock == -1){
                    goto sock_err;
                }
                connections ++;
                printf("connections: %d, fd: %d\n", connections, sock);
            }
        }
    }

    return 0;
sock_err:
    printf("error: %s\n", strerror(errno));
    return 0;
}
```

注意, 服务器监听了 10 个端口, 这是为了测试方便. 因为只有一台客户端测试机, 最多只能跟同一个 IP 端口创建 30000 多个连接, 所以服务器监听了 10 个端口, 这样一台测试机就可以和服务器之间创建 30 万个连接了.

#### 客户端

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <arpa/inet.h>
#include <netinet/tcp.h>

int main(int argc, char **argv){
    if(argc <=  2){
        printf("Usage: %s ip port\n", argv[0]);
        exit(0);
    }

    struct sockaddr_in addr;
    const char *ip = argv[1];
    int base_port = atoi(argv[2]);
    int opt = 1;
    int bufsize;
    socklen_t optlen;
    int connections = 0;

    bzero(&addr, sizeof(addr));
    addr.sin_family = AF_INET;
    inet_pton(AF_INET, ip, &addr.sin_addr);

    char tmp_data[10];
    int index = 0;
    while(1){
        if(++index >= 10){
            index = 0;
        }
        int port = base_port + index;
        printf("connect to %s:%d\n", ip, port);

        addr.sin_port = htons((short)port);

        int sock;
        if((sock = socket(AF_INET, SOCK_STREAM, 0)) == -1){
            goto sock_err;
        }
        if(connect(sock, (struct sockaddr *)&addr, sizeof(addr)) == -1){
            goto sock_err;
        }

        connections ++;
        printf("connections: %d, fd: %d\n", connections, sock);

        if(connections % 10000 == 9999){
            printf("press Enter to continue: ");
            getchar();
        }
        usleep(1 * 1000);
        /*
           bufsize = 5000;
           setsockopt(serv_sock, SOL_SOCKET, SO_SNDBUF, &bufsize, sizeof(bufsize));
           setsockopt(serv_sock, SOL_SOCKET, SO_RCVBUF, &bufsize, sizeof(bufsize));
         */
    }

    return 0;
sock_err:
    printf("error: %s\n", strerror(errno));
    return 0;
}
```

我测试 10 万个连接, 这些连接是空闲的, 什么数据也不发送也不接收. 这时, 进程只占用了不到 1MB 的内存. 但是, 通过程序退出前后的 free 命令对比, 发现操作系统用了 200M(大致)内存来维护这 10 万个连接! 如果是百万连接的话, 操作系统本身就要占用 2GB 的内存! 也即 2KB 每连接.

可以修改

```
/proc/sys/net/ipv4/tcp_wmem
/proc/sys/net/ipv4/tcp_rmem
```

来控制 TCP 连接的发送和接收缓冲的大小(多谢 @[egmkang](http://www.cnblogs.com/egmkang)).

### [3. 应用程序维持百万连接需要多少内存?](http://www.ideawu.net/blog/archives/740.html#q3)

通过上面的测试代码, 可以发现, 应用程序维持百万个空闲的连接, 只会占用操作系统的内存, 通过 ps 命令查看可知, 应用程序本身几乎不占用内存.

### [4. 百万连接的吞吐量是否超过了网络限制?](http://www.ideawu.net/blog/archives/740.html#q4)

假设百万连接中有 20% 是活跃的, 每个连接每秒传输 1KB 的数据, 那么需要的网络带宽是 0.2M x 1KB/s x 8 = 1.6Gbps, 要求服务器至少是万兆网卡(10Gbps).

### 总结

Linux 系统需要修改内核参数和系统配置, 才能支持 C1000K. C1000K 的应用要求服务器至少需要 2GB 内存, 如果应用本身还需要内存, 这个要求应该是至少 10GB 内存. 同时, 网卡应该至少是万兆网卡.

当然, 这仅仅是理论分析, 实际的应用需要更多的内存和 CPU 资源来处理业务数据.

### 测试工具

测试操作系统最大连接数的工具: <https://github.com/ideawu/c1000k>



For a TCP connection memory consumed depends on

1. size of sk_buff (internal networking structure used by linux kernel)
2. the read and write buffer for a connection

the size of buffers can be tweaked as required

```
root@x:~# sysctl -A | grep net | grep mem
```

check for these variables

these specify the maximum default memory buffer usage for all network connections in kernel

```
net.core.wmem_max = 131071

net.core.rmem_max = 131071

net.core.wmem_default = 126976

net.core.rmem_default = 126976
```

these specify buffer memory usage specific to tcp connections

```
net.ipv4.tcp_mem = 378528   504704  757056

net.ipv4.tcp_wmem = 4096    16384   4194304

net.ipv4.tcp_rmem = 4096    87380   4194304
```

the three values specified are " min default max" buffer sizes. So to start with linux will use the default values of read and write buffer for each connection. As the number of connection increases , these buffers will be reduced [at most till the specified min value] Same is the case for max buffer value.

These values can be set using this `sysctl -w  KEY=KEY VALUE`

eg. The below command ensures the read and write buffers for each connection are 4096 each.

```
sysctl -w net.ipv4.tcp_rmem='4096 4096 4096'

sysctl -w net.ipv4.tcp_wmem='4096 4096 4096'
```