##### 使用 context 系列函数实现用户级别的上下文切换



In a System V-like environment, one has the type *ucontext_t* defined in *<ucontext.h>* and the four functions **getcontext**(3),**setcontext**(3), **makecontext**() and **swapcontext**() that allow user-level context switching between multiple threads of control within a process.

在一个类V系统环境下，<ucontext.h> 中定义了 ucontext_t 类型和四个函数 getcontext, setcontext, makecontext 以及 swapcontext，这个四个函数允许同一个进程多个线程之间用户级别的上下文切换。

For the type and the first two functions, see **getcontext**(3).

The **makecontext**() function modifies the context pointed to by *ucp* (which was obtained from a call to **getcontext**(3)). Before invoking**makecontext**(), the caller must allocate a new stack for this context and assign its address to *ucp->uc_stack*, and define a successor context and assign its address to *ucp->uc_link*.

When this context is later activated (using **setcontext**(3) or **swapcontext**()) the function *func* is called, and passed the series of integer (*int*) arguments that follow *argc*; the caller must specify the number of these arguments in *argc*. When this function returns, the successor context is activated. If the successor context pointer is NULL, the thread exits.

The **swapcontext**() function saves the current context in the structure pointed to by *oucp*, and then activates the context pointed to by *ucp*.



```
#include <ucontext.h>
#include <stdio.h>
#include <stdlib.h>

static ucontext_t uctx_main, uctx_func1, uctx_func2;

#define handle_error(msg) \
    do { perror(msg); exit(EXIT_FAILURE); } while (0)

static void
func1(void)
{
    printf("func1: started\n");
    printf("func1: swapcontext(&uctx_func1, &uctx_func2)\n");
    if (swapcontext(&uctx_func1, &uctx_func2) == -1)
        handle_error("swapcontext");
    printf("func1: returning\n");
}

static void
func2(void)
{
    printf("func2: started\n");
    printf("func2: swapcontext(&uctx_func2, &uctx_func1)\n");
    if (swapcontext(&uctx_func2, &uctx_func1) == -1)
        handle_error("swapcontext");
    printf("func2: returning\n");
}

int
main(int argc, char *argv[])
{
    char func1_stack[16384];
    char func2_stack[16384];

   if (getcontext(&uctx_func1) == -1)
        handle_error("getcontext");
    uctx_func1.uc_stack.ss_sp = func1_stack;
    uctx_func1.uc_stack.ss_size = sizeof(func1_stack);
	// 如果 uctx_func1 退出后，会切换到 uctx_main 所指向的上下文
    uctx_func1.uc_link = &uctx_main;
    makecontext(&uctx_func1, func1, 0);

   if (getcontext(&uctx_func2) == -1)
        handle_error("getcontext");
    uctx_func2.uc_stack.ss_sp = func2_stack;
    uctx_func2.uc_stack.ss_size = sizeof(func2_stack);
    /* Successor context is f1(), unless argc > 1 */
    // 如果命令行参数大于一个，那么 uc_link 指向空，这里会导致 uctx_func2 结束时不会切换到 uctx_func1 上下文中，从而 func1 不能退出，并且程序会直接退出
    uctx_func2.uc_link = (argc > 1) ? NULL : &uctx_func1;
    makecontext(&uctx_func2, func2, 0);

   printf("main: swapcontext(&uctx_main, &uctx_func2)\n");
    if (swapcontext(&uctx_main, &uctx_func2) == -1)
        handle_error("swapcontext");

   printf("main: exiting\n");
    exit(EXIT_SUCCESS);
}


```



```
root@7007ba4c4a7c:/libtask# ./testcontext 
main: swapcontext(&uctx_main, &uctx_func2)
func2: started
func2: swapcontext(&uctx_func2, &uctx_func1)
func1: started
func1: swapcontext(&uctx_func1, &uctx_func2)
func2: returning
func1: returning
main: exiting

// 程序直接退出，不会执行 printf("main: exiting\n"); 语句
root@7007ba4c4a7c:/libtask# ./testcontext  1
main: swapcontext(&uctx_main, &uctx_func2)
func2: started
func2: swapcontext(&uctx_func2, &uctx_func1)
func1: started
func1: swapcontext(&uctx_func1, &uctx_func2)
func2: returning

```

