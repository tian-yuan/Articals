#部署 Docker Registry 服务

【编者的话】本文阐释了怎样部署私有的 Docker Registry 服务 —— 或为公司私用，或公开给其他用户使用。例如，你公司可能需要私人的 Registry 来支持持续集成（CI）。又或，你的公司可能有大量镜像方式的产品或服务，你想以公司品牌的方式来整体提供和呈现。

本文阐释了怎样部署私有的 Docker Registry 服务 —— 或为公司私用，或公开给其他用户使用。例如，你公司可能需要私人的 Registry 来支持持续集成（CI）。又或，你的公司可能有大量镜像方式的产品或服务，你想以公司品牌的方式来整体提供和呈现。

Docker 公共的 Registry 里维护有一个默认的 registry 镜像可以用以协助该部署过程。该 Registry 镜像对本地测试足矣，但不能用于生产环境。对于生产环境，应以 docker/distribution 为基础，自行配置并构建自定义 Registry 镜像。

>注意：本文中的例子在 Ubuntu 14.04 下编写及测试。如果你在不同的操作系统中运行Docker，你或许需要“翻译”这些命令，以适合你运行环境的需要。

##官方镜像下的简单示例

本节中，将创建一个 Container 来运行 Docker 的官方 Registry 镜像。你将推送（Push）一个镜像到这个 Registry 服务器，然后再从该 Registry 中拉取（Pull）同一个镜像。

这是个很好的练习，有助于理解客户端与本地 Registry 的基本交互。

*	安装 Docker。

*	从 Docker 公共 Registry 中运行 hello-world 镜像。

```	
$ docker run hello-world
run 命令自动从 Docker 的官方镜像库中将 hello-world 镜像 pull 下来。
```

*	在 localhost 上启动 Registry 服务。
	
```
$ docker run -p 5000:5000 registry:2.0
这将在 DOCKER_HOST 上启动一个 Registry 服务，并在 5000 端口监听。
```

>注意，网上很多教程里面使用 docker run -p 5000:5000 registry 命令安装 Registry 服务，这个本身是没有错的，但是这个安装的是1.0版本，不支持 HTTPS，会出现很多问题

*	列出镜像。

```
$ docker images
REPOSITORY     TAG     IMAGE ID      CREATED       VIRTUAL SIZE
registry       2.0     bbf0b6ffe923  3 days ago    545.1 MB
golang         1.4     121a93c90463  5 days ago    514.9 MB
hello-world    latest  e45a5af57b00  3 months ago  910 B
这个列表应当包括一个由先前运行而得来的 hello-world 镜像。
```

*	为本地 repoistory 重新标记 hello-world 镜像。

```
$ docker tag hello-world:latest localhost:5000/hello-mine:latest
此命令使用 [REGISTRYHOST/]NAME[:TAG] 格式为 hello-world:latest 重新打标。REGISTRYHOST 在此例中是 localhost。在 Mac OSX 环境中，得把 localhost 换成 $(boot2docker ip):5000。
```

*	列出新镜像。

```
$ docker images
REPOSITORY                  TAG          IMAGE ID      CREATED       VIRTUAL SIZE
registry                    2.0     bbf0b6ffe923  3 days ago    545.1 MB
golang                      1.4     121a93c90463  5 days ago    514.9 MB
hello-world                 latest  e45a5af57b00  3 months ago  910 B
localhost:5000/hello-mine   latest  ef5a5gf57b01  3 months ago  910 B
可以看到，新镜像已经出现在列表中。
```

*	推送新镜像到本地 Registry 中。

```
$ docker push localhost:5000/hello-mine:latest
The push refers to a repository [localhost:5000/hello-mine] (len: 1)
e45a5af57b00: Image already exists
31cbccb51277: Image successfully pushed
511136ea3c5a: Image already exists
Digest: sha256:a1b13bc01783882434593119198938b9b9ef2bd32a0a246f16ac99b01383ef7a
```

*	使用 curl 命令及 Docker Registry 服务 API v2 列出 Registry 中的镜像：

```
$ curl -v -X GET http://localhost:5000/v2/hello-mine/tags/list
* Hostname was NOT found in DNS cache
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 5000 (#0)
> GET /v2/hello-mine/tags/list HTTP/1.1
> User-Agent: curl/7.35.0
> Host: localhost:5000
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Docker-Distribution-Api-Version: registry/2.0
< Date: Sun, 12 Apr 2015 01:29:47 GMT
< Content-Length: 40
<
{"name":"hello-mine","tags":["latest"]}
* Connection #0 to host localhost left intact
```

也可以通过在浏览器中访问以下地址来获取这些信息：http://localhost:5000/v2/hello-mine/tags/list

*	从你的本地环境中移除所有未使用的镜像：
	
