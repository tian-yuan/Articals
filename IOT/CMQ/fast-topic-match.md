### 快速 TOPIC 匹配

消息中间件存在一个共性问题，如何有效地匹配订阅者主题。比如，我们有三个订阅者主题，分别为 1 到 3：

| **Subscriber** | **Match Request** |
| -------------- | ----------------- |
| 1              | forex.usd         |
| 2              | forex.*           |
| 3              | stock.nasdaq.msft |

同时我们有一串消息，分别为 1 到 N：

| **Message** | **Topic**         |
| ----------- | ----------------- |
| 1           | forex.gbp         |
| 2           | stock.nyse.ibm    |
| 3           | stock.nyse.ge     |
| 4           | forex.eur         |
| 5           | forex.usd         |
| …           | …                 |
| N           | stock.nasdaq.msft |

我们现在的任务是路由这些消息到对应的订阅者，"*" 通配符表示匹配任意单词。这通常是像ZeroMQ，RabbitMQ，ActiveMQ，TIBCO EMS等面向消息的中间件的性能瓶颈。这种问题有一些已知解决方案。本文将描述其中几种方案，并试图通过基准测试来评估以及量化各个方案的性能。对应代码在 [GitHub](https://github.com/tylertreat/fast-topic-matching).

> 订阅者主题命名为订阅主题，如 x.*，发布者主题命名为发布主题，如x.y。订阅者主题里面才能包含通配符，订阅者主题不能包含通配符

### 原生解决方案

原生解决方案是非常简单的：用一个哈希映射表来映射主题和订阅者。订阅一个主题时，往哈希表中增加一个一项。当需要匹配一个消息的Topic时，需要扫描整个哈希表，然后返回对应的订阅者列表。

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/naive.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/naive.png)

添加一个记录接近O(1)的时间复杂度，但是查找匹配的时间复杂度是O(n*m)，n 为订阅者数量，m是订阅主题中单词的数量。这意味着性能严重依赖于哈希表中订阅主题的数量。但是大部分情况下搜索比更新量要大的多。因此原生解决方案并不是一个好的方案。

如下是对订阅，取消订阅和查找的基准测试，我们开始使用空哈希表（我们叫 cold），然后使用一个包含一千个随机生成的有五个单词的订阅主题（我们叫 hot）。从下图可以看出，在哈希表不为空的情况下，查找的速度慢了三个数量级。

|          | **subscribe** | **unsubscribe** | **lookup** |
| -------- | ------------- | --------------- | ---------- |
| **cold** | 172ns         | 51.2ns          | 787ns      |
| **hot**  | 221ns         | 55ns            | 815,787ns  |

### [![img](https://bravenewgeek.com/wp-content/uploads/2016/12/naive-1.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/naive-1.png) 

### 反转位图

反转位图技术是建立在查找比更新更加频繁的基础上并且假设搜索空间是有限的。相应的，反转位图技术牺牲写性能获取更高的读性能。每一个发布主题一个位图，订阅主题被分配到一个从0开始的整数。我们分析每一个发布主题，如果匹配订阅主题，相应的位图对应的bits设置为1。如下，假设我们的搜索空间由下面一些列发布主题组成：

- forex.usd
- forex.gbp
- forex.jpy
- forex.eur
- stock.nasdaq
- stock.nyse

然后有如下的订阅主题：

- 0 = forex.* (匹配 forex.usd, forex.gbp, forex.jpy, and forex.eur)
- 1 = stock.nyse (匹配 stock.nyse)
- 2 = *.* (匹配所有)
- 3 = stock.* (匹配 stock.nasdaq 和 stock.nyse)

当我们分析完发布主题后，得到如下位图表：

| **发布主题** | **0** | **1** | **2** | **3** |
| ------------ | ----- | ----- | ----- | ----- |
| forex.usd    | 1     | 0     | 1     | 0     |
| forex.gbp    | 1     | 0     | 1     | 0     |
| forex.jpy    | 1     | 0     | 1     | 0     |
| forex.eur    | 1     | 0     | 1     | 0     |
| stock.nasdaq | 0     | 0     | 1     | 1     |
| stock.nyse   | 0     | 1     | 1     | 1     |

