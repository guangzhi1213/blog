---
title: "Tomcat"
date: 2021-04-25T11:19:30+08:00
lastmod: 2021-04-25T11:19:30+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

from [segmentfault](https://segmentfault.com/a/1190000009319829)

环境:

* java version 1.8
* tomcat 8.5
* jmeter 3.1

jmeter参数:

* 300线程
* 1000循环
* url: localhost:8080

tomcat server.xml 参数:

* protocol="org.apache.coyote.http11.Http11Nio2Protocol"
* acceptCount="5000"
* maxConnections="20000"

tomcat JVM 参数:

* -server -Xms4g -Xmx4g

JIT的介入
===

Tomcat server.xml 保持默认值

在不重启Tomcat 的情况想， 两次benchmark得到的throughput 数据相差较大
* 1次 9000/s
* 2次 17000/s

说明，在tomcat存在warmup机制，性能表现在warmup之后会更好

使用 `jvisualvm` 收集两次benchmark的相关数据， 发现这和JDK的compierThread存在关系，

![123](/images/redis_compierthread.png)

在第一次benchmark的前期，compilerThread动作比较多，此时NIO的吞吐量一直为维持在比较低的水平，在第一次benchmark后期， compilerThread降下来了， NIO 的吞吐量就上去了

第二次benchmark几乎没有compilerThread动作， 此时NIO的吞吐量一直维持在比较高的水平。

在经过一番调查之后， 发现这是由于JIT对代码做了优化， JIT会将执行频繁的代码的bytecode编译成nativecode， 从而加快执行速度。

参考文章:

* [http://blog.takipi.com/java-on-steroids-5-super-useful-jit-optimization-techniques/](http://blog.takipi.com/java-on-steroids-5-super-useful-jit-optimization-techniques/)
* [https://www.beyondjava.net/blog/a-close-look-at-javas-jit-dont-waste-your-time-on-local-optimizations/](https://www.beyondjava.net/blog/a-close-look-at-javas-jit-dont-waste-your-time-on-local-optimizations/)
* [https://blog.jooq.org/2016/07/19/the-java-jit-compiler-is-darn-good-in-optimization/](https://blog.jooq.org/2016/07/19/the-java-jit-compiler-is-darn-good-in-optimization/)


maxThreads
===

根据tomcat文档的说法， 此参数决定了同时最多能够处理多少请求， 默认值是 `200`, 而我们的Jmeter脚本是300并发，显然是不够的， 所以调整为 `server.xml` 追加 `Connector` 参数:
```
maxThread="500"
minSpareThreads="500"
```

这么配置是提高Tomcat处理线程数， `maxThreads` 和 `minSpareThreads` 设置为一样是为了让Tomcat在启动时就创建 1000 线程，这个和把JVM参数 `-Xms, -Xmx` 设置为一样是一个道理。

在不重启Tomcat的情况下， 两次benchmark得到的throughput 数据:

* 第一次: 12000/s
* 第二次: 17000/s

这说明在没有warmup的情况下， `maxThreads` 的提升能够提升Tomcat 的性能表现

这说明在没有warmup的情况下， `maxThreads` 的提升能够提升tomcat的性能表现

关于maxThread 应该设置为多少合适可以见So的回答

* [http://stackoverflow.com/a/6781781/1287790](http://stackoverflow.com/a/6781781/1287790)

<pre>
the "how many threads problem" is quite a big and complicated issue, and cannot be answered with a simple rule of thumb.

Considering how many cores you have is useful for multi threaded applications that tend to consume a lot of CPU, like number crunching and the like. This is rarely the case for a web-app, which is usually hogged not by CPU but by other factors.

One common limitation is lag between you and other external systems, most notably your DB. Each time a request arrive, it will probably query the database a number of times, which means streaming some bytes over a JDBC connection, then waiting for those bytes to arrive to the database (even is it's on localhost there is still a small lag), then waiting for the DB to consider our request, then wait for the database to process it (the database itself will be waiting for the disk to seek to a certain region) etc...

During all this time, the thread is idle, so another thread could easily use that CPU resources to do something useful. It's quite common to see 40% to 80% of time spent in waiting on DB response.

The same happens also on the other side of the connection. While a thread of yours is writing its output to the browser, the speed of the CLIENT connection may keep your thread idle waiting for the browser to ack that a certain packet has been received. (This was quite an issue some years ago, recent kernels and JVMs use larger buffers to prevent your threads for idling that way, however a reverse proxy in front of you web application server, even simply an httpd, can be really useful to avoid people with bad internet connection to act as DDOS attacks :) )

Considering these factors, the number of threads should be usually much more than the cores you have. Even on a simple dual or quad core server, you should configure a few dozens threads at least.

So, what is limiting the number of threads you can configure?

First of all, each thread (used to) consume a lot of resources. Each thread have a stack, which consumes RAM. Moreover, each Thread will actually allocate stuff on the heap to do its work, consuming again RAM, and the act of switching between threads (context switching) is quite heavy for the JVM/OS kernel.

This makes it hard to run a server with thousands of threads "smoothly".

Given this picture, there are a number of techniques (mostly: try, fail, tune, try again) to determine more or less how many threads you app will need:

1) Try to understand where your threads spend time. There are a number of good tools, but even jvisualvm profiler can be a great tool, or a tracing aspect that produces summary timing stats. The more time they spend waiting for something external, the more you can spawn more threads to use CPU during idle times.

2) Determine your RAM usage. Given that the JVM will use a certain amount of memory (most notably the permgen space, usually up to a hundred megabytes, again jvisualvm will tell) independently of how many threads you use, try running with one thread and then with ten and then with one hundred, while stressing the app with jmeter or whatever, and see how heap usage will grow. That can pose a hard limit.

3) Try to determine a target. Each user request needs a thread to be handled. If your average response time is 200ms per "get" (it would be better not to consider loading of images, CSS and other static resources), then each thread is able to serve 4/5 pages per second. If each user is expected to "click" each 3/4 seconds (depends, is it a browser game or a site with a lot of long texts?), then one thread will "serve 20 concurrent users", whatever it means. If in the peak hour you have 500 single users hitting your site in 1 minute, then you need enough threads to handle that.

4) Crash test the high limit. Use jmeter, configure a server with a lot of threads on a spare virtual machine, and see how response time will get worse when you go over a certain limit. More than hardware, the thread implementation of the underlying OS is important here, but no matter what it will hit a point where the CPU spend more time trying to figure out which thread to run than actually running it, and that numer is not so incredibly high.

5) Consider how threads will impact other components. Each thread will probably use one (or maybe more than one) connection to the database, is the database able to handle 50/100/500 concurrent connections? Even if you are using a sharded cluster of nosql servers, does the server farm offer enough bandwidth between those machines? What else will run on the same machine with the web-app server? Anache httpd? squid? the database itself? a local caching proxy to the database like mongos or memcached?

