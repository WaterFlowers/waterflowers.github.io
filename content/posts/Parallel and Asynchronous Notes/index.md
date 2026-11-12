---
title: "Parallel and Asynchronous Notes"
date: 2026-01-05
draft: false
tags: ["Parallel", "Asynchronous", "c++"]
summary: "Parallel and Asynchronous Notes"
---


写程序时，完全不用管核心数，想创建多少线程就创建多少（当然，太多会因切换开销而变慢）。

- 当“真正需要同时干活的线程数” ≤ 核心数时，它们趋向于**并行（同时）**。
- 当“真正需要同时干活的线程数” > 核心数时，它们趋向于**并发**。

所以，一个 4 核 CPU 可以同时运行上百个线程，只是任何一个时刻，最多只有 4 个线程在真正被 CPU 执行。

- **计算密集型任务**（如复杂的数学运算）：通常把线程数设为核心数（或核心数*2）就够了，设多了反而因切换开销变慢。
- **I/O 密集型任务**（如网络爬虫）：一个线程等待网页下载时，CPU 完全可以去处理另一个线程。所以开几百个线程也没问题，它们大多数时间都在等 I/O，不占用 CPU。

## 一、并发和并行

### 1、并发
并发的意思就是在一段时间内可以同时处理多个任务，但在同一时间点只有一个任务在执行，并且多个任务之间的执行不一定是顺序的。在并发应用程序中，两个任务可以在一段时间内交替启动、运行和完成。例如，任务2可以在任务1完成之前就启动。

在计算机科学中，不同的 CPU 实现并发的方式是不同的：
- 在单核 CPU 环境中，并发是通过切换进程的上下文实现的。
- 如果是多核心的 CPU，并发可以通过并行实现。

![替代文字](/images/task-1.jpg)

*(图示：Task-1 和 Task-2 在时间轴上交替运行)*

### 2、并行
假设你现在有两个任务：做饭以及和朋友打电话。你可以同时做这两件事情。你可以在做饭的同时通过手机给朋友打电话。那么你现在就是在并行的做事情。

并行意味着在同一时间点能同时执行多个任务。在计算机科学中，并行计算是指同时进行多个计算的过程。在同一时间周期内两个任务同时执行。

![替代文字](/images/task-1-2.jpg)

*(图示：Task-1 和 Task-2 在同一时间段内并排运行)*

### 3、并行和并发之间有什么关系呢？
并发性和并行性关注的是在计算机架构中任务或计算是如何被执行的。
- 在单核 CPU 环境中，并发是通过在同一时间周期内上下文切换来实现的。即在一个特定的时间内，只有一个任务在执行。
- 在多核环境下，并发是通过并发的执行多个任务来实现的。

## 二、线程 & 进程

### 1、线程
线程是单一顺序执行的代码，该段代码可独立的执行。它是能被操作系统可执行的最小单元。一个程序既可以是单线程的，也可以是多线程的。

### 2、进程
一个进程就是一个程序运行的实例。一个程序可以有多个进程（把一个程序启动多次，每个进程都有自己的运行环境，相互不影响）。一个进程启动的时候通常只会启一个线程，例如一个主要的线程。但随着程序的执行，它可以创建更多的线程。


![替代文字](/images/process.png)


*(图示：Application 包含 Process，Process 包含 Thread-1 和 Thread-2)*

## 三、同步和异步

### 1、同步（按顺序执行）
想象一下，你要写两封信，一封给你妈妈，另一封给你最好的朋友。你不能同时写两封信，除非你的双手能同时书写。在同步程序模型中，任务是一个接着一个执行的。每个任务需要等待前一个任务执行完成后才能开始。

### 2、异步
想象一下你有这样两个任务：制作三明治和在洗衣机中洗衣服。你可以把你的衣服放到洗衣机中，然后不需要等着它完成，你就可以去制作三明治了。这两个任务就是异步执行。在异步程序模型中，当一个任务已经执行了，你无需等待该任务执行完成，就可以切换到另外一个任务上。

