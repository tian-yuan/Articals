linux常用指令

1.scp -Pport2 bin/auth_proxy tianyuan@ip:/home/tianyuan/

2.strings ./core.9194 | grep onError

3.valgrind --track-fds=yes --leak-check=full --undef-value-errors=yes ./a.out

4.CLOSE_WAIT 是被动关闭，TIME_WAIT 是主动关闭；

5.查看GIT地址 cat .git/config