```	
	$ docker rmi -f $(docker images -q -a )
```

此命令仅用于说明目的；移除镜像强制 run 从 Registry 而不是从本地缓存拉取。如果在这之后运行docker images，在你的镜像列表中，应该看不到任何 hello-world 或 hello-mine 的实例。

```
$ docker images
REPOSITORY      TAG      IMAGE ID      CREATED       VIRTUAL SIZE
registry         2.0     bbf0b6ffe923  3 days ago    545.1 MB
golang           1.4     121a93c90463  5 days ago    514.9 MB
```

*	试运行 hello-mine。

```
$ docker run hello-mine
Unable to find image 'hello-mine:latest' locally
Pulling repository hello-mine
FATA[0001] Error: image library/hello-mine:latest not found
```

命令 run 运行失败，因为你的新镜像在 Docker 公共 Registry 中是不存在的。

*	现在，尝试指定镜像的 Registry 来运行镜像：
	
	$ docker run localhost:5000/hello-mine

如果你在这之后运行 docker images， 你会发现里面多了一个 hello-mine 的实例。

#使 Docker 官方 Registry 镜像做好生产环境准备

Docker 的官方镜像只为简单的测试或除错准备。其配置对多数生产环境来讲都不适用。例如，任何能访问服务器 IP 的客户端，都能推送及拉取镜像。参看下一节，获取使该镜像做好生产环境准备的信息。

##理解生产环境的部署

当部署一个用于生产环境发布的 Registry 时，须考虑如下因素：

*	BACKEND STORAGE 应在何处存储镜像？
*	ACCESS AND/OR AUTHENTICATION 用户是否应拥有全部或受控的访问权限？这取决于你为公众提供镜像服务，还是只为公司内部提供。
*	DEBUGGING 当问题或状况发生时，是否有解决这些它们的方法。日志由于可以看到问题动向，这使其很有用。
*	CACHING 快速提取镜像可能至关重要，如果依赖镜像进行测试、构建，或有其他自动化系统的话。

我们可以配置 Registry 功能特性，用以调整适配这些因素。可以在命令行里指定选项来干这个，或者更通常地，用一个 Registry 配置文件来完成此事。配置文件是 YAML 格式的。

```
Docker 的官方 Repository 镜像用以下配置文件做了预置：
version: 0.1
log:
level: debug
fields:
service: registry
environment: development
storage:
cache:
  layerinfo: inmemory
filesystem:
  rootdirectory: /tmp/registry-dev
http:
addr: :5000
secret: asecretforlocaldevelopment
debug:
  addr: localhost:5001
redis:
addr: localhost:6379
pool:
maxidle: 16
maxactive: 64
idletimeout: 300s
dialtimeout: 10ms
readtimeout: 10ms
writetimeout: 10ms
notifications:
endpoints:
  - name: local-8082
    url: http://localhost:5003/callback
    headers:
       Authorization: [Bearer <an example token>]
    timeout: 1s
    threshold: 10
    backoff: 1s
    disabled: true
  - name: local-8083
    url: http://localhost:8083/callback
    timeout: 1s
    threshold: 10
    backoff: 1s
    disabled: true
```

这个配置非常基本，可以看到这在生产环境下会有一些问题。例如，http 区块详述了运行 Registry 的主机的 HTTP 服务器配置，但服务器没有使用甚至是最低要求的传输层安全性（TLS）配置。接下来我们将配置这些东西。

##在 Registry 服务器上配置 TLS

在本节中，将在服务器上配置 TLS，使能通过 https 协议来通信。在服务器上启用 TLS 是在企业防火墙内运行 Registry 推荐的最低安全配置。达成这一目标的方法之一就是构建你自己的 Registry 镜像。

###下载 source 并生成 certificates

*	下载 Registry 源码。

	当然，也可以使用 git clone 命令。

*	把下载的包解压到本地目录。

	解压后创建 distribution 目录。

*	进入 distribution 目录。

	$ cd distribution

*	新建 certs 子目录。

	$ mkdir certs


*	使用 SSL 生成自签名证书。

	$ openssl req -newkey rsa:2048 -nodes -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt

	此命令将提示你回答一些基本的信息，用于创建证书。

*	列出 certs 目录的内容。
	
	$ ls certs

	domain.crt domain.key

	当你构建这个 Container 时，certs 目录及其内容也会自动被复制。

###将 TLS 加入配置

distribution 软件库在 cmd 子目录中包含有示例 Registry 配置。在本节中，您可以编辑这些配置之一，以添加 TLS 支持。

*	编辑 ./cmd/registry/config.yml 文件。

	$ vi ./cmd/registry/config.yml

*	定位到 http 区块。

```
http:
    addr: :5000
    secret: asecretforlocaldevelopment
    debug:
            addr: localhost:500
```