当需要匹配一条消息时，只需要查找消息中发布主题对应的位图，并检索出来对应的位，就可以获取所有的匹配的订阅主题。如下表所示，订阅和取消订阅比原生方案要消耗更多的时间，但是查找只需要不要0.5毫秒，性能非常好。

|          | **subscribe** | **unsubscribe** | **lookup** |
| -------- | ------------- | --------------- | ---------- |
| **cold** | 3,795ns       | 198ns           | 380ns      |
| **hot**  | 3,863ns       | 198ns           | 395ns      |

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/inverted_bitmap.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/inverted_bitmap.png)

在读负载非常高的情况下，反转位图比哈希表是更好的选择。但是反转位图需要预先知道搜素空间，或者说需要预先检索位图表，也是比较费的。主要应用于静态订阅场景。

### 优化的反转位图

反转位图性能非常好，但是需要发布主题和订阅主题相对静态。另外在主题空间和订阅主题是非常大的情况下性能也会急剧下降。

优化反转位图算法，把主题拆分成各组成部分（如，x.y拆分成 x 和 y），每一个组成部分有对应的位图。

假设我们最大主题大小为 2（主题最多有两层），有如下订阅主题：

- 0 = forex.*
- 1 = stock.nyse
- 2 = index
- 3 = stock.*

第一部分的反转位图如下：

|           | **forex.\*** | **stock.nyse** | **index** | **stock.\*** |
| --------- | ------------ | -------------- | --------- | ------------ |
| **null**  | 0            | 0              | 0         | 0            |
| **forex** | 1            | 0              | 0         | 0            |
| **stock** | 0            | 1              | 0         | 1            |
| **index** | 0            | 0              | 1         | 0            |
| **other** | 0            | 0              | 0         | 0            |

第二部分的位图如下：

|           | **forex.\*** | **stock.nyse** | **index** | **stock.\*** |
| --------- | ------------ | -------------- | --------- | ------------ |
| **null**  | 0            | 0              | 1         | 0            |
| **nyse**  | 1            | 1              | 0         | 1            |
| **other** | 1            | 0              | 0         | 1            |

"null" 表示没有对应的组成部分，比如 "index"，没有第二层，所以第二部分的位图中 "index" 对应的位被置为 1。"other"表示其他不包含在订阅主题里面的单词。

让我们看一下一个例子，假设发布主题为 forex.eur。我们把它拆分为两部分："forex" 和 "eur"。我们从第一个组成部分位图中找到 "forex" 对应的一行，从第二个组成部分位图中找到 "other" 对应的一样，然后我们执行与操作。

|           | **forex.\*** | **stock.nyse** | **index** | **stock.\*** |
| --------- | ------------ | -------------- | --------- | ------------ |
| 1 = forex | 1            | 0              | 0         | 0            |
| 2 = other | 1            | 0              | 0         | 1            |
| **AND**   | 1            | 0              | 0         | 0            |

最终我们得到 forex.* 订阅主题匹配我们的发布主题。

我们看一下另外一个例子 stock.nyse.

|           | **forex.\*** | **stock.nyse** | **index** | **stock.\*** |
| --------- | ------------ | -------------- | --------- | ------------ |
| 1 = stock | 0            | 1              | 0         | 1            |
| 2 = nyse  | 0            | 1              | 0         | 1            |
| **AND**   | 0            | 1              | 0         | 1            |

匹配的订阅主题为 stock.nyse 和 stock.*.

基准测试如下所示，从下表可以看出订阅性能，优化版的反转位图比反转位图要快很多。

|          | **subscribe** | **unsubscribe** | **lookup** |
| -------- | ------------- | --------------- | ---------- |
| **cold** | 1,053ns       | 330ns           | 2,724ns    |
| **hot**  | 1,076ns       | 371ns           | 3,337ns    |

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/optimized_inverted_bitmap.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/optimized_inverted_bitmap.png)

### 单词查找树

优化的反转位图虽然降低了空间复杂度，但是时间复杂度上依然不尽如人意。反转位图有非常好的查询效率，但是非常浪费内存空间，即使使用高压缩比的 roaring bitmaps 算法。是否存在一个方案，同时满足低时间复杂度和空间复杂度呢？

