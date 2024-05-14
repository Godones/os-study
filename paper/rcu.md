# RCU

同步、RCU、调研、

### 幻灯片

[Unraveling RCU-Usage Mysteries (Additional Use Cases)](https://events.linuxfoundation.org/wp-content/uploads/2022/02/RCUusageAdditional.2022.02.22b.LF-1.pdf)：这个幻灯片讲解的主要还是RCU在内核中的各种应用

[Mentorship Session - Unraveling RCU-Usage Mysteries (Fundamentals)](https://www.bilibili.com/video/BV1iU4y1d7jz/)

Mentorship Session: Unraveling RCU-Usage Mysteries (Fundamentals) > Intro | [Class Central Classroom](https://www.classcentral.com/classroom/youtube-mentorship-session-unraveling-rcu-usage-mysteries-fundamentals-157518)

[Mentorship Session - Unraveling RCU-Usage Mysteries (Fundamentals)_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1iU4y1d7jz/)

[what is RCU 2013 Paul McKenny_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1yv4y1u7kq/)

[What is RCU, Fundamentally?](https://www.slideserve.com/jena-garrison/what-is-rcu-fundamentally-powerpoint-ppt-presentation)（nice）：这一个比较好的RCU讲解幻灯片。

Jonathan Walpole: [Read-Copy Update \(RCU\)](https://web.cecs.pdx.edu/~theod/walpole/class/cs510/winter2018/slides/12.pdf) ：57页的幻灯片；

[(Download PPTX Powerpoint) What is RCU, Fundamentally?](https://dokumen.tips/download/link/what-is-rcu-fundamentally.html)：这个版本是可以下载的。这个幻灯片的难度与我想要的类似，是我上课的主要参考。

[What is RCU?](http://www2.rdrop.com/users/paulmck/RCU/RCU.2018.11.21c.PSU-full.pdf)：这是一个有241页的幻灯片。

[Read-Copy Update (RCU)](https://www.cs.unc.edu/~porter/courses/cse506/f12/slides/rcu.pdf)：这是一个课程中关于RCU的幻灯片，长度为18页。它只在说RCU的用法，没有说RCU的实现方法。

[Read-copy-update - HandWiki](https://handwiki.org/wiki/Read-copy-update)：这里的RCU介绍比较简洁。

### papers

M. Desnoyers, P. E. McKenney, A. S. Stern, M. R. Dagenais, and J. Walpole. User-level implementations of read-copy update. IEEE Trans. Parallel Distrib. Syst., 23(2):375–382, 2012.

[Introducing technology into the Linux kernel: A case study](https://www.researchgate.net/profile/Paul-Mckenney/publication/220624239_Introducing_technology_into_the_Linux_kernel_A_case_study/links/02bfe5141f77c79d4a000000/Introducing-technology-into-the-Linux-kernel-A-case-study.pdf)：这是一篇介绍RCU的历史的文章。有时间，可以好好看看。

[Is Parallel Programming Hard, And, If So, What Can You Do About It?](https://mirrors.edge.kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook.html)：这是一个开源的电子书。其中的附录B给出10种不同的RCU实现方法；

### RCU in Rust

[Userspace RCU](http://liburcu.org)

[urcu](https://docs.rs/urcu/latest/urcu/): Safe wrapper of the memb variant of the userspace RCU library.

### 插图

[深入理解RCU|核心原理](https://mp.weixin.qq.com/s/2ie-fnUabSfuvAbPl_Aaww)：这里的插图很不错。

[深入理解RCU | RCU源码剖析](https://blog.csdn.net/lianhunqianr1/article/details/119259755)：这里在有很详细的流程图。

[一文带你深入解析Linux内核-RCU机制（超详细~）](https://zhuanlan.zhihu.com/p/516304206)：这里也有一些插图可用。

### 代码分析

[linux RCU详细教程](https://zhuanlan.zhihu.com/p/376541044)：这里有对应代码的RCU分析。

Linux中的RCU机制[一] - [原理与使用方法](https://zhuanlan.zhihu.com/p/89439043)

Linux中的RCU机制[二] - [GP的处理](https://zhuanlan.zhihu.com/p/104581361)

Linux中的RCU机制[三] - [性能与实时性](https://zhuanlan.zhihu.com/p/90223380)

[RCU Concepts — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/RCU/rcu.html)

[more information on RCU](https://docs.google.com/document/d/1GCdQC8SDbb54W1shjEXqGZ0Rq8a6kIeYutdSIajfpLA/edit?usp=sharing)

[RCU Usage In the Linux Kernel: One Decade Later](https://pdos.csail.mit.edu/6.828/2017/readings/rcu-decade-later.pdf)

[爆肝三天整理的RCU同步机制最全介绍](https://zhuanlan.zhihu.com/p/538261127)

### LWN.net

[What is RCU? -- "Read, Copy, Update" — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html)

[What is RCU, Fundamentally?](https://lwn.net/Articles/262464/)

[What is RCU? Part 2: Usage](https://lwn.net/Articles/263130/)

[RCU part 3: the RCU API](https://lwn.net/Articles/264090/)

[The RCU API, 2019 edition](https://lwn.net/Articles/777036/)

[RCU: The Bloatwatch Edition](https://lwn.net/Articles/323929/)

[Simplifying RCU](https://lwn.net/Articles/541037/)

[The design of preemptible read-copy-update](https://lwn.net/Articles/253651/): a preemptible RCU implementation with two waitlist stages per grace period

[Sleepable RCU](https://lwn.net/Articles/202847/)

[Hierarchical RCU](https://lwn.net/Articles/305782/)

[User-space RCU](https://lwn.net/Articles/573424/)

### RCU测试

[Using Promela and Spin to verify parallel algorithms [LWN.net]](https://lwn.net/Articles/243851/)

[Stupid RCU Tricks: A tour through rcutorture](https://paulmck.livejournal.com/61432.html)：rcutorture是一个RCU测试工具。

### 代码

#### 与RCU相关的Linux补丁

[Kernel index [LWN.net]](https://lwn.net/Kernel/Index/#Read-copy-update)

#### Patch for QRCU

[QRCU with lockless fastpath [LWN.net]](https://lwn.net/Articles/223752/)

### URCU

[User-space RCU [LWN.net]](https://lwn.net/Articles/573424/)

[Userspace RCU](https://liburcu.org/)

[GitHub - urcu/userspace-rcu: This repo is a mirror of the official userspace-rcu git found at git://git.lttng.org/userspace-rcu.git. liburcu is a LGPLv2.1 userspace RCU (read-copy-update) library. This data synchronization library provides read-side access which scales linearly with the number of cores.](https://github.com/urcu/userspace-rcu)

[urcu - Rust](https://docs.rs/urcu/latest/urcu/)

[userspace-rcu](https://github.com/urcu/userspace-rcu)

User-Level Implementations of Read-Copy Update

[http://www.rdrop.com/users/paulmck/RCU/urcu-main-accepted.2011.08.30a.pdf](http://www.rdrop.com/users/paulmck/RCU/urcu-main-accepted.2011.08.30a.pdf)

[Userspace RCU原理](https://blog.csdn.net/chenmo187j3x1/article/details/80992945)

URCU的代码分析；

Enabling Fast Per-CPU User-Space Algorithms with Restartable Sequences

[https://blog.linuxplumbersconf.org/2016/ocw/system/presentations/3873/original/presentation-rseq-lpc2016.pdf](https://blog.linuxplumbersconf.org/2016/ocw/system/presentations/3873/original/presentation-rseq-lpc2016.pdf)

#### QSBR

[Quiescent-State and Epoch based reclamation](https://github.com/rmind/libqsbr)

[Practical lock-freedom](https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-579.pdf)

Making Lockless Synchronization Fast: Performance Implications of Memory Reclamation

[http://csng.cs.toronto.edu/publication_files/0000/0165/ipdps06.pdf](http://csng.cs.toronto.edu/publication_files/0000/0165/ipdps06.pdf)

[Making Lockless Synchronization Fast Performance Implications of Memory](https://slidetodoc.com/making-lockless-synchronization-fast-performance-implications-of-memory/)

对应的幻灯片；本地下载：rcu_pengyuan.ppt

### 存储一致性模型

[计算机体系结构——多核处理器——存储一致性模型_fence 多核_KGback的博客-CSDN博客](https://blog.csdn.net/qq_39815222/article/details/107029271)

[内存屏障（Memory Barrier）究竟是个什么鬼？](https://zhuanlan.zhihu.com/p/125737864)

1. 顺序一致性模型SC：对memory的访问时原子化的，串行化的。
2. 完全存储定序模型TSO：
   1. **store操作**在store buffer中顺序执行
   2. **load**同样按顺序执行，但可穿插到多人store执行过程中
3. 部分存储定序模型PSO：
   1. 同一core中地址不相关的store->store指令可以互相穿插执行
   2. **load**按顺序执行，但可穿插到多个store执行过程中
4. 处理器一致性模型PC：
   1. 在某**取数操作load**执行之前，所有在同一处理器中先于该条load指令的取数操作（load）都已完成。
   2. 在某**存数操作store**执行之前，所有在同一处理器中先于该条store的访存操作（load和store）都已完成。
5. 弱一致性模型WO
   1. **同步操作**的执行满足顺序一致性条件
   2. 在任意普通**访存操作**执行之前，所有在同一core中先于该操作的同司步操作都已执行完成
   3. 在任一同步操作执行之前，所有在同一core中先于该操作的普通访存操作都已执行完成
6. 释放一致性模型RC
   1. 同步操作的执行满足顺序一致性条件
   2. 在任意普通访存操作执行之前，所有在同一core中先于该操作的**acquire操作**都已完成
   3. 在任**意release操作**执行之前，所有在同一core中先于该操作的普通访存操作都已完成
7. 松散一致性模型RMO
   1. 只要是地址无关的指令在读写访存时都可以打乱访存顺序；
   2. 允许多核结构中的每个单核改变memory访问指令（不同访问地址）的执行顺序；

RISC-V的fence指令

[RISC-V fence指令](https://zhuanlan.zhihu.com/p/372433134)



# Linux RCU

**原始的RCU思想**

在多线程场景下，经常我们需要并发访问一个数据结构，为了保证线程安全我们会考虑使用互斥设施来进行同步，更进一步我们会根据对这个数据结构的读写比例而选用读写锁进行优化。但是读写锁不是唯一的方式，我们可以借助于COW技术来做到写操作不需要加锁，也就是在读的时候正常读，写的时候，先加锁拷贝一份，然后进行写，写完就原子的更新回去，使用COW实现避免了频繁加读写锁本身的性能开销。

**优缺点**

由于 RCU 旨在最小化读取端开销，因此仅在以更高速率使用同步逻辑进行读取操作时才使用它。如果更新操作超过10%，性能反而会变差，所以应该选择另一种同步方式而不是RCU。

- 好处
  - 几乎没有读取端开销。零等待，零开销
  - 没有死锁问题
  - 没有优先级倒置问题（优先级倒置和优先级继承）
  - 无限制延迟没有问题
  - 无内存泄漏风险问题
- 缺点
  - 使用起来有点复杂
  - 对于写操作，它比其他同步技术稍慢
- 适用场景

![图片](./assert/640.webp)

基于 RCU 的更新程序通常利用这样一个事实，即在现代 CPU 上写入单个对齐指针是原子的，允许在链接结构中原子插入、删除和替换数据，而不会中断读取器。然后，并发 RCU 读取器可以继续访问旧版本，并且可以省去原子读-修改-写指令、内存屏障和高速缓存未命中，这些在现代[SMP](https://en.wikipedia.org/wiki/Symmetric_multiprocessing)计算机系统上非常昂贵，即使在没有锁争用的情况下也是如此



实现方式：QSBR算法

这个算法的核心思想就是识别出线程的不活动(quiescent)状态

<img src="./assert/640-1715609000921-3.webp" alt="图片" style="zoom:50%;" />

- **Removal**：在写端临界区部分，读取（Read()），进行复制（Copy），并执行更改（Update）操作；

- **Grace Period**：这是一个等待期，以确保所有与执行删除的数据相关的reader访问完毕；

- **Reclamation**：回收旧数据；





**静止状态QS(Quiescent State):** CPU发生了上下文切换称为经历一个quiescent state

**宽限期GP(Grace Period):** grace period就是所有CPU都经历一次quiescent state所需要的等待的时间，也即系统中所有的读者完成对共享临界区的访问；

**读侧临界部分RCS(Read-Side Critical Section):** 保护禁止其他CPU修改的代码区域，但允许多个CPU同时读；





**读者reader**：

- 安全访问临界区资源；
- 负责标识进出临界区；

**写者updater**：

- 复制一份数据，然后更新数据；
- 用新数据覆盖旧数据，然后进入grace period；

**回收者reclaimer**：

- 等待在grace period之前的读者退出临界区；
- 在宽限期结束后，负责回收旧资源；