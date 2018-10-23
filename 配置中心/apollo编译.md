### apollo 配置中心

#### 工程地址

https://g.hz.netease.com/push-server/apollo
此工程是从 github 上 fork 过来

#### 编译

apollo 配置中心分为三个主要模块:

* apollo-adminservice
* apollo-configservice
* apollo-portal

##### 模块描述

- Config Service提供配置的读取、推送等功能，服务对象是Apollo客户端
- Admin Service提供配置的修改、发布等功能，服务对象是Apollo Portal（管理界面）
- Config Service和Admin Service都是多实例、无状态部署，所以需要将自己注册到Eureka中并保持心跳
- 在Eureka之上我们架了一层Meta Server用于封装Eureka的服务发现接口
- Client通过域名访问Meta Server获取Config Service服务列表（IP+Port），而后直接通过IP+Port访问服务，同时在Client侧会做load balance、错误重试
- Portal通过域名访问Meta Server获取Admin Service服务列表（IP+Port），而后直接通过IP+Port访问服务，同时在Portal侧会做load balance、错误重试
- 为了简化部署，我们实际上会把Config Service、Eureka和Meta Server三个逻辑角色部署在同一个JVM进程中

##### apollo-adminservice

在 apollo/apollo-adminservice/src/main/docker 下有 Dockerfile

编译方式：

```
# Dockerfile for apollo-adminservice
# Build with:
# docker build -t apollo-adminservice .
# Run with:
# docker run -p 8090:8090 --env-file env -d -v /tmp/logs:/opt/logs --name apollo-adminservice apollo-adminservice
```

由于 apollo-adminservice 也会连接 mysql ApolloConfigDB 数据库，在启动时需要提供环境变量：

````
env:
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_USERNAME=root
MYSQL_PASSWORD=123456
````



#### apollo-configservice（包含Eureka和Meta Server）

在 apollo/apollo-configservice/src/main/docker 下有 Dockerfile

编译方式：

```
# Dockerfile for apollo-configservice
# Build with:
# docker build -t apollo-configservice .
# Run with:
# docker run -p 8080:8080 --env-file env -d -v /tmp/logs:/opt/logs --name apollo-configservice apollo-configservice
```

由于 apollo-configservice 也会连接 mysql ApolloConfigDB 数据库，在启动时需要提供环境变量：

```
env:
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_USERNAME=root
MYSQL_PASSWORD=123456
```



#### apollo-portal

在 apollo/apollo-portal/src/main/docker 下有 Dockerfile

编译方式：

```
# Dockerfile for apollo-portal
# Build with:
# docker build -t apollo-portal .
# Run with:
# docker run -p 8080:8080 --env-file env -d -v /tmp/logs:/opt/logs --name apollo-portal apollo-portal
# Or if 8080 was taken:
# docker run -p 8070:8080 --env-file env -d -v /tmp/logs:/opt/logs --name apollo-portal apollo-portal
```

由于 apollo-portal 也会连接 mysql ApolloPortalDB 数据库，在启动时需要提供环境变量：

```
env:
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_USERNAME=root
MYSQL_PASSWORD=123456

DEV_META=http://172.17.0.4:8080 （这里实际上配置的是 apollo-configservice 的地址）
APOLLO_ENVS=DEV （通过这个配置环境）
```

其中 DEV_META 是 apollo-configservice 的地址，端口对应的是 apollo-configservice 中子模块 Eureka 的监听地址



#### apollo-quick-start

https://github.com/nobodyiam/apollo-build-scripts

> 目前 apollo-quick-start 在 mac 和 docker 容器里面能正常运行，但是在 ubuntu14 和 16 上面会出现如下错误，具体原因待查

```
start-stop-daemon: warning: this system is not able to track process names
longer than 15 characters, please use --exec instead of --name.
start-stop-daemon: user '501' not found
```

