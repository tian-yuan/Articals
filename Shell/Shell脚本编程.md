## date

	DEVSTACK_START_TIME=$(date +%s)
	Needs a leading '+' to invoke formattin
	%s yields number of seconds since "UNIX epoch" began
	参见：
		http://tldp.org/LDP/abs/html/timedate.html
		
		
## code

```
[ -a FILE ]  如果 FILE 存在则为真。  
[ -b FILE ]  如果 FILE 存在且是一个块特殊文件则为真。  
[ -c FILE ]  如果 FILE 存在且是一个字特殊文件则为真。  
[ -d FILE ]  如果 FILE 存在且是一个目录则为真。  
[ -e FILE ]  如果 FILE 存在则为真。  
[ -f FILE ]  如果 FILE 存在且是一个普通文件则为真。  
[ -g FILE ]  如果 FILE 存在且已经设置了SGID则为真。  
[ -h FILE ]  如果 FILE 存在且是一个符号连接则为真。  
[ -k FILE ]  如果 FILE 存在且已经设置了粘制位则为真。  
[ -p FILE ]  如果 FILE 存在且是一个名字管道(F如果O)则为真。  
[ -r FILE ]  如果 FILE 存在且是可读的则为真。  
[ -s FILE ]  如果 FILE 存在且大小不为0则为真。  
[ -t FD ]  如果文件描述符 FD 打开且指向一个终端则为真。  
[ -u FILE ]  如果 FILE 存在且设置了SUID (set user ID)则为真。  
[ -w FILE ]  如果 FILE 如果 FILE 存在且是可写的则为真。  
[ -x FILE ]  如果 FILE 存在且是可执行的则为真。  
[ -O FILE ]  如果 FILE 存在且属有效用户ID则为真。  
[ -G FILE ]  如果 FILE 存在且属有效用户组则为真。  
[ -L FILE ]  如果 FILE 存在且是一个符号连接则为真。  
[ -N FILE ]  如果 FILE 存在 and has been mod如果ied since it was last read则为真。  
[ -S FILE ]  如果 FILE 存在且是一个套接字则为真。  
[ FILE1 -nt FILE2 ]  如果 FILE1 has been changed more recently than FILE2, or 如果 FILE1 exists and FILE2 does not则为真。  
[ FILE1 -ot FILE2 ]  如果 FILE1 比 FILE2 要老, 或者 FILE2 存在且 FILE1 不存在则为真。  
[ FILE1 -ef FILE2 ]  如果 FILE1 和 FILE2 指向相同的设备和节点号则为真。  
[ -o OPTIONNAME ]  如果 shell选项 “OPTIONNAME” 开启则为真。  
[ -z STRING ]  “STRING” 的长度为零则为真。  
[ -n STRING ] or [ STRING ]  “STRING” 的长度为非零 non-zero则为真。  
[ STRING1 == STRING2 ]  如果2个字符串相同。 “=” may be used instead of “==” for strict POSIX compliance则为真。  
[ STRING1 != STRING2 ]  如果字符串不相等则为真。 
[ STRING1 < STRING2 ]  如果 “STRING1” sorts before “STRING2” lexicographically in the current locale则为真。  
[ STRING1 > STRING2 ]  如果 “STRING1” sorts after “STRING2” lexicographically in the current locale则为真。  
[ ARG1 OP ARG2 ] “OP” is one of -eq, -ne, -lt, -le, -gt or -ge. These arithmetic binary operators return true if “ARG1” is equal to, not equal to, less than, less than or equal to, greater than, or greater than or equal to “ARG2”, respectively. “ARG1” and “ARG2” are integers. 		
```

## UID && EUID

	linux系统中每个进程都有2个ID，分别为用户ID和有效用户ID，UID一般表示进程的创建者（属于哪个用户创建），而EUID表示进程对于文件和资源的访问权限（具备等同于哪个用户的权限）。可以通过函数getuid()和geteuid（）或者进程的两个ID值。
	
	root 用户下， UID 和 EUID 都为 0；所以可以通过 $EUID == 0 来判断当前用户是不是 root 用户；

	linux 文件权限为 User + Group + Other
	
	
## source 

```
LINUX中一个文件是根据其是否具有执行属性来判断他是否可以直接运行的。就像windows下的exe一样。如果我们要执行某一个文件，可以先将其权限修改为可执行(必须是所有者或者root才能修改)。然后，通过用sh来执行该脚本或者./脚本名。

但有时候我们并不想修改文件权限，可能我们也没有那个权限，所以我们可以使用.(点号)+文件名来临时执行一个脚本而无须修改权限。

在Linux系统中存在大量的脚本，其中你会看到大量这个用source命令(从 C Shell 而来)执行bash shell的内置命令。点命令，就是个点符号(从Bourne Shell而来)是source的另一名称。同样的，当前脚本中配置的变量也将作为脚本的环境，source(或点)命令通常用于重新执行刚修改的初始化文档，如 .bash_profile 和 .profile 等等。例如，假如在登录后,对.bash_profile中的 EDITER 和 TERM 变量做了修改，则能够用source命令重新执行.bash_profile中的命令而不用注销并重新登录。

source命令的作用就是用来执行一个脚本，那么： source a.sh 同直接执行 ./a.sh 有什么不同呢？比如您在一个脚本里export $KKK=111，假如您用./a.sh执行该脚本，执行完毕后，您运行echo $KKK，发现没有值，假如您用source来执行，然后再echo ,就会发现KKK=111。因为调用./a.sh来执行shell是在一个子shell里运行的，所以执行后，结果并没有反应到父shell里，但是source不同他就是在本shell中执行的，所以能够看到结果。	
```
	
## type

```
type命令的基本使用方式就是直接跟上命令名字。
type -a可以显示所有可能的类型，比如有些命令如pwd是shell内建命令，也可以是外部命令。
type -p只返回外部命令的信息，相当于which命令。
type -f只返回shell函数的信息。
type -t 只返回指定类型的信息。

liuzebodeMacBook-Pro-2:ShellWorkspace liuzebo$ type -a pwd
pwd is a shell builtin
pwd is /bin/pwd	
```





>参见：<https://bash.cyberciti.biz/guide/Main_Page>

