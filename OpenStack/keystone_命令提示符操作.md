# Keystone 命令提示符操作

## show user list

```
stack@ubuntu:~/devstack$ export OS_URL=http://127.0.0.1:35357/v2.0/
stack@ubuntu:~/devstack$ export OS_TOKEN=secrete_token
stack@ubuntu:~/devstack$ openstack user list
The request you have made requires authentication. (Disable debug mode to suppress these details.) (HTTP 401) (Request-ID: req-f9a47a8a-d3ee-471a-83fa-221ca2503016)
stack@ubuntu:~/devstack$ vim /etc/
Display all 275 possibilities? (y or n)
stack@ubuntu:~/devstack$ vim /etc/ke
keepalived/      kernel/          kernel-img.conf  kerneloops.conf  keystone/
stack@ubuntu:~/devstack$ vim /etc/keystone/
keystone.conf       keystone-paste.ini  policy.json
stack@ubuntu:~/devstack$ vim /etc/keystone/keystone.conf
stack@ubuntu:~/devstack$ export OS_TOKEN=abcdefghijklmnopqrstuvwxyz
stack@ubuntu:~/devstack$ openstack user list
+----------------------------------+----------+
| ID                               | Name     |
+----------------------------------+----------+
| 07ec8e89e8034aa0bbe81bd687e842fe | nova     |
| 097f0232657d4c95b7a33911c93b4b37 | neutron  |
| 1ad7bfbe7a87405c8cc3439d974f4a2c | alt_demo |
| 2209b20e92c94825b5af5cb2b4094c34 | admin    |
| 29fa1b4ec3854b6fadb9088428baf51b | glance   |
| d069750b83654d43a1527308a82d81a3 | cinder   |
| e532fbc102dd40faa0e5ccabaff71ee3 | demo     |
+----------------------------------+----------+
stack@ubuntu:~/devstack$
```

```
OS_TOKEN 从 /etc/keyston/keyston.conf 中找到；

[DEFAULT]
max_token_size = 16384
logging_exception_prefix = %(process)d TRACE %(name)s %(instance)s
logging_debug_format_suffix = %(funcName)s %(pathname)s:%(lineno)d
logging_default_format_string = %(process)d %(levelname)s %(name)s [-] %(instance)s%(message)s
logging_context_format_string = %(process)d %(levelname)s %(name)s [%(request_id)s %(user_identity)s] %(instance)s%(message)s
debug = True
admin_token = abcdefghijklmnopqrstuvwxyz
rpc_backend = rabbit
```

## show project list

```
stack@ubuntu:~/devstack$ openstack project list
+----------------------------------+--------------------+
| ID                               | Name               |
+----------------------------------+--------------------+
| 298c4d4948c549f2a81139716be1a0b5 | service            |
| 5f62d26490bd4d7b8a59cbd05c3f9416 | alt_demo           |
| 9a61b9519a104f3f903aa96cf5b22981 | admin              |
| b1c2c1f3317542448b8f9578c3d6b450 | demo               |
| cf3dec7d216e4b5b84c3902ab21dbe20 | invisible_to_admin |
+----------------------------------+--------------------+
```
## create project 

