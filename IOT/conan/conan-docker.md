#### conan server 的安装

* docker pull cguenther/conan-server
* 也可以参照如下 URL 进行安装：

```
https://docs.conan.io/en/latest/using_packages/conanfile_txt.html
```

### docker-conan-server

Docker image definition for a [Conan.io](https://conan.io/) server

#### Running the server

First create the server container

```
mkdir conan_server
docker run \
    -p 9300:9300 \
    -v conan_server:/var/lib/conan \
    --name conan_server \
    cguenther/conan-server:latest

```

The configure the server by editting the `.conan_server/server.conf`
file. The `host_name` and `public_port` parameters are of particular
importance and must match the name of the docker host and the public
port exposed for the container.
For more information of the server.conf, see the <http://conanio.readthedocs.io/en/latest/server.html#server-configuration>.

#### Example docker-compose.yml file

```
version: '2' 
services:
  conan:
    image: cguenther/conan-server
    container_name: conan
    volumes:
      - ./conan:/var/lib/conan
    ports:
      - 9300:9300
```

docker run -itd -p 9300:9300 -v conan_server:/var/lib/conan --name conan_server cguenther/conan-server:latest

#### conan_server config

```
[server]
# WARNING! Change default variable of jwt_secret. You should change it periodically
# It only affects to current authentication tokens, you can change safetely anytime
# When it changes user are just forced to log in again
jwt_secret: dkfnrvHuilFVyumNYesNxXjE
jwt_expire_minutes: 120

ssl_enabled: False
port: 9300
# Public port where files will be served. If empty will be used "port"
public_port:
host_name: localhost

# Choose file adapter, "disk" for disk storage
# Authorize timeout are seconds the client has to upload/download files until authorization expires
store_adapter: disk
authorize_timeout: 1800

# Just for disk storage adapter
# updown_secret is the key used to generate the upload/download authorization token
disk_storage_path: ~/.conan_server/data
disk_authorize_timeout: 1800
updown_secret: oRRYYvyBESGRZNzqwQYiMzGH

# Check docs.conan.io to implement a different authenticator plugin for conan_server
# if custom_authenticator is not specified, [users] section will be used to authenticate
# the users.
#
# custom_authenticator: my_authenticator

# name/version@user/channel: user1, user2, user3
#
# The rules are applied in order.
# If a rule matches your package, then the server wont look further.
# Place your more restrictive rules first.
#
# Example: All versions of opencv package from lasote user in testing channel is only
# writeable by default_user and default_user2. Rest of packages are not writtable by anything
# except the author.
#
#   opencv/2.3.4@lasote/testing: default_user, default_user2
#
[write_permissions]
*/*@*/*: *

# name/version@user/channel: user1, user2, user3
# The rules are applied in order. If a rule applies to a conan, system wont look further.
#
# Example:
#  All versions of opencv package from lasote user in testing channel are only
#    readable by default_user and default_user2.
#  All versions of internal package from any user/channel are only readable by
#    authenticated users.
#  Rest of packages are world readable.
#
#   opencv/*@lasote/testing: default_user default_user2
#   internal/*@*/*: ?
#   *:*@*/*: *
#
# By default all users can read all blocks
#
[read_permissions]
*/*@*/*: *

[users]
#default_user: defaultpass
demo: demo
```

在上传本地仓库时，有可能出现权限错误，需要放开 write_permissions 的权限



* 查看本地远程仓库

```
➜  openssl_package conan remote list
conan-center: http://127.0.0.1:9300 [Verify SSL: False]
conan-transit: https://conan-transit.bintray.com [Verify SSL: True]
```

* 修改本地远程仓库

```
➜  openssl_package conan remote update conan-transit https://conan-transit.bintray.com
```

* 下载依赖库

```
➜  build git:(master) ✗ conan install zlib/1.2.8@heksesang/stable --build zlib
zlib/1.2.8@heksesang/stable: Installing package
```

* 上传到本地仓库

```
➜  build git:(master) ✗ conan upload --remote conan-center zlib/1.2.8@heksesang/stable
WARN: A new conan version (v1.2.3) is available in current remote. Please, upgrade conan client to avoid deprecation.
Uploading zlib/1.2.8@heksesang/stable
```

