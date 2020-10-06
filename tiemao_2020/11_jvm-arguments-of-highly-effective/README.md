# 7 JVM arguments of Highly Effective Applications


＃ 最重要的JVM性能调优参数

At the time (March 2020) of writing this article there are 600+ arguments that you can pass to JVM just around Garbage collection and memory. If you include other aspects, total JVM arguments count will easily cross 1000+. It’s way too many arguments for anyone to digest and comprehend. In this article, we are highlighting seven important JVM arguments that you may find it useful.

截止目前, 可配置的JVM参数已达1000多个，其中GC和内存相关的配置参数有600多个。
参数太多是个麻烦，让我们难以下手，学习和理解起来也很费事。
本文主要介绍最重要的七个JVM参数，学会之后你会发现水平提高了一大截。

## 1. `-Xmx` and `-XX:MaxMetaspaceSize`

`-Xmx` is probably the most important JVM argument. `-Xmx` defines the maximum amount of heap size you are allocating to your application. (To learn about different memory regions in a JVM, you may watch [this short video clip](https://www.youtube.com/watch?v=uJLOlCuOR4k&t=9s) ). You can define your application’s heap size like this:

## 1. 限制最大堆内存

`-Xmx` 可以说是最重要的JVM参数，用于指定应用程序可以使用的堆内存最大值。
关于JVM内存的基础知识，可以参考这个英语版的youtube视频: [JVM Memory - Learn Easily](https://www.youtube.com/watch?v=uJLOlCuOR4k&t=9s)。
使用示例:

```
-Xmx4g
```

Heap size plays a critical role in determining your

- a. Application performance
- b. Bill, that you are going to get from your cloud provider (AWS, Azure,…)

This brings question, what is the right heap size for my application? Should I allocate a large heap size or small heap size for my application? Answer is: ‘It depends’. In [this article](https://blog.heaphero.io/2019/06/21/large-or-small-memory-size-for-my-app/), we have shared our thoughts whether you need to go with large or small heap size.

You might also consider reading this article: [advantages of setting `-Xms` and `-Xmx` to same value](https://blog.gceasy.io/2020/03/09/advantages-of-setting-xms-and-xmx-to-same-value/).

堆内存的大小对系统性能有很大影响， 当然，提高系统配置也会增加运行成本， 需要综合考虑。
给应用程序配置的堆内存太大或太小都会有副作用，那我们应该设置多大? 请参考: [LARGE OR SMALL MEMORY SIZE FOR MY APP?](https://blog.heaphero.io/2019/06/21/large-or-small-memory-size-for-my-app/)

另一个参数 `-Xms` 决定了初始堆内存大小，详情请参考: [advantages of setting `-Xms` and `-Xmx` to same value](https://blog.gceasy.io/2020/03/09/advantages-of-setting-xms-and-xmx-to-same-value/).


Metaspace is the region where JVM’s metadata definitions, such as class definitions, method definitions, will be stored.  By default, the amount of memory that can be used to store this metadata information is unlimited (i.e. limited by your container or machine’s RAM size). You need to use `-XX:MaxMetaspaceSize`  argument to specify an upper limit on the amount of memory that can be used to store metadata information.


JDK8引入了 Metaspace 空间，用于存储JVM中的公用信息，比如class定义，方法定义等等。
JVM默认没有限制 Metaspace，但受限于宿主机的物理内存大小， 如果程序出BUG，JVM使用的内存超出限制，可能会被操作系统给杀了。
参考案例: [OOM终结者参数调优](https://renfufei.blog.csdn.net/article/details/80468217)
我们可以使用参数 `-XX:MaxMetaspaceSize` 来限制Meta区的大小：

```
-XX:MaxMetaspaceSize=2g
```

## 2. GC Algorithm

As on date (March 2020), there are 7 different GC algorithms in OpenJDK:

- a. Serial GC
- b. Parallel GC
- c. Concurrent Mark & Sweep GC
- d. G1 GC
- e. Shenandoah GC
- f. Z GC
- g. Epsilon GC


## 2. 指定GC算法

截至目前， OpenJDK 支持的所有 GC 算法，一共有 7 类：

- 1. 串行 GC（Serial GC）：单线程执行，应用需要暂停；
- 1. 并行 GC（ParNew、Parallel Scavenge、Parallel Old）：多线程并行地执行垃圾回收，关注与高吞吐；
- 1. CMS（Concurrent Mark-Sweep）：多线程并发标记和清除，关注与降低延迟；
- 1. G1（G First）：通过划分多个内存区域做增量整理和回收，进一步降低延迟；
- 1. ZGC（Z Garbage Collector）：通过着色指针和读屏障，实现几乎全部的并发执行，几毫秒级别的延迟，线性可扩展；
- 1. Epsilon：实验性的 GC，供性能分析使用；
- 1. Shenandoah：G1 的改进版本，跟 ZGC 类似。

详细情况可参考: [JVM 核心技术 32 讲](https://gitbook.cn/gitchat/column/5de76cc38d374b7721a15cec/topic/5df0bea044f0aa237c287868)


If you don’t specify the GC algorithm explicitly, then JVM will choose the default algorithm. Until Java 8, Parallel GC is the default GC algorithm. Since Java 9, G1 GC is the default GC algorithm.

Selection of the GC algorithm plays a crucial role in determining the application’s performance. Based on our research, we are observing excellent performance results with Z GC algorithm. If you are running with JVM 11+, then you may consider using Z GC algorithm (i.e. -XX:+UseZGC). More details about Z GC algorithm can be found here.

Below table summarizes the JVM argument that you need to pass to activate each type of Garbage Collection algorithm.

如果没有明确指定，则JVM会使用默认的GC算法。
在Java8及之前的版本，默认使用成熟稳定的 Parallel GC 算法。
从Java8开始， 默认的GC算法为G1。

GC算法对系统性能的影响也很明显，直接影响了一些关键的性能指标: `响应延迟` 以及 `吞吐量` 。
根据业界的研究，ZGC是目前最棒的GC算法。
如果系统使用JDK 11 及以上版本，大部分系统都可以考虑使用ZGC。

需要提醒的是, JDK11中 `NotificationListener` 通知的 `GarbageCollectionNotificationInfo#duration` 并不是ZGC的暂停时间，而是ZGC的GC周期消耗的时间，这一点和其他GC算法不同。 具体案例可参考： [ZGC简介](https://github.com/cncounter/translation/tree/master/tiemao_2020/23_zgc_intro)。



各种GC算法对应的JVM启动参数见下表:

| GC 算法	 | JVM 参数 |
| -------------- | ------------ |
| Serial GC	 | `-XX:+UseSerialGC` |
| Parallel GC	 | `-XX:+UseParallelGC` |
| CMS GC	 | `-XX:+UseConcMarkSweepGC` |
| G1 GC	 | `-XX:+UseG1GC` |
| Shenandoah GC	 | `-XX:+UseShenandoahGC` |
| Z GC	 | `-XX:+UnlockExperimentalVMOptions -XX:+UseZGC` |
| Epsilon GC	 | `-XX:+UseEpsilonGC` |


## 3. Enable GC Logging

Garbage Collection logs contain information about Garbage Collection events, memory reclaimed, pause time duration, … You can enable Garbage collection log by passing following JVM arguments:


## 3. 开启GC日志

GC日志中包含了很多重要的信息： GC事件、内存回收、暂停时间等等。
很多疑难杂症的诊断和排查都需要分析GC日志才能得出正确的结论。

From JDK 1 to JDK 8:


从 JDK 1 到 JDK 8 版本都使用以下参数来输出GC日志:

```
-XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:{file-path}
```

From JDK 9 and above:

在 JDK 9 及之后的版本中，启动参数有一些变化，继续使用原来的参数配置可能会在启动时报错。
不过也不用担心，如果碰到，一般都可以从错误提示中找到对应的处置措施和解决方案。
一般格式为:

```
-Xlog:gc*:file={file-path}
```

Example:

使用示例:

```
-XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:/opt/workspace/gc.log

-Xlog:gc*=info:file=/opt/workspace/gc.log:time:filecount=0
```

Typically GC logs are used for tuning garbage collection performance. However, GC logs contain vital micro metrics. These metrics can be used for forecasting application’s availability and performance characteristics. In this article we would like to highlight one such micrometric: ‘GC Throughput‘ (to read more on other available micrometrics, you may refer to this article). GC Throughput is the amount of time your application spends in processing customer transactions vs the amount of time it spends in processing GC activities. Say if your application’s GC throughput is 98%, then it means application is spending 98% of its time in processing customer activity, and the remaining 2% is spent in GC activity.

Now let’s look at the heap usage graph of a healthy JVM:

一般来说，打印GC日志的主要目的是为了调优垃圾回收的性能。
当然，GC日志中包含很多详细的指标信息。
这些指标可用于分析应用程序的性能特征，预测系统的可用性。
下面着重介绍的指标是： `GC吞吐量(Throughput)`。 其他性能指标信息可参考: [应用程序微观性能指标简介](https://blog.gceasy.io/2019/03/13/micrometrics-to-forecast-application-performance/)。
GC吞吐量是指： 业务处理所占的CPU时间，与GC活动所消耗的CPU时间之间的比例。
例如应用程序的GC吞吐量为98％，表示应用程序将98％的时间用于处理正常业务，其余2％用于GC活动。

我们来看一个健康状态的JVM，其堆内存的使用情况如下图所示：

![](https://i1.wp.com/blog.gceasy.io/wp-content/uploads/2020/03/healthy-jvm-heap.png?w=960&ssl=1)

> Fig: Healthy JVM’s heap usage graph (generated by https://gceasy.io)

> 健康状态的JVM堆内存使用情况图（由 https://gceasy.io 生成）

You can see a perfect saw-tooth pattern. You can notice that when Full GC (red triangle) runs, memory utilization drops all the way to bottom.

Now let’s look at the heap usage graph of a sick JVM:

这是一个完美的锯齿型图案。 可以看到，当执行 Full GC时（图中用红色三角形标识），内存使用率会直接下降到谷底。

再看有问题的JVM堆使用情况：

![](https://i1.wp.com/blog.gceasy.io/wp-content/uploads/2020/03/sick-jvm-heap-1.png?w=1075&ssl=1)

> Fig: Sick JVM’s heap usage graph (generated by https://gceasy.io)

> 问题状态的JVM堆内存使用情况图（由 https://gceasy.io 生成）

You can notice towards the right end of the graph, even though GC repeatedly runs, memory utilization isn’t dropping. It’s a classic indication that the application is suffering from some sort of memory problem.

If you take a closer look at the graph, you will notice that repeated full GC’s started to happen right around 8 am. However, the application starts to get OutOfMemoryError only around 8:45 am. Till 8 am, the application’s GC throughput was about 99%. But right after 8 am, GC throughput started to drop down to 60%. Because when repeated GC runs, the application wouldn’t be processing any customer transactions and it will only be doing GC activity. As a proactive measure, if you notice GC throughput starts to drop, you can take out the JVM from the load balancer pool. So that unhealthy JVM will not process any new traffic. It will minimize the customer impact.

在图的右边，我们可以注意到，即使反复运行了多次GC，内存使用率也没有下降。
这就是程序存在某些内存问题的典型症状。

仔细观察，可以看到从8点左右，就开始产生大量的FullGC， 但该应用直到 8:45 左右才有 OutOfMemoryError。
在8点之前，该应用的GC吞吐量约为99％。
但在8点之后，GC吞吐量指标下降到60％左右。 因为连续执行大量的GC时，应用就不再处理实际业务，只会死命地执行GC。
作为一种主动措施，如果发现GC吞吐量开始下降，则可以将JVM从负载均衡中摘除。
这样，运行状况不佳的JVM将不会处理新的流量， 最大程度地减少对客户的影响。

![](https://i0.wp.com/blog.gceasy.io/wp-content/uploads/2020/03/repeated-full-gc-oom.png?w=1047&ssl=1)

> Fig: Repeated Full GC happens way before OutOfMemoryError

> 先是连续执行了很多次Full GC，然后才抛出 OutOfMemoryError

You can monitor GC related micrometrics in real time, using [GCeasy REST API](https://www.youtube.com/watch?v=6G0E4O5yxks).

当然，我们也可以通过API或者指标监控系统，实时监测GC相关的指标。 例如 [GCeasy REST API](https://www.youtube.com/watch?v=6G0E4O5yxks).

## 4. `-XX:+HeapDumpOnOutOfMemoryError`, `-XX:HeapDumpPath`

OutOfMemoryError is a serious problem that will affect your application’s availability/performance SLAs. To diagnose OutOfMemoryError or any memory-related problems, one would have to capture heap dump right at the moment or few moments before the application starts to experience OutOfMemoryError. As we don’t know when OutOfMemoryError will be thrown, it’s hard to capture heap dump manually at the right around the time when it’s thrown. However, capturing heap dumps can be automated by passing following JVM arguments:

## 4. 内存溢出时自动执行堆转储

OutOfMemoryError 是一个很严重的问题， 会影响应用系统的可用性和性能， 继而影响我们的SLA。
为了诊断 OutOfMemoryError 或者其他内存相关的问题，必须在抛出OutOfMemoryError的那一个瞬间获取堆内存快照(heap dump)。
当然我们肯定不知道何时会发生 OutOfMemoryError，所以很难手工在内存溢出时执行堆转储。
官方肯定考虑到了这种情形，所以我们可以指定以下JVM启动参数来自动获取堆转储：

```
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath={HEAP-DUMP-FILE-PATH}
```

In `-XX:HeapDumpPath`, you need to specify the file path where heap dump should be stored. When you pass these two JVM arguments, heap dumps will be automatically captured and written to a defined file path, when OutOfMemoryError is thrown. Example:


指定 `-XX:HeapDumpPath` 参数时需要带上保存堆转储的具体文件名或者目录，JVM会自动判断. 例如:

```
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/crashes/my-heap-dump.hprof
```

By default the heap dump is created in a file called `java_pidpid.hprof` in the working directory of the VM, as in the example above. You can specify an alternative file name or directory with the -XX:HeapDumpPath= option. For example -XX:HeapDumpPath=/disk2/dumps will cause the heap dump to be generated in the /disk2/dumps directory.

Once heap dumps are captured, you can use tools like [HeapHero](https://heaphero.io/), [EclipseMAT](https://www.eclipse.org/mat/) to analyze heap dumps.

More details around the OutOfMemoryError JVM arguments can be found in [this article](https://blog.heaphero.io/2019/06/21/outofmemoryerror-related-jvm-arguments/).

如果不指定 `-XX:HeapDumpPath`, 则会默认使用Java程序的启动目录，堆转储文件名格式为 `java_pid{pid}.hprof`。

如果指定的目录不存在或无权限则会报错。

获取到堆转储之后，可使用 [HeapHero](https://heaphero.io/)， [EclipseMAT](https://www.eclipse.org/mat/) 等工具来分析。

更多关于 OutOfMemoryError 的JVM参数可参考: [OutOfMemoryError related JVM Arguments](https://blog.heaphero.io/2019/06/21/outofmemoryerror-related-jvm-arguments/)。

## 5. `-Xss`

Each application will have tens, hundreds, thousands of threads. Each thread will have its own stack. In each thread’s stack following information are stored:

- a. Methods/functions that are currently executed
- b. Primitive datatypes
- c. Variables
- d. object pointers
- e. return values.

Each one of them consumes memory. If their consumption goes beyond a certain limit, then StackOverflowError is thrown. More details about StackOverflowError & it’s solution can be found in this article. However, you can increase the thread’s stack size limit by passing `-Xss` argument. Example:

## 5. 指定线程栈大小

每个程序一般会有几十上百个线程，多的可能还会有上千个。
每个线程都有专用的线程栈。
在每个线程的线程栈中，需要存储以下信息：


- a. 当前执行的方法/函数
- b. 原生数据类型
- c. 变量
- d. 对象指针
- e. 返回值

每个部分都会占用内存。
如果它们的使用量超出某个限制值，则会引发 StackOverflowError。
更多信息可参考: [StackOverflowError 的原因和解决方案](https://blog.fastthread.io/2018/09/24/stackoverflowerror/)。
如果确实是程序正常运行所需， 我们可以设置 `-Xss` 参数来增加线程栈的大小限制。


```
-Xss256k
```

If you set this `-Xss` value to a huge number, then memory will be blocked and wasted. Say suppose you are assigning `-Xss` value to be `2mb` whereas, it needs only 256kb, then you will end up wasting huge amount of memory, not just 1792kb (i.e. `2mb – 256kb`). Do you wonder why?

Say your application has 500 threads, then with `-Xss` value to be `2mb`, your threads will be consuming 1000mb of memory (i.e. `500 threads x 2mb/thread`). On the other hand, if you have allocated `-Xss` only to be `256kb`, then your threads will be consuming only `125mb` of memory (i.e. `500 threads x 256kb/thread`). You will save `875mb` (i.e. `1000mb – 125mb`) of memory per JVM. Yes, it will make such a huge difference.

Note: Threads are created outside heap (i.e. `-Xmx`), thus this `1000mb` will be in addition to `-Xmx` value you have already assigned. To understand why threads are created outside heap, you can watch this short video clip.

Our recommendation is to start from a low value (say `256kb`). Run thorough regression, performance, and AB testing with this setting. Only if you experience StackOverflowError then increase the value, otherwise consider sticking on to a low value.

如果 `-Xss` 的值设置太大，则会造成内存占用和浪费。
假设指定 `-Xss2m`，而程序正常运行只需要256kb，则会浪费大量的内存， 算法不是 `2mb – 256kb = 1792kb`, 可以先想一想为什么？

比如程序有 500个线程，设置 `-Xss` 值为 `2mb`，那么线程栈部分则会消耗 1000mb的内存（即 `500 threads x 2mb/thread` ）。
如果 `-Xss` 值设置为 `256kb`， 那么线程栈部分则只会消耗 `125mb` 的内存（即`500 threads x 256kb/thread`）。
这样的话每个JVM实例就能节省 `875mb`的内存 (`1000mb – 125mb = 875mb`)。
差别还是很明显的。

注意： 线程栈所占的内存和堆内存的大小 (`-Xmx`)没关系.   
如果已经指定了 `-Xmx`，这时候JVM占用的内存还要加上栈内存的这 `1000mb`。
要了解为什么在堆外创建线程，可以观看 [此短片](https://www.youtube.com/watch?v=uJLOlCuOR4k&t=9s)。

我们建议先设置一个较小的值, 例如 `-Xss256k`。 然后进行完整的回归测试，性能性能和AB测试。
只有在遇到 StackOverflowError 时才增加这个配置， 否则请坚持使用较小的值。

> 有些操作系统会有优化, 先分配地址空间，实际使用才分配物理内存，当然，和操作系统的内存页大小有关系，看具体情况。


## 6. `-Dsun.net.client.defaultConnectTimeout` and `-Dsun.net.client.defaultReadTimeout`

Modern applications use numerous protocols (i.e. SOAP, REST, HTTP, HTTPS, JDBC, RMI…) to connect with remote applications. Sometimes remote applications might take a long time to respond. Sometimes it may not respond at all.

If you don’t have proper timeout settings, and if remote applications don’t respond fast enough, then your application threads/resources will get stuck. Remote applications unresponsiveness can affect your application’s availability. It can bring down your application to grinding halt. To safeguard your application’s high availability, appropriate timeout settings should be configured.

You can pass these two powerful timeout networking properties at the JVM level that can be globally applicable to all protocol handlers that uses `java.net.URLConnection`:

- `sun.net.client.defaultConnectTimeout` specifies the timeout (in milliseconds) to establish the connection to the host. For example, for HTTP connections, it is the timeout when establishing the connection to the HTTP server.
- `sun.net.client.defaultReadTimeout` specifies the timeout (in milliseconds) when reading from the input stream when a connection is established to a resource.

Example, if you would like to set these properties to 2 seconds:

## 6. 默认连接超时时间

现在的应用系统之间通过各种协议相互连接（比如 SOAP，REST，HTTP，HTTPS，JDBC，RMI等等）。
但有时候远程服务器的响应时间很长，甚至客户端一直收不到响应信息。

如果程序没有设置合适的连接超时时间，而且远程服务的响应速度不够快的话，那么应用线程/资源会被阻塞，继而影响系统的可用性。
严重的拖慢甚至拖死我们的系统。 要保证系统高可用，创建连接时应该配置合适的超时时间。

我们可以通过设置JVM启动参数，来指定虚拟机级别的最大超时时间，下面这两个属性适用于所有使用 `java.net.URLConnection` 的协议处理器：

- `sun.net.client.defaultConnectTimeout` 指定创建连接时的超时时间（单位：毫秒）。 例如，与HTTP服务器建立连接时的超时时间。
- `sun.net.client.defaultReadTimeout` 读超时时间，即： 连接成功之后，连续多少毫秒没有读取到数据则超时。

例如，将这些属性都设置为2秒：

```
-Dsun.net.client.defaultConnectTimeout=2000
-Dsun.net.client.defaultReadTimeout=2000
```

Note, by default values for these 2 properties is -1, which means no timeout is set. More details on these properties can be found in this article.

需要注意的是，这两个属性的默认值都是 `-1`，表示不设置超时。 关于这些属性的详细信息，请参见本文。

## 7. `-Duser.timeZone`

Your application might have sensitive business requirements around time/date. For example, if you are building a trading application, you can’t take transaction before 9:30 am. To implement those time/date related business requirements, you might be using `java.util.Date`, `java.util.Calendar` objects. These objects, by default, picks up time zone information from the underlying operating system. This will become a problem; if your application is running in a distributed environment. Look at the below scenarios:

- a. If your application is running across multiple data centers, say, San Francisco, Chicago, Singapore – then JVMs in each data center would end up having different time zone. Thus, JVMs in each data center would exhibit different behaviors. It would result in inconsistent results.

- b. If you are deploying your application in a cloud environment, applications could be moved to different data centers without your knowledge. In that circumstance also, your application would end up producing different results.

- c. Your own Operations team can also change the time zone without bringing to the development team’s knowledge. It would also skew the results.

To avoid these commotions, it’s highly recommended to set the time zone at the JVM using the `-Duser.timezone` system property. Example if you want to set EDT time zone for your application, you will do:

## 7. 设置时区

有些业务需要在某个具体的时间/日期执行。
例如，很多股票交易系统不允许在上午9:30之前进行交易。
为了实现这些与时间/日期相关的业务需求，可能会用到 `java.util.Date`，`java.util.Calendar`对象。
默认情况下，这些对象从底层操作系统获取时区信息。
如果系统在分布式集群环境中运行, 可能会产生一些问题。
可能会碰到这些问题：

- 1. 如果系统部署在多个数据中心运行（例如，旧金山，芝加哥，新加坡等等）， 这些数据中心跨域了多个时区。 有可能会导致JVM产生不一致的行为, 造成结果的不一致。
- 2. 如果在云环境中部署应用系统，则可能会在用户无感知的情况下将应用迁移到其他数据中心。在这种情况下，也有可能会产生不同的结果。
- 3. Ops团队可以会更改默认时区，如果没有与开发团队沟通。也会造成某些不可预料的结果。

所以、为了减少麻烦，建议使用系统属性 `-Duser.timezone` 明确指定JVM的时区，。

设置时区的示例：


```shell
-Duser.timezone=GMT
-Duser.timezone=GMT+08
```

或者 Java代码:

```java
import java.util.*;
// 查看可用的系统时区
System.out.println(Arrays.toString(TimeZone.getAvailableIDs()));
// 结果很多, 示例:
/*
Etc/GMT, Etc/GMT+0, Etc/GMT+1, Etc/GMT+10, Etc/GMT+11, Etc/GMT+12,
Etc/GMT+2, Etc/GMT+3, Etc/GMT+4, Etc/GMT+5, Etc/GMT+6, Etc/GMT+7, Etc/GMT+8, Etc/GMT+9,
Etc/GMT-0, Etc/GMT-1, Etc/GMT-10, Etc/GMT-11, Etc/GMT-12, Etc/GMT-13, Etc/GMT-14,
Etc/GMT-2, Etc/GMT-3, Etc/GMT-4, Etc/GMT-5, Etc/GMT-6, Etc/GMT-7, Etc/GMT-8, Etc/GMT-9,
Etc/GMT0, Etc/Greenwich, Etc/UCT, Etc/UTC
*/
```

## Conclusion

In this article, we have attempted to summarize some of the important JVM arguments and their positive impacts. We hope you may find it helpful.

## 小结

本文介绍了最重要的几个JVM启动参数， 建议读者倒回去再快速过一遍。

## 相关链接


- <https://blog.gceasy.io/2020/03/18/7-jvm-arguments-of-highly-effective-applications/>

- [Java HotSpot VM Troubleshooting Guide Command-Line Options](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/clopts001.html)
