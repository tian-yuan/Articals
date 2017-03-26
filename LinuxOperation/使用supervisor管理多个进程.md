#使用supervisor管理多个进程

##创建Dockerfile

```
parallels@ubuntu:~$ cat ./Dockerfile
FROM ubuntu:14.04

MAINTAINER Quinten Krijger "https://github.com/Krijger"

ENV DEBIAN_FRONTEND noninteractive

##### make sure the package repository is up to date and update ubuntu
RUN \
  sed -i 's/# \(.*multiverse$\)/\1/g' /etc/apt/sources.list && \
  apt-get update && \
  apt-get -y upgrade && \
  locale-gen en_US.UTF-8
RUN apt-get install -y curl git htop man software-properties-common unzip vim wget
RUN apt-get install -y openssh-server

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL en_US.UTF-8
ENV HOME /root

RUN mkdir -p /var/run/sshd		#sshd 需要使用的目录
RUN mkdir -p /var/log/supervisor	#supervisor 需要使用的目录

##### supervisor installation &&
##### create directory for child images to store configuration in
RUN apt-get -y install supervisor && \
  mkdir -p /var/log/supervisor && \
  mkdir -p /etc/supervisor/conf.d

##### supervisor base configuration
ADD supervisor.conf /etc/supervisor.conf	#添加本地文件到容器目录

##### default command
CMD ["supervisord", "-c", "/etc/supervisor.conf"] #在容器控制台执行supervisord
```

Dockerfile 的 reference ： <https://docs.docker.com/engine/reference/builder/>

>Dockerfile 可以在任意目录，如果在其他目录，那么创建镜像时需要制定 Dockerfile 的文件路径。eg:docker build -f /path/to/a/Dockerfile .

##创建镜像

```
parallels@ubuntu:~$ sudo docker build -t test/supervisord .
[sudo] password for parallels:
Sending build context to Docker daemon 743.9 MB
Step 1 : FROM ubuntu:14.04
 ---> 0ccb13bf1954
Step 2 : MAINTAINER Quinten Krijger "https://github.com/Krijger"
 ---> Using cache
 ---> 90a490bec11a
Step 3 : ENV DEBIAN_FRONTEND noninteractive
 ---> Using cache
 ---> f4e19906484d
Step 4 : RUN sed -i 's/# \(.*multiverse$\)/\1/g' /etc/apt/sources.list &&   apt-get update &&   apt-get -y upgrade &&   locale-gen en_US.UTF-8
 ---> Using cache
 ---> 12dd4b789445
Step 5 : RUN apt-get install -y curl git htop man software-properties-common unzip vim wget
 ---> Using cache
 ---> 473b845ec7d0
Step 6 : RUN apt-get install -y openssh-server
 ---> Using cache
 ---> 5ecc104db427
Step 7 : ENV LANG en_US.UTF-8
 ---> Using cache
 ---> 5f96a71a59bd
Step 8 : ENV LANGUAGE en_US.UTF-8
 ---> Using cache
 ---> dd1a57d6284f
Step 9 : ENV LC_ALL en_US.UTF-8
 ---> Using cache
 ---> 5946247cd650
Step 10 : ENV HOME /root
 ---> Using cache
 ---> d8f301cee4ed
Step 11 : RUN mkdir -p /var/run/sshd
 ---> Using cache
 ---> ba044446573c
Step 12 : RUN mkdir -p /var/log/supervisor
 ---> Using cache
 ---> 25a48b15194a
Step 13 : RUN apt-get -y install supervisor &&   mkdir -p /var/log/supervisor &&   mkdir -p /etc/supervisor/conf.d
 ---> Using cache
 ---> efdd8c8f4ace
Step 14 : ADD supervisor.conf /etc/supervisor.conf
 ---> Using cache
 ---> e49f4acd4cc1
Step 15 : CMD supervisord -c /etc/supervisor.conf
 ---> Using cache
 ---> 5befb09a866a
Successfully built 5befb09a866a
```

>实际操作这个指令时，会安装相应的程序，创建的镜像和sudo docker pull 下来的镜像都保存在 /var/lib/docker 下面

##启动容器

```
parallels@ubuntu:~$ sudo docker run -p 22 -t -i test/supervisord
2016-08-22 06:38:03,333 CRIT Supervisor running as root (no user in config file)
2016-08-22 06:38:03,338 INFO supervisord started with pid 1
2016-08-22 06:38:04,340 INFO spawned: 'sshd' with pid 8
2016-08-22 06:38:05,343 INFO success: sshd entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
```

##测试容器中的 ssh 服务

```
parallels@ubuntu:~$ ssh 172.17.0.3
The authenticity of host '172.17.0.3 (172.17.0.3)' can't be established.
ECDSA key fingerprint is 44:e0:7c:0d:49:78:6a:d6:1d:eb:33:ea:fe:1d:03:ee.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.17.0.3' (ECDSA) to the list of known hosts.
parallels@172.17.0.3's password:
```