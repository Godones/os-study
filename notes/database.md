# 数据库 AS 文件系统

BonsaiDb performance update: [A deep-dive on file synchronization](https://bonsaidb.io/blog/durable-writes/) - 本文深入讲解了 file synchronization 机制以及他如何影响性能.

[BonsaiDb](https://github.com/thedodd/trunk)是一个新的数据库，旨在成为最适合开发人员的 Rust 数据库.



## 资料阅读

| 嵌入式数据库                 | 数据库服务器                               |
| ---------------------------- | ------------------------------------------ |
| SQLite、Berkeley DB          | Oracle、Sybase、MySQL                      |
| 位于同一个应用程序地址空间中 | C/S架构，客户端与服务端位于不能进程中      |
| 体积小，可自由定制           | 需要去安装服务器并部署，应用于文件系统困难 |

### SQLite

1. 支持事件，不需要繁杂的配置(单个文件-->单个磁盘)
2. 最大支持数据库到2T，字符和BLOB的支持仅限制于可用内存
3. 整个系统少于3万行代码，少于250KB的内存占用(gcc)，大部分应用比目前常见的客户端/服务端的数据库快，没有其它依赖
4. 功能完善：支持ACID(Atomicity 、Consistency、Isolation、Durability）事务， Atomicity（原子性）、Consistency（一致性）、Isolation（隔离性）和Durability（持久性）,SQLite支持大多数的SQL92，即支持触发器、多表和索引、事务、视图，还支持嵌套的SQL。

* [SQLite数据库 简介、特点、优势、局限性及使用](https://www.cnblogs.com/l199616j/p/10694036.html)
* [SQLite教程](https://www.runoob.com/sqlite/sqlite-tutorial.html)
* [SQLite Download Page](https://www.sqlite.org/download.html)



### Berkeley DB

1. key-value数据库，一个键对应多个数据(支持最大256TB)
2. 更高的并发度: 在页面级上的加锁 / SqlLite是在整个数据库加锁
3. 提供高效的接口







### FlashDB：超轻量级嵌入式数据库

1. 针对flash设备做了优化，具有较强的性能及可靠性
2. 提供两种数据库模式：键值/时序
3. FlashDB 底层的 Flash 管理及操作依赖于 RT-Thread 的 FAL (Flash Abstraction Layer) Flash 抽象层开源软件包 ，该开源库也支持运行在==裸机平台==所以只需要将所用到的 Flash 对接到 FAL ，即可完成整个移植工作
4. 将这个转为rust实现可能需要如下的步骤
   1. 底层设备的读写接口差异不大
   2. FAL可能没有rust的实现，需要定义这个抽象层
   3. 将flashDB重写

### SQLRite

1. rust实现的Sqlite，但没有完全实现（个人练习项目
2. 提供的CLI(REPL)实现可以借鉴

### KuiBaDB

1. OLAP数据库

一个用Rust写的异步数据库

[可能是最快的基于 io-uring 的异步 IO 框架](https://blog.hidva.com/2021/09/14/kbio/)：[仓库](https://github.com/KuiBaDB/kbio)

加上用户态中断，这个应该会更快。





## 参考链接

1. [嵌入式数据库（Embedded Database）](https://zhuanlan.zhihu.com/p/109227826)：比较了几种嵌入式数据库系统的特点。嵌入式数据库应该可以用于支持一个文件系统。SQLite的代码量还太大，运行开销相对较小，可以放在内核中。

2. [FlashDB](https://gitee.com/Armink/FlashDB)：一款支持 KV 数据和时序数据的超轻量级数据库。它的[文档](http://armink.gitee.io/flashdb/#/)。

3. [SQLRite](https://github.com/joaoh82/rust_sqlite) - Simple embedded database modeled off SQLite in Rust

4. [Let's Build a Simple Database](https://cstack.github.io/db_tutorial/)

5. [Database implementations](https://lib.rs/database-implementations): Database management systems implemented in Rust. Store and query large amounts of data in an efficient manner

6. [sled - Rust (docs.rs)](https://docs.rs/sled/latest/sled/)

7. [WinFS - Wikipedia](https://en.wikipedia.org/wiki/WinFS) 

8. [samzyy/DB-based-replicated-filesystem (github.com)](https://github.com/samzyy/DB-based-replicated-filesystem)  mysqlfs

9. Extending SQLite with Rust

   出处：[Extending SQLite with Rust](https://ricardoanderegg.com/posts/extending-sqlite-with-rust/)

10. 

    