#### (1) 在单线程和多线程环境中的同步和异步

- **① 同步-单线程**：任务挨个执行。每个任务需要等待前一个任务执行完毕。（Task 1 -> Task 2 -> Task 3）
![替代文字](/images/task-1-2-3.jpg)

- **② 同步-多线程**：每个任务在不同线程中执行，但需要等待前置任务的完成。
![替代文字](/images/thread-1.jpg)

- **④ 异步-单线程**：任务执行不需要等待其他任务的完成。但在一个时间点只能有一个任务执行。。
   ![替代文字](/images/start.jpg)

- **④ 异步-多线程**：任务执行不需要等待其他任务的完成。但在同一个时间点可以有多个任务执行。（Thread-1 运行 Task-1 时，Thread-2 同时运行 Task-2）

   ![替代文字](/images/thread-2.jpg)
  

在开发和并行中同步和异步程序是什么样的角色？异步程序模型帮助我们实现并发，多线程中的异步程序模型是一种实现并发的方式。

#### (2) 多进程
每个进程拥有独立的内存空间。进程A不能直接访问进程B的变量，必须通过专门的“进程间通信（IPC）”机制来交换数据，比如管道、消息队列、共享内存、socket等。这种隔离让进程更安全：一个进程崩溃一般不会影响其他进程。

C++标准目前没有跨平台的进程管理库，需要借助操作系统API或第三方库：
- Linux/Unix：`fork()`、`exec()` 等系统调用。
- Windows：`CreateProcess()` 等 API。

#### (3) 多线程编程
同一个进程内的所有线程共享该进程的内存空间（全局变量、堆内存等）。线程之间可以直接读写同一份数据，通信非常方便。但这也带来了**数据竞争**问题，必须用锁（`std::mutex`）、原子操作等同步手段来保护共享数据。一个线程的严重错误（如访问无效地址）可能导致整个进程崩溃。

C++标准库从 C++11 起直接支持，使用 `<thread>`、`<mutex>`、`<future>` 等头文件。

#### (4) Async
让你从“管理线程”变成“管理任务”，把更多脏活交给库去做。在 C++ 中，`std::async` 是一个标准库函数，定义在 `<future>` 头文件中，位于 `std` 命名空间。

## 四、总结

1、开发和并行指的是任务执行的方式。

2、同步和异步指的是通讯编程模型。

3、单线程和多线程指的是任务执行的环境。

## 五、案例 (基本语法)

### 1、创建线程
```cpp
#include <iostream>
#include <thread>

void task(int x) {
    // 线程要执行的代码
}

int main() {
    // 创建线程，立刻执行 task(10)
    // 第一个参数是可调用对象（函数、lambda、函数对象等），后面的参数是传给函数的实参
    std::thread t(task, 10); 
    return 0;
}
```
### 2、等待线程结束 join()
t.join(); // 主线程会在这里等待，直到t执行完
// join 相当于“等它干完活再继续”
// 如果不 join 也不 detach，线程对象析构时程序会直接终止（std::terminate）
### 3、分离线程 detach()
t.detach(); // 线程在后台独立运行，主线程不再等待
// 使用 detach 要小心：必须保证线程访问的资源（变量等）在它结束前一直有效。
### 4、互斥锁 std::mutex
防止多个线程同时访问共享数据。
```cpp
#include <mutex>

std::mutex mtx; // 一把锁

void safe_print() {
    mtx.lock();   // 加锁
    // ... 临界区，只有一个线程能进入
    mtx.unlock(); // 解锁
}
或者，更推荐的方式：用 std::lock_guard 自动加锁/解锁，不会忘。
void safe_print() {
    std::lock_guard<std::mutex> lock(mtx); // 构造时加锁
    // ... 临界区
    // 离开作用域自动解锁，即使发生异常也安全
}
```
### 5、原子变量 std::atomic
对简单的整型操作，用原子变量更轻量，不用锁。
```cpp
#include <atomic>

std::atomic<int> counter(0);

void add() {
    ++counter; // 原子操作，线程安全，无需 mutex
}
// std::atomic支持 ++、--、+= 等操作，保证中间状态不会被其他线程看到。
```
### 6、综合例子
```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void print(char c) {
    for (int i = 0; i < 10; ++i) {
        std::lock_guard<std::mutex> lock(mtx);
        std::cout << c; // 保证每次输出不会被其他线程打断
    }
}

int main() {
    std::thread t1(print, 'A');
    std::thread t2(print, 'B');
    t1.join();
    t2.join();
    return 0;
}
// 输出类似 AAAAAAAABBBBBBBBBB（不同运行顺序可能不同，但不会出现ABAB交叉错乱）。
```

