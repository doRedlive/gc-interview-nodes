# C++ 面试题与答案

> 共 86 题。按主题分类，答案直接放在题目后。

[返回题库总目录](面试题库总目录.md)

## 目录

| 模块 | 题号 |
| --- | --- |
| [一、版本特性与类型基础](#%E4%B8%80%E3%80%81%E7%89%88%E6%9C%AC%E7%89%B9%E6%80%A7%E4%B8%8E%E7%B1%BB%E5%9E%8B%E5%9F%BA%E7%A1%80) | 1–11 |
| [二、源码处理与链接](#%E4%BA%8C%E3%80%81%E6%BA%90%E7%A0%81%E5%A4%84%E7%90%86%E4%B8%8E%E9%93%BE%E6%8E%A5) | 12–14 |
| [三、内存与对象布局](#%E4%B8%89%E3%80%81%E5%86%85%E5%AD%98%E4%B8%8E%E5%AF%B9%E8%B1%A1%E5%B8%83%E5%B1%80) | 15–21 |
| [四、继承与多态](#%E5%9B%9B%E3%80%81%E7%BB%A7%E6%89%BF%E4%B8%8E%E5%A4%9A%E6%80%81) | 22–28 |
| [五、所有权与异常安全](#%E4%BA%94%E3%80%81%E6%89%80%E6%9C%89%E6%9D%83%E4%B8%8E%E5%BC%82%E5%B8%B8%E5%AE%89%E5%85%A8) | 29–34 |
| [六、值类别与移动语义](#%E5%85%AD%E3%80%81%E5%80%BC%E7%B1%BB%E5%88%AB%E4%B8%8E%E7%A7%BB%E5%8A%A8%E8%AF%AD%E4%B9%89) | 35–40 |
| [七、容器与视图](#%E4%B8%83%E3%80%81%E5%AE%B9%E5%99%A8%E4%B8%8E%E8%A7%86%E5%9B%BE) | 41–46 |
| [八、模板与可调用对象](#%E5%85%AB%E3%80%81%E6%A8%A1%E6%9D%BF%E4%B8%8E%E5%8F%AF%E8%B0%83%E7%94%A8%E5%AF%B9%E8%B1%A1) | 47–49 |
| [九、多线程与协程](#%E4%B9%9D%E3%80%81%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B8%8E%E5%8D%8F%E7%A8%8B) | 50–61 |
| [十、STL 高频补充](#%E5%8D%81%E3%80%81STL%20%E9%AB%98%E9%A2%91%E8%A1%A5%E5%85%85) | 62–67 |
| [十一、C++ 进阶追问](#%E5%8D%81%E4%B8%80%E3%80%81C%2B%2B%20%E8%BF%9B%E9%98%B6%E8%BF%BD%E9%97%AE) | 68–74 |
| [十二、设计模式与解耦](#%E5%8D%81%E4%BA%8C%E3%80%81%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B8%8E%E8%A7%A3%E8%80%A6) | 75–78 |
| [十三、面经高频补充](#%E5%8D%81%E4%B8%89%E3%80%81%E9%9D%A2%E7%BB%8F%E9%AB%98%E9%A2%91%E8%A1%A5%E5%85%85) | 79–86 |

## 一、版本特性与类型基础

### 1. C++11 的核心特性有哪些？

**答案：**

可以分五个层面回答：

- 语法与类型：`auto`、`decltype`、`nullptr`、范围 `for`、lambda、强类型枚举 `enum class`。
- 对象与资源：右值引用与移动语义、`unique_ptr`／`shared_ptr`／`weak_ptr`、委托构造与继承构造。
- 接口契约：`override` 显式标记重写、`final` 禁止重写或继承、`= default`／`= delete` 控制特殊成员函数。
- 常量与模板：`constexpr`、`static_assert`、可变参数模板、列表初始化与 `std::initializer_list`。
- 标准库：`thread`、`mutex`、`condition_variable`、`atomic`、`future`，以及 `array`、`tuple`、`unordered_map/set`、`regex`、`chrono`。

`nullptr` 的类型是 `std::nullptr_t`，解决了 `NULL`（本质是整数 0）在重载解析中混淆指针与整数的问题。

```cpp
enum class Color { Red, Green, Blue };
int* p = nullptr;
std::vector<int> values{1, 2, 3};
for (auto& value : values) value *= 2;
std::sort(values.begin(), values.end(),
          [](int a, int b) { return a > b; });

struct Owner {
    Owner() = default;
    Owner(const Owner&) = delete;
    Owner& operator=(const Owner&) = delete;
};
```

### 2. C++14 相比 C++11 补充了什么？

**答案：**

- 泛型 lambda：`[](auto x) { return x * x; }`。
- lambda 初始化捕获：`[p = std::move(value)]`，可以把只可移动的资源移入闭包。
- 函数返回类型推导：`auto add(int a, int b) { return a + b; }`，以及 `decltype(auto)`。
- 变量模板、二进制字面量 `0b1010`、数字分隔符 `1'000'000`。
- `constexpr` 函数放宽：允许局部变量、循环与分支。
- 标准库：`std::make_unique`、`std::shared_timed_mutex`、`[[deprecated]]`。

```cpp
auto square = [](auto x) { return x * x; };
auto add(int a, int b) { return a + b; }
template<class T> constexpr T pi = T(3.14159265358979);
int mask = 0b1010;
int count = 1'000'000;
auto value = std::make_unique<int>(42);
auto callback = [p = std::move(value)] { return *p; };
```

初始化捕获把资源移入闭包后，闭包对象本身也只可移动、不可复制。

### 3. C++17 的常见面试特性有哪些？

**答案：**

语言特性：结构化绑定、`if`／`switch` 初始化语句、`if constexpr`、折叠表达式、类模板实参推导（CTAD）、内联变量、嵌套命名空间。

标准库：`optional`、`variant`、`any`、`string_view`、`filesystem`、`scoped_lock`、`shared_mutex`。

```cpp
if (auto it = table.find(key); it != table.end()) {
    use(it->second);
}
auto [id, score] = std::make_pair(1, 2.0);
std::pair pair(1, 2.0); // 类模板实参推导
```

- `optional<T>`：表达"可能没有值"，替代输出参数和魔法值。
- `variant<A, B>`：类型安全的联合体，配合 `std::visit` 做静态分派。
- `if constexpr`：编译期裁剪模板分支，是替代 SFINAE 的首选手段。

### 4. C++20 的主要新特性有哪些？

**答案：**

| 特性 | 主要用途 | 说明 |
| --- | --- | --- |
| Concepts／`requires` | 约束模板参数 | 报错信息友好、约束可复用 |
| Modules | 模块化组织接口 | 减少头文件文本包含依赖，加快编译 |
| Coroutines | `co_await`／`co_yield`／`co_return` | 语言级挂起恢复机制，调度需库支持 |
| Ranges | 管道式视图与算法 | 惰性求值、操作可组合 |
| `std::format` | 类型安全格式化 | 替代 `sprintf` 与流拼接 |
| `<=>` | 三路比较 | `= default` 一次生成全套比较 |
| `std::span` | 连续内存的非拥有视图 | 配合数组／`vector` 传参 |
| `std::jthread` | 自动 join 的线程 | 内置协作式停止（`stop_token`） |

```cpp
template<class T>
concept Addable = requires(T a, T b) { a + b; };

template<Addable T>
auto add(T a, T b) { return a + b; }
```

### 5. auto、decltype 和 decltype(auto) 的推导规则是什么？

**答案：**

- `auto` 按模板实参推导规则：按值推导时忽略引用和顶层 cv 限定，数组、函数退化为指针；`auto&` 保留 const；`auto&&` 是转发引用，按引用折叠决定最终类型。
- `decltype`：对不加括号的名字直接给出声明类型；对一般表达式按值类别推导——左值给 `T&`，将亡值给 `T&&`，纯右值给 `T`。
- `decltype(auto)`：用 `decltype` 的规则做 `auto` 推导，用于函数返回类型精确转发。

```cpp
const int x = 1;
auto a = x;                // int
auto& b = x;               // const int&
auto&& c = x;              // const int&，引用折叠
auto&& d = 1;              // int&&，但具名表达式 d 是左值
const int* source = &x;
auto p = source;           // const int*
decltype(auto) e = x;      // const int
decltype(auto) f = (x);    // const int&，加括号后成为左值表达式
```

另外 `auto xs = {1, 2};` 会推导为 `std::initializer_list<int>`。

### 6. const、constexpr 和 static_assert 有什么区别？

**答案：**

- `const`：承诺不通过该名字修改，初始化值可以运行期确定，用于接口约束。
- `constexpr`：变量要求编译期常量初始化；函数表示具备编译期求值能力——在常量上下文中编译期执行，普通调用也可以运行期执行。
- `static_assert(常量表达式)`：编译期断言，用于验证模板约束、类型属性和布局假设。
- C++20 补充：`consteval` 声明立即函数，要求每次调用都产生编译期常量；`constinit` 保证静态变量静态初始化，规避静态初始化顺序问题，但变量本身可写。

### 7. lambda 有哪些捕获方式？

**答案：**

lambda 产生闭包对象，捕获即闭包的状态成员。`[x]` 按值捕获，`[&x]` 按引用捕获，`[=]`／`[&]` 是默认捕获；C++14 初始化捕获可把资源移动进闭包。

- 引用捕获不延长对象生命周期，异步回调用值捕获或按值捕获指针并管理所有权。
- 值捕获的副本默认只读，加 `mutable` 才能在闭包内修改，且修改的是副本、不影响外部变量。
- `[this]` 捕获的是 this 指针；C++17 的 `[*this]` 才是复制当前对象。
- 无捕获 lambda 可转换为函数指针；有捕获 lambda 只能用 `auto`、模板参数或 `std::function` 持有。

### 8. 四种显式类型转换如何选择？

**答案：**

| 写法 | 用途 | 失败方式 |
| --- | --- | --- |
| `static_cast` | 数值转换、上行转换、明确安全的转换 | 向下转换不检查实际类型，错误使用是未定义行为 |
| `dynamic_cast` | 多态类型的向下／交叉转换 | 指针失败返回 `nullptr`，引用失败抛 `std::bad_cast` |
| `const_cast` | 增删 cv 限定 | 修改原本为 const 的对象是未定义行为 |
| `reinterpret_cast` | 底层位模式重解释、指针reinterpretation | 不保证可安全解引用 |

选择原则：数值转换和安全上行用 `static_cast`；多态向下转换用 `dynamic_cast`（要求源类型是多态类型，有 RTTI 开销）；`const_cast` 仅用于适配旧接口；`reinterpret_cast` 只用于序列化、硬件寄存器访问等底层操作。

### 9. `static` 的几种用法及线程安全边界是什么？

**答案：**

- 局部静态变量：静态存储期，首次执行到声明处初始化，只初始化一次。
- 文件级静态函数／变量：内部链接，仅当前翻译单元可见。
- 静态数据成员：属于类本身，所有对象共享；C++17 可用 `inline static` 类内定义。
- 静态成员函数：没有 `this` 指针，不能是虚函数，不能访问非静态成员。

线程安全边界：C++11 起局部 static 变量的初始化由标准保证线程安全（并发首个初始化会同步），但这只针对初始化一次；对象创建后的并发读写仍需自行加锁或用原子操作。

### 10. 顶层 `const`、底层 `const` 与 const 成员函数如何区分？

**答案：**

```cpp
const int* p = nullptr;       // 底层 const：指向 const int，p 本身可改指向
int value = 1;
int* const q = &value;        // 顶层 const：q 本身不可改指向，可通过 q 修改 value
const int* const r = &value;  // 顶层 + 底层 const
```

顶层 const 修饰对象本身，底层 const 修饰指针／引用所指向的内容。拷贝时顶层 const 可以忽略，底层 const 必须双方一致。

const 成员函数：`this` 指针类型为 `const T*`，不能修改非 `mutable` 成员，只能被 const 对象和 const 引用／指针调用。`mutable` 成员可在 const 成员函数中修改，常用于缓存和锁，但并发访问仍需同步。

### 11. 引用与指针有什么区别？

**答案：**

- 引用必须初始化并绑定到一个已存在的对象，绑定后不可更改；指针可以空、可以改变指向、支持算术运算。
- 对引用赋值是修改被引用对象；指针赋值是改变指向。
- `sizeof(引用)` 得到被引用类型的大小，`sizeof(指针)` 得到指针自身大小（4 或 8 字节）。
- 引用没有"空引用"，但仍可能悬垂（绑定对象销毁后），生命周期分析与指针同样重要。

参数传递惯例：小对象按值传递；大对象只读输入用 `const T&`；需要修改用引用或指针；需要表达"可空"或"可重新指向"用指针。

## 二、源码处理与链接

### 12. C++ 源码如何变成可执行程序？

**答案：**

四个阶段（可用 `gcc -E/-S/-c` 分别观察中间产物）：

1. 预处理（Preprocessing）：展开头文件与宏、处理条件编译，输出 `.i` 文件。
2. 编译（Compilation）：词法、语法、语义分析，优化后生成汇编代码 `.s`。
3. 汇编（Assembly）：将汇编翻译为机器指令，生成目标文件 `.o`／`.obj`。
4. 链接（Linking）：符号解析与重定位，合并各目标文件和库，生成可执行文件或库。

### 13. 静态链接与动态链接有哪些取舍？

**答案：**

| 维度 | 静态链接 | 动态链接 |
| --- | --- | --- |
| 时机 | 链接期把库代码复制进可执行文件 | 运行期加载 `.dll`／`.so` |
| 体积 | 可执行文件大 | 可执行文件小 |
| 部署 | 单文件、无运行时依赖 | 需管理依赖库的版本与搜索路径 |
| 升级 | 库更新需重新链接发布 | ABI 兼容前提下直接替换库文件 |
| 内存 | 每个进程各有一份副本 | 多进程共享同一份代码页 |
| 启动 | 无额外加载解析 | 加载器需定位、重定位符号 |

选择依据：追求部署简单、隔离性好用静态链接；追求体积小、共享内存、库独立升级用动态链接。

### 14. `extern "C"` 如何支持 C 与 C++ 互操作？

**答案：**

`extern "C"` 指定 C 语言链接：关闭 C++ 名字修饰（name mangling），让符号名保持 C 的平坦形式，从而与 C 代码、动态库和其他语言互调。

```cpp
#ifdef __cplusplus
extern "C" {
#endif

int library_init(void);
void library_shutdown(void);

#ifdef __cplusplus
}
#endif
```

跨语言接口的惯例：只暴露普通函数和不透明句柄，C++ 侧用包装层管理类与资源。跨 DLL 边界还需约定调用约定、数据布局、内存释放方，且不能让 C++ 异常穿越 C 边界。

## 三、内存与对象布局

### 15. C++ 程序的内存区域有哪些？

**答案：**

以典型 Linux 进程地址空间为例（低地址到高地址）：

| 区域 | 内容 |
| --- | --- |
| Text 代码段 | 机器指令、只读常量，只读可执行 |
| Data 数据段 | 已初始化的全局／静态变量 |
| BSS | 未初始化的全局／静态变量，加载时清零，不占可执行文件体积 |
| Heap 堆 | `new`／`malloc` 动态分配，向高地址增长 |
| MMAP 区 | 共享库、内存映射文件、大块分配 |
| Stack 栈 | 函数调用帧、局部变量，向低地址增长，通常几 MB |

语言层面更严谨的划分是按存储期：自动存储期（栈）、静态存储期（全局／静态变量）、动态存储期（`new`／`malloc`）、线程存储期（`thread_local`）。

### 16. 什么是内存对齐？为什么要对齐？

**答案：**

内存对齐：把数据放在其对齐系数整数倍的地址上。结构体会按成员对齐要求插入填充，总大小补齐到最大对齐系数的整数倍。

```cpp
struct A {
    char c;  // 偏移 0
             // 填充 3 字节
    int i;   // 偏移 4，对齐 4
};           // 总大小 8，不是 5
```

对齐的三个原因：

1. 硬件要求：部分架构（ARM、SPARC）非对齐访问直接触发总线错误，x86 允许但性能下降。
2. 性能：对齐的数据一次总线事务即可读取；跨缓存行的访问需要两次内存访问。
3. 原子性：对齐的字长访问在部分平台可用单条指令保证原子性。

工具：`alignof(T)` 查询对齐要求，`alignas(N)` 指定对齐，`#pragma pack` 压缩填充。

### 17. 指针、空类和含虚函数的类一般占多少内存？

**答案：**

- 指针：32 位程序 4 字节，64 位程序 8 字节。
- 空类：`sizeof` 为 1，保证每个对象有独立地址；空基类优化（EBO）下作为基类不占空间。
- 只含虚函数的类：一个 vptr 的大小（64 位下 8 字节）。
- 有成员和继承时：基类子对象 + 成员 + 填充对齐；静态成员属于类而非实例，不计入 `sizeof`。

### 18. `malloc/free` 与 `new/delete` 有什么区别？

**答案：**

| 维度 | `malloc`／`free` | `new`／`delete` |
| --- | --- | --- |
| 本质 | 分配／释放原始内存 | 分配内存 + 调用构造／析构 |
| 类型 | 按字节分配，返回 `void*`，需强转 | 按类型工作，返回具体类型指针 |
| 失败 | 返回 `NULL` | 抛 `std::bad_alloc`（nothrow 版本返回空指针） |
| 来源 | C 库函数 | C++ 运算符，可重载 |
| 配对 | `malloc` ↔ `free` | `new` ↔ `delete`，`new[]` ↔ `delete[]` |

补充：`operator new` 是可重载的分配函数，`new` 表达式 = `operator new` 分配 + 构造函数调用。混用两个家族（如 `malloc` 的指针交给 `delete`）是未定义行为。实践准则：优先使用容器与智能指针。

### 19. 栈上缓冲区与 `alloca` 有什么风险？

**答案：**

栈上固定数组具有自动存储期，函数返回自动回收，高效但大小必须编译期确定。

`alloca` 是平台扩展（非标准 C++），在当前栈帧上动态分配，函数返回时自动释放。风险：

- 栈空间有限（通常 1–8 MB），分配过大会栈溢出。
- 循环中多次 `alloca` 会累积占用，无法在迭代间释放。
- 不能返回指向该区域的指针（返回后立即失效）。

可变大小数据应使用 `std::vector` 等拥有型容器。

### 20. C++ 内存泄漏有哪些原因？如何检查和处理？

**答案：**

常见原因：

1. `new` 后遗漏 `delete`，异常或提前返回跳过清理路径。
2. `shared_ptr` 循环引用，强计数无法归零。
3. 资源所有权不清晰，多方都以为对方负责释放。

排查工具：Visual Studio 内存快照、Valgrind、ASan／LeakSanitizer，查看存活分配及分配调用栈；结合业务场景反复进出同一功能，观察资源是否单调增长。

解决方案：RAII 全面接管——裸资源用智能指针，循环引用中非拥有方向改 `weak_ptr`；注意内存泄漏、悬垂访问、重复释放是三类不同的错误。

### 21. placement new 是什么？内存池怎么使用它？

**答案：**

placement new 在已有内存上构造对象，不分配内存：`new (ptr) T(args...)`。调用方负责保证空间大小、对齐满足要求，并必须显式调用析构函数、按内存来源回收。

```cpp
#include <new>
struct Widget {
    explicit Widget(int n) : value(n) {}
    int value;
};

alignas(Widget) unsigned char buffer[sizeof(Widget)];
Widget* object = new (buffer) Widget(42);
object->~Widget(); // 显式析构；buffer 是栈内存，不能 delete
```

内存池用法：批量申请大块内存，用 placement new 在块内构造对象，归还时显式析构并标记块可复用。收益是减少频繁 `malloc` 的开销和外部碎片；代价是峰值内存驻留、对齐和生命周期管理复杂度。

## 四、继承与多态

### 22. 静态多态与动态多态有什么区别？

**答案：**

- 静态多态（编译期绑定）：函数重载、模板、CRTP。调用目标在编译期确定，可内联优化，零运行时开销；代价是模板实例化膨胀、接口与实现耦合。
- 动态多态（运行期绑定）：虚函数 + 继承。通过基类指针／引用调用时按对象实际动态类型分派；支持运行期扩展和异构容器，代价是间接调用、难以内联。

注意：派生对象按值拷贝给基类对象会发生对象切片，丢失派生部分，多态失效——多态必须通过指针或引用。

### 23. 虚函数是怎么实现多态的？

**答案：**

实现机制（主流编译器一致）：

1. 每个含虚函数的类有一张虚函数表 vtable（函数指针数组），编译期生成，同类型对象共享。
2. 每个多态对象头部有一个虚表指针 vptr，在构造函数中由编译器插入代码设置。
3. 虚调用 `ptr->f()` 编译为：取对象 vptr → 查 vtable 中 f 的槽位 → 间接调用。派生类重写虚函数即替换对应槽位，因此调用落到实际类型的实现。

多继承下对象可能有多个 vptr，分别指向各基类子对象的虚表；具体布局由 ABI 决定（Itanium ABI 与 MSVC 不同）。

### 24. 多继承下有几个虚表指针？为什么基类指针地址会变化？

**答案：**

多继承下，每个多态基类子对象各有一个 vptr，因此对象内可能有多个虚表指针。

地址变化的原因：`Derived*` 转 `Base2*` 时，指针必须从对象起始处偏移到 Base2 子对象的位置，所以 `(void*)d != (void*)base2` 可能成立。

虚调用时通过 thunk（跳板代码）调整 this 指针，把基类子对象地址还原为完整对象地址后再进入派生类实现。通过基类指针 delete 时，虚析构同样经 thunk 定位到完整对象正确释放。

### 25. 虚继承如何解决菱形继承？构造顺序是什么？

**答案：**

菱形继承 `D : B, C`，`B`、`C` 都继承 `A` 时，D 中含有两份 A 子对象，访问 A 成员二义。将 B、C 对 A 的继承声明为 `virtual` 后，D 中只保留一份共享的 A 子对象。

```cpp
struct A { int value = 0; };
struct B : virtual A {};
struct C : virtual A {};
struct D : B, C {};
```

构造顺序：由最派生类 D 直接负责初始化虚基类 A；顺序为——虚基类 → 按声明顺序的直接基类 → 按声明顺序的成员 → 构造函数体；析构严格相反。

实现上通过 vbptr／vbtable（MSVC）或 vtable 中的偏移（Itanium）定位虚基类，带来一次间接寻址开销。

### 26. 构造和析构函数中调用虚函数会怎样？

**答案：**

不会崩溃，但失去多态：构造／析构期间对象的虚调用只分派到"当前正在构造／析构的那一层"。

原因：Base 构造时，vptr 先指向 Base 的 vtable，此时派生部分还不存在；进入 Derived 构造阶段后才把 vptr 换成 Derived 的表。析构按相反顺序回退。因此在基类构造／析构中调用虚函数，执行的是基类版本，无法触达派生类的重写，更不能调用纯虚函数（链接错误或纯虚调用崩溃）。

设计上：不要在基类构造／析构中依赖派生类的虚函数行为。

### 27. 为什么构造函数不能是虚函数？析构函数什么时候必须是虚函数？

**答案：**

构造函数不能虚：虚调用依赖对象中已设置好的 vptr 和 vtable，而 vptr 恰恰是在构造过程中设置的——调用构造函数前对象尚不存在，没有动态类型可供分派；创建哪种对象由调用处静态决定。

基类析构必须是虚：只要对象会通过基类指针 `delete`（如 `unique_ptr<Base>` 持有派生对象）。虚析构保证从最派生类析构开始逐层向上执行；非虚析构下经基类指针删除派生对象是未定义行为，典型表现为派生类资源泄漏。若基类明确禁止多态删除，可将非虚析构设为 `protected`。

补充：纯虚函数可以提供类外定义；纯虚析构即使 `= 0` 也必须提供定义。静态成员函数、构造函数、自由函数都不能是虚函数。

### 28. 虚函数有哪些开销？什么时候考虑其他多态方式？

**答案：**

开销：

1. 阻止内联——这是核心代价：单次虚调用只多两次内存读（vptr → vtable → 函数地址），分支预测器对稳定虚调用预测准确率很高，直接开销几乎可忽略；真正的损失是调用目标编译期未知导致无法内联，而内联是常量传播、死代码消除、跨函数向量化的入口，这条优化链被虚调用断掉。小函数（getter）虚化后开销放大十倍是常态。
2. 空间：每个对象一个 vptr（多继承更多），vtable 与 RTTI 元数据常驻内存。
3. 缓存：调用路径涉及两次间接访存，可能增加 cache miss。
4. ABI：vtable 布局参与二进制兼容，跨动态库场景下新增／调整虚函数会破坏 ABI，导致跨模块崩溃。

替代方案：模板／CRTP（编译期分派、可内联）、`std::variant` + `visit`（类型集合封闭时的高效分派）、函数指针／`std::function`（回调注入）、手写类型擦除。编译器在能证明调用目标时会去虚拟化（devirtualization）为直接调用。原则：先测量热点，抽象层接口的虚函数开销通常可接受，热路径再考虑替代。

## 五、所有权与异常安全

### 29. RAII 是什么？有什么好处？

**答案：**

RAII（Resource Acquisition Is Initialization）：把资源生命周期绑定到对象生命周期——构造时获取资源，析构时释放资源。对象无论正常返回、提前 return 还是异常栈展开都会析构，资源必然被释放。

好处：

1. 自动清理，杜绝提前返回和异常路径的泄漏。
2. 所有权显式化：看类型就知道谁负责释放（`unique_ptr` 独占、`shared_ptr` 共享）。
3. 异常安全的基础设施。

典型例子：`unique_ptr` 管内存、`lock_guard` 管锁、`fstream` 管文件句柄。RAII 不限于内存，GPU 资源、句柄、事务同样适用。

### 30. unique_ptr、shared_ptr 和 weak_ptr 的实现与生命周期有什么区别？

**答案：**

- `unique_ptr`：独占所有权，大小等于裸指针（无状态删除器时），只能移动不能拷贝；移动后源指针置空。零开销抽象。
- `shared_ptr`：共享所有权。对象指针 + 控制块指针；控制块含强引用计数、弱引用计数、删除器、分配器。拷贝强计数原子加一，析构原子减一，最后一个强引用析构对象。
- `weak_ptr`：只增加弱计数，不延长对象生命周期；`lock()` 原子地尝试提升为 `shared_ptr`，对象已销毁则返回空。用于打破 `shared_ptr` 循环引用、缓存观察。

生命周期规则：强计数归零 → 销毁对象；强弱计数都归零 → 释放控制块。分两步的原因：`weak_ptr::lock()`／`use_count()` 需要读写控制块，若强计数归零时就释放控制块，剩余 weak_ptr 全是悬垂指针，所以控制块必须活到所有观察者结束。

`make_shared` 把对象和控制块合并为一次分配，更高效且异常安全；副作用是只要还有 weak_ptr 存活，已析构对象占用的那块内存也无法归还，大对象 + 长寿 weak_ptr 场景需要注意。

### 31. shared_ptr 的引用计数如何保证线程安全？为什么对象本身仍可能数据竞争？

**答案：**

三层模型回答（口诀：计数安全、变量不安全、对象不安全）：

1. 控制块引用计数——安全：强／弱计数用原子操作维护，多个线程同时拷贝／析构各自持有的、指向同一对象的 `shared_ptr` 副本，计数不会丢失更新。
2. 同一个 `shared_ptr` 变量——不安全：一个线程写（赋值、reset）、另一个线程读同一变量是数据竞争，需加锁或使用 `std::atomic<std::shared_ptr<T>>`（C++20）。
3. 托管对象本身——不安全：计数只管何时析构对象，不保护其成员；多线程并发读写 `ptr->member` 且至少一方写，必须另行加锁或用原子字段。

补充：最后一个引用在哪个线程归零，对象就在哪个线程析构，析构线程需在设计时明确。

### 32. 深拷贝和浅拷贝有什么区别？

**答案：**

浅拷贝逐成员复制值：指针成员只复制地址，两个对象指向同一块资源，析构时重复释放。深拷贝为指针成员另行分配内存并复制内容，两个对象完全独立。

C++ 默认拷贝构造／赋值是逐成员拷贝：值语义成员（`string`、`vector`）自动执行各自的深拷贝，`shared_ptr` 拷贝共享所有权，只有裸指针成员才是浅拷贝风险点。

管理裸资源的类必须三选一：禁止拷贝（`= delete`）、实现深拷贝、或实现移动语义转移所有权——即三／五法则。

### 33. 三法则、五法则和零法则是什么？

**答案：**

- 三法则：拥有原始资源的类，析构函数、拷贝构造、拷贝赋值三个必须一起考虑——需要自定义其中一个，通常三个都需要。
- 五法则：C++11 起加入移动构造、移动赋值，五个特殊成员函数一起设计，保持所有权语义一致。
- 零法则：优先把资源交给 `vector`、`string`、智能指针等 RAII 成员管理，业务类不手写任何特殊成员函数，编译器生成的默认版本即正确。

注意隐式生成规则：用户声明了析构函数，移动构造／赋值不再隐式生成；声明了移动操作，拷贝操作被删除。`= default` 后仍可能因成员不可拷贝而被定义为删除。

### 34. C++ 异常处理机制、异常安全等级和 noexcept 是什么？

**答案：**

异常机制：`throw` 抛出后沿调用栈向上寻找匹配的 catch；栈展开（stack unwinding）过程中按逆构造顺序析构已完成构造的局部对象，RAII 资源因此被释放。

异常安全四个等级：

1. 不抛保证（noexcept）：绝不抛出。
2. 强保证：失败后回滚到调用前状态（copy-and-swap 是常用手法）。
3. 基本保证：失败后状态合法、无泄漏，但具体值不保证。
4. 无保证。

`noexcept`：承诺函数不抛异常，违反时调用 `std::terminate`。移动构造／移动赋值确实不抛时应标记 `noexcept`——容器扩容等场景据此选择移动而非拷贝；析构函数默认 noexcept，不应向外抛异常。

## 六、值类别与移动语义

### 35. 左值、右值和右值引用有什么区别？

**答案：**

C++11 值类别三分：

- 左值（lvalue）：有身份、可取地址的表达式，如变量名 `x`。
- 将亡值（xvalue）：资源可被转移的对象，如 `std::move(x)` 的结果。
- 纯右值（prvalue）：用于初始化的字面量和临时值，如 `42`、返回非引用的函数调用。

组合关系：glvalue = lvalue + xvalue；rvalue = prvalue + xvalue。

右值引用 `T&&` 是类型，只能绑定右值，表示"该对象的资源可以被搬走"；值类别是表达式的属性——具名的右值引用变量本身是左值。

```cpp
int&& reference = 42;
// reference 声明类型是 int&&，但作为表达式是左值
auto&& again = reference; // int&，引用折叠
```

### 36. `i++` 与 `++i` 有什么区别？

**答案：**

- 前置 `++i`：先自增，再返回对象自身的引用（左值），不产生副本。
- 后置 `i++`：先记录旧值副本，自增，再返回旧值副本（纯右值）。

对内建整数，两者生成的代码经优化后无差别；对迭代器等类类型，后置版本需要构造并返回副本。因此不需要旧值时（尤其是遍历容器）惯用前置 `++i`。

### 37. std::move 的原理和使用场景是什么？

**答案：**

`std::move` 是一次到右值引用的 `static_cast` 转换，本身不移动任何资源，只是把表达式标记为"可被移动"，使重载决议选择移动版本。

使用场景：转移 `unique_ptr` 所有权、把不再使用的对象移入容器、移动构造／赋值中转交成员资源。

注意：

1. 不移除 `const`：对 const 对象 `move` 后仍匹配拷贝重载。
2. 移动后的标准库对象处于"有效但未指定"状态，可以安全析构或重新赋值。
3. 返回局部对象直接 `return result;`，不要 `return std::move(result);`——多余的 move 会禁用 NRVO。

### 38. std::move、std::forward 和右值引用的本质区别是什么？

**答案：**

- 右值引用 `T&&` 是引用类型：绑定右值、表达"资源可转移"，用于区分移动／拷贝重载。
- `std::move(x)`：无条件转换为右值引用，强制走移动重载。
- `std::forward<T>(x)`：条件转换——`T` 推导为 `U&`（传入左值）时转发为左值，`T` 为 `U`（传入右值）时转发为右值，完整保留调用方的值类别，专用于转发引用参数的透传。

一句话：move 是强制转右值，forward 是还原原值类别。

```cpp
template<class T>
void relay(T&& value) {
    consume(std::forward<T>(value));
}
```

原理是引用折叠：`& &`、`& &&`、`&& &` 都折叠为 `&`，只有 `&& &&` 得 `&&`；左值传入时 `T` 推导为 `U&`，右值传入时 `T` 为 `U`。

### 39. RVO 和 NRVO 是什么？

**答案：**

- RVO（返回值优化）：`return Widget{};` 这类返回纯右值时，C++17 起强制拷贝消除——直接在调用方的存储上构造对象，零拷贝零移动。
- NRVO（具名返回值优化）：`return result;` 返回具名局部变量时，编译器可以把该局部对象直接构造在返回存储上，消除拷贝／移动；这是可选优化，主流编译器均支持。

```cpp
struct Widget {};
Widget direct() { return Widget{}; }  // C++17 强制消除
Widget named() {
    Widget result;
    return result;            // NRVO 候选
}
Widget discouraged() {
    Widget result;
    return std::move(result); // 禁用 NRVO，且按右值处理
}
```

不适用 NRVO 的情形：返回函数参数、多分支返回不同局部变量。结论：返回局部对象直接 `return` 名字，不要加 `std::move`。

### 40. 移动构造为什么常声明 noexcept？

**答案：**

核心场景是 `vector` 扩容：把元素搬到新存储时，若移动构造可能抛异常，搬到一半抛出会破坏已搬元素、无法回滚，容器无法维持强异常保证。因此标准规定：移动构造非 noexcept 且类型可拷贝时，扩容使用拷贝；标记 `noexcept` 后容器才放心使用移动，性能显著更好。

补充：若类型只可移动（不可拷贝），容器只能用移动，此时若移动抛异常，强保证不再成立。可用 `std::is_nothrow_move_constructible_v<T>` 在编译期检测该属性；成员可能抛异常时可写条件 noexcept。

## 七、容器与视图

### 41. STL 包含什么？常见容器如何选择？

**答案：**

传统 STL 六大组件：容器、算法、迭代器、函数对象、适配器、分配器。

| 容器 | 特点／复杂度 | 适用场景 |
| --- | --- | --- |
| `array<T, N>` | 固定长度连续存储，随机访问 O(1) | 大小编译期已知 |
| `vector<T>` | 连续存储，尾插摊还 O(1)，中间插删 O(n) | 默认序列容器 |
| `deque<T>` | 分段连续，随机访问 O(1)，两端插删 O(1) | 双端操作 |
| `list<T>` | 双向链表，已知位置插删 O(1)，查找 O(n) | 频繁中间插删、需要稳定节点地址 |
| `map`／`set` | 红黑树，查找插删 O(log n)，有序 | 有序遍历、范围查询 |
| `unordered_map`／`unordered_set` | 哈希表，平均 O(1)，最坏 O(n) | 无序键值查询的默认选择 |
| `stack`／`queue`／`priority_queue` | 容器适配器 | LIFO／FIFO／优先级 |

选择建议：默认 `vector`（缓存局部性好，实测常快于理论复杂度更优的节点容器）；频繁两端操作用 `deque`；需要有序和范围查询用 `map`，否则用 `unordered_map`。

### 42. vector 如何扩容和缩容？

**答案：**

`size()` 是元素数量，`capacity()` 是已分配存储可容纳的元素数。扩容流程：申请更大内存（增长倍率由实现决定，GCC/Clang 2 倍、MSVC 1.5 倍）→ 把旧元素搬移到新存储（移动构造若 noexcept，否则拷贝）→ 析构旧元素、释放旧内存。扩容后所有迭代器、指针、引用失效。

- `reserve(n)`：预留容量，不改变 size，可避免多次扩容。
- `clear()`／`pop_back()`：只销毁元素，不释放内存，capacity 不变。
- `shrink_to_fit()`：非强制的缩容请求。
- 释放内存的可靠做法：`std::vector<T>().swap(v);` 与空容器交换。

### 43. 哈希表的负载因子和 rehash 是什么？

**答案：**

负载因子 = `size() / bucket_count()`，衡量哈希表的拥挤程度。超过 `max_load_factor`（默认 1.0）时触发 rehash：增加桶数（通常翻倍），逐个重新计算哈希并把元素分配到新桶。

影响：

- rehash 是 O(n) 操作且需重算哈希，扩容瞬间产生延迟尖刺，实时场景应预留容量。
- rehash 使所有迭代器失效；指向元素的指针和引用保持有效。
- 相等键必须哈希相同，性能依赖哈希函数的质量与分布。

预先 `reserve(n)` 可一次性备足桶数，避免渐进式 rehash。

### 44. 容器操作后哪些迭代器、指针和引用会失效？

**答案：**

| 容器与操作 | 失效规则 |
| --- | --- |
| vector 重分配 | 全部迭代器、指针、引用失效 |
| vector 尾插未重分配 | 仅旧 `end()` 失效，元素引用有效 |
| vector 中间插入 | 插入点及之后的迭代器、指针、引用失效 |
| vector erase | 删除点及之后失效 |
| deque 两端插入 | 所有迭代器失效，元素引用／指针仍有效 |
| deque 中间插入／删除 | 所有迭代器及引用失效 |
| list／map／set 插入 | 不失效任何已有元素 |
| list／map／set 删除 | 仅被删元素失效 |
| unordered 容器 rehash | 所有迭代器失效，元素引用仍有效 |
| unordered 容器 erase | 仅被删元素失效 |

安全的边遍历边删除写法：

```cpp
for (auto it = values.begin(); it != values.end(); ) {
    if (*it < 0) it = values.erase(it);
    else ++it;
}
// vector 更优的 erase-remove 惯用法
values.erase(std::remove_if(values.begin(), values.end(),
                            [](int x) { return x < 0; }),
              values.end());
// C++20 可直接用 std::erase_if(values, pred);
```

### 45. push_back 和 emplace_back 有什么区别？

**答案：**

- `push_back`：接收一个已存在的对象，拷贝或移动进容器。
- `emplace_back`：转发构造参数，直接在容器内存上原地构造，省去临时对象。

```cpp
std::vector<std::pair<int, std::string>> items;
items.push_back(std::make_pair(1, "hello")); // 构造临时 pair 再移动
items.emplace_back(1, "hello");              // 原地构造
```

传已有对象时两者等价。emplace 版本仍受扩容成本影响，且直接初始化可匹配 explicit 构造函数，注意参数含义。`map` 的 `emplace`／`try_emplace` 同理。

### 46. string_view 和 span 是什么？适合什么场景？

**答案：**

- `string_view`（C++17）：字符串的非拥有只读视图，{指针 + 长度}，接受 `string`、字面量、字符数组而不发生拷贝，传参替代 `const string&` 避免临时 string 构造。
- `span`（C++20）：连续内存序列的非拥有视图，统一接受 C 数组、`vector`、`std::array`，配合范围 for 和算法使用。

```cpp
int sum(std::span<const int> values) {
    int result = 0;
    for (int value : values) result += value;
    return result;
}
```

注意：视图不拥有数据、不延长底层生命周期——不要保存视图超过数据存活期，不要返回指向局部数据的视图；`string_view` 不保证以 `\0` 结尾，不能直接传给 C 接口。

## 八、模板与可调用对象

### 47. 模板全特化、偏特化和重载有什么区别？

**答案：**

- 全特化：为某个具体的模板实参组合提供专门定义，如 `Traits<void>`。
- 偏特化：为一类参数模式提供专门定义，如 `Traits<T*>` 匹配所有指针；只有类模板和变量模板可以偏特化。
- 函数模板不能偏特化，用重载或 `if constexpr`／Constraints 替代。

```cpp
template<class T> struct Traits { static constexpr bool pointer = false; };
template<class T> struct Traits<T*> { static constexpr bool pointer = true; };  // 偏特化
template<> struct Traits<void> { static constexpr bool pointer = false; };      // 全特化
```

选择顺序：主模板匹配前，编译器先选最特化的特化版本；函数重载则在重载决议中选最佳匹配。接口行为只依赖少量条件时，`if constexpr` 或 Concepts 比多层特化更清晰。

### 48. 可变参数模板与折叠表达式如何展开？

**答案：**

`template<class... Args>` 声明参数包，展开方式有递归和折叠表达式（C++17）两种：

```cpp
template<class... Args>
auto sum(Args... values) {
    return (0 + ... + values); // 二元左折叠，空包返回 0
}

template<class... Args>
void print_all(const Args&... values) {
    ((std::cout << values << ' '), ...); // 一元右折叠的逗号形式
}
```

四种折叠形式：一元右折叠 `(pack op ...)` = `a op (b op c)`；一元左折叠 `(... op pack)` = `(a op b) op c`；加上初始值即为二元形式。空包时只有逗号、`&&`、`||` 的折叠合法。

典型应用：`make_unique`／`make_shared` 的完美转发、`emplace` 系列、tuple 实现。注意左右结合方向会影响浮点误差和求值行为。

### 49. 函数指针和 std::function 有什么区别？

**答案：**

- 函数指针：只指向静态函数入口，不携带状态；调用经间接跳转；大小固定、可比较。
- lambda／函数对象 + 模板参数：保留具体类型，调用可内联，性能最优，但不同类型无法统一存储。
- `std::function<R(Args...)>`：通过类型擦除统一持有任意签名匹配的可调用对象（含带捕获的 lambda）；代价是可能堆分配、间接调用、难以内联；C++23 前不能持有仅可移动的闭包（如捕获 `unique_ptr`）。

选择：模板参数（`template<class F>`）优先，性能最好；需要运行期注册回调、存储异构可调用对象时用 `std::function`；纯 C 接口用函数指针 + `void*` 上下文。

## 九、多线程与协程

### 50. 进程与线程有什么区别？

**答案：**

- 进程是资源分配的基本单位：独立地址空间、文件描述符等资源相互隔离。
- 线程是 CPU 调度的基本单位：拥有独立的栈、寄存器上下文、程序计数器；同进程内所有线程共享地址空间和资源。

通信与同步：进程间用 IPC（管道、共享内存、消息队列、socket）；线程间直接读写共享变量，配合锁、原子变量等同步原语。

隔离性：进程崩溃不影响其他进程；线程的非法访问可能危害整个进程。

切换开销：线程切换不必切换页表和地址空间，上下文切换比进程更轻；同进程线程间通信无需内核拷贝。

### 51. std::thread 如何创建、等待和分离？

**答案：**

```cpp
void worker(int id) { /* 处理任务 */ }
std::thread thread(worker, 42);
thread.join();
```

- 参数按值拷贝传入；要传引用必须用 `std::ref`／`std::cref`，并保证对象存活到线程结束。
- `join()`：阻塞等待线程结束，之后 thread 对象不再关联。
- `detach()`：分离执行，线程后台独立运行，thread 对象不再可控——必须保证线程访问的数据生命周期覆盖其执行期。

关键规则：析构时仍 `joinable()` 的 `std::thread` 会调用 `std::terminate`，因此必须 join 或 detach；异常路径下用 RAII 包装保证 join。C++20 的 `jthread` 析构时自动请求停止并 join，工作函数通过 `stop_token` 配合停止。

线程内未捕获异常不会传播到创建线程，须线程内捕获或经 `promise` 传递。

### 52. mutex、lock_guard、unique_lock 和 scoped_lock 有什么区别？

**答案：**

| 类型 | 用途 |
| --- | --- |
| `mutex` | 基础互斥锁 |
| `recursive_mutex` | 同线程可重复加锁，解锁次数须匹配 |
| `timed_mutex` | 支持 `try_lock_for` 限时尝试 |
| `shared_mutex`（C++17） | 读写锁，读多写少场景 |
| `lock_guard` | 作用域 RAII 加锁，不可移动、不可中途解锁 |
| `unique_lock` | 可移动、可延迟加锁、可中途 unlock，条件变量必需 |
| `scoped_lock`（C++17） | RAII 管理多个锁，内部用 `std::lock` 死锁避免算法 |
| `shared_lock` | RAII 共享模式加锁，配合 `shared_mutex` |

选择：普通作用域加锁用 `lock_guard`／`scoped_lock`；配合条件变量或需要中途解锁用 `unique_lock`；读多写少用 `shared_mutex` + `shared_lock`。

### 53. 死锁怎么产生？如何避免？

**答案：**

死锁四个必要条件：互斥、占有且等待、不可剥夺、循环等待——破坏任意一个即可预防。

工程手段：

1. 统一加锁顺序（按锁的地址或编号排序），破坏循环等待。
2. 多锁用 `std::lock`／`std::scoped_lock` 一次性获取，内部死锁避免算法。
3. `try_lock` + 限时退避，避免永久等待。
4. 不持锁调用外部回调、不在持锁时等待其他任务。

区分三个概念：死锁是互相等待无法前进；活锁是不断重试却无法推进；饥饿是某方长期得不到资源。排查工具：调试器查看各线程调用栈、等待链。

### 54. 条件变量如何实现生产者—消费者？

**答案：**

核心两点：条件变量存在虚假唤醒，必须用谓词循环等待；共享状态（队列、停止标志）由同一把 mutex 保护。

```cpp
std::queue<int> queue;
std::mutex mutex;
std::condition_variable changed;
bool stopped = false; // 读写都持有 mutex

void produce(int value) {
    {
        std::lock_guard<std::mutex> lock(mutex);
        queue.push(value);
    }
    changed.notify_one();
}

bool consume(int& value) {
    std::unique_lock<std::mutex> lock(mutex);
    changed.wait(lock, [] { return stopped || !queue.empty(); });
    if (queue.empty()) return false;
    value = queue.front();
    queue.pop();
    return true;
}

void stop() {
    {
        std::lock_guard<std::mutex> lock(mutex);
        stopped = true;
    }
    changed.notify_all();
}
```

要点：`wait(lock, pred)` 等价于 `while (!pred()) wait(lock);`；停止流程先置标志再 `notify_all` 唤醒全部消费者；通知前释放锁可减少被唤醒线程立即阻塞的开销。

### 55. atomic、内存序和 volatile 有什么区别？

**答案：**

`std::atomic` 提供不可分割的读写及读改写操作，并按指定内存序约束指令重排。注意 `fetch_add` 是单次原子读改写，先 `load` 再 `store` 的组合不具原子性。

内存序三档：

- `relaxed`：只保证本操作的原子性，无跨对象同步语义，适合计数器。
- `release`／`acquire`：写方 release、读方 acquire 配对，建立 happens-before，写方之前的写对读方可见——典型发布／获取模式。
- `seq_cst`：默认序，额外保证所有 seq_cst 操作存在单一全序，最直观也最贵。

```cpp
int payload = 0;
std::atomic<bool> ready{false};
// 生产者：
payload = 42;
ready.store(true, std::memory_order_release);
// 消费者：
// if (ready.load(std::memory_order_acquire)) use(payload); // 可见 payload=42
```

`volatile`：只保证访问不被优化掉、按程序顺序执行可观察读写，用于内存映射 IO；不提供原子性、不建立线程同步，不能替代 atomic。

### 56. future、promise、async 与 packaged_task 如何协作？

**答案：**

- `promise<T>`／`future<T>`：一对一的结果通道。promise 端 `set_value`／`set_exception`，future 端 `get()` 阻塞获取，异常在 get 时重新抛出。
- `packaged_task`：包装可调用对象，执行时把返回值或异常写入共享状态，可投递给线程池执行。
- `std::async`：按策略在线程上或延迟执行任务，直接返回持有结果的 future。

```cpp
auto result = std::async(std::launch::async, [] { return 42; });
int value = result.get();
```

注意：

1. `future::get()` 只能调用一次；多方读取用 `shared_future`。
2. `std::launch::async` 强制新线程；`deferred` 延迟到 get 时执行；默认策略由实现二选一。
3. `std::async` 返回的 future 析构时会阻塞等待任务完成——不要在需要异步放手的场景误用。

### 57. thread_local 有什么作用和使用场景？

**答案：**

`thread_local` 声明线程存储期变量：每个线程拥有独立实例，线程启动时构造、线程退出时析构，天然避免共享竞争。

典型场景：每线程的随机数引擎、缓存、错误码（类似 `errno`）、临时工作区。

注意：线程池中线程跨任务复用，thread_local 状态需要在任务间重置；线程局部变量内部若引用共享数据，访问仍需同步；指针传出线程外则失去隔离意义。

### 58. 数据竞争和伪共享是什么？

**答案：**

数据竞争：两个线程访问同一内存位置、至少一个是写、且无同步，属于未定义行为。解决靠锁、原子操作或线程本地化。

伪共享：两个线程修改各自独立的不同变量，但它们落在同一缓存行（通常 64 字节），缓存一致性协议导致缓存行在核心间反复失效，性能大幅下降。

```cpp
struct Counters {
    alignas(64) std::atomic<int> a;  // 各占一个缓存行
    alignas(64) std::atomic<int> b;
};
```

排查：数据竞争用 ThreadSanitizer；伪共享用 perf／VTune 观察缓存命中率与 `LOCAL_TO_REMOTE` 事件。缓存行大小可用 `std::hardware_destructive_interference_size`（C++17）查询。

### 59. CAS、无锁编程和 ABA 问题是什么？

**答案：**

CAS（Compare-And-Swap）：比较内存值与预期值，相等则写入新值并成功，否则失败返回当前值。通常配合重试循环构建无锁算法：`compare_exchange_weak/strong`。

无锁（lock-free）的严格定义：系统整体保证有操作持续取得进展，而不是"没用 mutex"——一个自旋的原子标志仍然是锁。

ABA 问题：值从 A 改成 B 又改回 A，CAS 只比较值、发现不了中间变化，导致误判"未被修改"。解决办法：版本号／标签指针（携带代数计数）、hazard pointer、延期回收（epoch/memory reclamation）。

另外原子操作未必无锁，`std::atomic<T>::is_lock_free()` 可查询。

### 60. 线程池与游戏任务系统设计要考虑什么？

**答案：**

线程池：预先创建固定数量工作线程，反复从任务队列取任务执行，避免频繁创建销毁线程的开销。

基础组成：任务队列（mutex + condition_variable 或无锁队列）、工作线程、提交接口（`packaged_task` + `future` 返回结果）、停止协议（标志 + notify_all）。

设计要点：

1. 队列容量与背压：有界队列满时的拒绝策略。
2. 任务粒度：过细的任务调度开销超过收益；可用任务窃取（work stealing）平衡负载。
3. 异常传播：任务异常经 future 传出，不能杀死工作线程。
4. 关闭语义：排空剩余任务还是丢弃；避免任务间相互等待造成关闭死锁。
5. 线程数：CPU 密集型取核数或核数-1（给主线程留余量），`hardware_concurrency()` 只是提示；IO 密集型可更多或改用异步 IO。

游戏场景补充：渲染提交必须留在图形 API 约定的线程；帧内短任务用图驱动的任务系统（fork-join），按依赖构建 DAG 调度。

### 61. 协程的原理是什么？与线程有什么区别？

**答案：**

协程是可挂起与恢复的函数：挂起时把执行状态保存到协程帧（参数、局部变量、挂起点），恢复时从挂起点继续。

| 维度 | 有栈协程（如 ucontext、goroutine） | C++20 无栈协程 |
| --- | --- | --- |
| 状态保存 | 独立调用栈 | 协程帧（堆上的结构体） |
| 挂起点 | 任意调用深度 | 函数体内的 `co_await` 处 |
| 内存 | 每个协程独占整块栈 | 帧大小按需，部分场景可优化掉堆分配 |
| 切换成本 | 保存／恢复寄存器切换栈 | 同步操作级别，远低于线程切换 |

与线程的区别：线程由内核抢占式调度，切换进内核；协程是用户态协作式调度，挂起不阻塞所在线程，单线程可运行大量协程；但协程本身不提供并行——并行仍需多线程配合执行器。

C++20 机制三件套：`promise_type` 定制行为、`coroutine_handle` 控制执行、awaiter 协议（`await_ready`／`await_suspend`／`await_resume`）决定挂起恢复。注意协程帧的生命周期管理（handle 必须 destroy）以及引用参数和捕获必须存活到协程结束。

## 十、STL 高频补充

### 62. STL Allocator 和内存池的作用是什么？

**答案：**

Allocator 抽象容器的内存来源：定义 `allocate`／`deallocate`／`construct`／`destroy` 接口，把分配策略与容器实现解耦；默认 `std::allocator` 转发到 `operator new`／`delete`。

内存池：批量申请大块内存后切分复用，减少频繁小块分配的系统调用开销与外部碎片；代价是内部碎片和内存驻留。

C++17 的 PMR（polymorphic memory resources）：改用 `std::pmr::vector<T>`，通过 `memory_resource` 切换分配来源——`monotonic_buffer_resource` 只增不减适合帧分配，`unsynchronized_pool_resource` 适合池化。注意 resource 的生命周期必须覆盖使用它的容器。

### 63. map 和 unordered_map 如何选择？

**答案：**

| 维度 | map | unordered_map |
| --- | --- | --- |
| 底层 | 红黑树 | 哈希表 |
| 查找 | O(log n) | 平均 O(1)，最坏 O(n) |
| 顺序 | 按 key 有序 | 无序 |
| 范围查询 | `lower_bound`／`upper_bound` | 不支持 |
| 键要求 | `operator<` 严格弱序 | 相等键哈希相同 |

选择：需要有序遍历、范围查询、稳定迭代顺序用 `map`；纯键值查询用 `unordered_map`。小数据量可用排序 `vector`（缓存友好）。注意 `operator[]` 在键不存在时会插入默认值——只查询用 `find`，需要异常用 `at`。

### 64. stack、queue 和 priority_queue 的底层和复杂度是什么？

**答案：**

- `stack`：默认基于 `deque`（也可用 `vector`、`list`），LIFO，push/pop/top 均 O(1)。
- `queue`：默认基于 `deque`（需 `pop_front`，`vector` 不可用），FIFO，两端操作 O(1)。
- `priority_queue`：默认 `vector` + `std::make_heap` 堆算法，`less<T>` 为大顶堆。push/pop 为 O(log n)，top 为 O(1)，底层扩容时有一次 O(n) 搬移，批量构建为 O(n)。

```cpp
std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;
minHeap.push(3);
minHeap.push(1);
int smallest = minHeap.top(); // 1
minHeap.pop();                // pop 不返回元素，先 top 再 pop
```

priority_queue 不支持任意元素的更新和删除；需要 decrease-key 操作时改用索引堆或惰性删除配对。

### 65. sort、stable_sort、partial_sort 和 nth_element 有什么区别？

**答案：**

| 需求 | 算法 | 复杂度／性质 |
| --- | --- | --- |
| 全排序 | `sort` | O(n log n)，introsort（快排+堆排+插入排序），不稳定 |
| 稳定全排序 | `stable_sort` | 等价元素保持原序，通常需 O(n) 辅助空间 |
| 前 k 个有序 | `partial_sort` | O(n log k)，堆实现 |
| 第 k 小／划分 | `nth_element` | 平均 O(n)，仅保证第 n 位就位、左侧不大于它 |

注意：

1. 比较器必须是严格弱序（用 `<`，不能用 `<=`），否则越界崩溃。
2. `list` 不支持随机访问迭代器，须用成员 `list::sort`。
3. `nth_element` + `partial_sort` 组合是 Top-K 问题的高效解。

### 66. lower_bound 和 upper_bound 有什么区别？

**答案：**

在有序区间上二分查找：

- `lower_bound(first, last, v)`：第一个 **>= v** 的位置。
- `upper_bound(first, last, v)`：第一个 **> v** 的位置。
- 两者的差即等于 v 的元素区间 `[lower, upper)`。
- `equal_range` 一次返回上述 pair。

找不到时返回 `end()`，解引用前须判空。前提：区间按同一比较器有序。注意 `map`／`set` 应使用成员版本 `lower_bound`（利用树结构），比通用算法对迭代器的要求更低。非随机访问迭代器上这些算法仍是 O(n) 次迭代。

### 67. STL 容器是线程安全的吗？

**答案：**

STL 容器不做任何内部同步，线程安全规则如下：

1. 多线程并发只读同一容器：安全。
2. 至少一个线程写（插入／删除／修改元素），其他线程同时访问：数据竞争，必须外部加锁。
3. 各线程操作各自独立的容器：安全。

例外注意：`vector<bool>` 相邻元素共享同一存储字，并发写不同下标也是竞争。实践方案：锁粒度按业务拆分、线程本地容器汇总、或采用专门的并发容器（TBB `concurrent_hash_map`、folly ConcurrentHashMap 等）。

## 十一、C++ 进阶追问

### 68. 什么是对象切片？如何避免多态失效？

**答案：**

对象切片：派生对象按值赋值／拷贝给基类对象（按值传参、按值返回、值容器存基类）时，只复制基类子对象，派生部分被切掉，结果是一个真正的基类对象，虚调用退化为基类行为。

```cpp
void draw(Base value);        // 按值传参发生切片
void draw(const Base& value); // 引用保留动态类型
```

避免：多态对象一律用基类指针或引用持有；容器存 `std::unique_ptr<Base>`；需要值语义多态时实现虚 `clone()` 返回基类智能指针。

### 69. vector 扩容时如何搬迁元素并保证异常安全？

**答案：**

扩容搬迁流程：申请新存储 → 在新存储上逐个构造旧元素的副本 → 全部成功后析构旧元素、释放旧内存。

异常安全决策：

- T 的移动构造 `noexcept`：使用移动，最快且异常安全。
- 移动可能抛而可拷贝：使用拷贝——拷贝失败时旧容器完整，可维持强保证。
- 只可移动且移动可能抛：无法提供强保证。

因此给移动构造／赋值标记 `noexcept` 直接影响 vector 的扩容性能。已知规模先 `reserve` 可完全避免扩容；增长倍率由实现决定（GCC/Clang 2 倍，MSVC 1.5 倍）。

### 70. unordered_map 在高并发服务中有哪些风险？如何设计稳定访问？

**答案：**

风险：

1. 无同步的并发读写是数据竞争，rehash 使迭代器全部失效。
2. 恶意构造的键可造成哈希碰撞攻击（哈希洪水），查询退化为 O(n)。
3. 扩容 rehash 瞬间延迟尖刺，影响长尾。

设计方案：

1. 预留 `reserve`、控制 `max_load_factor`，业务低峰预扩容。
2. 按 key 分片：多张表配多把锁，降低锁粒度。
3. 读多写少场景用 `shared_mutex` 或写时复制的快照（`shared_ptr` 原子换指针）。
4. 高吞吐场景采用成熟的并发哈希表（分段锁或无锁实现）。
5. 缩小锁范围时保证节点生命周期——不返回可能被其他线程删除的引用。

### 71. 什么是大端和小端？跨平台协议如何处理字节序？

**答案：**

`0x12345678` 在内存中的存放：

- 小端（x86、ARM 默认）：低地址 `78 56 34 12`，低字节在前。
- 大端（网络字节序）：低地址 `12 34 56 78`，高字节在前。

判断方法：把 `int` 拷贝到字节数组看首字节，或 C++20 `std::endian::native`。

跨平台协议准则：逐字段按约定字节序显式编码（`htons`／`htonl` 或手动移位），不直接 `memcpy` 结构体内存发送——结构体还有填充、对齐、整数宽度、浮点表示差异，需要一并显式定义并做版本兼容。

### 72. 为什么不能用 char* 强行解析任意类型？memcpy 与类型转换有什么区别？

**答案：**

把任意字节地址强转成 `T*` 解引用，需同时满足：对齐要求、该地址存在 T 类型的对象（生命周期）、字节构成 T 的有效表示、别名规则（只能用 `char*`／`unsigned char*` 观察任意对象，反向不行）。违反即未定义行为。

- `memcpy`：纯字节复制。对 trivially copyable 类型，在合法对象间复制对象表示是定义良好的；解析外部数据仍需按协议显式解码。
- `static_cast`：按语言规则转换值，如整数与浮点的表示转换。
- `std::bit_cast`（C++20）：按位重新解释，编译期可用，要求源目标大小相同且目标可平凡构造。

### 73. 函数参数上的 const 会影响重载吗？

**答案：**

按值参数的顶层 const 不参与函数签名：`void foo(int)` 与 `void foo(const int)` 是同一个函数，不能构成重载。

指针／引用所指向内容的 const 属于类型的一部分：`void foo(int&)` 与 `void foo(const int&)` 是合法重载。重载决议：非常量左值优先匹配 `int&`，常量左值和右值匹配 `const int&`。

这也是"接口只读参数用 `const T&`"的原因——同一版本接受左值和临时值。

### 74. 符号重定义和符号未定义是怎样产生的？

**答案：**

符号重定义：同一个外部符号在多个目标文件中都有强定义。典型原因：非 inline 函数的定义写在头文件中被多个 `.cpp` 包含，违反 ODR。解决：定义移到源文件、头文件只留声明，或声明为 `inline`。

符号未定义：链接器找不到符号的定义。典型原因：只声明未实现、漏链接目标文件或库、函数签名／命名空间不匹配（声明与定义不一致）、静态库依赖顺序错误。

`extern "C"` 通过关闭名字修饰保持 C 风格符号名，是 C/C++ 混合链接及动态库导出符号的基础。

## 十二、设计模式与解耦

### 75. 单例模式有什么缺陷？游戏中如何缓解？

**答案：**

缺陷：

1. 全局可变状态：依赖关系隐藏在实现里，调用方不可见，问题难以定位。
2. 测试困难：无法注入替身，单测之间通过单例相互污染。
3. 初始化与销毁顺序：跨编译单元的静态初始化顺序不确定，析构中访问其他全局对象同样危险。
4. 生命周期锁死：一旦出现第二实例需求（多房间、多世界），全局访问点全部要重构。

缓解方案：把唯一性约束在使用上下文而非全局——依赖注入（构造时传入）、引擎子系统模式（UE 的 GameInstance Subsystem 提供受控生命周期与注入点）、局部上下文对象逐层传递。

补充：C++11 起局部 static 的懒汉初始化线程安全（编译器自动加锁或原子标志），Meyers 单例是标准写法；但这只解决并发初始化，不解决上述结构性缺陷。

### 76. 模板方法模式解决什么问题？与策略模式如何区分？

**答案：**

模板方法：把算法骨架固定在基类的非虚公有函数中（NVI 惯用法），子类只重写其中的虚钩子步骤。典型骨架：加载流程"打开→解析→校验→注册"、AI"感知→决策→执行"。

与策略模式的区别：

- 模板方法用继承，编译期固定流程框架，变化点是流程内的步骤。
- 策略用组合，运行期注入整个可替换算法。

选择：流程稳定、步骤多变用模板方法；算法整体可替换用策略。模板方法的代价是继承耦合，变化点过多时退化为"改基类牵动全部子类"，此时应转向组合。

### 77. 原型模式是什么？适合什么场景？

**答案：**

原型模式通过克隆已有实例创建新对象，而不是从零构造：基类声明 `virtual Clone()`，各派生类返回自身的深拷贝。

适合场景：

1. 构建成本高或初始化复杂的对象，克隆比重新配置便宜。
2. 运行时按配置选择变体、类数量由数据表驱动——怪物、子弹、Buff 的模板实例配好后克隆生成战斗实例。

实现要点：克隆必须深拷贝资源；`std::shared_ptr<T>(new T(*this))` 复用拷贝构造是最小实现；多态克隆返回基类智能指针。游戏中常与对象池配合：池解决分配频率，原型解决"实例如何从模板生成"。

### 78. 事件／消息中心如何实现系统解耦？有什么代价？

**答案：**

发布—订阅结构：中心维护"事件类型 → 订阅者回调"的注册表，发布方只投递事件对象，订阅方注册／注销，双方互不持有。典型用法：成就系统监听"击杀"事件，战斗系统无需认识成就系统。

代价与陷阱：

1. 生命周期：订阅者销毁前必须反注册，否则回调悬垂。
2. 顺序与重入：多订阅者的执行顺序、订阅者再发布事件的调用链扩散，都难以追踪。
3. 性能与调试：整数 ID + 裸数据的事件总线不如强类型事件可查，高频分发成本需测量。

工程结论：低频业务事件用总线换解耦，热路径（每帧物理、渲染）用直接调用或数组遍历。

## 十三、面经高频补充

### 79. 为什么构造函数不能是虚函数？为什么基类析构函数要声明为虚函数？

**答案：**

构造函数不能虚：虚调用依赖对象 vptr 指向的 vtable，而 vptr 恰恰在构造过程中才按基类→派生的顺序逐步设定；调用构造函数前对象尚不存在，没有可用的虚机制来选择构造函数，编译器只能按静态类型确定。

基类析构必须是虚：通过基类指针 `delete` 派生对象时，只有虚析构才能经 vtable 调到最派生类的析构，再按构造的逆序链式上溯；非虚析构下该操作是未定义行为，典型表现是只执行基类析构、派生资源泄漏。

补充：`shared_ptr` 的控制块记录创建时的删除器，因此能正确删除经基类指针持有的对象，但这不改变裸指针与 `unique_ptr` 下"基类析构必须虚"的规则。

### 80. 移动构造与移动赋值如何实现？被移动后的对象处于什么状态？

**答案：**

实现要点：接管源对象的资源句柄，把源置空，标记 `noexcept`；移动赋值先释放自身资源并处理自赋值。

```cpp
struct Buffer {
    char* data = nullptr;
    size_t size = 0;
    Buffer(Buffer&& other) noexcept
        : data(std::exchange(other.data, nullptr)),
          size(std::exchange(other.size, 0)) {}
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = std::exchange(other.data, nullptr);
            size = std::exchange(other.size, 0);
        }
        return *this;
    }
    ~Buffer() { delete[] data; }
};
```

被移动后的对象处于"有效但未指定"状态：可以安全析构、可以重新赋值，但不承诺具体值。标准库类型如此约定，用户类型应在文档中显式定义移动后契约。容器扩容、返回值、`swap` 都依赖移动的廉价性——移动越廉价（且 noexcept），容器越倾向用移动替代拷贝。

### 81. 如何实现简化版 `std::move` 与 `std::forward`？两者区别是什么？

**答案：**

```cpp
template <class T>
constexpr std::remove_reference_t<T>&& my_move(T&& v) noexcept {
    return static_cast<std::remove_reference_t<T>&&>(v);
}

template <class T>
constexpr T&& my_forward(std::remove_reference_t<T>& v) noexcept {
    return static_cast<T&&>(v);
}
```

区别一句话：move 是无条件强制转右值，forward 是按模板参数 T 有条件还原值类别——左值传入时 `T` 推导为 `U&`，折叠回左值；右值传入时 `T` 为 `U`，转为右值。forward 专用于转发引用参数透传。两者都只是编译期类型转换，运行时零开销。

### 82. C++11 以后 `std::string` 的实现有哪些要点？SSO 是什么？

**答案：**

- SSO（Short String Optimization）：短字符串直接存放在 string 对象内部的栈缓冲区，不分配堆内存；超过阈值才转到堆。阈值因实现而异（libstdc++/MSVC 15 字节，libc++ 22 字节）。SSO 带来的推论：短字符串的移动并不比拷贝便宜，且移动后原字符串内容通常保持不变。
- C++11 起禁止 COW（写时复制）：标准要求 `operator[]` 返回可写引用且不使其他迭代器失效、并发读不产生数据竞争，COW 无法满足；主流实现均为 SSO + 堆双模式。
- 移动廉价：移动只接管堆指针和长度，标记 noexcept 以配合容器扩容。
- 使用建议：已知长度先 `reserve`；大量拼接用 `append`／格式化库；只读传参用 `string_view`，注意其不拥有数据的悬垂风险。

手写实现答题框架：对象布局（指针 + size + capacity，SSO 用 union 复用空间）→ 移动语义与异常安全 → SSO／堆双路径切换 → 与 string_view 的边界。

### 83. inline 的作用是什么？头文件里的成员函数不写 inline 会怎样？

**答案：**

现代 C++ 中 inline 的首要语义是链接层面的 ODR 豁免：允许多个翻译单元出现同一函数的相同定义，链接器合并为一份；"建议编译器内联展开"只是次要优化提示，展开与否由编译器决定。

头文件场景分两种：

1. 类内定义的成员函数隐式 inline，写在头文件不标 inline 也安全。
2. 类内声明、类外定义的成员函数，若定义放在头文件里，必须手动标 inline，否则被多个 `.cpp` 包含时链接器报重复定义。

`constexpr` 函数与模板实例同样隐式满足 inline 要求。性能层面的内联展开应交给编译器、LTO 与热点分析决定。

### 84. 如何防范栈溢出攻击？

**答案：**

先区分两种"栈溢出"：递归过深耗尽栈空间是资源问题；栈溢出攻击指利用无边界写覆盖栈上返回地址、劫持控制流。防范后者分三层：

- 编译器与系统层：Stack Protector 栈金丝雀（`-fstack-protector`，返回前校验哨兵值）、ASLR 地址空间随机化、NX／DEP 数据页不可执行、控制流完整性 CFI（`/guard:cf`）。
- 代码层：禁用无边界检查的 C 函数（`gets`、`sprintf`、`strcpy`），改用带长度的版本或 `std::string`／`std::vector`；所有外部输入先验长度再拷贝；数组访问走带边界检查的接口。
- 工程层：静态分析与 ASan 纳入 CI；重点审计解析网络数据和文件格式的历史模块。

多层叠加是标准答案：运行时机制降低可利用性，代码层消除根源，没有单点完全防御。

### 85. 如何手写一个简易 `shared_ptr`？控制块里放什么？`weak_ptr` 如何配合？

**答案：**

```cpp
template <class T>
class SharedPtr {
    T* ptr = nullptr;
    ControlBlock* cb = nullptr;
public:
    explicit SharedPtr(T* p) : ptr(p), cb(new ControlBlock(p)) {}
    SharedPtr(const SharedPtr& o) : ptr(o.ptr), cb(o.cb) {
        if (cb) cb->strong.fetch_add(1, std::memory_order_relaxed);
    }
    ~SharedPtr() { release(); }
    void release() {
        if (!cb) return;
        if (cb->strong.fetch_sub(1) == 1) {
            cb->destroy();
            if (cb->weak.load() == 0) delete cb;
        }
        ptr = nullptr; cb = nullptr;
    }
};
```

控制块内容：强引用计数、弱引用计数、类型擦除的删除器与分配器；`make_shared` 把对象本体也放进同一块内存，一次分配完成。

销毁分两步：强计数归零销毁对象；弱计数也归零才释放控制块——`weak_ptr` 不增加强计数，但通过持有控制块把控制块生命周期延长到所有观察者结束。`weak_ptr::lock()` 需原子地把强计数从 0 竞争加回 1，失败即对象已销毁，这一竞态处理是关键点。

完整实现还需：移动语义、自赋值、aliasing 构造、计数操作的内存序（递增用 relaxed，归零判断用 acquire/release）。

### 86. 如何用模板元编程实现一个类型列表？

**答案：**

类型列表是编译期的类型容器，基于可变参数模板与特化匹配：

```cpp
template <class... Ts> struct TypeList {};

template <class L> struct Length;
template <class... Ts>
struct Length<TypeList<Ts...>> : std::integral_constant<size_t, sizeof...(Ts)> {};

template <class L, class T> struct Contains;
template <class... Ts, class T>
struct Contains<TypeList<Ts...>, T> : std::bool_constant<(std::is_same_v<T, Ts> || ...)> {};

template <class L, class... Ts> struct Append;
template <class... Us, class... Ts>
struct Append<TypeList<Us...>, Ts...> { using type = TypeList<Us..., Ts...>; };
```

原理：偏特化把 `TypeList<Us...>` 拆包，用折叠表达式或递归继承在编译期完成计算，结果通过 `::value`／`::type` 访问。C++17 折叠表达式让"存在／计数"类运算不再需要递归终止条件。

游戏中的典型用途：ECS 的组件集合、反射注册的类型清单、按类型过滤事件的订阅列表。代价是编译时间随规模增长、报错信息冗长，属于编译期计算的固有成本。
