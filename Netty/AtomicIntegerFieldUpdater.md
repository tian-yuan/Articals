#### AtomicIntegerFieldUpdater使用



假设现在有这样的一个场景： 一百个线程同时对一个int对象进行修改，要求只能有一个线程可以修改。

看看下面程序是否正确：

```
    private static int a = 100;
    private static  volatile boolean ischanged = false;
    public static void main(String[] args){
        for(int i=0; i<100;i++){
            Thread t = new Thread(new Runnable() {
                @Override
                public void run() {
                    if(!ischanged){
                        ischanged = true;
                        a = 120;
                    }
                }
            });
            t.start();
        }
    }
```

对于volatile变量，写的时候会将线程本地内存的数据刷新到主内存上，读的时候会将主内存的数据加载到本地内存里，所以可以保证可见行和单个读/写操作的原子性。但是上例中先 1. 判断!ischanged 2.ischanged=true  该组合操作就不能保证原子性了，也就是说线程A A1->A2  线程B B1->B2(第一个操作为volatile读或者第二个操作为volatile写的时候，编译器不会对两个语句重排序，所以最后的执行顺序满足顺序一致性模型的)，但是最后的执行结果可能是A1->B1->A2->B2。不满足需求

AtomicIntegerFieldUpdater:  基于反射的工具，可用CompareAndSet对volatile int进行原子更新:

修改上个场景的代码，如下：

```
public class Test{
    private static AtomicIntegerFieldUpdater<Test> update = AtomicIntegerFieldUpdater.newUpdater(Test.class, "a");
    private static Test test = new Test();
    public volatile int a = 100;
    public static void main(String[] args){
        for(int i=0; i<100;i++){
            Thread t = new Thread(new Runnable() {
                @Override
                public void run() {
                    if(update.compareAndSet(test, 100, 120)){
                        System.out.print("已修改");
                    }
                }
            });
            t.start();
        }
    }
}
```

```
用AtomicIntegerFieldUpdater.newUpdater指定类里面的属性。这里我们要更新Test类里面的A字段（必须是volatile且不是static对象）。update.compareAndSet()方法使用cas机制，每次提交的时候都比较下test.a是不是100，如果是，则更新。
```

