---
title: 'rust: 使用多线程'
date: 2024-06-13 14:33:12
categories: rust
tags: rust
---

## 使用多线程

### 多线程编程的风险

由于多线程的代码是同时运行的，所以无法保证线程间的执行顺序，这会导致一些问题：

- 竞态条件(race conditions)，多个线程以非一致性的顺序同时访问数据资源
- 死锁(deadlocks)，两个线程都想使用某个资源，但是又都在等待对方释放资源后才能使用，结果最终都无法继续执行
- 一些因为多线程导致的很隐晦的 BUG，难以复现和解决

注意：在 rust 线程中借用外部的引用必须拥有 `'static` 生命周期。

### spawn 创建线程

使用 thread::spawn 可以创建线程，它与主线程的执行顺序和次数依赖操作系统如何调度线程，总之，**千万不要依赖线程的执行顺序**：

```rust
thread::spawn(|| {
    for i in 1..10 {
        println!("spawned thread, index = {}", i);
    }
});

for j in 1..5 {
    println!("main thread, index = {}", j);
    thread::sleep(Duration::from_millis(1)); // thread::sleep() 可以强制线程停止执行一段时间
}
```

有几点值得注意：

- 线程内部的代码使用**闭包**来执行
- main 线程一旦结束，程序就立刻结束，如果其它子线程需要完成自己的任务，就需要保证主线程的存活
- thread::sleep 会让当前线程休眠指定的时间，随后其它线程会被调度运行，如果是在单核心处理上，那就会形成并发

输出

```shell
main thread, index = 1
spawned thread, index = 1
spawned thread, index = 2
spawned thread, index = 3
spawned thread, index = 4
spawned thread, index = 5
spawned thread, index = 6
main thread, index = 2
spawned thread, index = 7
spawned thread, index = 8
main thread, index = 3
main thread, index = 4
```

### 线程的结束方式

main 线程是程序的主线程，一旦结束则程序随之结束，同时各个子线程也将被强行终止。
如果父线程不是 main 线程，那么父线程的结束后子线程是继续运行还是被强行终止？

答案是**当父线程不是 main 线程时，父线程的结束不会强制终止子线程，只有线程的代码执行完，线程才会自动结束**。
这是因为虽然在系统编程中，操作系统提供了直接杀死线程的接口，但是**粗暴地终止一个线程可能会引发资源没有释放、状态混乱等不可预期的问题**，所以 Rust 并没有提供功能。

因此**非主线程的子线程的代码不会执行完时(阻塞、死循环)，子线程就不会结束，此时只有主动关闭或结束主线程才能关闭子线程**。

**阻塞和死循环**

- 线程的任务是一个循环 IO 读取，任务流程类似：IO 阻塞，等待读取新的数据 -\> 读到数据，处理完成 -\> 继续阻塞等待 ··· -\> 收到 socket 关闭的信号 -\> 结束线程，在此过程中，绝大部分时间线程都处于阻塞的状态，因此虽然看上去是循环，CPU 占用其实很小，也是网络服务中最常见的模型
- 线程的任务是一个循环，没有任何阻塞（也不包括休眠），此时如果没有设置终止条件，该线程将持续跑满一个 CPU 核心，并且不会被终止，直到 main 线程的结束

```rust
// 线程的关闭方式
let handle = thread::spawn(|| {
    thread::spawn(|| loop {
        println!("sub sub thread running");
    });
    println!("sub thread end");
});
handle.join().unwrap();

// 睡眠一段时间，看子线程创建的子线程是否还在运行
thread::sleep(Duration::from_millis(10));
```

### join 等待线程结束

在使用 spawn 创建线程中，由于主线程结束，导致依赖主线程的新创建线程并没有执行完整。
为了能让线程安全的结束执行，需要保证**主线程**在依赖线程后结束，使用 join 可以达到目的。

join 可以阻塞当前线程，直到调用 join 方法的线程执行完成后才会解除当前线程的阻塞。

