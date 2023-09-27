---
title: Closure
date: 2023-09-27 11:35:01
tags: rust
---

有些语言中没有 closure 和普通函数的区分，但 Rust 有。对 Rust 来说普通函数就是一段代码。而 closure 和 C++ 类似：每个 closure 会创建一个匿名的 struct，编译器会在当前上下文捕获 closure 代码中的外部变量然后塞进这个**结构体**里面。

为什么要有闭包？
在 rust 中函数是不能捕获**动态环境变量**，这是因为在 Rust 中，函数是静态的，而不是动态的。函数定义时，所有的变量和参数都必须是已知的、固定的值。这意味着，函数不能从其所在的环境中捕获任何动态变量。而闭包可以。下面展示使用函数调用作用域变量的错误：

```rust
fn main() {
    let x = 4;

    fn equal_to_x(z: i32) -> bool { z == x }

    let y = 4;

    assert!(equal_to_x(y));
}

// 无法通过编译

error[E0434]: can't capture dynamic environment in a fn item; use the || { ...
} closure form instead
 --> src/main.rs
  |
4 |     fn equal_to_x(z: i32) -> bool { z == x }
  |
```

闭包是可以捕获环境变量的，捕获的方式有 3 种**Trait**：FnOnce, FnMut, Fn

FnOnce 表示：捕获到的环境变量**所有值转移**
FnMut 表示：捕获到的环境变量**可变借用**
Fn 表示：捕获到的环境变量**不可变借用**

上面所说的并不是绝对的，注意：**一个闭包实现了哪种 Fn 特征取决于该闭包如何使用被捕获的变量，而不是取决于闭包如何捕获它们**，入下面的代码：

```rust
fn main() {
    let s = String::new();

    let update_string =  move || println!("{}",s);

    exec(update_string);
}

fn exec<F: FnOnce()>(f: F)  {
    f()
}
```

虽然声明了特征值：FnOnce，但是内部使用的是不可变借用，所以该闭包也实现了 Fn 特征。所以上面的代码: FnOnce 改为 Fn 也是可以的：

```rust
fn main() {
    let s = String::new();

    let update_string =  move || println!("{}",s);

    exec(update_string);
}

fn exec<F: Fn()>(f: F)  {
    f()
}
```
