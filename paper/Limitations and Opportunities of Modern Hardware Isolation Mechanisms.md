# Limitations and Opportunities of Modern Hardware Isolation Mechanisms

商用 CPU 中的隔离机制仍然缺乏对支持低开销执行隔离边界、零拷贝数据交换和安全撤销访问权限至关重要的多项设计原则的实现。

## 概述

数十年来，由于缺乏硬件支持，细粒度隔离的成本过高。 例如，高度优化的基于页面的隔离实现需要 771 个周期才能在 Intel CPU 上进行跨子系统调用（ARM 上需要 818 个周期）。对于网络处理框架等现代 I/O 密集型工作负载（每个 I/O 请求花费的周期少于一百个周期）的隔离来说，开销过高 。

现代 Intel CPU 中部署的内存保护键 (MPK) 和扩展页表 (EPT) 与 VM 函数 (VMFUNC)  交换为内存隔离提供支持，其开销逐渐接近函数调用的开销，X86 和 ARM 都在探索对子页面隔离的硬件支持。

| 技术  | 平台 | 功能                                                         |
| ----- | ---- | ------------------------------------------------------------ |
| MTE   | ARM  | 16字节的内存隔离                                             |
| SPP   | x86  | 128字节子页面的内存隔离                                      |
| EPT   | x86  | VMFUNC/扩展页表，需要虚拟化基础                              |
| PAC   | ARM  | 指针身份验证：硬件中指针的加密签名，可用于实现控制流完整性 和子系统隔离 |
| CHERI | ARM  | 一个完整的模型，用于灵活实施各种隔离手段                     |

**新的隔离机制进一步降低了隔离的开销，但这通常需要依赖复杂的编译器和二进制工具来强制进行隔离，并且需要软件或者硬件方案来支持权限撤销**。

对几种方案的总结：

1. 所有隔离方案都受到保存和恢复通用寄存器和扩展寄存器的开销的限制，这显着影响了跨子系统调用的成本
2. Intel MPK 的问题是无法反映零复制内存区域在系统所有核心之间的传递从而导致编程模型受到限制（在一个核心上传递的缓冲区无法从其他核心访问
3. MPK 和 MTE 等基于标签的方案支持数量少得不切实际的独立子系统
4. MTE 还受到重新标记的开销的影响
5.  像 CHERI 这样的功能方案本质上受到缺乏对撤销权限和“移动”语义的支持的限制，即确保调用者失去对传递给被调用者的堆上对象的访问权限
6.  将访问权限保留在寄存器中的硬件架构，例如Intel MPK、CHERI，甚至PAC，面临着另一个固有的限制：不可能跨核心执行权限撤销（active capabilities 可以保留在其他核心的寄存器中）。

基于这些观察和实验，文章提出了一些设计实用的、低开销的隔离机制以及支持高效的零拷贝通信至关重要的原则：

1. **软件透明度**（Software transparency）：架构机制应尽量减少昂贵的编译器工具的使用
2. **跨核心一致的权限同步**(Core-coherent synchronization of rights)：硬件应该支持系统核心之间的访问权限同步，这对于实现通用编程模型至关重要，在该模型中，跨隔离边界交换的内存区域可用于所有线程，对于全局撤销权限甚至更重要（ARM MTE是实现此原则的唯一隔离扩展）
3. **隔离子系统的数量**：硬件隔离原语实际上应该支持大量的隔离子系统
4. **撤销**：支持撤销应该是一个一流的设计原则，面对频繁的通信，撤销对特定内存区域的访问权限的能力与授予访问权限同样重要。



## 背景

 软件故障隔离（SFI）通过在所有内存访问指令之前进行额外的边界检查来强制执行类似段的边界。



## 实现有效隔离的设计准则

硬件隔离机制用于在相互不信任的子系统之间实现类似进程的隔离边界。隔离架构保护各个子系统的状态：堆、栈等等免受意外和恶意访问，同时提供一种方式来控制子系统之间的接口调用。

- 低开销实施

- 隔离边界快速切换

- 数据的零拷贝传递

- **移动**(move)语义和撤销

  > 零拷贝带来两个挑战是：
  >
  > 1. 当在子系统之间传递对象的引用时，需要撤销或者将权限从调用者转换到被调用者
  > 2. 权限的撤销应该在多个核心之间同步，这可能需要昂贵的处理器核间中断



## 评估