```rust
// 使用 join，可以使当前线程阻塞，直到调用 join 方法的线程执行完成后才会解除当前线程的阻塞
let handle1 = thread::spawn(|| {
    for i in 1..10 {
        println!("spawned1 thread, index = {}", i);
    }
});
let handle2 = thread::spawn(|| {
    for j in 1..10 {
        println!("spawned2 thread, index = {}", j);
    }
});
handle1.join().unwrap(); // spawned1 和 spawned2 不确定的轮换执行
for k in 1..5 {
    println!("main thread, index = {}", k);
    thread::sleep(Duration::from_millis(1));
}
```

### move 和多线程

线程的启动时间点和结束时间点是不确定的，与代码的创建点无关。
由于这种无序和不确定性，**被依赖线程**不一定在依赖线程后结束，换句话说，存在**被依赖线程结束运行，而依赖线程仍在运行**的情况。

这意味依赖线程访问**被依赖线程**的环境时，需要有一个限制条件：
当依赖线程的闭包访问被依赖线程的变量时，需要将被依赖线程的变量所有权转移到依赖线程的闭包内，否则容易出现被依赖线程结束运行后变量被释放，而依赖线程还在访问该变量的问题。

因此使用 move 关键字拿走访问变量的所有权，被依赖线程就无法再使用该变量，也就是不会再释放该变量的值。

```rust
let v = vec![1, 2, 3];
let handle = thread::spawn(move || {
    println!("spawned thread, index = {:?}", v);
});
handle.join().unwrap();
```

### 多线程的性能

不精确估算，创建一个线程大概需要 0.24 毫秒，随着线程的变多，这个值会变得更大，因此线程的创建耗时是不可忽略的，**只有真的值得用线程去处理的任务时，才使用线程**。

#### 创建合适的线程数量

合适的线程数是提高并发性能的关键，并发性能关键在于 CPU 负载利用率的高低。

当任务是 CPU 密集型时，就算线程数超过了 CPU 核心数，也不能获得更好的性能，因为每个线程的任务都可以轻松让 CPU 的某个核心跑满，CPU 负载利用率的高，此时让线程数等于 CPU 核心数是最好的。

当任务大部分时间都处于阻塞状态时，就可以考虑增多线程数量，这样当某个线程处于阻塞状态时会被切走，系统运行其它的线程。
典型就是网络 IO 操作，可以为每一个进来的用户连接创建一个线程去处理，该连接绝大部分时间都是处于 IO 读取阻塞状态，因此有限的 CPU 核心完全可以处理成百上千的用户连接线程。
但事实上网络 IO 一般都不再使用多线程的方式了，毕竟操作系统的线程数是有限的，意味着并发数也很容易达到上限，而且过多的线程也会导致线程上下文切换的代价过大，使用 async/await 的 M:N 并发模型就没有这个烦恼。

CPU 密集型任务尽量让线程数量等于 CPU 核心数，阻塞型任务可以选择 async/await 的 M:N 模型。
如果不确定任务是 CPU 密集型还是阻塞型，就需要通过测试来确定选择合适的线程数量。

#### 多线程的开销

阅读：https://course.rs/advance/concurrency-with-threads/thread.html#多线程的开销

性能不会随着线程数的增加而线性增长，锁、数据竞争、缓存失效这些限制了现代化软件系统随着 CPU 核心的增多性能也线性增加的野心。

- 线程过多时，CPU 缓存的命中率会显著下降，同时多个线程竞争一个 CPU Cache-line 的情况也会经常发生
- 大量读写可能会让内存带宽也成为瓶颈
- 读和写不一样，无锁数据结构的读往往可以很好地线性增长，但是写不行，因为写竞争太大

### 线程屏障(Barrier)

在 Rust 中，可以使用 Barrier 让多个线程**都执行到某个位置**后，才继续一起往后执行，Barrier 对于需要所有线程同时开始执行某部分代码的场景非常有用。

