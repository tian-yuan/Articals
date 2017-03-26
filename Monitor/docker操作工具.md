#nsenter命令连接虚拟机

1、安装 nsenter

	$ cd /tmp; curl https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz | tar -zxf-; cd util-linux-2.24;
	$ ./configure --without-ncurses
	$ make nsenter && sudo cp nsenter /usr/local/bin

	or 
	
	$ wget https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz; tar xzvf util-linux-2.24.tar.gz
	$ cd util-linux-2.24
	$ ./configure --without-ncurses && make nsenter
	$ sudo cp nsenter /usr/local/bin

2、连接容器

	PID=$(sudo docker inspect --format "{{ .State.Pid }}" <container>)
	
	<container> 需要使用 sudo docker run -idt ubuntu 输出结果中的 CONTAINER ID 替换
	
	eg: PID=$(sudo docker inspect --format "{{ .State.Pid }}" f43d6713fe99)
	
	连接：
		sudo nsenter --target $PID --mount --uts --ipc --net --pid
		
		
#安装docker register

	sudo pip install docker-registry
	
	如果出现 unable to execute 'swig': No such file or directory 错误，执行如下指令
	sudo apt-get install swig		