---
layout: post
category: "计算机"
title:  "进程和线程"
---

**进程是资源分配的最小单位，线程是任务调度的最小单位。**

### 进程

对于操作系统来说，一个任务就是一个进程（Process），比如打开一个浏览器就是启动一个浏览器进程，打开一个记事本就启动了一个记事本进程，打开两个记事本就启动了两个记事本进程，打开一个Word就启动了一个Word进程。

### 线程

有些进程还不止同时干一件事，比如Word，它可以同时进行打字、拼写检查、打印等事情。在一个进程内部，要同时干多件事，就需要同时运行多个“子任务”，我们把进程内的这些“子任务”称为线程（Thread）。

### 多进程

比如我们写论文的时候，既要打开浏览器查资料，又要复制粘贴到Word加以改造，这里面有两个进程，他们是怎么同时运行的呢？

单核CPU是通过分时技术实现多进程的。任务1执行0.01秒，然后切换到任务2执行0.02秒，由于CPU的执行速度实在太快了，在用户看来就好像是两个任务同时在运行一样。只有多核CPU才能实现真正意义上的多进程。

### 多线程

由于每个进程至少要干一件事，所以，**一个进程至少有一个线程**。当然，像Word这种复杂的进程可以有多个线程，多个线程可以同时执行，多线程的执行方式和多进程是一样的，也是由操作系统在多个线程之间快速切换，让每个线程都短暂地交替运行，看起来就像同时执行一样。

### 切换的代价

无论是多进程还是多线程，既然要切换着执行，就要保存现场（上下文环境），下一次才能继续执行。所以，多任务一旦多到一个限度，就会消耗掉系统所有的资源，结果效率急剧下降，所有任务都做不好。

### 多进程 VS 多线程

多进程的优点是稳定性高，一个子进程挂了，不会影响主进程和其他子进程。缺点是创建进程的开销比较大，另外，操作系统能同时运行的进程数也是有限的，在内存和CPU的限制下，如果有几千个进程同时运行，操作系统连调度都会成问题。

多线程通常比多进程快一点，但致命的缺点是任何一个线程挂掉都可能造成整个进程崩溃，因为**所有线程共享进程的内存空间**。

在Thread和Process中，应当优选Process，因为Process更稳定，而且，Process可以分布到多台机器上，而Thread最多只能分布到同一台机器的多个CPU上。

### 怎么用好多任务

我们可以把任务分为计算密集型和IO密集型。计算密集型主要消耗CPU，IO密集型主要消耗网络和磁盘

- 计算密集型

	计算密集型任务的特点是要进行大量的计算，消耗CPU资源，比如计算圆周率、对视频进行高清解码等等，全靠CPU的运算能力。这种计算密集型任务虽然也可以用多任务完成，但是任务越多，花在任务切换的时间就越多，CPU执行任务的效率就越低，所以，要最高效地利用CPU，计算密集型任务同时进行的数量应当等于CPU的核心数。

	计算密集型任务由于主要消耗CPU资源，因此，代码运行效率至关重要。Python这样的脚本语言运行效率很低，完全不适合计算密集型任务。对于计算密集型任务，最好用C语言编写。

- IO密集型

	涉及到网络、磁盘IO的任务都是IO密集型任务，这类任务的特点是CPU消耗很少，任务的大部分时间都在等待IO操作完成（因为IO的速度远远低于CPU和内存的速度）。对于IO密集型任务，任务越多，CPU效率越高，但也有一个限度。常见的大部分任务都是IO密集型任务，比如Web应用。

### 异步IO

考虑到CPU和IO之间巨大的速度差异，一个任务在执行的过程中大部分时间都在等待IO操作，单进程单线程模型会导致别的任务无法并行执行，因此，我们才需要多进程模型或者多线程模型来支持多任务并发执行。

现代操作系统对IO操作已经做了巨大的改进，最大的特点就是支持异步IO。如果充分利用操作系统提供的异步IO支持，就可以用单进程单线程模型来执行多任务，这种全新的模型称为事件驱动模型，Nginx就是支持异步IO的Web服务器，它在单核CPU上采用单进程模型就可以高效地支持多任务。在多核CPU上，可以运行多个进程（数量与CPU核心数相同），充分利用多核CPU。由于系统总的进程数量十分有限，因此操作系统调度非常高效。用异步IO编程模型来实现多任务是一个主要的趋势。对应到Python语言，单线程的异步编程模型称为协程。

### 参考资料
1. [进程和线程-廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014319272686365ec7ceaeca33428c914edf8f70cca383000)