I've seen systems in production with only 4 threads + 4 spare threads, cause the work done by that server was merely to resize images, so it was nearly 100% CPU intensive, and others configured on more or less the same hardware with a couple of hundreds threads, cause the webapp was doing a lot of SOAP calls to external systems and spending most of its time waiting for answers.

Oce you've determined the approx. minimum and maximum threads optimal for you webapp, then I usually configure it this way :

1) Based on the constraints on RAM, other external resources and experiments on context switching, there is an absolute maximum which must not be reached. So, use maxThreads to limit it to about half or 3/4 of that number.

2) If the application is reasonably fast (for example, it exposes REST web services that usually send a response is a few milliseconds), then you can configure a large acceptCount, up to the same number of maxThreads. If you have a load balancer in front of your web application server, set a small acceptCount, it's better for the load balancer to see unaccepted requests and switch to another server than putting users on hold on an already busy one.

3) Since starting a thread is (still) considered a heavy operation, use minSpareThreads to have a few threads ready when peak hours arrive. This again depends on the kind of load you are expecting. It's even reasonable to have minSpareThreads, maxSpareThreads and maxThreads setup so that an exact number of threads is always ready, never reclaimed, and performances are predictable. If you are running tomcat on a dedicated machine, you can raise minSpareThreads and maxSpareThreads without any danger of hogging other processes, otherwise tune them down cause threads are resources shared with the rest of the processes running on most OS.
</pre>



processCache
===

`processCache` 控制Tomcat内部 `RequestProcessor` 的缓存池大小， 当并发量大于此值时， Tomcat 会创建新的 `RequestProcessor` 实例， 如果并发量持续大于此值， 则会持续创建 `RequestProcessor` 实例

在这次压力测试过程中， 发现如果 `processorCache=100` 时， Tomcat 只会保留100个 `RequestProcessor` 并且会不断创建其实例， 见下图：

![processorcache](/images/redis_processorcache.png)

如果 `processorCache=400` 则tomcat 会保留300个 `RequestProcessor`, 并且不会创建新的实例， 主要是因为Jmeter的线程数为300， 也就是说同时最多只会有300个connection， 每个connection 对应一个 `RequestProcessor`

gzip 压缩
===

和gzip压缩相关的参数有 : `compression`, `compressibleMimeType`, `compressionMinSizze`, Tomcat 默认是关闭gzip压缩的， gzip压缩能够显著降低带宽。

一下是在warmup之后的benchmark结果， 开启gzip能够得到17%的提升：
* `compression="on"`, 17073/s
* `compression="off"`, 14548.3/s

但是在bencchmark多次之后，发现开启和不开启的结果相近

参考文档：
* [https://tomcat.apache.org/articles/performance.pdf](https://tomcat.apache.org/articles/performance.pdf)
* [https://medium.com/netflix-techblog/tuning-tomcat-for-a-high-throughput-fail-fast-system-e4d7b2fc163f](https://medium.com/netflix-techblog/tuning-tomcat-for-a-high-throughput-fail-fast-system-e4d7b2fc163f)





jvisualvm 监控 tomcat

===

*[https://github.com/oracle/visualvm](https://github.com/oracle/visualvm)*

```
# vi setenv.sh
JAVA_OPTS="-Xms312m -Xmx768m 
-Dcom.sun.management.jmxremote.ssl=false 
-Djava.rmi.server.hostname=118.89.229.182
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.port=18999"
```

jmx
====

远程pc 使用 `Java\jdk1.8.0_161\bin\jvisualvm.exe`  使用 jmx的方式连接 `118.89.229.182:18999` 查看jvm状态， 性能分析只能在本机启动，监控本机tomcat性能， 

jstat
====

好像查看的东西不多， 应该是配置不对

`nohup ./jstatd -J-Djava.security.policy=jstatd.all.policy -J-Djava.rmi.server.hostname=118.89.229.182 -p 54081 &> /dev/null &`
