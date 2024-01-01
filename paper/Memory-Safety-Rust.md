# Memory-Safety Challenging In Rust

本研究通过对所有Rust CVEs的深入研究，对内存安全问题进行了全面的分析。研究发现，Rust在内存安全方面取得了显著的成果，但仍存在一些问题。主要的内存安全问题包括缓冲区溢出/读取、使用已释放的内存、重复释放内存和未初始化内存访问。研究还对这些问题的成因进行了分类，包括自动内存回收、不安全的函数、不安全的泛型或特性以及其他错误。这些研究结果对于进一步提高Rust的内存安全性具有重要意义。



结论1：除了编译器错误之外，Rust 的所有内存安全错误都需要不安全的代码。主要的魔力在于 Rust 对 Rust 标准库或第三方库提供的 API 强制执行健全性要求。因此，我们数据集中的许多错误都是轻微的健全性问题，并且这些错误并未引发严重的内存安全问题。此外，编译器错误的趋势也非常稳定。因此，我们可以得出结论，Rust 在防止内存安全错误方面是有效的。

Do all memory-safety bugs require unsafe code?

根据Rust的内存安全承诺，如果不使用不安全的Rust，开发人员不应该能够编写包含内存安全bug的Rust代码。**我们的结果证实，在我们的数据集中，除了编译器错误之外，所有内存安全错误都需要不安全的代码**

How severe are these memory-safety bugs？

现有的cve都是库错误。这意味着这些错误不会直接导致不好的结果，除非有bug的api被其他可执行程序使用。相比之下，许多现有的C/ c++ cve都是可执行的bug。**这些错误的一个重要现象是，其中许多错误内部并不包含错误，而只是引入了不健全，违反了Rust的内存安全设计**。因此，它们为用户留下了在没有任何不安全标记的情况下编写内存安全漏洞的可能性。

简而言之，我们的分析表明，我们的数据集中的许多内存安全错误都是轻微的，它们只会在没有不安全代码的情况下引入内存安全问题的可能性。然而，由于 Rust 提倡内存安全，社区对此类风险更加严格。

How robust is Rust compiler？

我们可以观察到，新问题出现的趋势虽然没有下降，但还是稳定了不少。为什么不健全问题难以避免？随着Rust的功能不断丰富，引入新的不健全问题是不可避免的。此外，一些与语言设计相关的不健全问题也很难修复



结论2： Rust 典型的三大类内存安全错误

1. 自动内存回收：揭示了 Rust OBRM 的副作用
2. 不健全的函数：揭示了 Rust 开发人员避免代码不健全所面临的基本挑战
3. 不健全的泛型或特征：泛型或特征为 Rust 开发人员带来了另一个高级挑战，即在使用多态性和继承等复杂的语言功能时提供健全的 API

自动内存回收：

为了实现OBRM并避免内存泄漏，Rust采用了RAII，即在初始化时自动获取内存，如果不再使用则释放内存[31]。 Rust 通过在编译期间在中级中间表示 (MIR) 代码中添加对特定变量的 drop() 函数调用来实现自动内存回收。然而，由于 Rust 编译器对于不安全的代码来说是不健全的，这种自动内存回收方案使得 Rust 程序容易出现悬空指针。

Bad Drop at Normal Execution Block：一般来说，自动内存回收只有在存在不安全代码时才会触发内存安全问题，这会导致与 Rust OBRM 不一致并破坏 Rust 编译器的健全性。

```rust
fn genvec() -> Vec<u8>{
	let mut s= String::from("a	tmp	string");
	/*fix2: let mut s = ManuallyDrop::new(String::from("a tmp string"));*/
	let ptr = s.as_mut_ptr();
	unsafe {
		let v= Vec::from_raw_parts(ptr, s.len(), s.len());
		/*fix1: mem::forget(s);*/
		return v;
		/*s is freed when the function returns*/
	}
}
fn main() {
	let v = genvec();
	assert_eq!('a' as u8,v[0]); /*use-after-free*/
	/*double free: v is released when the function returns*/
}
```

Bad Drop at Cleanup Block: Rust 编译器有两种运行时错误处理模式：panic 或 abort。 Panic 是默认模式，自动执行堆栈展开并回收分配的资源； abort 只是直接结束进程。此类错误仅存在于恐慌模式下。



不健全的函数: 不健全的函数可能会受到用户的攻击，并在没有不安全代码的情况下导致未定义的行为

bad function signature

- Function safety: 如果开发人员错误地将不安全函数声明为安全函数，则该函数的用户可能会在不使用不安全代码的情况下触发未定义的行为
- Parameter mutability: 如果开发人员错误地定义了不可变参数，但在内部对其进行了更改（例如，通过调用 FFI），则该函数的用户可能会错误地认为参数值未更改并遭受未定义的行为
- Lifetime bound: 函数的参数应该与生命周期正确绑定。使用这些函数时，生命周期界限缺失或不足可能会导致悬空指针
- Function visibility: 在极端情况下，整个函数如果是不健全的（例如，已弃用或私有），不应暴露给用户

FFI:  FFI 只允许在不安全的 Rust 中使用，因为它们可能会使 Rust 编译器变得不健全。为了满足 Rust 的安全要求，开发人员应该确保 FFI 的所有行为都得到正确处理或声明其调用函数不安全。否则，FFI 的不健全性将传播到安全的 Rust 代码并引发未定义的行为

**在std-lib中，有许多问题都是FFI导致的。**



不健全的泛型或特征

Insufficient Bound of Generic： 在 Rust 中，函数或结构体可以接受泛型类型参数作为输入.然而，实际的函数可能无法支持具有任何生命周期的所有类型的变量，因此需要边界来进行此类限制。如果指定的边界不足，则可能会因传递不支持类型的参数而引发内存安全问题

Generic Vulnerable to Specific Type: 数据集中的大多数此类错误都是处理特殊类型时的 std-lib 错误，例如 void或零大小类型

Unsound Trait: 派生安全特征或重新实现该特征并导致未定义的行为，而不使用不安全代码



结论3：分析表明，Rust 与其他编程语言之间的主要区别在于 API 的健全性承诺。为此，Rust 编译器已经实现了安全 Rust 的健全性分析，但主要挑战在于不安全代码。



Best Practice for Code Suggestion：

- Generic Bound Declaration.  数据集中最常见的错误是在为自定义类型或结构实现发送或同步特征时缺少与泛型绑定的发送或同步。建议开发人员在实现发送或同步特征时考虑强制执行此类限制。我们可以开发一个规则来自动检测此类问题，即，如果为具有通用参数的类型实现了 Send 或 Sync 特征，则所有通用参数都应相应地绑定到 Send 或 Sync
- Avoiding Bad Drop at Cleanup Block: 避免使用一些容易出现内存安全问题的易受攻击的 API（例如，forget() 和 uninitialized()）。当使用不安全代码缩小或扩展缓冲区时，应将生成的缓冲区的长度设置在适当的程序点

- Function Safety Declaration： 如果一个函数无条件地使用不安全代码并且它不是成员函数，那么该函数很可能被声明为不安全。
  - 不安全函数意味着使用不安全代码的函数被声明为不安全。如果函数包含unsafe，则函数安全性将取决于危险输入在到达不安全代码之前是否已被naturalized(是否被检查过)

静态分析

- 针对不安全构造函数相关的指针使用问题
- 检测那些不健全的API