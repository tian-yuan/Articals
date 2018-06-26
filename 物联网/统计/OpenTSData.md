#### OpenTSData 

##### 安装

https://hub.docker.com/r/petergrace/opentsdb-docker/

下载 OpenTSData 镜像，按照文档启动 docker 容器

```
CONTAINER ID    IMAGE         COMMAND        CREATED     STATUS      PORTS  NAMES
27918ce20f86        petergrace/opentsdb-docker   "/tini -- /entrypoin…"   14 hours ago        Up 14 hours         16010/tcp, 16070/tcp, 60000/tcp, 60010/tcp, 0.0.0.0:4242->4242/tcp, 60030/tcp   naughty_thompson

```

##### 使用

在浏览器中登录 http://localhost:4242 可以查看查询界面

##### API

http://opentsdb.net/docs/build/html/api_http/version.html

测试 version 接口



```
request:

GET /api/version HTTP/1.1
Host: 10.242.83.109:4242
Cache-Control: no-cache
Postman-Token: 56eb39e1-ccb6-a682-da96-92fcfba382ad

response:
{
    "timestamp": "1527358102",
    "host": "be6d1f0eb05e",
    "repo": "/opt/opentsdb/opentsdb-2.3.1/build",
    "branch": "",
    "full_revision": "",
    "short_revision": "",
    "user": "root",
    "repo_status": "MODIFIED",
    "version": "2.3.1"
}
```



测试 put 接口

```
分别准备三个数据文件：
put1.json

{
    "metric": "sys.cpu.nice",
    "timestamp": 1346846400,
    "value": 18,
    "tags": {
       "host": "web01",
       "dc": "lga"
    }
}

put2.json

{
    "metric": "sys.cpu.nice",
    "timestamp": 1346846410,
    "value": 18,
    "tags": {
       "host": "web01",
       "dc": "lga"
    }
}

put3.json

{
    "metric": "sys.cpu.nice",
    "timestamp": 1346846410,
    "value": 18,
    "tags": {
       "host": "web02",
       "dc": "lga"
    }
}

➜  OpenTSData curl -i -d "@put2.json" -X POST http://localhost:4242/api/put -vvv                  
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 4242 (#0)
> POST /api/put HTTP/1.1
> Host: localhost:4242
> User-Agent: curl/7.54.0
> Accept: */*
> Content-Length: 134
> Content-Type: application/x-www-form-urlencoded
> 
* upload completely sent off: 134 out of 134 bytes
< HTTP/1.1 204 No Content
HTTP/1.1 204 No Content
< Content-Type: application/json; charset=UTF-8
Content-Type: application/json; charset=UTF-8
< Content-Length: 0
Content-Length: 0

< 
* Connection #0 to host localhost left intact

分别把 put1.json put2.json put3.json 数据导入到 OpenTSData 中。

查询结果如下所示：
➜  OpenTSData curl -i "http://localhost:4242/api/query?start=1346846400&m=sum:host:sys.cpu.nice"
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
Content-Length: 112

[{"metric":"sys.cpu.nice","tags":{"dc":"lga"},"aggregateTags":["host"],"dps":{"1346846400":18,"1346846410":36}}]
```

