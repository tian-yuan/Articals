linux常用指令

1.scp -P10022 bin/auth_proxy tianyuan@10.19.19.28:/home/tianyuan/

2.strings ./core.9194 | grep onError

3.valgrind --track-fds=yes --leak-check=full --undef-value-errors=yes ./a.out

4.CLOSE_WAIT 是被动关闭，TIME_WAIT 是主动关闭；

5.查看GIT地址 cat .git/config