*	给服务器的自签名证书新增一个 tls 区块：

```
http:
    addr: :5000
    secret: asecretforlocaldevelopment
    debug:
            addr: localhost:5001
    tls:
        certificate: /go/src/github.com/docker/distribution/certs/domain.crt
        key: /go/src/github.com/docker/distribution/certs/domain.key
```

提供 Container 内到证书的路径。如果你希望跨 Layer 使用两步认证，那么你可以增加一个 clientcas 区块选项。

*	保存并关闭该文件。

	构建并运行你的 Registry 镜像

*	构建你的 Registry 镜像。

	$ docker build -t secure_registry .

*	运行你的新镜像。

```
$ docker run -p 5000:5000 registry_local:latest
time="2015-04-12T03:06:18.616502588Z" level=info msg="endpoint local-8082 disabled, skipping" environment=development instance.id=bf33c9dc-2564-406b-97c3-6ee69dc20ec6 service=registry
time="2015-04-12T03:06:18.617012948Z" level=info msg="endpoint local-8083 disabled, skipping" environment=development instance.id=bf33c9dc-2564-406b-97c3-6ee69dc20ec6 service=registry
time="2015-04-12T03:06:18.617190113Z" level=info msg="using inmemory layerinfo cache" environment=development instance.id=bf33c9dc-2564-406b-97c3-6ee69dc20ec6 service=registry
time="2015-04-12T03:06:18.617349067Z" level=info msg="listening on :5000, tls" environment=development instance.id=bf33c9dc-2564-406b-97c3-6ee69dc20ec6 service=registry
time="2015-04-12T03:06:18.628589577Z" level=info msg="debug server listening localhost:5001"
2015/04/12 03:06:28 http: TLS handshake error from 172.17.42.1:44261: remote error: unknown certificate authority
```

观察启动时的信息。你应该可以看到 tls 在运行。

*	使用 curl 验证可以通过 https 连接。

```
$ curl -v https://localhost:5000
* Rebuilt URL to: https://localhost:5000/
* Hostname was NOT found in DNS cache
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 5000 (#0)
* successfully set certificate verify locations:
*   CAfile: none
CApath: /etc/ssl/certs
* SSLv3, TLS handshake, Client hello (1):
* SSLv3, TLS handshake, Server hello (2):
* SSLv3, TLS handshake, CERT (11):
* SSLv3, TLS alert, Server hello (2):
* SSL certificate problem: self signed certificate
* Closing connection 0
curl: (60) SSL certificate problem: self signed certificate
More details here: http://curl.haxx.se/docs/sslcerts.html
```
配置适合 v1 及 v2 版本 Registry 的 Nginx

本节介绍了如何使用 docker-compose ，在 nginx 代理背后运行版本 1 和 2 并存的 Registry 服务。并存的 Registry 服务都用 localhost:5000 访问。如果 docker 客户端版本小于 1.6，那么 Nginx 将其请求路由到 1.0 版本的 Registry 服务。从更新版本客户端发来的请求，将路由到 2.0 版本的 Registry 服务。

此过程使用您在上面最后一个过程中创建的 distribution 目录。该目录包含有一个 compose 配置示例。

安装 Docker Compose

1、在你 distribution 目录所在主机上打开一个新的终端窗口。

2、获取 docker-compose 二进制可执行文件。
$ sudo wget https://github.com/docker/compose/releases/download/1.1.0/docker-compose-`uname -s`-`uname -m` -O /usr/local/bin/docker-compose

此命令将二进制可执行文件安装到 /usr/local/bin 目录。

3、添加可执行权限到二进制文件。
$ sudo chmod +x /usr/local/bin/docker-compose

做一些清理

1、移除早先的镜像。
$ docker rmi -f $(docker images -q -a )

该步骤是一个内部管理步骤。这可以防止你在这个例子里错误地选取了旧的镜像。

2、编辑 distribution/cmd/registry/config.yml 文件，并移除 tls 区块。

如果沿用了前面例子里的东西，你就会有一个 tls 区块。

3、保存变更并关闭文件。

配置 SSL

1、进入 distribution/contrib/compose/nginx 目录。

该目录包含了 Nginx 及 Registry 的配置文件。

2、使用 SSL 生成自签名证书。
$ openssl req \
     -newkey rsa:2048 -nodes -keyout domain.key \
     -x509 -days 365 -out domain.crt

此命令将提示你回答一些问题，供证书创建使用。

3、编辑 Dockerfile 并添加以下行。
COPY domain.crt /etc/nginx/domain.crt
COPY domain.key /etc/nginx/domain.key

当你全部搞完的时候，这个文件看上去像下面。
FROM nginx:1.7

