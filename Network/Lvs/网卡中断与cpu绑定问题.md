# [巴蛮子的烂笔头](http://www.cnblogs.com/bamanzi/)

## 好记性不如烂笔头啊。 跟我联系: ba.manzi AT yahoo.com [更多关于巴蛮子的信息](https://www.cnblogs.com/bamanzi/articles/1871009.html)

- [博客园](http://www.cnblogs.com/)
- [首页](http://www.cnblogs.com/bamanzi/)
- ​
- [联系](https://msg.cnblogs.com/send/%E5%B7%B4%E8%9B%AE%E5%AD%90)
- [管理](https://i.cnblogs.com/)
- [订阅](http://www.cnblogs.com/bamanzi/rss)[![订阅](https://www.cnblogs.com/images/xml.gif)](http://www.cnblogs.com/bamanzi/rss)

随笔- 120  文章- 5  评论- 578 

# [[Linux 性能调优\] 网卡中断与CPU的绑定问题](https://www.cnblogs.com/bamanzi/p/linux-irq-and-cpu-affinity.html)

在Linux的网络调优方面，如果你发现网络流量上不去，那么有一个方面需要去查一下：网卡处理网络请求的中断是否被绑定到单个CPU（或者说跟处理其它中断的是同一个CPU）。

### 先说一下背景

网卡与操作系统的交互一般有两种方式，

- 一种是中断（IRQ，网卡在收到了网络信号之后，主动发送中断到CPU，而CPU将会立即停下手边的活以便对这个中断信号进行分析），
- 另一种叫DMA（Direct Memory Access, 也就是允许硬件在无CPU干预的情况下将数据缓存在指定的内存空间内，在CPU合适的时候才处理）

——记得在10多年前开始玩电脑的时候就知道DMA是效率比较高的方式了。但是，这么多年过去了，在网卡方面，大部分还是在用IRQ方式（据说DMA技术仅仅被应用在少数高端网卡上; 另一个说法是：DMA方式会使外部设备的控制器独占PCI总线，从而CPU无法与外部设备进行交互，这对通用型操作系统Linux来说，是很难接收的，所以DMA方式在Linux内核里使用得很少）。

但是（再来一个但是），在现在的对称多核处理器（SMP）上，一块网卡的IRQ还是只有一个CPU来响应，其它CPU无法参与，如果这个CPU还要忙其它的中断（其它网卡或者其它使用中断的外设（比如磁盘）），那么就会形成瓶颈。

### 问题判定

网上不少讲这个问题的文章都是直接让查询IRQ跟CPU的绑定情况，甚至直接修改。但我们应该先判断我们的系统是不是受这个问题影响，然后再来看怎么解决。

首先，让你的网络跑满（比如对于MySQL/MongoDB服务，可以通过客户端发起密集的读操作; 或者执行一个i大文件传送任务）

第一个要查明的是：**是不是某个CPU在一直忙着处理IRQ？**

这个问题我们可以从 `mpstat -P ALL 1` 的输出中查明：里面的 `%irq`一列即说明了CPU忙于处理中断的时间占比

```
18:20:33     CPU   %user   %nice    %sys %iowait    %irq   %soft  %steal   %idle    intr/s
18:20:33     all    0,23    0,00    0,08    0,11    6,41    0,02    0,00   93,16   2149,29
18:20:33       0    0,25    0,00    0,12    0,07    0,01    0,05    0,00   99,49    127,08
18:20:33       1    0,14    0,00    0,03    0,04    0,00    0,00    0,00   99,78      0,00
18:20:33       2    0,23    0,00    0,02    0,03    0,00    0,00    0,00   99,72      0,02
18:20:33       3    0,28    0,00    0,15    0,28   25,63    0,03    0,00   73,64   2022,19
```

上面的例子中，第四个CPU有25.63%时间在忙于处理中断（这个数值还不算高，如果高达80%（而同时其它CPU这个数值很低）以上就说明有问题了），后面那个 intr/s 也说明了CPU每秒处理的中断数（从上面的数据也可以看出，其它几个CPU都不怎么处理中断）。

然后我们就要接着查另外一个问题：**这个忙于处理中断的CPU都在处理哪个（些）中断？**

```
cat /proc/interrupts 
           CPU0       CPU1       CPU2       CPU3       
  0:        245          0          0    7134094    IO-APIC-edge  timer
  8:          0          0         49          0    IO-APIC-edge  rtc
  9:          0          0          0          0   IO-APIC-level  acpi
 66:         67          0          0          0   IO-APIC-level  ehci_hcd:usb2
 74:     902214          0          0          0         PCI-MSI  eth0
169:          0          0         79          0   IO-APIC-level  ehci_hcd:usb1
177:          0          0          0    7170885   IO-APIC-level  ata_piix, b4xxp
185:          0          0          0      59375   IO-APIC-level  ata_piix
NMI:          0          0          0          0 
LOC:    7104234    7104239    7104243    7104218 
ERR:          0
MIS:          0
```

这里记录的是自启动以来，每个CPU处理各类中断的数量（第一列是中断号，最后一列是对应的设备名）[详细说明: [E.2.10 /proc/interrupts - Deployment Guide - RedHat Enterprise Linux 6](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s2-proc-interrupts.html) )，从上面可以看到： `eth0`所出发的中断全部都是 `CPU0`在处理，而CPU0所处理的中断请求中，主要是eth0和LOC中断。（有时我们会看到几个CPU对同一个中断类型所处理的的请求数相差无几（比如上面的LOC一行），这并不一定是说多个CPU会轮流处理同一个中断，而是因为这里记录的是“自启动以来”的统计，中间可能因为irq balancer重新分配过处理中断的CPU——当然，也可能是谁手工调节过）。

### 解决问题

首先说明几点：

1. 首先应该根据上面的诊断方法查明当前系统是不是受这个原因影响，如果不是，那么就没有必要往下看了;
2. 现在的多数Linux系统中已经有了IRQ Balance这个服务（服务程序一般是 `/usr/sbin/irqbalance`），它可以自动调节分配各个中断与CPU的绑定关系，以避免所有中断的处理都集中在少数几个CPU上;
3. 在某些情况下，这个IRQ Balance反而会导致问题，会出现 irqbalance 这个进程反而自身占用了较高的CPU（当然也就影响了业务系统的性能）[参考: [mongodb性能问题及原理分析](http://blog.csdn.net/lmh12506/article/details/46859651) ]

下面来说手工将中断限定到少数几个CPU的方法。

首先当然要查明，该网卡的中断当前是否已经限定到某些CPU了？具体是哪些CPU？

根据上面 `/proc/interrupts` 的内容我们可以看到 eth0 的中断号是74，然后我们来看看该中断号的CPU绑定情况（或者说叫亲和性 affinity）

```
$ sudo cat /proc/irq/74/smp_affinity
ffffff
```

这个输出是一个16进制的数值，`0xffffff = ‘0b111111111111111111111111’`，这就意味着这里有24个CPU，所有位都为1表示所有CPU都可以被该中断干扰。

另一个例子:

```
$ sudo cat /proc/irq/67/smp_affinity
00000001
```

这个例子说明，只有CPU0处理编号为67的中断。

**修改配置的方法**：

我们可以用 `echo 2 > /proc/irq/74/smp_affinity` 的方法来修改这个设置（设置为2表示将该中断绑定到CPU1上，0x2 = 0b10，而第一个CPU为CPU0）

### 参考

- [简单介绍下linux下的中断（interrupt）- 一切皆有可能 - 51CTO技术博客](http://noican.blog.51cto.com/4081966/1355357)
- [把网卡中断绑定到CPU,最大化网卡的吞吐量 - SegmentFault](https://segmentfault.com/a/1190000006178824)
- [Linux 多核下绑定硬件中断到不同 CPU（IRQ Affinity） | vpsee.com](http://www.vpsee.com/2010/07/load-balancing-with-irq-smp-affinity/)
- [Linux系统CPU的性能监控及调优 - 简书](http://www.jianshu.com/p/6beacca6fdcd)