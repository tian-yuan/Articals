### Futex

futex (fast userspace mutex) 是Linux的一个基础构件，可以用来构建各种更高级别的同步机制，比如锁或者信号量等等，[POSIX信号量](http://linuxperf.com/?p=25)就是基于futex构建的。大多数时候编写应用程序并不需要直接使用futex，一般用基于它所实现的系统库就够了。

futex的性能非常优异，它是怎样做到的呢？这要从它的设计思想谈起。传统的SystemV IPC(inter process communication)进程间同步机制都是通过内核对象来实现的，以 semaphore 为例，当进程间要同步的时候，必须通过系统调用semop(2)进入内核进行PV操作。系统调用的缺点是开销很大，需要从user mode切换到kernel mode、保存寄存器状态、从user stack切换到kernel stack、等等，通常要消耗上百条指令。事实上，有一部分系统调用是可以避免的，因为现实中很多同步操作进行的时候根本不存在竞争，即某个进程从持有semaphore直至释放semaphore的这段时间内，常常没有其它进程对同一semaphore有需求，在这种情况下，内核的参与本来是不必要的，可是在传统机制下，持有semaphore必须先调用semop(2)进入内核去看看有没有人和它竞争，释放semaphore也必须调用semop(2)进入内核去看看有没有人在等待同一semaphore，这些不必要的系统调用造成了大量的性能损耗。futex就为了解决这个问题而生的，它的办法是：在无竞争的情况下，futex的操作完全在user space进行，不需要系统调用，仅在发生竞争的时候进入内核去完成相应的处理(wait 或者 wake up)。所以说，futex是一种user mode和kernel mode混合的同步机制，需要两种模式合作才能完成，futex变量必须位于user space，而不是内核对象，futex的代码也分为user mode和kernel mode两部分，无竞争的情况下在user mode，发生竞争时则通过sys_futex系统调用进入kernel mode进行处理，具体来说：

futex变量是位于user space的一个整数，支持原子操作。futex同步操作都是从user space开始的：

- 当要求持有futex的时候，对futex变量执行”down”操作，即原子递减，如果变量变为0，则意味着没有竞争发生，进程成功持有futex并继续在user mode运行；如果变量变为负数，则意味着有竞争发生，需要通过sys_futex系统调用进入内核执行futex_wait操作，让进程进入休眠等待。
- 当释放futex的时候，对futex变量进行”up”操作，即原子递增，如果变量变成1，则意味着没有竞争发生，进程成功释放futex并继续在user mode执行；否则意味着有竞争，需要通过sys_futex系统调用进入内核执行futex_wake操作，唤醒正在等待的进程。

如果需要在多个进程之间共享futex，那就必须把futex变量放在共享内存中，并确保这些进程都有访问共享内存的权限；如果仅需在线程之间使用futex的话，那么futex变量可以位于进程的私有内存中，比如普通的全局变量即可。

更详细的信息请参阅futex作者的论文：
[Fuss, Futexes and Furwocks: Fast Userlevel Locking in Linux](https://www.kernel.org/doc/ols/2002/ols2002-pages-479-495.pdf)



