---
title: 'rust: 闭包 Closure（三）当闭包作为函数参数或函数返回值时正确标注函数签名'
date: 2024-03-26 23:20:39
categories:
tags:
---

## 闭包 Closure（三）当闭包作为函数参数或函数返回值时正确标注函数签名

> 本章内容较长，且在本章内容尾部更新了对闭包的认识，读者应读完全章，不要取其中部分。

闭包是一种匿名函数，它可以赋值给变量也可以作为参数传递给其它函数，不同于函数的是，它允许捕获调用者作用域中的值。

Rust 闭包在形式上借鉴了 Smalltalk 和 Ruby 语言，与函数最大的不同就是它的参数是通过 |parm1| 的形式进行声明，如果是多个参数就 |param1, param2,...|，闭包的形式定义：

```rust
|param1, param2,...| {
    语句1;
    语句2;
    返回表达式
}
```

### 闭包作为函数返回值

在实现 `Cacher` 中我们将闭包作为参数传递给函数（方法），现在考虑如何将闭包应用在函数的返回值上，因为只有这样，才能将内部值传递出去。

闭包在 Rust 中有一个独特的特性：每个闭包都有其自己独特的匿名类型（这个类型就是类似 `i32 String` 的一种数据格式类型），这是因为**闭包类型不仅仅是由其参数和返回类型定义**的，还包括它捕获的环境。每个闭包根据其捕获的环境（变量、生命周期等）具有不同的类型。
即使两个闭包有相同的签名，它们也被认为是不同的类型。这意味着直接返回闭包类型（`Fn(i32) -> i32`）是不可能的，因为闭包的具体类型是未知的，且无法直接命名。

#### 正确标注闭包作为函数返回值

