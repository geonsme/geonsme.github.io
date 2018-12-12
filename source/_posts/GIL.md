---
title:  GIL对多线程的影响
toc: true
description:  
date: 2017-11-19
category: 
 - python
tag:
 - python
---

## GIL是什么

首先需要明确的一点是GIL并不是Python的特性，它是在实现Python解析器(CPython)时所引入的一个概念。就好比C++是一套语言（语法）标准，但是可以用不同的编译器来编译成可执行代码。有名的编译器例如GCC，INTEL C++，Visual C++等。Python也一样，同样一段代码可以通过CPython，PyPy，Psyco等不同的Python执行环境来执行。像其中的JPython就没有GIL。然而因为CPython是大部分环境下默认的Python执行环境。所以在很多人的概念里CPython就是Python，也就想当然的把GIL归结为Python语言的缺陷。所以这里要先明确一点：GIL并不是Python的特性，Python完全可以不依赖于GIL。

那么CPython实现中的GIL又是什么呢？GIL全称Global Interpreter Lock为了避免误导，我们还是来看一下官方给出的解释

```
In CPython, the global interpreter lock, or GIL, is a mutex that prevents multiple native threads from executing Python bytecodes at once. This lock is necessary mainly because CPython’s memory management is not thread-safe. (However, since the GIL exists, other features have grown to depend on the guarantees that it enforces.)
```

在CPython中，全局解释器锁是一个排他锁，防止多个线程在同一时刻同时解释Python字节码。这个锁是必要的，否则CPython的内存管理不是安全的。但是，自从GIL存在后，其他后续的库也因此这么设计，导致GIL至今无法被排除掉。

## 为什么会有GIL
由于物理上得限制，各CPU厂商在核心频率上的比赛已经被多核所取代。为了更有效的利用多核处理器的性能，就出现了多线程的编程方式，而随之带来的就是线程间数据一致性和状态同步的困难。即使在CPU内部的Cache也不例外，为了有效解决多份缓存之间的数据同步时各厂商花费了不少心思，也不可避免的带来了一定的性能损失。

Python当然也逃不开，为了利用多核，Python开始支持多线程。而解决多线程之间数据完整性和状态同步的最简单方法自然就是加锁。 于是有了GIL这把超级大锁，而当越来越多的代码库开发者接受了这种设定后，他们开始大量依赖这种特性（即默认python内部对象是thread-safe的，无需在实现时考虑额外的内存锁和同步操作）。

慢慢的这种实现方式被发现是蛋疼且低效的。但当大家试图去拆分和去除GIL的时候，发现大量库代码开发者已经重度依赖GIL而非常难以去除了。有多难？做个类比，像MySQL这样的“小项目”为了把Buffer Pool Mutex这把大锁拆分成各个小锁也花了从5.5到5.6再到5.7多个大版为期近5年的时间，并且仍在继续。MySQL这个背后有公司支持且有固定开发团队的产品走的如此艰难，那又更何况Python这样核心开发和代码贡献者高度社区化的团队呢？

所以简单的说GIL的存在更多的是历史原因。如果推到重来，多线程的问题依然还是要面对，但是至少会比目前GIL这种方式会更优雅。

## GIL的影响

从上文的介绍和官方的定义来看，GIL无疑就是一把全局排他锁。毫无疑问全局锁的存在会对多线程的效率有不小影响。甚至就几乎等于Python是个单线程的程序。 那么读者就会说了，全局锁只要释放的勤快效率也不会差啊。只要在进行耗时的IO操作的时候，能释放GIL，这样也还是可以提升运行效率的嘛。或者说再差也不会比单线程的效率差吧。理论上是这样，而实际上呢？Python比你想的更糟。

## GIL的缺陷

按照Python社区的想法，操作系统本身的线程调度已经非常成熟稳定了，没有必要自己搞一套。所以Python的线程就是C语言的一个pthread，并通过操作系统调度算法进行调度（例如linux是CFS）。为了让各个线程能够平均利用CPU时间，python会计算当前已执行的微代码数量，达到一定阈值后就强制释放GIL。而这时也会触发一次操作系统的线程调度（当然是否真正进行上下文切换由操作系统自主决定）。

这种模式在只有一个CPU核心的情况下毫无问题。任何一个线程被唤起时都能成功获得到GIL（因为只有释放了GIL才会引发线程调度）。但当CPU有多个核心的时候，问题就来了。从伪代码可以看到，从release GIL到acquire GIL之间几乎是没有间隙的。所以当其他在其他核心上的线程被唤醒时，大部分情况下主线程已经又再一次获取到GIL了。这个时候被唤醒执行的线程只能白白的浪费CPU时间，看着另一个线程拿着GIL欢快的执行着。然后达到切换时间后进入待调度状态，再被唤醒，再等待，以此往复恶性循环。

## 避免GIL的影响

用multiprocessing替代Thread
multiprocessing库的出现很大程度上是为了弥补thread库因为GIL而低效的缺陷。它完整的复制了一套thread所提供的接口方便迁移。唯一的不同就是它使用了多进程而不是多线程。每个进程有自己的独立的GIL，因此也不会出现进程之间的GIL争抢。

当然multiprocessing也不是万能良药。它的引入会增加程序实现时线程间数据通讯和同步的困难。就拿计数器来举例子，如果我们要多个线程累加同一个变量，对于thread来说，申明一个global变量，用thread.Lock的context包裹住三行就搞定了。而multiprocessing由于进程之间无法看到对方的数据，只能通过在主线程申明一个Queue，put再get或者用share memory的方法。这个额外的实现成本使得本来就非常痛苦的多线程程序编码，变得更加痛苦了。具体难点在哪有兴趣的读者可以扩展阅读这篇文章

## 改进

将切换颗粒度从基于opcode计数改成基于时间片计数

避免最近一次释放GIL锁的线程再次被立即调度

新增线程优先级功能（高优先级线程可以迫使其他线程释放所持有的GIL锁）

## 总结

因为GIL的存在，只有IO Bound场景下得多线程会得到较好的性能，因此IO密集型可以给GIL释放的有效时间，而计算密集型会一直占用CPU计算，没有给予充足的调度时间给其他线程获取GIL，因此浪费了线程调度时间，计算密集型使用多线程效果相反。IO密集型在处于等待时间会给予比较长的等待时间，比如有time.sleep这种也能比较好的效果。

如果对并行计算性能较高的程序可以考虑把核心部分也成C模块，或者索性用其他语言实现

GIL在较长一段时间内将会继续存在，但是会不断对其进行改进