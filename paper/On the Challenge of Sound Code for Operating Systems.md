# On the Challenge of Sound Code for Operating Systems

内存安全系统编程语言 Rust 在操作系统开发社区中越来越受到关注，因为它在不牺牲性能或控制的情况下提供内存安全。 然而，这些安全保证仅适用于 Rust 的安全子集，而裸机编程则要求程序的某些部分用不安全的 Rust 编写。 为软件的这些部分编写健全的抽象（意味着它们保证不存在未定义的行为，从而维护安全 Rust 的不变量）可能具有挑战性。



## 概述

事实上，通常情况下，在用这些非内存安全语言编写的大型软件项目中，65% 以上的安全漏洞都是由内存不安全引起的 [1-4]。Rust是一种新的内存安全系统编程语言，并在快速发展中。**其成功的最关键因素是通过复杂的类型系统和借用检查器进行内存管理的新颖方法，从而在不牺牲对内存的控制的情况下实现内存安全**。这两个方法将对内存管理的工作转移到编译时而不是类似Go/Jave在运行时利用收集来解决这个问题。

在Rust中编写裸机应用程序(如操作系统)需要使用不安全的Rust，它允许在安全Rust中不可能实现的某些必要的低级操作，例如通过专门的汇编指令或内存映射寄存器访问配置硬件时。在这些情况下，Rust的安全保证必须手工维护，编译器不能保证没有未定义的行为。**在Rust代码中保证不存在未定义行为被称为健全性。这是操作系统的基本特性，因为必须不惜一切代价防止崩溃和漏洞**。



## 背景

编译器在假设程序不存在未定义行为的情况下优化程序。 这意味着确实包含未定义行为的程序在优化后可能包含损坏的不变量。对于 Rust 来说，没有未定义的行为是至关重要的，因为该语言从明确定义和维护的安全不变量中获得了内存安全属性。

**Safe Rust 保证不存在任何类型的未定义行为（在 [25] 中对语言的大部分部分进行了正式证明）**。在编写不安全的 Rust 代码时，必须维护 Rust 的健全性属性：“无论如何，安全的 Rust 都不会导致未定义的行为。” [26]为了能够对此进行推理，必须为不安全代码创建可靠的抽象，这些抽象不能用于导致未定义的行为，或者，如果不可能，则将抽象标记为不安全，并记录该代码所依赖的所有不变量

值得一提的是，程序看起来没有行为不当这一事实并不意味着它是健全的或没有未定义的行为。 对不安全抽象的使用方式的任何更改或编译器的升级都可能会带来此类问题。 因此，努力提高可靠性对于利用 Rust 的核心优势至关重要。

## 典型的健全性问题

1. 不同步的全局状态：通过使用锁来同步
2. C风格的抽象
3. 可变引用别名：对原始指针取引用会出现多个可变引用
4. 重新实现内存访问：对

**健全性有一个根问题，如何保证提供健全性部分的健全性？**

## 参考文献

[1] The Chromium projects authors. [n. d.] Memory safety. Publisher: The Chromium Projects. Retrieved Feb. 19, 2023 from https://www.c hromium.org/Home/chromium-security/memory-safety/.

2] Paul Kehrer. 2019. Memory unsafety in apple’s operating systems. Publisher: langui.sh. (July 23, 2019). Retrieved Feb. 19, 2023 from https://langui.sh/2019/07/23/apple-memory-safety/.

[3] Evgenii Stepanov. 2020. Detecting memory corruption bugs with HWASan. Publisher: Android Developers Blog. (Feb. 13, 2020). Retrieved Feb. 19, 2023 from https://android-developers.googleblog.co m/2020/02/detecting-memory-corruption-bugs-with-hwasan.htm l.

[4] Gavin Thomas. 2019. A proactive approach to more secure code. Publisher: Microsoft Security Response Center Blog. (July 16, 2019). Retrieved Feb. 19, 2023 from https://msrc.microsoft.com/blog/2019 /07/a-proactive-approach-to-more-secure-code/.

[25] Jung R, Jourdan J H, Krebbers R, et al. RustBelt: Securing the foundations of the Rust programming language[J]. Proceedings of the ACM on Programming Languages, 2017, 2(POPL): 1-34.

[26] [n. d.] The rustonomicon. Retrieved Aug. 1, 2023 from https://web.arc hive.org/web/20230619130041/https://doc.rust-lang.org/nomicon/.