Rust 的 Barrier 类型是一个同步原语，Barrier 数量对应线程数量，如果两者不相等则会导致线程无法自动停止：

```rust
let count = 6;
let mut handles: Vec<JoinHandle<()>> = Vec::with_capacity(count);
let barrier = Arc::new(Barrier::new(7));
for i in 0..count {
    let b = barrier.clone();
    handles.push(thread::spawn(move || {
        println!("before wait, {i}");
        b.wait();
        println!("after wait, {i}");
    }))
}
// 等待线程结束才允许关闭主线程
for t in handles {
    t.join().unwrap();
}
```

### 线程局部变量（Thread Local Variable）

对于多线程编程，线程局部变量在一些场景下非常有用，而 Rust 通过标准库和三方库对此进行了支持。

rust 标准库中提供 `thread_local!` 宏来生成线程内部变量 Thread Local Variable，有一点值得注意：
`thread_local!` 宏声明线程局部存储 (Thread Local Storage, TLS) 变量需要使用 static 关键字，这是由于 TLS 变量的内存布局、线程安全性、编译时求值以及语义等原因决定的。
`thread_local!` 宏本质上是在创建**一个静态变量**，所以在修改后，同一个线程访问的都是被修改后的值。

- 内存布局：静态变量在程序的整个生命周期内存在，而不是在函数调用时创建和销毁，这使得 TLS 变量可以在多个函数和模块中访问
- 线程安全：静态变量的初始化是线程安全的。当多个线程同时访问 TLS 变量时，每个线程都会得到自己独立的变量副本
- 编译时求值：使用 static 可以让编译器在编译时就确定变量的内存布局和初始化表达式，这提高了性能和安全性
- 语义：将 TLS 变量声明为 static 更加明确地表达了它们的作用域和生命周期

线程内部使用该变量的 with 方法获取变量值，每个新的线程访问它时，都会使用它的初始值作为开始：

```rust
// 线程内部变量（Thread Local Variable），每个新的线程访问它时，都会使用它的初始值作为开始
thread_local! {
    static N: RefCell<i32> = RefCell::new(1);
}
thread_local!(static FOO: RefCell<u32> = RefCell::new(1)); // 另一种声明形式

// 使用该变量的 with 方法获取变量值，thread_local! 的本质是创建一个全局静态变量，所以在修改后，同一个线程访问的都是被修改后的值
N.with(|n| {
    *n.borrow_mut() = 2;
});
let handle = thread::spawn(|| {
    // 使用该变量的 with 方法获取变量值，thread_local! 的本质是创建一个全局静态变量，所以在修改后，同一个线程访问的都是被修改后的值
    N.with(|n| {
        *n.borrow_mut() = 4;
    });
    // 子线程打印的值是4
    println!("{:?}", N.take());
});
handle.join().unwrap();
// 主线程打印的值是2
println!("{:?}", N.take());
```

此外还可以在结构中声明线程内部变量，与普通声明一样，每个新的线程访问它时都会使用它的初始值作为开始：

```rust
// 结构体使用 thread_local! 创建线程局部变量
struct Node {
    value: i32,
}
impl Node {
    thread_local! {
        static V: RefCell<i32> = RefCell::new(1);
    }
}
let handle = thread::spawn(|| {
    // 使用该变量的 with 方法获取变量值，thread_local! 的本质是创建一个全局静态变量，所以在修改后，同一个线程访问的都是被修改后的值
    Node::V.with(|v| {
        *v.borrow_mut() = 4;
        println!("{v:?}");
    });
    println!("{}", Node::V.take()); // 4，本质是一个全局变量，被修改后同一线程访问的数据是被修改后的值
});
handle.join().unwrap();
println!("{}", Node::V.take()); // 1
```

当然结构体也可以通过引用的方式使用线程局部变量：

