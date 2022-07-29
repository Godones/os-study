# 各章节记录

## 第三章

todo!

## 第四章

todo!

## 第五章
问题1： 为什么普通的`map`循环查找最小的pass的进程会造成错误，而切换到min_by_key算法则不会？

```rust
 if self.ready_queue.is_empty() {
            return None
        }
let mut min_pass = usize::MAX;
let mut index = 0;
let _ = self.ready_queue.iter().enumerate().map(|(i,task)| {
    let inner = task.inner_exclusive_access();
    if inner.pass < min_pass {
    	min_pass = inner.pass;
    	index = i;
    }
 });
Some(self.ready_queue.remove(index))
```

解决方法：由于`map`是惰性求值,会导致上述的循环不起作用，因此要更换为普通的`for`循环