## 六、C++ 在数据科学中应用

**C++ 在数据科学中的重点图谱**

|`层次`|	`重点内容`|	`与数据科学的连接`|
| :--- | :---: | ---: | 
|生存基础|	指针、STL、内存管理、编译|	把 Python 慢循环改写成高效 C++|
|价值桥梁|	pybind11、Cython|	让 Python 无缝调用 C++ 核心|
|性能倍增器|	多线程 (OpenMP/thread)、SIMD、CUDA|	榨干多核 CPU/GPU，加速模拟、推理、数据预处理|
|造轮工坊|	模板元编程、表达式模板|	设计通用高效数值库（极少人需要）|

## 七、常用的异步函数模版
### 1.	常用头文件：
#### 1. <future> —— 异步任务与结果传递

|`组件`|	`种类`|	`作用`|
| :--- | :---: | ---: | 
|std::async|	函数模板|	启动异步任务，返回 std::future|
|std::future<T>	|类模板|	持有异步任务的“未来结果”，通过 get() 获取|
|std::shared_future<T>|	类模板|	允许多个线程等待同一结果（可多次 get()）|
|std::promise<T>|	类模板|	手动设置值或异常，与 future 配对，用于线程间通信|
|std::packaged_task<F>|	类模板|	包装可调用对象，生成 future，可传给线程执行|

典型用法：
std::async 快速启动任务。
std::promise / std::future 实现更灵活的结果传递。
std::packaged_task 把任务封装起来，搭配 std::thread 使用。

#### 2. <thread> —— 线程管理
|`组件`|	`种类`|	`作用`|
| :--- | :---: | ---: | 
|std::thread|	类|	创建和管理线程|
|std::this_thread::sleep_for|	函数模板|	让当前线程休眠指定时间|
|std::this_thread::yield|	函数|	主动让出 CPU 时间片|
|std::this_thread::get_id|	函数|	获取当前线程 ID|

常用配合：用 std::thread 显式创建线程，再用 <future> 传递结果。
#### 3. <chrono> —— 时间工具

|`组件`|	`种类`|	`作用`|
| :--- | :---: | ---: | 
|std::chrono::seconds|	类型别名|	表示秒级时长|
|std::chrono::milliseconds|	类型别名|	毫秒|
|std::chrono::steady_clock|	类|	单调时钟，适合测量间隔|

常和 sleep_for 一起用，指定等待时长。
#### 4. 同步工具（多线程共享数据时用）

|`头文件`|	`核心类/模板`	|`作用`|
| :--- | :---: | ---: | 
|<mutex>|	std::mutex, std::lock_guard, std::unique_lock|	互斥锁，保护临界区|
|<condition_variable>|	std::condition_variable|	线程等待/通知机制|
|<atomic>|	std::atomic<T>|	原子操作，无锁同步|
#### 5、总结