```rust
thread_local! {
    static M: RefCell<i32> = RefCell::new(1);
}
struct Person {
    m: &'static LocalKey<RefCell<i32>>, // 结构体通过引用的方式使用线程局部变量
}
let handle = thread::spawn(|| {
    let p = Person { m: &M }; // 与普通声明一样，每个新的线程访问它时都会使用它的初始值作为开始
    p.m.with(|m| {
        *m.borrow_mut() = 4;
    });
    println!("{:?}", p.m.take());
});
handle.join().unwrap();
let p = Person { m: &M };
println!("{:?}", p.m.take());
```

线程中对 FOO 的使用是通过借用的方式，如果需要每个线程独自获取它的拷贝，最后进行汇总，`thread_local!` 宏就不太满足需求了。
借助第三方库 thread-local 很容易实现：https://course.rs/advance/concurrency-with-threads/thread.html#三方库-thread-local

```rust
// 使用第三方包 thread_local
// 为了便于修改基础数据，使用 Cell
// let tls: Arc<ThreadLocal<Cell<i32>>> = Arc::new(ThreadLocal::new());
let tls = Arc::new(ThreadLocal::<Cell<i32>>::new());
// 创建多个线程，每个线程更新第三方包生成的线程局部变量（TLS），最后统计总和
let count = 5;
let mut handles: Vec<JoinHandle<()>> = Vec::with_capacity(count);
for i in 0..count {
    let _tls = Arc::clone(&tls);
    handles.push(thread::spawn(move || {
        let cell = _tls.get_or(|| Cell::new(0));
        cell.set(cell.get() + 1);
    }))
}
// 等待所有的线程都执行完成
for h in handles {
    h.join().unwrap();
}
// 统计线程中的数据
let tls = Arc::try_unwrap(tls).unwrap();
let total = tls
    .into_iter()
    .reduce(|acc, item| Cell::new(acc.get() + item.get()));
println!("{:?}", total.unwrap());
```

最终可以得到每个线程的线程局部变量的值的总和。

### 条件变量控制线程的挂起和执行

**条件变量(Condition Variables)** 经常和 **Mutex** 一起使用，可以让**线程挂起**，直到某个条件发生后再继续执行。

#### Mutex（Mutual Exclusion）

Mutex 是一个互斥锁（Mutual Exclusion），同时是一个同步原语，用于在多线程环境中**保护共享数据**。
当多个线程需要访问同一数据时，Mutex 确保任何时刻只有一个线程可以访问该数据，从而防止数据竞争和不一致的问题。

使用 Mutex 时，需要先锁定它访问数据，然后再解锁让其他线程可以访问该数据。
锁定和解锁的过程通常是自动的，通过 Rust 的作用域管理来实现。当 Mutex 的锁超出作用域时，它会自动释放。

不同于线程局部变量的每一个线程都有单独的数据拷贝，**Mutex 用于多线程访问同一个实例**，因为用于多线程，所以常常和 **Arc** 搭配使用：

```rust
// Mutex 需要手动上锁，超过作用于后自动解锁
let count = 5;
let mutex = Arc::new(Mutex::new(Cell::new(1)));
let mut handles: Vec<JoinHandle<()>> = Vec::with_capacity(count);
for i in 0..count {
    let _mutex = Arc::clone(&mutex);
    handles.push(thread::spawn(move || {
        // 上锁，其他线程不能访问该数据
        let cell = _mutex.lock().unwrap();
        cell.set(i.into())
        // 超出作用域后自动解锁
    }))
}
// 等待所有线程完成
for h in handles {
    h.join().unwrap();
}
println!("mutex.lock().unwrap() = {:?}", mutex.lock().unwrap());
```

#### Condvar（Condition Variable）

Condvar（Condition Variable）是一个同步原语，用于在多线程环境中协调线程的执行，可以视为一个信号。