单词查找数在这种情况下可以更加有效地节省空间使用率。

给定一个订阅者集合，单词查找数就像下图所示：

- forex.*
- stock.nyse
- index
- stock.*

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/trie.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/trie.png)

从测试结果可以看出，单词查找树在空间复杂度和时间复杂度上取得了均衡。

|          | **subscribe** | **unsubscribe** | **lookup** |
| -------- | ------------- | --------------- | ---------- |
| **cold** | 406ns         | 221ns           | 2,145ns    |
| **hot**  | 443ns         | 257ns           | 2,278ns    |

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/trie-1.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/trie-1.png)



### 并发主题订阅树

迄今为止，有一个问题一直被我们忽视，那就是并发调用。订阅、取消订阅和检索通常发生在不同的线程。特别是我们需要充分利用多核优势时，高并发支持是非常重要的。

一般应对高并发，多线程问题，大部分的算法都是使用全局锁来保证线程安全。但是全局锁极大的影响了应用的吞吐量。为了解决这个问题，演化出了并发主题订阅树算法。它集成了主题匹配树和并发树的思想。

并发主题订阅树使用一个中间节点 I-nodes，当订阅和取消订阅时，创建 I-nodes 指向节点的副本，执行完修改后，对 I-nodes 执行 CAS 操作。通过引入中间节点的方式实现无锁和线下操作。

对于给定的订阅者集合，x， y和 z，并发主题订阅树表示如下：

- x = foo, bar, bar.baz
- y = foo, bar.qux
- z = bar.*

[![img](https://bravenewgeek.com/wp-content/uploads/2015/07/matchbox.png)](https://bravenewgeek.com/wp-content/uploads/2015/07/matchbox.png)

CS-trie 基准测试如下：

|          | **subscribe** | **unsubscribe** | **lookup** |
| -------- | ------------- | --------------- | ---------- |
| **cold** | 412ns         | 245ns           | 1,615ns    |
| **hot**  | 471ns         | 280ns           | 1,637ns    |

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/cs_trie.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/cs_trie.png)

### 耗时对比

下图显示了对于以上所述的五中算法的主题匹配耗时的对比：

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/operations_cold.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/operations_cold.png)

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/operations_hot.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/operations_hot.png)

### 吞吐量对比

迄今为止，我们对比了主题匹配的耗时。接下来我们对比一下并发情况下的吞吐量对比和内存使用对比。

| **algorithm**             | **msg/sec**  |
| ------------------------- | ------------ |
| naive                     | 4,053.48     |
| inverted bitmap           | 1,052,315.02 |
| optimized inverted bitmap | 130,705.98   |
| trie                      | 248,762.10   |
| cs-trie                   | 340,910.64   |

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/throughput.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/throughput.png)

从图标看，反转位图算法性能是最高的，但是它扩展性很差，同时如下图所示，占用内存资源很大。

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/memory.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/memory.png)

### 高并发情况下的扩展性

最后，我们对比一下各算法并发情况下的性能。我们使用 GO 做压力测试，使用不同的协程数量和不同的操作数量。读写比为1：1，协程数量从 2 到 16。使用2.6GHZ 8 核英特尔 i7 处理器。每个协程处理 1000 读操作或者 1000 写操作，比如 2 个协程处理 1000 读操作和 1000 写操作，4 个协程执行 2000 读操作和 2000 写操作。最后我们统计执行所消耗的时间如下：

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/multithreaded_5050.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/multithreaded_5050.png)

由于 trie 和 CS-trie 数值比较小，单独显示在下图：

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/tries_5050.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/tries_5050.png)

很显然 cs-trie 随着负载和并发数量的增加，性能依然表现良好。

因为在大部分情况下，读操作要远远高于写操作，所以我们做了读写比为 9：1 的性能测试，如下所示：

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/multithreaded_9010.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/multithreaded_9010.png)

[![img](https://bravenewgeek.com/wp-content/uploads/2016/12/tries_9010.png)](https://bravenewgeek.com/wp-content/uploads/2016/12/tries_9010.png)

从结果可以看出，cs-trie 依然表现最好。

### 结论

并发订阅主题树是最好的选择，唯一的缺点是实现复杂。