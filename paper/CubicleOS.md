# CubicleOS

 A Library OS with Soﬅware Componentisation for Practical Isolation

## 背景

本篇文章的研究背景是关于库操作系统（Library OS）的隔离性问题。库操作系统是一种将操作系统功能封装为库的设计模式，可以提供更高的灵活性和可扩展性。然而，由于库操作系统中的组件之间共享相同的地址空间，存在数据完整性和隐私泄露的风险。因此，本文旨在探索一种模块化和分隔化的库操作系统设计，以实现组件之间的实际隔离，并保持与现有应用程序的兼容性。研究目标是在不对应用程序或库操作系统进行源代码更改的情况下，通过最小的代码修改来实现空间和时间内存隔离，并提供可接受的性能开销。文章提出了CubicleOS，它通过引入cubicles、windows和跨cubicle调用等核心抽象来实现内存隔离策略，并使用Intel的Memory Protection Keys（MPK）扩展来提供高效的实现。通过实验评估，文章证明了CubicleOS在开发工作量和性能方面的优势。

## 思路

本研究旨在设计一个模块化和分隔的库操作系统，以实现现有第三方组件之间的实际隔离。研究问题是如何在不对应用程序或库操作系统进行侵入性源代码更改的情况下，实现空间和时间内存隔离，并保持与功能丰富的（Linux）应用程序的兼容性。

为了解决这个问题，本研究提出了CubicleOS，一个新的库操作系统，具有以下三个核心贡献：

1. **互相隔离的组件**：CubicleOS通过相互隔离现有的第三方组件来提供数据完整性和隐私保护。

2. **最小代码更改**：CubicleOS通过最小的代码更改来表达组件的隔离策略，使得开发人员能够轻松管理隔离策略。

3. **高效实现**：CubicleOS基于现有硬件进行高效实现，只需要进行微小的修改。它利用Intel的Memory Protection Keys（MPK）扩展来实现内存隔离，并通过动态管理访问权限来确保交叉组件调用的完整性。


CubicleOS通过引入三个核心抽象来实施每个组件定义的内存隔离策略：**cubicles**、**windows**和**cross-cubicle calls**。Cubicles为每个库操作系统和应用程序组件提供透明的<font color=#FF0000>空间内存隔离</font>,Windows提供用户管理的<font color = #FF0000> 时间内存隔离 </font>，使得cubicles可以临时共享数据而无需复制（例如，作为函数参数）。最后，cross-cubicle calls在跨cubicle边界调用函数时提供控制流完整性（CFI），确保只使用公共入口点，并强制执行内存隔离策略。

CubicleOS在现有的Unikraft库操作系统之上进行了原型实现。开发人员只需管理CubicleOS的windows来授予跨cubicle的内存访问权限。CubicleOS的构建系统自动识别每个组件的公共入口点，并为每个入口点生成可信的**cross-cubicle call trampolines**。CubicleOS利用Intel的Memory Protection Keys（MPK）扩展来实现低开销的内存隔离，将每个cubicle映射到单独的MPK标签，并动态管理窗口的访问权限。当跨cubicle调用通过窗口访问由另一个cubicle传递的参数时，CubicleOS重新分配该页面的标签给访问的cubicle。



![image-20231216171959002](C:\Users\godones\AppData\Roaming\Typora\typora-user-images\image-20231216171959002.png)

在不同的组件之间进行调用的形式：

1. 普通函数调用：不存在任何的隔离，性能最高
2. 微内核中使用的消息传递机制：需要额外的内核切换，以及可能的数据打包、拷贝开销，因此性能可能不是很理想，但隔离性很好
3. CubicleOS: 希望既拥有函数调用的性能，又有更好的隔离性。使用MPK技术来保护组件的数据，并在适当的时候打开权限。



## 结果

本研究通过对第三方库操作系统和应用程序开发人员的开发工作量和性能进行评估，证明了CubicleOS的可行性。实验结果表明，CubicleOS在具有隔离文件系统堆栈的SQLite数据库引擎上比最先进的微内核快3倍至5倍，并且开发工作量小，不影响现有第三方库操作系统和应用程序代码的组织或接口。与非隔离版本相比，CubicleOS与SQLite一起运行时速度较慢，速度为1.7倍至8倍；对于I/O密集型的NGINX Web服务器，速度较慢2倍