使用 impl Trait 语法允许我们返回一个实现了指定 trait 的类型，而不需要指定具体的类型。（impl Trait 形式来说明一个函数返回了一个类型，该类型实现了某个特征，外部使用时只能使用该特征已声明的属性）(函数返回中的 impl trait)[https://course.rs/basic/trait/trait.html#%E5%87%BD%E6%95%B0%E8%BF%94%E5%9B%9E%E4%B8%AD%E7%9A%84-impl-trait]
使用 `impl trait` 形式将闭包标识为实现了某个特征的类型后就可以返回。在闭包的场景中，Fn、FnMut、FnOnce 它们分别对应不同的闭包类型。通过返回 impl Fn(参数类型) -> 返回值类型，这将告诉 Rust 编译器，返回一个实现了 Fn trait 的类型，但不指定具体是哪个类型。
简而言之，impl 关键词的使用是因为闭包的类型是匿名且不可直接命名的，而 impl Trait 语法允许我们以一种抽象的方式返回实现了特定 trait 的闭包，而无需关心闭包的具体类型。这大大增加了代码的灵活性和可重用性。

为什么闭包作为函数的参数时不需要显式的指定 impl ？
当闭包作为参数传递给函数时，**不需要**使用 impl 关键字，是因为在这种情况下可以直接指定闭包参数遵循的特定 trait（如 Fn、FnMut 或 FnOnce）。
这是通过使用**trait 界定（trait bounds）**来实现的(简单理解为自动推断和实现)，它允许函数接受任何实现了指定 trait 的类型。这种方式提供了足够的灵活性，同时避免了 impl Trait 在参数位置的使用。

```rust
fn factory() -> impl Fn(i32) -> i32 {
    let num = 5;
    move |x| x + num
}

let f = factory();
let answer = f(1);
assert_eq!(6, answer);
```

用 `impl trait` 形式实现闭包作为返回值返回，最大的问题是 `impl trait` 要求返回只能有一个具体的类型。而闭包即使签名一致也可能是不同的类型。

```rust
编译错误 error
fn factory(x: i32) -> impl Fn(i32) -> i32 {
    let num = 5;
    if x > 1 {
        |x| x + num
     } else {
         |x| x - num
     }
}
```

与 impl trait 相对应的，动态特征对象不限制某一个具体的类型，因此可改为使用动态特征对象解决这个问题。

```rust
fn factory(x:i32) -> Box<dyn Fn(i32) -> i32> {
    let num = 5;
    if x > 1{
        Box::new(move |x| x + num) // 仅实现了Fn特征，需要将所有权强制移入
    } else {
        Box::new(move |x| x - num)
    }
}
```

当闭包作为函数参数或函数返回值时如何正确的标注函数签名？

首先，Rust 要求函数的参数和返回值的内存大小是固定的，因此闭包作为 Trait，它的内存大小不固定，如 `Fn(i32) -> i32` 是不能够直接作为参数和返回值的。
但有特殊情况，把闭包当作参数时，可以直接声明为闭包类型如 `Fn(i32) -> i32`，这是因为函数体内的**参数闭包**有且只有一种闭包类型，Rust 通过使用**trait 界定（trait bounds）**允许函数接受任何实现了指定 trait 的类型。
可以理解成 Rust 自动推断为 `impl trait` 形式。这种参数闭包直接声明为闭包类型的方式提供了足够的灵活性，同时避免了 impl Trait 直接在参数位置的使用。

其次，闭包作为返回值时，为了遵守 “Rust 要求函数的参数和返回值的内存大小是固定的” 的原则，不能直接将闭包类型作为函数的返回值声明，而应该使用 `impl trait` 或特征对象。
如果只有一种闭包，那么可以直接使用 `impl trait` 形式，如 `impl Fn(i32) -> i32`，表明返回一个实现了指定 trait 的类型，impl Trait 形式来说明一个函数返回了一个类型，该类型实现了某个特征，外部使用时只能使用该特征已声明的属性，因此是固定大小的。(函数返回中的 impl trait)[https://course.rs/basic/trait/trait.html#%E5%87%BD%E6%95%B0%E8%BF%94%E5%9B%9E%E4%B8%AD%E7%9A%84-impl-trait]
如果存在多个闭包类型，那么可以使用特征对象 `Box<dyn Fn(i32) -> i32>` 来作为函数的返回值。

> 多个闭包类型即多个位置声明定义的闭包，每个闭包都有其自己独特的匿名类型（这个类型就是类似 `i32 String` 的一种数据格式类型），这是因为**闭包类型不仅仅是由其参数和返回类型定义**的，还包括它捕获的环境。每个闭包根据其捕获的环境（变量、生命周期等）具有不同的类型。

同时，当函数返回闭包时，需要注意返回的闭包是否具有变量的所有权，因为只有闭包获取了变量的所有权，在当前函数执行结束返回后，变量才不会被释放，才不会出现引用问题。当闭包作为函数的返回值返回时，常见的闭包都会有 `move` 关键字强制获取变量的所有权。

### & mut &mut 的各种位置认识

阅读：https://github.com/sunface/rust-course/discussions/619#discussioncomment-2623736

- fn do1(c: String) {}：表示实参会将所有权传递给 c
- fn do2(c: &String) {}：表示实参的不可变引用（指针）传递给 c，实参需带 & 声明
- fn do3(c: &mut String) {}：表示实参可变引用（指针）传递给 c，实参需带 let mut 声明，且传入需带 &mut
- fn do4(mut c: String) {}：表示实参会将所有权传递给 c，且在函数体内 c 是可读可写的，实参无需 mut 声明
- fn do5(mut c: &mut String) {}：表示实参可变引用指向的值传递给 c，且 c 在函数体内部是可读可写的，实参需带 let mut 声明，且传入需带 &mut

一句话总结：在函数参数中，冒号左边的部分，如：mut c，这个 mut 是对 🪄 函数体内部有效 🪄；冒号右边的部分，如：&mut String，这个 &mut 是针对 🪄 外部实参传入时的形式（声明）说明 🪄。

### 阅读

- [泛型、特征、特征对象](https://course.rs/basic/trait/intro.html)
- [闭包的使用案例](https://www.cnblogs.com/lilpig/p/17022845.html#%E5%A4%8D%E6%9D%82%E4%B8%80%E7%82%B9%E6%8A%8A%E9%97%AD%E5%8C%85%E7%94%A8%E5%9C%A8%E5%87%BD%E6%95%B0%E4%B8%8A)

### Code

```rust
fn main {
    let x = 2;
    let closure = || println!("{}", x);

    fn factory() -> impl Fn(i32) -> i32 {
        let x = 2;
        let closure = move |a| x + a;
        closure
    }

    let f = factory();
    println!("{}", f(1));

    // impl trait 只允许一种具体的类型，在有多种类型返回值时不适用
    // fn factory(x: i32) -> impl Fn(i32) -> i32 {
    //     let num = 5;
    //     if x > 1 {
    //         |x| x + num
    //     } else {
    //         |x| x - num
    //     }
    // }

    fn factoryDyn(x: i32) -> Box<dyn Fn(i32) -> i32> {
        let num = 5;

        if x > 1 {
            Box::new(move |x| x + num)
        } else {
            Box::new(move |x| x - num)
        }
    }
}
```