```cpp
启动异步任务
│
├─ std::async           → 自动创建线程，返回 future
├─ std::packaged_task   → 包装任务，获取 future，可手动传给 thread
└─ std::promise         → 纯手动设置值，与 future 配对

获取结果
└─ std::future / std::shared_future  → 通过 get() 等待并取值

线程控制
└─ std::thread / std::this_thread

时间控制
└─ std::chrono + sleep_for

安全同步
└─ std::mutex / std::atomic / std::condition_variable
```
（1）只想要异步执行并拿结果 → std::async + std::future。  
（2）需要精细控制线程和结果传递 → std::thread + std::promise / std::packaged_task + std::future。  
（3）涉及共享数据 → 必须搭配 <mutex> 或 <atomic>。  

## 八、智能指针

### 1. 三巨头核心区别（选型铁律）

|`指针类型`|	`外号`|	`核心特征`|	`使用场景`|
| :--- | :---: | ---: | ---: | 
|unique_ptr|	独占指针|	独生子。不准拷贝，只能移动（std::move）|	绝大多数日常场景，代替裸指针管理资源。|
|shared_ptr	|共享指针	|引用计数。允许多人拷贝，最后一个销毁时释放资源。|	多个对象需要共享同一块数据（如多线程、图结构）。|
|weak_ptr	|弱引用指针	|小弟助攻。不增加引用计数，配合 shared_ptr 使用。|	专门用来打破循环引用（内存泄漏元凶），观察者模式。|

### 2. 创建方式（最重要的一行代码）

|`指针类型`|	`正确创建姿势（C++14 起`）|	`错误姿势（别碰）`|
| :--- | :---: | ---: | 
|unique_ptr	|auto p = std::make_unique<int>(10);|	❌ unique_ptr<int> p(new int(10));（啰嗦且异常不安全）|
|shared_ptr	|auto p = std::make_shared<int>(10);|	❌ shared_ptr<int> p(new int(10));（内存碎片多）|
|核心记忆：永远优先使用 make_unique 和 make_shared。|		

### 3. 常用成员函数速查表（必背！）

|`函数`|	`作用`|	`示例`|	`注意事项`|
| :--- | :---: | ---: | ---: | 
|get()|	获取内部的裸指针（仅供查看）|	int* raw = p.get();	|千万不要 delete raw，否则智能指针析构时会二次释放崩溃。|
|reset()	|重置指针。释放旧资源，指向新资源（或空）|	p.reset(); (置空)p.reset(new int(5)); (换新)		|释放后记得检查 if(p)。|	
|release()	|仅 unique_ptr 有。放弃所有权，返回裸指针，但不释放内存|int* raw = p.release();	|调用后 p 变为 nullptr，你必须手动 delete raw，否则内存泄漏。|
|swap()|	交换两个智能指针的内容|	p1.swap(p2);	|很少用，知道就行。|
|use_count()|	仅 shared_ptr 有。返回当前引用计数|	long cnt = p.use_count();	|只用于调试，不要依赖它做业务逻辑。|

### 4. 运算符重载（用起来像指针）

|操作|	写法|	说明|
| :--- | :---: | ---: | 
|解引用|	*p	|获取对象本身。|
|箭头访问|	p->func()	|访问成员函数或变量。|
|布尔判空|	if(p) 或 if(!p)|	判断是否为空指针（检查内部是否 nullptr）。|

### 5. 所有权转移（移动语义）

```cpp
unique_ptr：只能移动，不能拷贝。
std::unique_ptr<int> p1 = std::make_unique<int>(10);
std::unique_ptr<int> p2 = std::move(p1); // ✅ p1 变为空，p2 接手
// std::unique_ptr<int> p3 = p1; // ❌ 编译报错（拷贝被禁用）

shared_ptr：可拷贝（引用计数 +1）。
std::shared_ptr<int> p1 = std::make_shared<int>(10);
std::shared_ptr<int> p2 = p1; // ✅ 两者指向同一块内存，计数变为 2
```

### 6. 解决 shared_ptr 循环引用的杀手锏（weak_ptr）