`Condvar` 可以阻塞一个或多个线程，让线程等待某个信号发出后继续执行，因此具有阻塞性的 `Condvar` **不占用 CPU 资源**。
当 `Condvar` 信号发出时，可以唤醒等待的线程，让线程继续执行。这种机制可以帮助提高多线程程序的性能和响应性。

`Condvar` 通常配合 while 使用。while 一直循环，condvar wait 等待条件不满足时阻塞执行，不占用 CPU 资源。
当条件满足时，`Condvar` 唤醒等待的线程，while 循环继续执行。

因为用于多线程，并且阻塞需要给定条件和解除信号，所以 Condvar 所以常常和 **Arc、Mutex** 搭配使用。

#### Mutex 和 Condvar

`Condvar`（Condition Variable）与 `Mutex` （Mutual Exclusion）一起使用，允许线程阻塞等待特定条件成立，并在条件满足时被唤醒。

- 等待特定条件成立，即线程（多个线程）访问某一个数据并将其作为条件，用到 Mutex
- 在条件满足时发出信号，唤醒等待的线程，用到 Condvar

```rust
let count = 5;
let pair = Arc::new((Mutex::new(0), Condvar::new()));
let mut handles: Vec<JoinHandle<()>> = Vec::with_capacity(count);
for i in 0..count {
    let _pair = Arc::clone(&pair);

    handles.push(thread::spawn(move || {
        // 不应该在外部尝试解构 _pair，而是直接在闭包内部使用它
        let (mutex, condvar) = &*_pair;
        let mut single = mutex.lock().unwrap();
        if i == 4 {
            *single = 4;
            condvar.notify_all();
        }

        while *single != 4 {
            // 当条件不满足时进入循环，被 condvar.wait 阻塞，当 condvar.notify 发出信号时，改变 single
            // 此时线程执行，while 循环不再符合条件，代码继续往下执行
            single = condvar.wait(single).unwrap();
        }

        println!("condvar index = {i}");
    }))
}
// 等待所有线程完成
for h in handles {
    h.join().unwrap();
}
```

### 只调用一次的函数

如果希望一个变量无论被多少线程访问且无论访问的顺序如何，该变量只调用一次初始化代码，就需要用到 Once::call_one 方法。
回顾所学的两个特殊变量/类型，线程局部变量（Thread Local Variable）和 互斥锁 Mutex（Mutual Exclusion）：

- 线程局部变量（Thread Local Variable）在每个线程访问时都会拷贝一份初始化的值，不适合做能被多线程访问，且变量只初始化一次的场景
- Mutex 存储同一份数据，可以做到被多线程访问，且变量限制初始化一次的功能，这需要每个线程判断是否为**最初值**，比较麻烦

除了 Mutex 外，只要是一个所有线程可以访问的位置，都可以做到只调用一次初始化的功能。并且初始化的值一定是第一个访问的线程初始化的。

使用 Mutex 实现只初始化一次的功能：

```rust
// 用 Mutex 做一个只初始化一次的变量，这需要每个线程在访问变量时判断是否为**最初值**
// 这里有一点注意，当前程序中初始化的值一定是第一个访问的线程，也就是索引为0线程进行了唯一一次初始化流程
const init_value: i32 = 1;
let mutex = Arc::new(Mutex::new(init_value));
let count = 5;
let mut handles: Vec<JoinHandle<()>> = Vec::with_capacity(5);
for i in 0..count {
    let _mutex = Arc::clone(&mutex);
    handles.push(thread::spawn(move || {
        let mut num = _mutex.lock().unwrap();
        // 每个线程的初始化值不一样
        if *num == init_value {
            *num = i;
        }
    }))
}

for h in handles.into_iter() {
    h.join().unwrap();
}
println!("{:?}", mutex.lock().unwrap());
```

不确定顺序的线程：