COPY nginx.conf /etc/nginx/nginx.conf
COPY registry.conf /etc/nginx/conf.d/registry.conf
COPY docker-registry.conf /etc/nginx/docker-registry.conf
COPY docker-registry-v2.conf /etc/nginx/docker-registry-v2.conf
COPY domain.crt /etc/nginx/domain.crt
COPY domain.key /etc/nginx/domain.key

4、保存并关闭 Dockerfile 文件。

5、编辑 registry.conf 文件并增加以下配置。
ssl on;
ssl_certificate /etc/nginx/domain.crt;
ssl_certificate_key /etc/nginx/domain.key;

这是一个 nginx 配置文件。

6、保存并关闭 registry.conf 文件。

构建并运行

1、进到 distribution/contrib/compose 目录

此目录包含单一个 docker-compose.yml 配置。
nginx:
build: "nginx"
ports:
    - "5000:5000"
links:
    - registryv1:registryv1
    - registryv2:registryv2
registryv1:
image: registry
ports:
    - "5000"
registryv2:
build: "../../"
ports:
    - "5000"

此配置按 nginx/Dockerfile 所指定，构建一个新的 nginx 镜像。1.0 版本的 Registry 来自 Docker 的官方公开镜像。Registry 2.0 镜像将从你前面用到的 distribution/Dockerfile 来构建。

2、获取 Registry 1.0 镜像。
$ docker pull registry:0.9.1

Compose 的配置是在本地寻找此镜像。如果你不做这步，那后面的步骤会失败。

3、构建 nginx，Registry 2.0 镜像，以及
$ docker-compose build
registryv1 uses an image, skipping
Building registryv2...
Step 0 : FROM golang:1.4

...

Removing intermediate container 9f5f5068c3f3
Step 4 : COPY docker-registry-v2.conf /etc/nginx/docker-registry-v2.conf
---> 74acc70fa106
Removing intermediate container edb84c2b40cb
Successfully built 74acc70fa106

此命令将输出其执行过程，直到运行结束。

4、启动使用了 Compose 的配置。
$ docker-compose up
Recreating compose_registryv1_1...
Recreating compose_registryv2_1...
Recreating compose_nginx_1...
Attaching to compose_registryv1_1, compose_registryv2_1, compose_nginx_1
...

5、在另一个终端，显示运行中的配置。
$ docker ps
CONTAINER ID        IMAGE                       COMMAND                CREATED             STATUS              PORTS                                     NAMES
a81ad2557702        compose_nginx:latest        "nginx -g 'daemon of   8 minutes ago       Up 8 minutes        80/tcp, 443/tcp, 0.0.0.0:5000->5000/tcp   compose_nginx_1
0618437450dd        compose_registryv2:latest   "registry cmd/regist   8 minutes ago       Up 8 minutes        0.0.0.0:32777->5000/tcp                   compose_registryv2_1
aa82b1ed8e61        registry:latest             "docker-registry"      8 minutes ago       Up 8 minutes        0.0.0.0:32776->5000/tcp                   compose_registryv1_1

浏览一下

1、检查一下你 nginx 服务器上的 TLS。
$ curl -v https://localhost:5000
* Rebuilt URL to: https://localhost:5000/
* Hostname was NOT found in DNS cache
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 5000 (#0)
* successfully set certificate verify locations:
*   CAfile: none
CApath: /etc/ssl/certs
* SSLv3, TLS handshake, Client hello (1):
* SSLv3, TLS handshake, Server hello (2):
* SSLv3, TLS handshake, CERT (11):
* SSLv3, TLS alert, Server hello (2):
* SSL certificate problem: self signed certificate
* Closing connection 0
curl: (60) SSL certificate problem: self signed certificate
More details here: http://curl.haxx.se/docs/sslcerts.html

2、标记 v1 registry 镜像。
$ docker tag registry:latest localhost:5000/registry_one:latest

3、将其推送到 localhost。
$ docker push localhost:5000/registry_one:latest

如果你在使用 1.6 版本的 Docker 客户端，这将推送镜像到 v2 registry。

4、使用 curl 来列出 Registry 中的镜像。
$ curl -v -X GET http://localhost:32777/v2/registry1/tags/list
* Hostname was NOT found in DNS cache
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 32777 (#0)
> GET /v2/registry1/tags/list HTTP/1.1
> User-Agent: curl/7.36.0
> Host: localhost:32777
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Docker-Distribution-Api-Version: registry/2.0
< Date: Tue, 14 Apr 2015 22:34:13 GMT
< Content-Length: 39
<
{"name":"registry1","tags":["latest"]}
* Connection #0 to host localhost left intact


本例参照引用了分配给 2.0 Registry 服务的特定端口。早些时候，在使用 docker ps 命令显示正在运行的容器时，你应该看到过这个端口。