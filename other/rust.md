# Rust的一点碎碎念

记录Rust的一些值得思考的内容

## &/ref/*区别

对指针来说，解构（destructure）和解引用（dereference）要区分开

- 解引用使用 `*`
- 解构使用 `&`、`ref`、和 `ref mut`

获取一个值的引用有如下的方式:

```rust
let x = &3;
let ref x = 3;
```

引用一般来说就是指向数据的一个指针。`ref`的作用一般体现在模式匹配中，考虑下面这段代码:

```rust
fn main(){
   let str = Some(String::from("Hello"));
   match str {
      Some(s) => println!("{}", s),
      None => println!("None")
   }
   //println!("{:?}", str);
}
```

在注销掉最后一句时，程序可以编译通过，但是如果没有注释掉，则程序发生编译错误，原因在于模式匹配会移动变量的值，在第一个分支s已经获得了str的所有权，因此后续再对str使用将会失效，为了在进行模式匹配后可以继续使用原变量的值，有两者方法:

```rust
fn main(){
   let str = Some(String::from("Hello"));
   match &str {
      Some(s) => println!("{}", s),
      None => println!("None")
   }
   println!("{:?}", str);
}
```

第一种方法是在模式匹配时匹配变量的引用。

```rust
fn main(){
   let str = Some(String::from("Hello"));
   match str {
      Some(ref s) => println!("{}", s),
      None => println!("None")
   }
   println!("{:?}", str);
}
```

第二种方法是在匹配的模式中加入`ref`关键字。

`ref`是Rust的早期产物，为了向后兼容而保留。因此在多数时候我们会使用上面的第一种解决方案。

`*`解引用运算符通过解除引用获取到该引用所指向的原始值，Rust绝大多数时候不会自动地解除引用，但在某些时候会自动解除引用。



## 可变引用与不可变引用

Rust 对可变引用和不可变引用有两条规则：

- **规则 1**：如果你只有不可变的引用，你可以拥有任意数量的引用。
- **规则 2**：如果你有一个可变引用，你只能有一个。此外，您不能同时拥有不可变引用和可变引用

在实践中，可能会遇到同时存在可变引用和不可变引用也能通过编译的情况，比如说如下的代码:

```rust
 let mut number = 10;
 let number_ref = &number;
 let number_change = &mut number;
 *number_change += 10;
// println!("{}", number_ref);
```

在注释掉最后一句的情况下，可以正常编译，如果未注释，则会违反上述规则2从而编译错误。在早期的Rust中不管注释与否都会错误，但现在的编译器足够智能，在注释掉最后一句的情况下，可以推断出