```rust
let mutex = Arc::new(Mutex::new(init_value));
let mutex1 = Arc::clone(&mutex);
let mutex2 = Arc::clone(&mutex);
let mut handle = thread::spawn(move || {
    let mut num = mutex1.lock().unwrap();
    if *num == init_value {
        *num = 10
    }
});
let mut handle2 = thread::spawn(move || {
    let mut num = mutex2.lock().unwrap();
    if *num == init_value {
        *num = 20
    }
});
handle.join().unwrap();
println!("{}", mutex.lock().unwrap()); // 10 OR 20
```

使用 Mutex 做只调用一次初始化函数的功能，除了需要初始化代码，还需要手动判断是否已初始化的逻辑。`Once::call_once` 可以减少手动判断的步骤。

```rust
// Once::call_once，只调用一次的函数
static INIT: Once = Once::new();
let mutex = Arc::new(Mutex::new(init_value));
let mutex1 = Arc::clone(&mutex);
let mutex2 = Arc::clone(&mutex);
let mut handle = thread::spawn(move || {
    let mut num = mutex1.lock().unwrap();
    INIT.call_once(move || *num = 10);
});
let mut handle2 = thread::spawn(move || {
    let mut num = mutex2.lock().unwrap();
    INIT.call_once(move || *num = 20);
});
handle.join().unwrap();
println!("{}", mutex.lock().unwrap());
```

执行初始化过程一次，并且**只执行一次**。如果当前有另一个初始化过程正在运行，线程将阻止该方法被调用。
当这个函数返回时，保证一些初始化已经运行并完成，它还保证由执行的闭包所执行的任何内存写入都能被其他线程在这时可靠地观察到。

### Code

