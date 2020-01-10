### Go 多线程和协程



#### 设置线程数量



```
runtime.GOMAXPROCS(3)来设置最大的原生线程。
runtime.Gosched() 显式地让出CPU时间给其他goroutine
```



#### 绑定 goroutine 到 specify 线程



```
golang 的 runtime 提供了一个 LockOSThread 的函数，该方法的作用是可以让当前协程绑定并独立一个线程 M。绑定线程的那个协程 new 出来的子协程不会继承 lockOSThread 特性，new 出来的子协程不会绑定到 LockOSThread 的线程；另外需要注意的是，这个线程已经不能再处理其他协程。
```

