#### devstack 单机部署 openstack





#### FAQ

* ubuntu下su: Authentication failure的解决办法

```
$ sudo passwd root
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
```

* 设置sudo为不需要密码

有时候我们只需要执行一条root权限的命令也要su到root，是不是有些不方便？这时可以用sudo代替。默认新建的用户不在sudo组，需要编辑/etc/sudoers文件将用户加入，该文件只能使用visudo命令，

​	1）、首先需要切换到root, su - (注意有- ，这和su是不同的，在用命令"su"的时候只是切换到root，但没有把root的环境变量传过去，还是当前用乎的环境变量，用"su -"命令将环境变量也一起带过去，就象和root登录一样)

​	2）、 然后　visudo 或者 vi /etc/sudoers, visudo 这个和vi的用法一样，由于可能会有人不太熟悉vi，所以简要说一下步骤

移动光标，到一行root ALL=(ALL)   ALL的下一行，按a，进入append模式，输入
your_user_name ALL=(ALL)   ALL

然后按Esc，再输入:w保存文件，再:q退出

这样就把自己加入了sudo组，可以使用sudo命令了。

​	3）、默认5分钟后刚才输入的sodo密码过期，下次sudo需要重新输入密码，如果觉得在sudo的时候输入密码麻烦，把刚才的输入换成如下内容即可：
your_user_name ALL=(ALL) NOPASSWD: ALL

至于安全问题，对于一般个人用户，我觉得这样也可以的。

​	4）、如果你想设置只有某些命令可以sudo的话，your_user_name   ALL= (root) NOPASSWD: /sbin/mount, (root) NOPASSWD: /bin/umount, (root) NOPASSWD: /mnt/mount, (root) NOPASSWD: /bin/rm, (root) NOPASSWD: /usr/bin/make, (root) NOPASSWD: /bin/ln, (root) NOPASSWD: /bin/sh, (root) NOPASSWD: /bin/mv, (root) NOPASSWD: /bin/chown, (root) NOPASSWD: /bin/chgrp, (root) NOPASSWD: /bin/cp, (root) NOPASSWD: /bin/chmod

 

注意： 有的时候你的将用户设了nopasswd，但是不起作用，原因是被后面的group的设置覆盖了，需要把group的设置也改为nopasswd。

joe ALL=(ALL) NOPASSWD: ALL

%admin ALL=(ALL) NOPASSWD: ALL

 

参考： 

http://blog.163.com/love-love-l/blog/static/21078304201071232234518/

* Can not find nss.h

The correct package in Ubuntu is `libnss3-dev`. Although you may find some other packages, like `libnss3` and `libnss3-1g`. I suggest to install them all.

Anyway, run `sudo apt-get upgrade libnss3*` returns you a complete list.

May be  `sudo apt-get install libnss3-dev`

use `sudo apt-cache search xxx`

* sudo: /etc/sudoers is owned by uid 1002, should be 0

```
chown root:root /etc/sudoers /etc/sudoers.d -R
```

* /opt/stack/devstack/inc/python:146 Currently installed pip version 1 does not meet minimum requirements (>=6).

```
Looks like devstack might be splitting the pip version incorrectly:

http://git.openstack.org/cgit/openstack-dev/devstack/tree/inc/python?h=stable%2Fpike#n335

    local pip_version
    pip_version=$(python -c "import pip; \
                        print(pip.__version__.split('.')[0])")
    if (( pip_version<6 )); then
        die $LINENO "Currently installed pip version ${pip_version} does not" \
            "meet minimum requirements (>=6)."
    fi
```

* Cannot uninstall 'six'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.

```
pip install -r temp.txt --ignore-installed six --user
```

需要安装的 python package 保存到 temp.txt 中

* SyntaxError:operator not allowed in environment markers

```
出现这个问题，应该是setuptools版本太旧，需要更新升级，可以执行如下命令来进行升级：

pip install --upgrade pip

pip install --upgrade setuptools
```

* Unable to correct problems, you have held broken packages.



* 重启 openstack 服务

```
screen -c stack-screenrc
ctrl + a + n 查看下一个服务
```



> 注意：newton 版本安装，一般需要 4核 8G 的内存，使用 4G 的内存，部分服务会因为内存无法分配导致部分服务无法启动