如果两个类互相持有 shared_ptr，会造成内存泄漏（析构时你等我、我等你）。
破解法：将其中一方改为 weak_ptr。
```cpp
class B; // 前置声明
class A { std::shared_ptr<B> b_ptr; };
class B { std::weak_ptr<A> a_ptr; }; // 这里用 weak，不增加引用计数
使用 weak_ptr 前，必须调用 lock() 将其临时升级为 shared_ptr，以确保访问时对象还在
if (auto tmp = weak_a.lock()) { // 升级成功
    tmp->doSomething();
}
```

### 7、案例

#### Unique_ptr (独占指针)
```cpp
#include <iostream>
#include <memory>
class Product {
public:
    void show() { std::cout << "Product ready" << std::endl; }
};

// 工厂函数：直接返回 unique_ptr，所有权转给调用方
std::unique_ptr<Product> createProduct() {
    return std::make_unique<Product>(); 
}
//std::unique_ptr<Product> createProduct()是一个函数 createProduct()，其类型为std::unique_ptr<Product>.,类似于int max（），其返回值是一个独占智能指针，这个指针独占Product对象。
```

返回值是调用函数std::make_unique<Product>()，这是模版函数,即工厂函数，调用这个函数生成指向Product对象的智能指针unique_ptr。
也可以这样写:
```cpp
auto createProduct() {
    return std::make_unique<Product>(); 
}

int main() {
    auto p = createProduct(); // p 接管对象
    p->show();
    // 函数结束，p 自动 delete Product
    return 0;
}
```

#### Shared_Ptr 共享指针
```cpp
#include <iostream>
#include <memory>
class Resource {
public:
    ~Resource() { std::cout << "Resource released" << std::endl; }
    void work() { std::cout << "Working..." << std::endl; }
};
void foo(std::shared_ptr<Resource> r) { 
    r->work(); 
    std::cout << "foo 中计数: " << r.use_count() << std::endl;
}
int main() {
auto r1 = std::make_shared<Resource>(); // 计数 1
//调用模版函数，生成智能指针指向Resource对象。
    auto r2 = r1;  // 计数 2（共享同一份资源）
foo(r2);    // 传入函数，计数临时变 3；值传递
std::cout << "main 中计数: " << r1.use_count() << std::endl; // 输出 2
std::cout << "main 中计数: " << r2.use_count() << std::endl; // 输出 2
//两条输出一样
//r1和r2共享一个内存；计数器是2；当传入函数的时候，值传递，拷贝构造，这时候函数中的计数是3；再回到main的时候函数中的指针释放了，这时候main中还是2；
    // 离开 main，r2 先销毁（计数变1），r1 再销毁（计数变0，析构 Resource）
    return 0;
}
```
#### weak_ptr弱指针，不增加计数
如果父子都用 shared_ptr，双方析构时你等我、我等你，内存永远不释放。必须把其中一方改成 weak_ptr。
```cpp
#include <iostream>
#include <memory>
class Child; // 前置声明
class Parent {
public:
    std::shared_ptr<Child> child; // 父亲拥有孩子（强引用）
    ~Parent() { std::cout << "Parent 释放" << std::endl; }
};
class Child {
public:
    std::weak_ptr<Parent> parent; // 孩子只观察父亲（弱引用）！关键点
    ~Child() { std::cout << "Child 释放" << std::endl; }
};
int main() {
auto dad = std::make_shared<Parent>();
//调用模版函数，生成共享指针，指向parent对象
auto son = std::make_shared<Child>();
//调用模版函数，生成共享指针，指向Child对象
    
    dad->child = son;   // dad 强引用 son
    son->parent = dad;  // son 弱引用 dad（若写成强引用，这里必泄漏！）
    // 离开作用域，两个对象都正常析构（打印 Parent释放 和 Child释放）
    return 0;
}
```
<div align="center">
  <img src="E:\Hugo_site\static\images\ptr.jpg" width="80%" alt="图1">
  <p><em>图 1</em></p>
</div>