```
stack@ubuntu:~/devstack$ openstack project create tianyuan_demo
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| enabled     | True                             |
| id          | ba38ec1b782649b88433c4e3e36ef0d4 |
| name        | tianyuan_demo                    |
+-------------+----------------------------------+
stack@ubuntu:~/devstack$ openstack project create list
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| enabled     | True                             |
| id          | 4fcc4ae520214c09939b0c147ae06cf6 |
| name        | list                             |
+-------------+----------------------------------+
stack@ubuntu:~/devstack$ openstack project list
+----------------------------------+--------------------+
| ID                               | Name               |
+----------------------------------+--------------------+
| 298c4d4948c549f2a81139716be1a0b5 | service            |
| 4fcc4ae520214c09939b0c147ae06cf6 | list               |
| 5f62d26490bd4d7b8a59cbd05c3f9416 | alt_demo           |
| 9a61b9519a104f3f903aa96cf5b22981 | admin              |
| b1c2c1f3317542448b8f9578c3d6b450 | demo               |
| ba38ec1b782649b88433c4e3e36ef0d4 | tianyuan_demo      |
| cf3dec7d216e4b5b84c3902ab21dbe20 | invisible_to_admin |
+----------------------------------+--------------------+
stack@ubuntu:~/devstack$ openstack project delete list
stack@ubuntu:~/devstack$ openstack project list
+----------------------------------+--------------------+
| ID                               | Name               |
+----------------------------------+--------------------+
| 298c4d4948c549f2a81139716be1a0b5 | service            |
| 5f62d26490bd4d7b8a59cbd05c3f9416 | alt_demo           |
| 9a61b9519a104f3f903aa96cf5b22981 | admin              |
| b1c2c1f3317542448b8f9578c3d6b450 | demo               |
| ba38ec1b782649b88433c4e3e36ef0d4 | tianyuan_demo      |
| cf3dec7d216e4b5b84c3902ab21dbe20 | invisible_to_admin |
+----------------------------------+--------------------+
stack@ubuntu:~/devstack$
```

## GET /v3/projects

```
curl -s \
 -H "X-Auth-Token: $OS_TOKEN" \
 http://localhost:5000/v3/projects | python -mjson.tool
 
 stack@ubuntu:~/devstack$ curl -s \
>  -H "X-Auth-Token: $OS_TOKEN" \
>  http://localhost:5000/v3/projects | python -mjson.tool
{
    "links": {
        "next": null,
        "previous": null,
        "self": "http://localhost:5000/v3/projects"
    },
    "projects": [
        {
            "description": "",
            "domain_id": "default",
            "enabled": true,
            "id": "298c4d4948c549f2a81139716be1a0b5",
            "is_domain": false,
            "links": {
                "self": "http://localhost:5000/v3/projects/298c4d4948c549f2a81139716be1a0b5"
            },
            "name": "service",
            "parent_id": null
        },
        {
            "description": "",
            "domain_id": "default",
            "enabled": true,
            "id": "5f62d26490bd4d7b8a59cbd05c3f9416",
            "is_domain": false,
            "links": {
                "self": "http://localhost:5000/v3/projects/5f62d26490bd4d7b8a59cbd05c3f9416"
            },
            "name": "alt_demo",
            "parent_id": null
        },
        {
            "description": "",
            "domain_id": "default",
            "enabled": true,
            "id": "9a61b9519a104f3f903aa96cf5b22981",
            "is_domain": false,
            "links": {
                "self": "http://localhost:5000/v3/projects/9a61b9519a104f3f903aa96cf5b22981"
            },
            "name": "admin",
            "parent_id": null
        },
        {
            "description": "",
            "domain_id": "default",
            "enabled": true,
            "id": "b1c2c1f3317542448b8f9578c3d6b450",
            "is_domain": false,
            "links": {
                "self": "http://localhost:5000/v3/projects/b1c2c1f3317542448b8f9578c3d6b450"
            },
            "name": "demo",
            "parent_id": null
        },
        {
            "description": null,
            "domain_id": "default",
            "enabled": true,
            "id": "ba38ec1b782649b88433c4e3e36ef0d4",
            "is_domain": false,
            "links": {
                "self": "http://localhost:5000/v3/projects/ba38ec1b782649b88433c4e3e36ef0d4"
            },
            "name": "tianyuan_demo",
            "parent_id": null
        },
        {
            "description": "",
            "domain_id": "default",
            "enabled": true,
            "id": "cf3dec7d216e4b5b84c3902ab21dbe20",
            "is_domain": false,
            "links": {
                "self": "http://localhost:5000/v3/projects/cf3dec7d216e4b5b84c3902ab21dbe20"
            },
            "name": "invisible_to_admin",
            "parent_id": null
        }
    ]
}
```

>可以通过<http://10.211.55.3/dashboard/identity/>查询到当前的 project 的详细信息，同时通过上面的 cli 命令行工具进行查询；
