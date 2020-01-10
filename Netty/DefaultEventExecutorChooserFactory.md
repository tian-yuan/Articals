### DefaultEventExecutorChooserFactory

DefaultEventExecutorChooserFactory 继承于 EventExecutorChooserFactory，是一个 EventExecutor 选择器；

需要重点关注的一点是 DefaultEventExecutorChooserFactory 的两个实现类：

```
PowerOfTwoEventExecutorChooser
GenericEventExecutorChooser
```

* GenericEventExecutorChooser

```
@Override
        public EventExecutor next() {
            return executors[Math.abs(idx.getAndIncrement() % executors.length)];
        }
```



* PowerOfTwoEventExecutorChooser

```
@Override
        public EventExecutor next() {
            return executors[idx.getAndIncrement() & executors.length - 1];
        }
```

注意 "-" 符号比 "&" 的优先级要高，这两个实现类完成的功能是差不多的，都是 round robin 的方式分配 EventExecutor，唯一不同的是如果 EventExecutor 的数量为 2 的 n 次方时，选择 PowerOfTwoEventExecutorChooser 的性能会高一点，性能高的原因主要是 "&" 的性能比 "%" 的性能要高