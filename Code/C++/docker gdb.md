#### DOCKER 容器支持 gdb 调试

在 mac 上启动的 docker 容器，使用 gdb 调试会提示无权限问题。

需要设置相关参数才能启动 gdb 调试

##### 问题根因

主要问题是容器启动时没有使能 TRACE 能力，可以修改容器的配置来修改启动的容器的配置信息，也可以通过添加参数重新启动一个容器

添加容器启动参数的方式：

```
docker run --cap-add=SYS_PTRACE --security-opt seccomp=unconfined --security-opt apparmor=unconfined
```

下面为修改配置信息的方式

##### 登录docker内部的linux

```
cd ~/Library/Containers/com.docker.docker/Data/vms/0/
```

在这个目录下面，有一个`tty`的文件，通过这个文件我们能够登录到docker内部的linux界面，然后使用下面命令进行登录：

```
screen tty
```

遇到空白命令行，直接回车即可。

##### 编辑config.v2.json

```
cat /var/lib/docker/containers/5f29020b36c0971da5d50825f4132d0eaf40f1e5efaac02eadab23239ca77d7d/config.v2.json
```

具体效果如下图：

![获得config.v2.json原内容](https://oscimg.oschina.net/oscnet/679ec86bbf2b55356f22404a853926376f8.jpg)

其中 CapAdd=["SYS_PTRACE"], SecurityOpt=["seccomp=unconfined", "apparmor=unconfined"]

##### linux 系统下

1.Stop Container:

```
docker stop yourcontainer;
```

2.Get container id:

```
docker inspect yourcontainer;
```

3.Modify hostconfig.json(default docker path:/var/lib/docker, you can change yours)

```
vim /var/lib/docker/containers/containerid/hostconfig.json
```

4.Search "CapAdd", and modify null to ["NET_ADMIN"];

```
....,"VolumesFrom":null,"CapAdd":["NET_ADMIN"],"CapDrop":null,....
```

5.Restart docker in host machine;

```
service docker restart;
```

6.Start yourconatiner;

```
docker start yourcontainer;
```

和 mac 后面部分相同

