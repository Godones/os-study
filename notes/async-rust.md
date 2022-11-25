# Rust异步模型

## Async 和 多线程

两种方法都可以实现并发编程，但两者的使用和实现差距巨大。

**多线程**:

- 适合较少的并发数量
- 不破坏代码逻辑和编程模型
- 在可修改优先级的系统中对某些延迟敏感任务有很大的帮助
- 适用于CPU密集型任务
  - 线程切换是不必要的，线程数量==核心数量

**Async**:

- 高并发更适用于IO密集型任务
  - 使用多线程将会造成大量线程处于无事可做的状态
- 线程切换 ==> 任务切换
  - 保存的信息相对于线程切换更少，速度更快
  - 单个线程上可以存在大量的任务
- 每一个Async函数是一个状态机
  - 需要运行时帮助，代码体积增大



## 使用Async的一些必要条件

- 所必须的特征(例如 `Future` )、类型和函数，由标准库提供实现
- 关键字 `async/await` 由 Rust 语言提供，并进行了编译器层面的支持
- 众多实用的类型、宏和函数由官方开发的 [`futures`](https://github.com/rust-lang/futures-rs) 包提供(不是标准库)，它们可以用于任何 `async` 应用中。
- `async` 代码的执行、`IO` 操作、任务创建和调度等等复杂功能由社区的 `async` 运行时提供，例如 [`tokio`](https://github.com/tokio-rs/tokio) 和 [`async-std`](https://github.com/async-rs/async-std)
  - 裸机操作系统需要自定义运行时？



## Async内部机制

### Future特征

```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

每一个异步任务都会返回一个`Future`， 而这个`Future`是惰性的，只有被执行器`poll` 后才会真正去执行。若在当前 `poll` 中， `Future` 可以被完成，则会返回 `Poll::Ready(result)` ，反之则返回 `Poll::Pending`， 并且安排一个 `wake` 函数：当未来这个 `Future` 准备好继续执行时， 该函数会被调用，然后管理该 `Future` 的执行器会再次调用 `poll` 方法，此时 `Future` 就可以继续执行了。

==疑问==：怎么安排`wake`函数，在编写普通的异步函数的时候我们并没有去关注这个`wake`函数。 

#### 状态机转换

一个简单的异步函数如下所示:

```rust
async fn example(min_len: usize) -> String {
    let content = async_read_file("foo.txt").await;
    if content.len() < min_len {
        content + &async_read_file("bar.txt").await
    } else {
        content
    }
}
```

此异步函数被编译器重写为一个状态机：

![pic1](assert/pic1.svg)

在函数开始运行后，通过每次在每个状态上调用`poll`函数并根据返回值决定下一次该怎么进行状态转换。

![pic2](assert/pic2.svg)

`.await` 会触发调用`poll`进行询问，并阻塞直到其完成。

为了能够从上一个等待状态继续，状态机必须在内部跟踪当前状态。 此外，它必须保存在下一个`poll`调用中继续执行所需的所有变量。 这是编译器真正发挥作用的地方：它知道何时使用哪些变量，因此它可以自动生成具有所需变量的结构。

对于上面的这个函数，被编译器生成的状态机代码的简化形式如下:

```rust
struct StartState {
    min_len: usize,
}

struct WaitingOnFooTxtState {
    min_len: usize,
    foo_txt_future: impl Future<Output = String>,
}

struct WaitingOnBarTxtState {
    content: String,
    bar_txt_future: impl Future<Output = String>,
}

struct EndState {}
```

一般来说，开始状态的结构体中只包含了参数，结束状态则不包含内容。

```rust
enum ExampleStateMachine {
    Start(StartState),
    WaitingOnFooTxt(WaitingOnFooTxtState),
    WaitingOnBarTxt(WaitingOnBarTxtState),
    End(EndState),
}
impl Future for ExampleStateMachine {
    type Output = String; // return type of `example`

    fn poll(self: Pin<&mut Self>, cx: &mut Context) -> Poll<Self::Output> {
        loop {
            match self { 
                ExampleStateMachine::Start(state) => {…}
                ExampleStateMachine::WaitingOnFooTxt(state) => {…}
                ExampleStateMachine::WaitingOnBarTxt(state) => {…}
                ExampleStateMachine::End(state) => {…}
            }
        }
    }
}
```

将所有状态放到一个`enum`中，并为其实现`poll`，这里对真正的实现进行了简化，在此函数被调用后，状态机就处于初始状态，位于初始状态执行的代码可能如下所示:

```rust
let foo_txt_future = async_read_file("foo.txt");
// `.await` operation
let state = WaitingOnFooTxtState {
    min_len: state.min_len,
    foo_txt_future,
};
*self = ExampleStateMachine::WaitingOnFooTxt(state);
```

编译器会将遇见的第一个`.await`之前的代码执行掉，并将当前状态更新到下一个状态返回。下一个状态可能执行的代码如下：

```rust
ExampleStateMachine::WaitingOnFooTxt(state) => {
    match state.foo_txt_future.poll(cx) {
        Poll::Pending => return Poll::Pending,
        Poll::Ready(content) => {
            // from body of `example`
            if content.len() < state.min_len {
                let bar_txt_future = async_read_file("bar.txt");
                // `.await` operation
                let state = WaitingOnBarTxtState {
                    content,
                    bar_txt_future,
                };
                *self = ExampleStateMachine::WaitingOnBarTxt(state);
            } else {
                *self = ExampleStateMachine::End(EndState));
                return Poll::Ready(content);
            }
        }
    }
}
```

代码会首先检查`future`是否完成，如果没有完成，则其返回`Pending`，否则就会继续执行函数体的逻辑并在下一个`.await`生成新状态。

通过上述的状态机生成和转换，编译器就可以把一个异步函数转换成一个正确的普通形式的代码。

### Executor

虽然在一个异步函数内部的异步函数可以使用`.await`调用，但最外层的异步函数却不能这样使用，需要有一个管理员负责去调用这些没有父级函数的异步函数，这种管理员就是`Executor`，比如常见的`tokio/async_std`。这个执行器需要查看所有的异步任务是否完成，最简单的方式是直接轮询所有的任务，但这样就会白白浪费CPU，因为如果大多数任务没有完成，我们就没有必要时刻关注。另一种方式就是在任务完成时告知执行器，执行器再对其进行`poll`。



### Waker

`Waker` 是为了提高执行器执行任务的效率而产生，通过将一个`Waker`绑定到一个异步任务上，在这个异步任务完成后，其就可以唤醒调度器对它进行询问。



#### RawWaker

该[`RawWaker`](https://doc.rust-lang.org/stable/core/task/struct.RawWaker.html)类型要求程序员明确定义一个[*虚函数表*](https://en.wikipedia.org/wiki/Virtual_method_table)（*vtable*），该*表*指定了在`RawWaker`克隆，唤醒或删除它们时应调用的函数。此vtable的布局由[`RawWakerVTable`](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://doc.rust-lang.org/stable/core/task/struct.RawWakerVTable.html&usg=ALkJrhhw23YeIwD9TejZL0gPQapSx4FlKw)类型定义。每个函数接收一个`*const ()`参数，该参数基本上是指向某个结构的*类型被擦除掉的* `&self`指针，例如，堆上的内存分配。使用`*const ()`指针而不是适当的引用的原因是该`RawWaker`类型应该是非泛型的，但仍支持任意类型。传递给函数的指针值是[`RawWaker::new`](https://doc.rust-lang.org/stable/core/task/struct.RawWaker.html#method.new)中传入的`data`指针值。

通常，`RawWaker`为包裹在[`Box`](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://doc.rust-lang.org/stable/alloc/boxed/struct.Box.html&usg=ALkJrhh5TXJ5H6x82XpGXpGzQuQC88TCeg)或[`Arc`](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://doc.rust-lang.org/stable/alloc/sync/struct.Arc.html&usg=ALkJrhhytzcnWX7C38Bdbd_rI5EPjRlGig)类型中的一些堆分配的结构创建。对于此类类型，可以使用诸如[`Box::into_raw`](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://doc.rust-lang.org/stable/alloc/boxed/struct.Box.html&usg=ALkJrhh5TXJ5H6x82XpGXpGzQuQC88TCeg#method.into_raw)之类的方法将`Box<T>`转换为`*const T`指针。然后，可以将该指针转换为匿名`*const ()`指针并传递给`RawWaker::new`。由于每个vtable函数接收同样的的`*const ()`参数，因此这些函数可以安全地将指针强制转换回`Box<T>`或 `&T`进行操作。可以想象，此过程非常危险，很容易导致错误的不确定行为。因此，除非必要，否则不建议手动创建`RawWaker`。

### 





## 参考资料

[Async in depth | Tokio - An asynchronous Rust runtime](https://tokio.rs/tokio/tutorial/async)

200行解释Future

[Introduction - Futures Explained in 200 Lines of Rust (cfsamson.github.io)](https://cfsamson.github.io/books-futures-explained/introduction.html)

一个异步输入的示例同时包含其它讲解

[writing-an-os-in-rust/12-async-await.md at master · rustcc/writing-an-os-in-rust (github.com)](https://github.com/rustcc/writing-an-os-in-rust/blob/master/12-async-await.md)

https://github.com/YdrMaster/notebook/blob/main/translation/20221019-futures-in-200-lines/introduction.md

树莓派操作系统的讲解

[Demystifying Async/Await in Rust - Demystifying Async/Await in Rust (ruspiro.github.io)](https://ruspiro.github.io/ruspiro-async-book/cover.html)



https://github.com/YdrMaster/fast-trap