```rust
fn main() {
     // 一、初步使用 thread
    thread::spawn(|| {
        for i in 1..10 {
            println!("spawned thread, index = {}", i);
        }
    });
    for j in 1..5 {
        println!("main thread, index = {}", j);
        thread::sleep(Duration::from_millis(1)); // thread::sleep() 可以强制线程停止执行一段时间
    }

    // 二、线程的关闭方式
    let handle = thread::spawn(|| {
        thread::spawn(|| loop {
            // println!("sub sub thread running");
        });
        println!("sub thread end");
    });
    handle.join().unwrap();
    // 睡眠一段时间，看子线程创建的子线程是否还在运行
    thread::sleep(Duration::from_millis(10));

    // 三、使用 join，可以使当前线程阻塞，直到调用 join 方法的线程执行完成后才会解除当前线程的阻塞
    let handle1 = thread::spawn(|| {
        for i in 1..10 {
            println!("spawned1 thread, index = {}", i);
        }
    });
    let handle2 = thread::spawn(|| {
        for j in 1..10 {
            println!("spawned2 thread, index = {}", j);
        }
    });
    handle1.join().unwrap();
    for k in 1..5 {
        println!("main thread, index = {}", k);
        thread::sleep(Duration::from_millis(1));
    }

    // 四、move 和闭包
    let v = vec![1, 2, 3];
    let handle = thread::spawn(move || {
        println!("spawned thread, index = {:?}", v);
    });
    handle.join().unwrap();

    // 五、线程屏障 Barrier，Rust 的 Barrier 类型是一个同步原语。
    // Barrier 数量对应线程数量，如果两者不相等则会导致线程无法自动停止。
    // Barrier 对于需要所有线程同时开始执行某部分代码的场景非常有用。
    let count = 6;
    let mut handles: Vec<JoinHandle<()>> = Vec::with_capacity(count);
    let barrier = Arc::new(Barrier::new(count));
    for i in 0..count {
        let b = barrier.clone();
        handles.push(thread::spawn(move || {
            println!("before wait, {i}");
            b.wait();
            println!("after wait, {i}");
        }))
    }
    // 等待线程结束才允许关闭主线程
    for t in handles {
        t.join().unwrap();
    }

    // 六、线程内部变量（Thread Local Variable）
    thread_local! {
        static N: RefCell<i32> = RefCell::new(1);
    }
    thread_local!(static FOO: RefCell<u32> = RefCell::new(1));

    // 使用该变量的 with 方法获取变量值，thread_local! 的本质是创建一个全局静态变量，所以在修改后，同一个线程访问的都是被修改后的值
    N.with(|n| {
        *n.borrow_mut() = 2;
    });
    let handle = thread::spawn(|| {
        // 使用该变量的 with 方法获取变量值，thread_local! 的本质是创建一个全局静态变量，所以在修改后，同一个线程访问的都是被修改后的值
        N.with(|n| {
            *n.borrow_mut() = 4;
        });
        // 子线程打印的值是4
        println!("{:?}", N.take());
    });
    handle.join().unwrap();
    // 主线程打印的值是2
    println!("{:?}", N.take());

    // 结构体使用 thread_local! 创建线程局部变量
    struct Node {
        value: i32,
    }
    impl Node {
        thread_local! {
            static V: RefCell<i32> = RefCell::new(1);
        }
    }
    let handle = thread::spawn(|| {
        // 使用该变量的 with 方法获取变量值，thread_local! 的本质是创建一个全局静态变量，所以在修改后，同一个线程访问的都是被修改后的值
        Node::V.with(|v| {
            *v.borrow_mut() = 4;
            println!("{v:?}");
        });
        println!("{}", Node::V.take()); // 4，本质是一个全局变量，被修改后同一线程访问的数据是被修改后的值
    });
    handle.join().unwrap();
    println!("{}", Node::V.take()); // 1

    thread_local! {
        static M: RefCell<i32> = RefCell::new(1);
    }
    struct Person {
        m: &'static LocalKey<RefCell<i32>>, // 结构体通过引用的方式使用线程局部变量
    }
    let handle = thread::spawn(|| {
        let p = Person { m: &M }; // 与普通声明一样，每个新的线程访问它时都会使用它的初始值作为开始
        p.m.with(|m| {
            *m.borrow_mut() = 4;
        });
        println!("{:?}", p.m.take());
    });
    handle.join().unwrap();
    let p = Person { m: &M };
    println!("{:?}", p.m.take());

    // 使用第三方包 thread_local
    // 为了便于修改基础数据，使用 Cell
    // let tls: Arc<ThreadLocal<Cell<i32>>> = Arc::new(ThreadLocal::new());
    let tls = Arc::new(ThreadLocal::<Cell<i32>>::new());
    // 创建多个线程，每个线程更新第三方包生成的线程局部变量（TLS），最后统计总和
    let count = 5;
    let mut handles: Vec<JoinHandle<()>> = Vec::with_capacity(count);
    for i in 0..count {
        let _tls = Arc::clone(&tls);
        handles.push(thread::spawn(move || {
            let cell = _tls.get_or(|| Cell::new(0));
            cell.set(cell.get() + 1);
        }))
    }
    // 等待所有的线程都执行完成
    for h in handles {
        h.join().unwrap();
    }
    thread::sleep(Duration::from_millis(10));
    // 统计线程中的数据
    let tls = Arc::try_unwrap(tls).unwrap();
    let total = tls
        .into_iter()
        .reduce(|acc, item| Cell::new(acc.get() + item.get()));
    println!("{:?}", total.unwrap());

    // 不同于线程局部变量的每一个线程都有单独的数据拷贝，Mutex 用于多线程访问同一个实例，因为用于多线程，所以常常和 Arc 搭配使用
    // Mutex 需要手动上锁，超过作用于后自动解锁
    let count = 5;
    let mutex = Arc::new(Mutex::new(Cell::new(1)));
    let mut handles: Vec<JoinHandle<()>> = Vec::with_capacity(count);
    for i in 0..count {
        let _mutex = Arc::clone(&mutex);
        handles.push(thread::spawn(move || {
            // 上锁，其他线程不能访问该数据
            let cell = _mutex.lock().unwrap();
            cell.set(i.into())
            // 超出作用域后自动解锁
        }))
    }
    // 等待所有线程完成
    for h in handles {
        h.join().unwrap();
    }
    println!("mutex.lock().unwrap() = {:?}", mutex.lock().unwrap());

    // Condvar（Condition Variable）是一个同步原语，用于在多线程环境中协调线程的执行，可以视为一个信号。
    // `Condvar` 的主要作用是让一个或多个线程等待某个条件成立，而**不占用 CPU 资源**，通常配合 while 使用。
    // 让 while 一直循环，condvar wait 等待条件不满足时阻塞执行，因此不占用 CPU 资源。
    // 当条件满足时，其他线程可以通知 `Condvar` 唤醒等待的线程，while循环继续执行。这种机制可以帮助提高多线程程序的性能和响应性。
    let count = 5;
    let pair = Arc::new((Mutex::new(0), Condvar::new()));
    let mut handles: Vec<JoinHandle<()>> = Vec::with_capacity(count);
    for i in 0..count {
        let _pair = Arc::clone(&pair);

        handles.push(thread::spawn(move || {
            // 不应该在外部尝试解构 _pair，而是直接在闭包内部使用它
            let (mutex, condvar) = &*_pair;
            let mut single = mutex.lock().unwrap();
            if i == 4 {
                *single = 4;
                condvar.notify_all();
            }

            while *single != 4 {
                // 当条件不满足时进入循环，被 condvar.wait 阻塞，当 condvar.notify 发出信号时，改变 single
                // 此时线程执行，while 循环不再符合条件，代码继续往下执行
                single = condvar.wait(single).unwrap();
            }

            println!("condvar index = {i}");
        }))
    }
    // 等待所有线程完成
    for h in handles {
        h.join().unwrap();
    }

    // 用 Mutex 做一个只初始化一次的变量，这需要每个线程在访问变量时判断是否为**最初值**
    // 这里有一点注意，当前程序中初始化的值一定是第一个访问的线程，也就是索引为0线程进行了唯一一次初始化流程
    const init_value: i32 = 1;
    let mutex = Arc::new(Mutex::new(init_value));
    let count = 5;
    let mut handles: Vec<JoinHandle<()>> = Vec::with_capacity(count);
    for i in 0..count {
        let _mutex = Arc::clone(&mutex);
        handles.push(thread::spawn(move || {
            let mut num = _mutex.lock().unwrap();
            // 每个线程的初始化值不一样
            if *num == init_value {
                *num = i as i32;
            }
        }))
    }
    for h in handles.into_iter() {
        h.join().unwrap();
    }
    println!("{:?}", mutex.lock().unwrap());

    let mutex = Arc::new(Mutex::new(init_value));
    let mutex1 = Arc::clone(&mutex);
    let mutex2 = Arc::clone(&mutex);
    let mut handle = thread::spawn(move || {
        let mut num = mutex1.lock().unwrap();
        if *num == init_value {
            *num = 10
        }
    });
    let mut handle2 = thread::spawn(move || {
        let mut num = mutex2.lock().unwrap();
        if *num == init_value {
            *num = 20
        }
    });
    handle.join().unwrap();
    println!("{}", mutex.lock().unwrap()); // 10 OR 20

    // Once::call_once，只调用一次的函数
    static INIT: Once = Once::new();
    let mutex = Arc::new(Mutex::new(init_value));
    let mutex1 = Arc::clone(&mutex);
    let mutex2 = Arc::clone(&mutex);
    let mut handle = thread::spawn(move || {
        let mut num = mutex1.lock().unwrap();
        INIT.call_once(move || *num = 10);
    });
    let mut handle2 = thread::spawn(move || {
        let mut num = mutex2.lock().unwrap();
        INIT.call_once(move || *num = 20);
    });
    handle.join().unwrap();
    println!("{}", mutex.lock().unwrap());
}
```
