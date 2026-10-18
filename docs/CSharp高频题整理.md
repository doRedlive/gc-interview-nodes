# C# 面试题与答案

> 共 136 题。按主题分类，答案直接放在题目后。

[返回题库总目录](面试题库总目录.md)

## 目录

| 模块 | 题号 |
| --- | --- |
| [一、基础与类型语义](#%E4%B8%80%E3%80%81%E5%9F%BA%E7%A1%80%E4%B8%8E%E7%B1%BB%E5%9E%8B%E8%AF%AD%E4%B9%89) | 1–22 |
| [二、面向对象与多态](#%E4%BA%8C%E3%80%81%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E4%B8%8E%E5%A4%9A%E6%80%81) | 23–32 |
| [三、泛型、反射与比较规则](#%E4%B8%89%E3%80%81%E6%B3%9B%E5%9E%8B%E3%80%81%E5%8F%8D%E5%B0%84%E4%B8%8E%E6%AF%94%E8%BE%83%E8%A7%84%E5%88%99) | 33–44 |
| [四、委托、闭包与事件](#%E5%9B%9B%E3%80%81%E5%A7%94%E6%89%98%E3%80%81%E9%97%AD%E5%8C%85%E4%B8%8E%E4%BA%8B%E4%BB%B6) | 45–54 |
| [五、内存管理与资源释放](#%E4%BA%94%E3%80%81%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E4%B8%8E%E8%B5%84%E6%BA%90%E9%87%8A%E6%94%BE) | 55–77 |
| [六、集合与迭代器](#%E5%85%AD%E3%80%81%E9%9B%86%E5%90%88%E4%B8%8E%E8%BF%AD%E4%BB%A3%E5%99%A8) | 78–88 |
| [七、LINQ 查询与性能](#%E4%B8%83%E3%80%81LINQ%20%E6%9F%A5%E8%AF%A2%E4%B8%8E%E6%80%A7%E8%83%BD) | 89–108 |
| [八、异步与并发](#%E5%85%AB%E3%80%81%E5%BC%82%E6%AD%A5%E4%B8%8E%E5%B9%B6%E5%8F%91) | 109–129 |
| [九、异常处理](#%E4%B9%9D%E3%80%81%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86) | 130–132 |
| [十、设计与测试基础](#%E5%8D%81%E3%80%81%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%B5%8B%E8%AF%95%E5%9F%BA%E7%A1%80) | 133–134 |
| [十一、运行时与执行模型](#%E5%8D%81%E4%B8%80%E3%80%81%E8%BF%90%E8%A1%8C%E6%97%B6%E4%B8%8E%E6%89%A7%E8%A1%8C%E6%A8%A1%E5%9E%8B) | 135–136 |

## 一、基础与类型语义

### 1. 值类型和引用类型有什么区别？

**答案：**

- 值类型直接存储数据，赋值时复制整个值；引用类型的变量存储对象地址，赋值时复制引用，两个变量指向同一个对象。
- 值类型：`int`、`float`、`bool`、`struct`、`enum`。引用类型：`class`、`string`、数组、委托、接口。
- 存储：值类型局部变量通常在栈上，作为字段时内嵌在包含它的对象中；引用类型对象本身在托管堆，由 GC 管理。

选择依据：小而不可变、高频创建的数据用值类型，减少 GC 压力；有身份语义或体积较大的数据用引用类型。

### 2. `struct` 和 `class` 有什么区别？

**答案：**

| 维度 | struct | class |
| --- | --- | --- |
| 类型语义 | 值类型，赋值／传参复制整个值 | 引用类型，赋值复制引用 |
| 继承 | 不能继承类和结构体，可实现接口 | 支持单继承 + 多接口 |
| 存储 | 栈或内嵌于包含对象 | 托管堆，GC 管理 |
| 默认值 | 总有全零默认值 | null |

选择：小（经验值 16 字节以内）、不可变、生命周期短的数据用 `struct`；有身份、会被大量修改或体积大用 `class`。

### 3. `ref`、`out`、`in` 有什么区别？

**答案：**

- `ref`：调用前必须初始化，方法内可读可写。
- `out`：调用前不要求初始化，方法返回前必须赋值，用于多返回值。
- `in`：只读引用传入，避免大型结构体按值复制的开销。

```csharp
void A(ref int x) { x++; }
void B(out int x) { x = 10; }
void C(in decimal amount) { }
```

### 4. `const` 和 `readonly` 有什么区别？`readonly` 对象还能修改吗？

**答案：**

- `const`：编译期常量，声明时必须用编译期能算出的值初始化，隐式静态，直接内联到调用处——改 const 值必须重编译所有使用方。
- `readonly`：运行期常量，只能在声明处或构造函数中赋值，之后不可重新赋值。
- `readonly` 修饰引用类型字段：锁定的是引用本身，对象内部状态仍可修改。

```csharp
class Settings
{
    public const int MaxCount = 100;
    private readonly List<int> values = new List<int>();

    public void Add()
    {
        values.Add(1);                  // 可以修改对象内容
        // values = new List<int>();   // 不能重新赋值该字段
    }
}
```

### 5. `string` 为什么不可变？如何选择 `string` 和 `StringBuilder`？

**答案：**

string 创建后内容不可修改，任何"修改"操作都返回新对象。设计动机：

1. 内容与哈希值稳定，可安全作为字典键，且能驻留（interning）共享同一实例，节省内存。
2. 天然线程安全，无需同步即可跨线程共享。

代价：循环拼接产生大量中间字符串，增加 GC 压力。选择：少量拼接用 `+`；循环或大量拼接用 `StringBuilder`（内部维护可变缓冲区，应预设容量避免内部扩容）。

### 6. `==` 和 `Equals()` 有什么区别？

**答案：**

- `==` 是运算符，可重载；`Equals()` 是方法，可重写。二者默认行为都由类型决定。
- 引用类型默认比较引用；`string` 重载了 `==` 并重写了 `Equals`，都按内容比较。
- `ReferenceEquals(a, b)` 永远比较引用身份，不受类型重载影响。

```csharp
object a = "hello";
object b = "hello";
Console.WriteLine(a == b);        // False：object 的 == 按引用比较
Console.WriteLine(a.Equals(b));   // True：虚方法按实际类型 string 比较
```

### 7. 什么是装箱和拆箱？为什么泛型可以减少装箱？

**答案：**

- 装箱：值类型转换为 `object` 或接口类型时，在托管堆上创建装箱对象并复制值，产生分配与后续 GC 压力。
- 拆箱：从装箱对象中取出值类型，必须显式转换且目标类型必须与装箱类型严格一致，否则异常。

```csharp
int x = 10;
object obj = x;                        // 装箱
int y = (int)obj;                      // 拆箱

List<int> numbers = new List<int>();
numbers.Add(x);                        // 存储 int 不需要装箱
```

泛型集合为值类型生成专用代码，直接存储值本身，避免了非泛型集合把元素当 `object` 存储的装箱。注意把泛型值传给 `object` 参数、或调用值类型实现的接口方法时仍可能装箱。

### 8. `object`、`dynamic`、`var` 有什么区别？

**答案：**

| 写法 | 绑定时机 | 特点 |
| --- | --- | --- |
| `object` | 编译期静态类型为 object | 访问具体成员需要显式转换；接收值类型会装箱 |
| `var` | 编译期推断出确定的静态类型 | 语法糖，推断后类型固定，不是动态类型 |
| `dynamic` | 运行时绑定 | 编译器跳过成员检查，运行时由绑定器解析，失败抛 `RuntimeBinderException` |

```csharp
var x = 10;       // x 的静态类型是 int
dynamic y = 10;
y = "hello";      // 允许
object z = 10;    // int 装箱为 object
```

### 9. `is`、`as` 和强制类型转换有什么区别？

**答案：**

- `is`：判断类型兼容性，返回 bool；C# 7 起的模式匹配 `is T t` 判断并声明变量一步完成。
- `as`：尝试转换，失败返回 `null`（不抛异常），只适用于引用类型和可空值类型。
- 强制转换 `(T)obj`：失败抛 `InvalidCastException`。

```csharp
if (obj is Player player)
{
    player.Move();
}

Player player2 = obj as Player;   // 失败为 null，用前需判空
Player player3 = (Player)obj;     // 失败抛异常
```

### 10. `readonly struct` 有什么作用？

**答案：**

声明整个结构体不可变，所有实例成员都必须是只读的。收益：

1. 语义明确，调用方可信赖其值不会变。
2. 消除防御性复制：通过只读引用访问时，编译器确认成员不修改状态，不需要复制副本。

注意：字段若包含引用类型，指向的对象仍可修改，`readonly struct` 不等于深度不可变。

```csharp
readonly struct Position
{
    public readonly float X;
    public readonly float Y;

    public Position(float x, float y)
    {
        X = x;
        Y = y;
    }
}
```

### 11. 什么是防御性复制（Defensive Copy）？

**答案：**

当通过只读引用访问非只读结构体成员时，编译器无法保证成员不修改状态，于是先复制一份结构体再调用——这就是防御性复制，隐藏的分配与复制开销。

```csharp
struct Data
{
    public int Value;
    public void Print() => Console.WriteLine(Value);
}

readonly struct Wrapper
{
    public readonly Data Data;
}
```

通过 `wrapper.Data` 调用非只读的 `Print()` 会触发复制。解决：把结构体声明为 `readonly struct`，或把不改状态的成员标记为 `readonly`（`readonly void Print()`）。

### 12. `ref return` 和 `ref local` 是什么？

**答案：**

允许按引用返回和保存变量，操作的是原存储位置而非副本，用于高性能数据访问路径。

```csharp
static ref int GetRef(int[] array, int index)
{
    return ref array[index];
}

int[] values = { 1, 2, 3 };
ref int value = ref GetRef(values, 0);
value = 100; // values[0] 变为 100
```

约束：不能返回生命周期短于调用方的局部变量（栈安全规则保证不产生悬垂引用）。

### 13. in 参数适合哪些场景？

**答案：**

`in` 传只读引用，避免大型结构体（几十字节以上，如矩阵、大向量）按值复制的开销。

注意两点：

1. 小结构体用 `in` 反而吃亏——传引用本身也是开销。
2. 若结构体含非只读成员，通过 `in` 引用调用这些成员会触发防御性复制，收益归零——配合 `readonly struct` 使用才是正解。

### 14. `record` 和普通 `class` 有什么区别？

**答案：**

`record` 是 C# 9 引入的声明值语义数据类型：编译器自动生成基于成员的 `Equals`、`GetHashCode`、`ToString`，以及 `with` 表达式（非破坏性拷贝修改）和解构。

```csharp
record PlayerData(int HP, int Level);

var a = new PlayerData(100, 1);
var b = new PlayerData(100, 1);

Console.WriteLine(a == b);        // True：按值比较
var c = a with { HP = 50 };       // 拷贝并修改，a 不变
```

`record class` 是引用类型 + 值相等性；`record struct` 是 C# 10 的值类型版本。适合 DTO、配置、不可变数据模型。

### 15. C# 中的 `checked` 和 `unchecked` 有什么区别？

**答案：**

控制整数算术的溢出检查：`checked` 溢出抛 `OverflowException`，`unchecked` 静默回绕。默认行为由编译器设置决定（默认 unchecked，`for` 循环计数等场景高效）。

```csharp
int x = int.MaxValue;

int a = unchecked(x + 1); // int.MinValue，回绕
int b = checked(x + 1);   // 抛 OverflowException
```

浮点运算不受影响（IEEE 754 本身有 Infinity／NaN 语义）。

### 16. C# 默认按值传参，为什么方法仍能修改引用对象？

**答案：**

按值传参复制的是"值"本身：引用类型变量的值是对象地址——方法拿到地址副本，可以修改对象成员（调用方可见），但让参数指向另一个对象不影响调用方变量。

```csharp
static void Change(List<int> items)
{
    items.Add(1);                  // 调用方能看到元素变化
    items = new List<int>();      // 只改变方法内部的引用副本
}
```

要让方法能替换调用方变量本身，用 `ref` 传参。

### 17. `int?` 与 `string?` 有什么区别？

**答案：**

- `int?` 是 `Nullable<int>`，真实的运行时类型：在值类型外包一层"有无值"状态，`HasValue`／`Value` 访问，无值时取 `Value` 抛异常。
- `string?` 是可空引用类型标注（C# 8）：只参与编译期静态分析流程，提示"可能为 null"并产生警告；运行时 `string?` 与 `string` 是同一类型，无任何区别。

```csharp
int? score = null;
int actualScore = score ?? 0;
```

### 18. `?.`、`??`、`??=` 和空值宽容运算符 `!` 有什么区别？

**答案：**

- `?.`：接收者为 null 时短路返回 null，不执行成员访问。
- `??`：左侧为 null 时取右侧的值。
- `??=`：左侧为 null 时才赋值（惰性初始化惯用）。
- 后缀 `!`：压制可空警告的编译期标注，运行时什么都不做，实际为 null 照样抛异常。

```csharp
string name = user?.Name ?? "Guest";
cache ??= new Dictionary<string, int>();
```

### 19. `float`、`double` 和 `decimal` 如何选择？

**答案：**

| 类型 | 位宽 | 有效数字 | 适用 |
| --- | --- | --- | --- |
| float | 32 | 约 7 位 | 图形、游戏位置，空间换精度 |
| double | 64 | 15～16 位 | 科学计算、通用默认 |
| decimal | 128 | 28～29 位（十进制） | 金额、财务——二进制浮点无法精确表示 0.1 等十进制小数 |

浮点比较应用误差范围（`Math.Abs(a - b) < epsilon`）而非 `==`。`decimal` 计算比二进制浮点慢一个量级，且同样有限精度（`1m / 3m` 仍需舍入）。

### 20. 浅拷贝、深拷贝与 `record` 的 `with` 有什么区别？

**答案：**

- 浅拷贝：复制字段值——引用字段只复制引用，嵌套对象仍共享。
- 深拷贝：递归复制所有引用对象，产生完全独立的对象图，需处理循环引用。
- `MemberwiseClone()` 是浅拷贝；`record` 的 `with` 同样只做浅拷贝，引用成员仍然共享。

`ToList()`／`ToArray()` 也只是新容器装旧元素引用。深拷贝必须手动实现或序列化往返。

### 21. 下面的字符串代码会产生哪些对象？

**答案：**

```csharp
string a = new string(new[] { 'a', 'b', 'c' });
a = (a.ToUpperInvariant() + "123").Substring(0, 2);
```

堆分配清单：char 数组、`new string` 的结果、`ToUpperInvariant` 的大写串、`+` 拼接串、`Substring` 的子串，共 5 次；字面量 `"123"` 编译期已驻留，不计入。根因是 string 不可变，每步"修改"都产生新对象——这正是循环拼接要用 `StringBuilder` 的原因。

### 22. C# 和 C++ 有什么区别？

**答案：**

| 方面 | C# | C++ |
| --- | --- | --- |
| 执行环境 | 托管运行时（CLR），JIT／AOT | 编译为原生程序 |
| 内存管理 | GC 管托管内存，Dispose 管资源 | RAII、值语义、智能指针 |
| 类型机制 | class 引用语义、struct 值语义、泛型 | class／struct 均可值语义、模板 |
| 继承 | 类单继承 + 多接口 | 支持多继承 |
| 底层控制 | unsafe／P/Invoke 有限访问 | 布局、所有权、原生接口完全控制 |

性能差异的本质在内存布局与分配器控制权，而非语言本身——C++ 的优势是对数据布局和分配时机的完全掌控。

## 二、面向对象与多态

### 23. 面向对象的三大特性是什么？

**答案：**

- 封装：隐藏内部实现，只暴露必要接口，隔离变化。
- 继承：子类复用并扩展父类行为，建立 is-a 关系。
- 多态：同一接口调用，按对象实际类型执行不同实现，是开闭原则的基础。

### 24. 抽象类和接口有什么区别？

**答案：**

- 抽象类：可包含字段、构造函数、状态和已实现方法；表达 is-a 语义，共享基础实现。
- 接口：定义能力契约（can-do，如 `IDisposable`、`IComparable<T>`）；类可实现多个接口。
- 接口自 C# 8 起支持默认实现，但主要用于不破坏二进制兼容的演进，不改变"契约"定位。

选择：提炼共性状态与实现用抽象类；声明能力、需要多实现时用接口。

### 25. `virtual`、`override`、`new` 有什么区别？

**答案：**

- `virtual`：声明方法可被重写。
- `override`：重写基类虚方法，参与运行时多态——按对象实际类型分派。
- `new`：隐藏基类成员而非重写——通过基类引用调用时执行基类版本，按变量的编译期类型分派。

```csharp
class A
{
    public virtual void Run() { }
}

class B : A
{
    public override void Run() { }  // 多态
}

class C : A
{
    public new void Run() { }       // 隐藏：A a = new C(); a.Run() 执行 A 的版本
}
```

### 26. 什么是多态？虚方法如何实现运行时分派？

**答案：**

通过基类或接口引用调用方法时，按对象的实际运行时类型执行对应实现。

实现机制：每个托管对象头部有类型句柄指向该类型的 MethodTable（方法表），虚方法占据方法表的虚槽位；虚调用经对象类型句柄查表取目标方法地址执行。接口调用经接口映射定位实现。这与 C++ 的 vtable 思路一致。

优化：JIT 在能确定目标类型时（`sealed` 类型、静态类型即实际类型）执行去虚拟化，转直接调用甚至内联。

### 27. 什么是协变返回类型？

**答案：**

C# 9 起，重写虚方法时可以返回比基类声明更具体的引用类型。

```csharp
class Animal { }

class Dog : Animal { }

class Factory
{
    public virtual Animal Create() => new Animal();
}

class DogFactory : Factory
{
    public override Dog Create() => new Dog();  // 返回更具体的 Dog
}
```

调用方无需再转换即可使用更具体类型，同时保留多态能力。

### 28. 重载与重写有什么区别？

**答案：**

- 重载（Overload）：同一方法名、不同参数列表，编译期按调用处参数类型静态选择签名；不能仅凭返回类型重载。
- 重写（Override）：用 `override` 替换继承的虚成员实现，运行期按对象实际类型分派。

一句话：重载是编译期"选哪个签名"，重写是运行期"执行哪个类型的实现"。

### 29. `static` 类和单例有什么区别？

**答案：**

- static 类：不能实例化，只有静态成员，承载无状态工具方法；无法实现接口、无法作为参数传递、难以替换测试。
- 单例：控制一个类只有一个实例，实例仍是普通对象——可实现接口、可依赖注入、可替换为 mock 测试。

选择：纯工具函数用 static 类；有状态、需要生命周期管理或可测试性时用单例（或直接依赖注入）。静态字段的并发访问同样需要同步。

### 30. `sealed` 有什么作用？

**答案：**

- `sealed class`：禁止被继承，封闭类型层级。
- `sealed override`：该虚成员在更深层的子类中禁止再重写（类本身仍可被继承）。

收益：明确扩展边界；为 JIT 去虚拟化提供条件——sealed 类型没有派生类，虚调用可确定目标，转直接调用。`sealed` 与不可变、线程安全无关。

### 31. 扩展方法是什么？如何调用？

**答案：**

声明在静态类中的静态方法，第一个参数用 `this` 标记接收者类型，调用时以实例方法语法使用。

```csharp
public static class TextExtensions
{
    public static bool IsBlank(this string value)
        => string.IsNullOrWhiteSpace(value);
}

// 调用
bool blank = "  ".IsBlank();
```

要点：本质是编译器改写为静态调用，不修改原类型、不能访问私有成员；同名实例方法优先于扩展方法；按调用处的 using 指令解析，切换命名空间可能改变行为。LINQ 的 `Where`／`Select` 就是扩展方法。

### 32. `public`、`private`、`protected`、`internal` 及其组合有什么区别？

**答案：**

| 修饰符 | 访问范围 |
| --- | --- |
| public | 任何代码 |
| private | 所属类型内部 |
| protected | 所属类型及其派生类 |
| internal | 同一程序集 |
| protected internal | 同一程序集，或任何程序集的派生类（并集） |
| private protected | 同一程序集内的派生类（交集，C# 7.2） |

成员可访问性不能超过外层类型的可访问性；`internal` 的粒度是程序集，与项目、文件夹无关。

## 三、泛型、反射与比较规则

### 33. 什么是泛型？有什么好处？

**答案：**

泛型允许类型和方法参数化类型（`List<T>`、`Dictionary<K,V>`）。好处：

1. 编译期类型安全，错误在编译期暴露。
2. 消除显式类型转换。
3. 值类型避免装箱，减少 GC 压力。
4. 算法一份实现，多类型复用。

```csharp
List<int> numbers = new();
```

### 34. 什么是 `new()` 泛型约束？有什么限制？

**答案：**

```csharp
class Factory<T> where T : new()
{
    public T Create() => new T();
}
```

要求 T 是非抽象类型且有公共无参构造函数（值类型的默认构造也满足）。限制：无法约束带参构造——需要按参数创建时改用 `Func<T>` 工厂委托传入。

### 35. `where T : class`、`where T : struct`、`where T : unmanaged` 有什么区别？

**答案：**

- `class`：T 为引用类型。
- `struct`：T 为非可空值类型。
- `unmanaged`：T 为非托管类型——不含任何引用字段的结构（可传给指针、`stackalloc`、与原生内存互操作）。

`unmanaged` 常用于互操作与高性能缓冲场景；字段布局仍需 `StructLayout` 显式指定才能保证跨语言一致。

### 36. 什么是协变和逆变？为什么 `List<T>` 不支持协变？

**答案：**

- 协变（`out T`）：输出位置——`IEnumerable<string>` 可以当作 `IEnumerable<object>` 使用。
- 逆变（`in T`）：输入位置——`Action<object>` 可以当作 `Action<string>` 使用。

```csharp
IEnumerable<string> strings = new List<string>();
IEnumerable<object> objects = strings; // 协变，合法

Action<object> printObject = value => Console.WriteLine(value);
Action<string> printString = printObject; // 逆变，合法
```

`List<T>` 不支持协变的原因：它既读又写 T。若允许 `List<string>` 转 `List<object>`，就能向其中 `Add` 一个非字符串对象，类型安全崩塌。因此只有纯输出的 `IEnumerable<out T>`、纯输入的 `Action<in T>` 这类接口和委托可以声明变体。

### 37. 泛型代码在运行时如何处理？

**答案：**

CLR 泛型在运行时按类型参数实例化：

- 每个不同的值类型参数（`List<int>`、`List<float>`）各生成一份专用机器码，直接操作值，无装箱。
- 所有引用类型参数（`List<string>`、`List<object>`）共享同一份代码（引用统一按指针大小处理），每个类型只生成一份类型元数据。

这与 C++ 模板的编译期实例化不同：泛型定义编译一次，实例化发生在运行时加载阶段。

### 38. 泛型约束有哪些？有什么作用？

**答案：**

约束声明类型参数必须具备的能力，让编译器允许泛型代码使用相应成员：

- `class`／`struct`／`unmanaged`／`notnull`。
- 基类或接口约束：`where T : BaseEntity`、`where T : IDisposable`（可调用 Dispose）。
- `new()`：可实例化。
- `T : U`：裸类型约束（T 继承 U）。

收益：编译期检查、减少运行时强制转换和判断，泛型数学（`INumber<T>`）就是接口约束的典型应用。

### 39. 什么是表达式树？它与 Lambda 有什么区别？

**答案：**

同一个 Lambda，两种编译目标：

- 赋给委托（`Func<int,int>`）：编译成可执行代码。
- 赋给 `Expression<Func<int,int>>`：编译成描述代码结构的数据树（参数、常量、运算符节点），可遍历、分析、转换。

```csharp
Expression<Func<int, int>> expr = x => x + 1;
Func<int, int> func = expr.Compile();
```

表达式树是 EF 等框架把 C# 查询翻译成 SQL 的基础。代价：构建与分析有开销，编译成委托也有成本，不适合热点路径。

### 40. 什么是反射？原理、用途和性能优化方法是什么？

**答案：**

反射通过运行时元数据（程序集内的类型表）获取类型、字段、属性、方法信息，并动态创建对象、调用成员。入口：`Type`、`FieldInfo`、`PropertyInfo`、`MethodInfo`、`Assembly`。

用途：序列化、编辑器与工具链、插件加载、依赖注入容器。

性能要点：

1. 元数据查找和动态调用比直接调用慢一个数量级以上，值类型还会装箱。
2. 优化：缓存 `MethodInfo`／`PropertyInfo`，用 `Delegate.CreateDelegate` 转成强类型委托，高频创建用表达式树编译工厂。
3. 裁剪（Trimming）和 AOT 环境会移除静态不可达的类型——反射访问的成员需 `[DynamicallyAccessedMembers]` 标注或保留配置。

### 41. Activator.CreateInstance 如何创建对象？

**答案：**

按运行时 `Type` 或泛型参数动态调用构造函数创建实例，用于插件、反射工厂等编译期不知道具体类型的场景。

```csharp
object obj = Activator.CreateInstance(typeof(Player));
Player p = Activator.CreateInstance<Player>();
```

已知类型直接 `new`；高频动态创建先缓存工厂委托（`Func<T>`），避免每次都走反射查找。

### 42. `IEquatable<T>` 与 `IEqualityComparer<T>` 有什么区别？

**答案：**

- `IEquatable<T>`：类型自身定义强类型相等性（`Equals(T other)`，无装箱）。
- `IEqualityComparer<T>`：外部比较策略对象，同时提供 `Equals` 和 `GetHashCode`，可注入字典、集合、LINQ 操作符——同一数据支持多种比较规则（区分／忽略大小写）。

二者都必须满足契约：相等的对象必须返回相同的哈希值。`EqualityComparer<T>.Default` 会自动利用类型实现的 `IEquatable<T>`。

### 43. `IComparable<T>` 与 `IComparer<T>` 有什么区别？

**答案：**

- `IComparable<T>`：类型自带的默认排序规则，返回负数／零／正数表示小于／等于／大于。
- `IComparer<T>`：外部排序策略，支持按不同字段排序（按价格、按名称）。

注意：排序相等不等于对象相等——两个人年龄相同（比较相等）不代表是同一个人，需要 `Equals` 与 `CompareTo` 表达不同语义时分别实现。

### 44. 序列化和反序列化是什么？

**答案：**

序列化把对象编码为可存储／传输的字节或文本（JSON、二进制），反序列化恢复成对象。方案需明确：格式、字段命名与忽略规则、类型信息、循环引用处理、版本兼容。

安全要点：不信任外部输入——校验长度与取值范围，不允许可信域外的数据任意指定实例化类型（反序列化漏洞）。线程、文件句柄、委托等运行时状态不参与序列化，应保存稳定 ID 并在加载时重建。

## 四、委托、闭包与事件

### 45. 什么是委托和事件？

**答案：**

- 委托：类型安全的方法引用，可存储一个或多个匹配签名的方法，可加减组合。
- 事件：基于委托的发布—订阅机制，外部只能 `+=`／`-=`，不能触发、不能清空订阅列表——封装发布者对通知的控制权。

```csharp
public event Action<int> OnHealthChanged;

void TakeDamage(int damage)
{
    OnHealthChanged?.Invoke(damage);
}
```

事件用于 UI 刷新、战斗状态通知、模块解耦；订阅者销毁或不再需要时应及时取消订阅。

### 46. `Action`、`Func` 和自定义委托有什么区别？

**答案：**

- `Action<T...>`：无返回值的标准泛型委托。
- `Func<T..., TResult>`：有返回值，最后一个泛型参数是返回类型。
- 自定义委托：可命名业务语义（`DamageEventHandler`）、可定义自己的事件签名与扩展。

```csharp
Action<int> action = x => Console.WriteLine(x);
Func<int, int> func = x => x * 2;
```

### 47. 多播委托的返回值有什么特点？

**答案：**

多播委托按订阅顺序依次调用。有返回值时，只有最后一个方法的返回值成为最终结果，前面的返回值全部丢弃；需要收集全部返回值应使用 `GetInvocationList()` 逐个调用。

```csharp
Func<int> f = () => 1;
f += () => 2;

int result = f(); // result 为 2
```

另一个要点：链中某个方法抛出未处理异常，后续方法不再执行——多播委托没有内建的异常隔离。

### 48. 闭包是什么？为什么可能造成内存问题？

**答案：**

闭包是捕获了外部变量的 lambda。编译器把被捕获的变量提升到编译器生成的闭包类（display class）的字段中，lambda 变成该类的方法——变量生命周期因此延长到委托存活多久就活多久。

内存影响：

1. 每次执行包含"捕获局部变量 lambda"的方法都会分配一个闭包对象和一个委托对象。
2. 长期保存该委托（如注册到事件）会连带延长闭包引用的所有对象的生命周期——隐性泄漏源。

### 49. Lambda 的捕获与内存分配有什么关系？

**答案：**

分三种情况：

1. 不捕获任何变量：lambda 可被编译器缓存为静态实例，多次创建不产生新分配。
2. 只捕获 `this`（成员访问）：不需要闭包对象（直接经 this 访问字段），但委托本身仍会分配。
3. 捕获局部变量：编译器生成闭包类，每次执行方法分配闭包对象 + 委托对象，两次分配。

高频路径（每帧注册回调）应避免第 3 种写法：把 lambda 提出去缓存为字段，或改用对象池 + 预分配委托。

### 50. 闭包捕获的是变量还是变量的值？

**答案：**

捕获的是变量的存储位置本身，不是创建 lambda 时的值快照——lambda 执行时读取的是变量当前值。

```csharp
var actions = new List<Action>();

for (int i = 0; i < 3; i++)
    actions.Add(() => Console.WriteLine(i));

// 循环结束后依次调用，输出 3、3、3——共享同一个 i
```

要输出 `0、1、2`，在循环体内建立每次迭代的局部副本：

```csharp
for (int i = 0; i < 3; i++)
{
    int index = i;
    actions.Add(() => Console.WriteLine(index));
}
```

### 51. 为什么事件订阅可能造成内存泄漏？如何避免？

**答案：**

机制：事件发布者通过委托持有订阅者的强引用。发布者长寿（静态事件、全局单例、UI 主控件）而订阅者短寿时，订阅者即使业务上已废弃，GC 仍判定它可达——间接泄漏。

避免：

1. 订阅方销毁前显式 `-=` 取消订阅；生命周期复杂的用实现 `IDisposable` 的订阅句柄统一释放。
2. 取消 Lambda 订阅必须保存原委托引用，重新写一个字面相同的 Lambda 是不同的实例，`-=` 无效。

```csharp
EventHandler handler = (sender, args) => Refresh();
publisher.Changed += handler;
// 生命周期结束
publisher.Changed -= handler;
```

### 52. 如何设计一个避免重复分配的高频事件系统？

**答案：**

1. 事件参数对象复用：预分配参数实例或用对象池，派发前填充字段，避免每次 `new EventArgs`。
2. 避免临时委托：不用闭包 lambda 动态注册，订阅用缓存的委托或实现固定接口（监听器模式）。
3. 订阅列表预分配容量，增删用交换移除保持紧凑。
4. 多线程生产、主线程消费：无锁队列（`ConcurrentQueue`）缓冲，主线程统一派发。
5. 明确订阅生命周期：句柄化（`IDisposable`）或绑定到对象销毁回调自动清理。

原则：先有分配分析数据再优化，不为零分配牺牲可维护性。

### 53. event 和公开委托字段有什么区别？

**答案：**

公开委托字段：外部可以 `=` 重新赋值、置空、直接 Invoke——发布者失去对订阅列表的控制。

`event`：外部只允许 `+=`／`-=`，只有声明类内部能触发和访问底层委托；还可用自定义 `add`／`remove` 访问器接管订阅存储（如按 key 分桶、加锁保护）。

一句话：event 是对委托字段的封装，保护发布权与订阅列表。

### 54. `for` 与 `foreach` 中捕获循环变量有什么区别？

**答案：**

`foreach` 的迭代变量自 C# 5 起每次迭代都是新变量，闭包各自独立捕获；`for` 的循环变量在整个循环中是同一个，所有闭包共享它。

```csharp
var callbacks = new List<Action>();
foreach (int number in new[] { 0, 1, 2 })
    callbacks.Add(() => Console.WriteLine(number));

foreach (Action callback in callbacks)
    callback(); // 依次输出 0、1、2
```

`for` 循环要达到同样效果需在循环体内建局部副本（见第 50 题）。

## 五、内存管理与资源释放

### 55. C# 如何管理托管内存？什么是 GC Roots？

**答案：**

托管对象由 GC 自动回收：从 GC Roots 出发沿引用图标记所有可达对象，不可达对象被回收。

常见 GC Roots：

1. 各线程调用栈上的局部变量与参数。
2. 静态字段。
3. 传递给原生 API 的句柄（GCHandle、Interop 参数）。

关键认知："业务上不再使用"不等于"GC 不可达"——对象被静态集合、缓存、事件订阅链引用时就无法回收，托管内存泄漏的根源都是意外的引用链。

### 56. 什么情况下会触发 GC？

**答案：**

1. 分配预算耗尽：第 0 代空间满时触发 0 代回收（最常见）。
2. 内存压力：系统内存不足、大对象分配。
3. 显式调用 `GC.Collect()`（仅诊断场景使用）。

关注指标：分配速率（alloc/s）、GC 触发频率、单次暂停时间。分配并不立刻触发 GC，但持续的高分配必然转化为高频率回收。

### 57. 什么是 GC Alloc？如何减少？

**答案：**

GC Alloc 指代码执行过程中的托管堆分配。每次分配本身很快（指针递增），代价在于加速 0 代回收频率和暂停。

常见优化：

1. 高频路径避免创建临时对象：字符串拼接、LINQ 链、闭包、装箱。
2. 复用集合与缓冲（`Clear()` 后重用、`ArrayPool<T>`）。
3. 值类型替代小对象（struct、`Span<T>`）。
4. 对象池管理创建销毁频繁的对象。
5. 用分析工具（Unity Profiler 的 GC Alloc 列、dotMemory）定位分配调用栈。

### 58. 什么是托管堆和非托管内存？

**答案：**

- 托管堆：CLR 管理、GC 回收的对象内存，分配回收都由运行时接管。
- 非托管内存：原生代码 `malloc` 分配的内存、文件句柄、GPU 资源——GC 不知道它们的存在。

包装类（如 `FileStream` 持有文件句柄、纹理对象持有 GPU 内存）被 GC 回收的时机不确定，非托管资源不能等 GC——必须 `Dispose()` 或 `using` 确定性释放。

### 59. `IDisposable` 和析构函数有什么区别？

**答案：**

- `Dispose()`：确定性释放，调用时机由开发者控制，配合 `using` 在作用域结束时自动执行。
- 终结器（Finalizer，C# 的"析构函数"语法）：GC 回收对象前调用，时机不确定（可能永远不执行），且会让对象晋升一代、延迟回收。

```csharp
using (var stream = new FileStream(path, FileMode.Open))
{
    // 使用文件
} // 自动调用 Dispose()
```

标准释放模式：`Dispose()` 中释放资源并 `GC.SuppressFinalize(this)`；终结器只作为忘记 Dispose 时的兜底，且只释放非托管资源。

### 60. 什么是弱引用 `WeakReference`？

**答案：**

弱引用引用对象但不阻止 GC 回收——对象没有强引用时会被回收，弱引用自动变空。使用前必须检查 `TryGetTarget`（泛型版）判断目标是否存活。

用途：缓存（内存紧张时缓存项可被自动回收）、对象身份映射（不延长对象生命）。注意每次使用都要判活，且目标可能"复活"，逻辑要能接受对象随时消失。

### 61. GC 为什么采用分代回收？

**答案：**

基于弱分代假说：绝大多数对象朝生夕死。全堆扫描代价高，分代让 GC 优先扫描最小的新生代，用小成本回收大部分垃圾。

.NET GC 分 0、1、2 代：

1. 新对象进入 0 代；0 代回收存活的对象晋升 1 代，1 代存活晋升 2 代。
2. 回收某代会顺带回收更年轻的代。
3. 触发条件是各代分配预算，不是简单的"装满"。
4. 标记后执行清除与压缩（移动对象消除碎片，需修正引用），大对象走 LOH（见第 66 题）。

推论：长寿命对象多经历两次回收后进入 2 代，2 代回收最贵——控制长寿命对象引用链（事件、缓存）是减少 2 代回收的关键。

### 62. 为什么频繁创建小对象也会影响性能？

**答案：**

三个层面：

1. 加速 GC 触发：0 代预算固定，分配越快回收越频繁。
2. 暂停累积：每次 GC 都有暂停（0 代通常 <1ms，但帧敏感场景 60fps 下 16ms 帧预算里多次 0.5ms 也是负担）。
3. 缓存压力：新对象分散在内存各处，破坏局部性。

例：每次操作分配 1 KiB、每秒 60 次，每分钟累计分配约 3.5 MiB——瞬时存活不大，但 GC 频率被拉高。优化以"降低分配速率"为目标，而非只看峰值内存。

### 63. C# 中的对象在内存中通常包含什么？

**答案：**

托管堆对象布局（x64 典型）：

1. 对象头（8 字节）：同步块索引（lock 标记）、哈希码缓存。
2. 方法表指针（8 字节）：指向类型的 MethodTable，承担类型识别、虚方法分派。
3. 实例字段：按布局排列；引用类型字段存地址。

每个对象至少 16 字节开销——小对象批量创建时开销占比显著。值类型无这两个头部，内嵌于包含者中或栈上。

### 64. struct 作为局部变量、字段和数组元素时怎样存储？

**答案：**

- 局部变量：栈帧上（或被优化进寄存器）。
- 类的字段：内嵌在所属堆对象的布局中，`Vector2 Position` 不产生单独的堆分配。
- 数组元素：连续内嵌在数组内存中，遍历缓存友好。

```csharp
struct Vector2
{
    public float X, Y;
}

class Entity
{
    public Vector2 Position;
}
```

装箱是例外：值类型转 object／接口后复制到托管堆，带完整对象头。

### 65. `GCHandle` 有什么作用？

**答案：**

显式控制托管对象的 GC 可达性与固定状态，用于原生互操作：

- `Normal`：让原生代码持有的对象保持可达（防止 GC 回收正在被原生侧使用的对象）。
- `Pinned`：固定对象地址，GC 压缩时不移动，供原生指针直接访问。
- `Weak`：弱引用形式。

`Free()` 必须调用，否则对象被钉死或长期存活。`fixed` 语句是 Pinned 句柄的语法糖，优先使用。

### 66. 什么是 LOH？为什么大对象分配需要关注？

**答案：**

LOH（Large Object Heap）：约 85,000 字节以上的对象直接分配到 LOH（.NET Framework／Core 默认；.NET 9 起阈值可配置）。

特点：

1. 分配即视作 2 代，回收只在 2 代／显式请求时发生——大对象频繁分配意味着昂贵的 2 代回收。
2. 默认只清扫不压缩，反复分配释放产生碎片；`GCSettings.LargeObjectHeapCompactionMode` 可请求压缩。

优化：大缓冲复用（`ArrayPool<T>`）、切分成小块、流式处理避免整块加载。

### 67. 什么是内存碎片？

**答案：**

空闲内存总量足够但被分割成不连续小块，无法满足大的连续分配。托管堆的碎片来自：不同大小对象交错分配回收后留下的空洞、被 Pin 的对象阻断压缩。

缓解：启用 GC 压缩、减少长时间 Pin、复用定长缓冲。非托管堆（malloc）碎片更严重，需要内存池管理。

### 68. GC 为什么需要写屏障（Write Barrier）？

**答案：**

分代 GC 默认假设"老年代不常引用新生代"，回收新生代时只扫描新生代 + 根。若老对象获得一个指向新对象的引用而没有记录，新对象会被误回收。

写屏障在引用赋值时插入一小段代码，把"老→新"的引用记录到 remembered set／卡表（card table），下次新生代回收时补扫这些位置。增量／并发 GC 同样靠写屏障捕获标记期间的引用变化。代价是所有引用写入都变慢一点——这是分代 GC 的运行时成本。

### 69. 什么是逃逸分析？有什么优化作用？

**答案：**

逃逸分析判断对象的引用是否会离开当前作用域（返回、写入字段、传给其他线程）。不逃逸的对象生命周期确定，运行时可以栈分配、标量替换（拆成局部变量）或直接消除分配。

.NET 的逃逸分析能力有限且随版本演进（部分内联优化、stackalloc 场景），不能假设"不逃逸的对象一定在栈上"——确定性零分配要靠 `struct`、`Span<T>`、`stackalloc` 显式表达。

### 70. Span<T> 是什么？有什么作用？

**答案：**

`Span<T>` 是连续内存的类型安全视图（指针 + 长度），可以指向数组片段、栈内存（`stackalloc`）、非托管内存，切片不产生新数组、不复制数据。

```csharp
int[] data = { 1, 2, 3, 4 };
Span<int> part = data.AsSpan(1, 2);

part[0] = 10; // 修改原数组中的 data[1]
```

核心价值：子串／子数组操作零分配。约束：`Span<T>` 是 `ref struct`，只能存在于栈上——不能作为类字段、不能跨 `await`／`yield` 边界，以防悬垂。

### 71. `stackalloc` 是什么？有什么风险？

**答案：**

在栈上分配一段内存，配合 `Span<T>` 使用，随作用域自动回收、零 GC 开销。

```csharp
Span<int> buffer = stackalloc int[16];
```

风险：

1. 栈空间有限（通常 1 MB 量级），分配过大会栈溢出——建议只用于小的、大小有上限的缓冲。
2. 递归中使用会随深度累积。
3. 不能让引用逃逸出当前方法。

大小可能超限的缓冲用 `ArrayPool<T>` 或堆分配。

### 72. `fixed` 关键字的作用是什么？

**答案：**

在语句范围内固定托管对象地址，禁止 GC 压缩移动它，从而安全取指针交给原生代码。

```csharp
unsafe
{
    int[] data = { 1, 2, 3 };

    fixed (int* p = data)
    {
        // 在此范围内 p 指向的内存不会被移动
    }
}
```

注意：固定时间过长会阻碍 GC 压缩形成碎片；`fixed` 是同步阻塞的 Pin，与 `GCHandle.Pinned` 机制相同但作用域受限；能改用 `Span`／`Memory<T>` 传参时优先避免 Pin。

### 73. `ArrayPool<T>` 有什么用？

**答案：`

共享数组池，租借／归还临时缓冲，消除大数组反复分配的 GC 压力与 LOH 问题。

```csharp
var pool = ArrayPool<byte>.Shared;
byte[] buffer = pool.Rent(4096);

try
{
    // 使用 buffer，注意实际长度 >= 4096，用传入的 count 控制有效范围
}
finally
{
    pool.Return(buffer);
}
```

纪律：归还后不得继续使用（可能已被其他线程租走）；记录并传递有效长度（Rent 返回的数组可能更长）；敏感数据归还前清零。

### 74. `using` 语句、`using` 声明和 `await using` 有什么区别？

**答案：**

- `using (...) { ... }`：语句块结束释放。
- `using var resource = ...;`：C# 8 声明形式，所在作用域结束释放，减少嵌套。
- `await using`：异步释放，用于 `IAsyncDisposable` 资源。

三者语义等价于 `try/finally`——异常也能保证释放。注意 `using` 只是确定性调用 Dispose，与线程安全无关。

### 75. GC 和 Dispose 分别负责什么？什么时候调用 GC.Collect？

**答案：**

分工：GC 回收托管内存（自动、时机不定）；Dispose 释放非托管资源与时机敏感的托管资源（手动、确定）。

`GC.Collect()` 的合理场景几乎只有：切换大关卡／重载后主动回收旧世代对象、内存诊断时配合 `WaitForPendingFinalizers()` 分段调用。常规业务代码不应调用——触发完整回收代价比自然触发更贵，且可能不必要地提升对象代数。

### 76. unsafe 和 P/Invoke 有什么作用？

**答案：**

- `unsafe`：启用指针、取地址、固定内存等底层操作，用于高性能数据操作与互操作；不关闭 GC、不保证更快。
- P/Invoke（`[DllImport]`）：调用原生动态库导出函数，参数默认被 marshal（字符串转 char*、结构体按布局复制）。

工程要点：Pin 好所有被原生侧引用的内存、明确内存所有权（谁分配谁释放）、注意调用开销（marshal 与跳转，高频小调用应改批量接口）；能用 `Span`、`LibraryImport` 源生成器时优先。

### 77. 调用栈与托管堆有什么区别？

**答案：**

- 调用栈：线程私有，后进先出，保存调用帧、返回地址、局部变量；函数返回即回收，速度快、无碎片问题，但空间有限。
- 托管堆：进程共享，GC 管理，对象生命周期与创建它的方法调用期解耦（引用可被带出去）。

存储判断不是"值类型在栈、引用类型在堆"这么简单：值类型字段内嵌于堆对象、装箱后在堆、闭包捕获后在闭包对象里；引用类型对象的引用本身也可以只在栈上。分析内存时关注"分配到了堆上什么"而非类型分类。

## 六、集合与迭代器

### 78. `Array` 和 `List<T>` 有什么区别？`List<T>` 如何扩容？

**答案：**

| 对比项 | `Array` | `List<T>` |
| --- | --- | --- |
| 长度 | 创建后固定 | 动态 |
| 存储 | 连续，最紧凑 | 内部也是数组 |
| 性能 | 无扩容与额外间接 | 增删需处理容量 |
| 适用 | 大小固定、高性能、多维 | 通用动态序列 |

`List<T>` 扩容：容量不足时新建约为 2 倍的数组（默认首次扩到 4，之后倍增）并整体复制，单次 O(n)、尾插均摊 O(1)。已知规模用构造函数或 `Capacity` 预设容量，避免反复复制。

### 79. `Dictionary<TKey, TValue>` 的底层原理是什么？如何处理哈希冲突？

**答案：**

底层是 buckets + entries 双数组结构（链地址法）：

1. 查找：计算 key 哈希 → 定位 bucket → 沿该桶的冲突链比较哈希值与 `Equals` 找到条目。
2. 插入冲突：新条目挂到桶链头部。
3. 负载过高时扩容（翻倍并 rehash 所有条目）。

复杂度平均 O(1)，冲突严重或哈希分布差时退化为 O(n)。自定义键类型必须同时正确重写 `GetHashCode()` 与 `Equals()`，并保证：相等对象哈希相同、哈希在对象进入字典后不变。

### 80. `List<T>` 和 `LinkedList<T>` 如何选择？

**答案：**

- `List<T>`：连续数组存储，随机访问 O(1)、遍历缓存友好、尾插均摊 O(1)；中间插删 O(n) 但需要移动的只是内存拷贝，常数小。
- `LinkedList<T>`：双向链表，已持有节点时插删 O(1)，但查找节点 O(n)，每节点额外两个指针 + GC 压力 + 遍历跳跃缓存不友好。

工程结论：绝大多数场景选 `List<T>`——"链表插删快"只有在已经持有节点引用时才成立，遍历找节点本身就是 O(n)。

### 81. `IEnumerable<T>` 和 `IList<T>` 有什么区别？

**答案：**

- `IEnumerable<T>`：只能正向枚举，最小的消费接口——支持延迟执行、LINQ、任意序列（包括数据库查询、生成器）。
- `IList<T>`：支持索引访问、按位置增删改。

面向接口编程原则：参数和返回值用够用的最小接口（`IEnumerable<T>` 优先），把集合的真正实现与调用方解耦。需要多次枚举或随机访问时再升级要求。

### 82. 遍历 List 时如何安全删除元素？

**答案：**

`foreach` 中结构性修改集合，枚举器的版本检测会抛 `InvalidOperationException`。正确做法是倒序 for 遍历（删除不影响未遍历的下标）：

```csharp
for (int i = list.Count - 1; i >= 0; i--)
{
    if (list[i] < 0)
        list.RemoveAt(i);
}
```

或一次遍历标记、二次过滤（`RemoveAll`），或直接 `list.RemoveAll(x => x < 0)`。

### 83. `yield return` 的底层原理是什么？

**答案：**

编译器把迭代器方法改写成状态机类：状态字段保存执行位置，局部变量提升为字段；每次 `MoveNext()` 从上次挂起点继续执行到下一个 `yield return`，把值放进 `Current`。

效果：按需逐个产生元素（延迟执行、可表示无限序列），不需要先构建整个集合。代价：状态机对象与枚举器对象的分配（接口枚举时），热路径可用手写 struct 枚举器或 `Span` 遍历规避。

### 84. `HashSet<T>` 与 `List<T>` 有什么区别？

**答案：**

- `HashSet<T>`：按相等性去重的集合，`Contains` 平均 O(1)，适合成员检测、去重、集合运算（交并差）。
- `List<T>`：有序可重复，`Contains` 是 O(n) 遍历，支持索引访问。

高频"是否已存在"判断用 `HashSet`；需要顺序、索引、允许重复用 `List`。注意 `HashSet` 的枚举顺序不保证，勿依赖。

### 85. 为什么不应修改字典键中参与哈希或相等性计算的字段？

**答案：**

插入时字典按当时的哈希值把条目挂到对应桶。修改键的字段后哈希值改变，但条目还挂在旧桶上——再用这个键查找会去新桶找，永远找不到，形成"字典里有但查不到"的隐形丢失。

规则：字典键应为不可变类型（string、int、readonly struct）；确需修改时先删除旧键、改完再插入新键。

### 86. IReadOnlyList 与不可变集合有什么区别？

**答案：**

- `IReadOnlyList<T>`：只读视图接口——通过该接口不能改，但底层引用的集合可能被持有写权限的代码修改（快照不保证）。
- 不可变集合（`ImmutableList<T>` 等）：结构本身不可变，任何"修改"返回新集合，旧版本永远不变，天然线程安全。

需要稳定的多次枚举快照：复制、不可变集合或 `ToImmutableList()`；只是限制调用方修改：只读接口即可。

### 87. IEnumerable、IEnumerator 和 foreach 的原理是什么？

**答案：**

- `IEnumerable<T>`：提供枚举器（`GetEnumerator()`）。
- `IEnumerator<T>`：保存一次遍历的状态——`MoveNext()` 前进并返回是否有元素、`Current` 取当前元素、`Dispose()` 清理。

`foreach` 是模式匹配语法糖：编译器生成"取枚举器 → 循环 MoveNext/Current → finally Dispose"，不强制要求实现 IEnumerable（鸭子类型）。性能细节：数组与 `List<T>` 的 foreach 经 JIT 优化为索引循环且使用 struct 枚举器，零分配；把集合先转成接口变量再遍历会装箱 struct 枚举器并走虚调用——热路径保持具体类型。

### 88. `ArrayList` 与 `List<T>` 有什么区别？

**答案：**

`ArrayList` 是 .NET 1.x 的非泛型列表：元素以 `object` 存储，值类型存取装箱拆箱、取出需强转、无编译期类型安全。`List<T>` 是泛型版本：类型安全、值类型零装箱、性能更好。

新代码一律使用泛型集合；`ArrayList` 只在维护旧代码时出现。

## 七、LINQ 查询与性能

### 89. 什么是 LINQ？查询语法和方法语法有什么关系？

**答案：**

LINQ（Language Integrated Query）用统一的方式表达筛选、投影、排序、分组、聚合，操作符位于 `System.Linq`。

```csharp
int[] numbers = { 1, 2, 3, 4 };
var query1 = from n in numbers where n % 2 == 0 select n * 10;
var query2 = numbers.Where(n => n % 2 == 0).Select(n => n * 10);
// 枚举时二者都得到 20、40
```

查询语法在编译期被翻译成方法调用，两者完全等价；`Count()`、`ToList()` 等只有方法语法。默认讨论 LINQ to Objects（针对内存集合）。

### 90. 什么是延迟执行？修改数据源后查询结果会改变吗？

**答案：**

`Where`、`Select` 等操作符构建的是查询逻辑，不立即执行——每次枚举才遍历数据源求值。数据源或捕获的变量在枚举前变化，结果随之变化。

```csharp
var numbers = new List<int> { 1, 2, 3 };
int limit = 1;
var query = numbers.Where(n => n > limit);
limit = 2;
numbers.Add(4);
Console.WriteLine(string.Join(",", query)); // 3,4
```

`ToList()`／`ToArray()` 在调用时物化快照；`Count()`／`First()` 等终结操作立即求值，但可能提前结束。

### 91. 延迟执行与流式执行是同一回事吗？

**答案：**

不是。延迟执行描述"何时开始执行"，流式执行描述"是否需要先读完全部输入"。

| 操作 | 执行方式 | 典型行为 |
| --- | --- | --- |
| `Where`、`Select` | 延迟、流式 | 按需读取并产生结果 |
| `OrderBy`、`GroupBy` | 延迟、非流式 | 枚举时先处理完整输入再输出 |
| `ToList`、`ToArray` | 立即物化 | 保存枚举结果 |
| `Any`、`First` | 立即求值 | 得到答案即提前结束 |

推论：对无限序列可以 `Where().First()`，但 `OrderBy().First()` 永远无法完成。

### 92. `IEnumerable<T>` 与 `IQueryable<T>` 有什么区别？

**答案：**

- `IEnumerable<T>` + `Enumerable` 扩展：接收委托，在内存中逐元素执行。
- `IQueryable<T>` + `Queryable` 扩展：接收表达式树，由查询提供程序翻译成远程查询（如 SQL）——筛选在数据库执行，只传回结果。

```csharp
// db 是 DbContext
var result = db.Players.Where(p => p.Level > 10).ToList();  // WHERE level > 10 在数据库执行
var local = db.Players.AsEnumerable().Where(p => p.Level > 10);  // 全表拉回内存后过滤
```

注意：`IQueryable` 只翻译提供程序认识的表达式，任意 C# 方法调用会失败或退化；数据已在内存中时二者无区别。

### 93. `AsEnumerable()` 和 `ToList()` 有什么区别？

**答案：**

- `AsEnumerable()`：只把静态类型换成 `IEnumerable<T>`，让后续操作符走本地 `Enumerable` 逻辑；不枚举、不缓存，惰性。
- `ToList()`：立即枚举并物化成列表，消费时间与内存，换来回读与多次复用。

远程查询场景：先尽量下推数据库能执行的筛选，再 `AsEnumerable()` 切到本地处理本地逻辑，最后确需复用才 `ToList()`。

### 94. `Where()` 和 `Select()` 有什么区别？

**答案：**

- `Where`：谓词筛选，元素数量减少，类型不变。
- `Select`：投影映射，每个输入对应一个输出，类型可以改变。

```csharp
int[] values = { 1, 2, 3, 4 };
var result = values.Where(x => x > 2).Select(x => $"No.{x}");
// 枚举得到 No.3、No.4
```

注意：无人枚举的延迟查询不会执行——把副作用写进 `Select` 却从不枚举，代码根本不会跑；需要逐项执行副作用用明确的 foreach。

### 95. `Select()` 与 `SelectMany()` 有什么区别？

**答案：**

`Select` 是一对一投影（结果可以是集合本身）；`SelectMany` 是一对多再展平——把每个输入的子序列拍平成一个序列。

```csharp
int[][] groups = { new[] { 1, 2 }, new[] { 3 } };
IEnumerable<int[]> nested = groups.Select(g => g);
IEnumerable<int> flat = groups.SelectMany(g => g); // 1、2、3
```

典型场景：所有订单的所有订单项、所有玩家的所有装备——一层 SelectMany 展开一层嵌套。

### 96. Any、All 和 Count 有什么区别？

**答案：**

- `Any()`：存在性判断，找到第一个满足条件的元素立即返回 true；空序列返回 false。
- `All(predicate)`：遇到第一个反例立即返回 false；空序列返回 true（全称命题的空真）。
- `Count()`：必须数完整个序列才能给答案。

判断"有没有元素"：序列用 `Any()`（可提前结束），集合自带 `Count` 属性时直接读属性（O(1)）。业务要求"非空且全部满足"需 `Any() && All(...)` 两次遍历，注意重复枚举成本。

### 97. `First`、`FirstOrDefault`、`Single`、`SingleOrDefault` 如何选择？

**答案：**

| 方法 | 0 个 | 1 个 | 多个 |
| --- | --- | --- | --- |
| `First()` | 抛异常 | 返回该元素 | 返回第一个 |
| `FirstOrDefault()` | 返回 `default(T)` | 返回该元素 | 返回第一个 |
| `Single()` | 抛异常 | 返回该元素 | 抛异常 |
| `SingleOrDefault()` | 返回 `default(T)` | 返回该元素 | 抛异常 |

要求"唯一"（业务上不该有多条）用 `Single` 系列，多数据视为异常暴露问题；只要首个用 `First` 系列。`default(int)` 是 0，无法区分"没有元素"与"元素恰好是 0"，值类型需要区分时用可空版本 `FirstOrDefault(x => ...) ?? (int?)null` 的判断模式或改用 TryGet 风格 API。

### 98. `OrderBy()` 与 `ThenBy()` 有什么区别？

**答案：**

`OrderBy` 建立主排序；`ThenBy` 在主键相等时按次级键排序；再次 `OrderBy` 会丢弃已有排序建立新主排序。

```csharp
var sorted = employees
    .OrderBy(e => e.Department)
    .ThenByDescending(e => e.Salary)
    .ThenBy(e => e.Id);
```

LINQ to Objects 的排序是稳定的（相等元素保持原顺序）；`List<T>.Sort()` 不稳定。分页必须带上唯一的末级排序键，否则翻页结果可能重复／跳过。

### 99. `GroupBy()` 与 `ToLookup()` 有什么区别？

**答案：**

- `GroupBy`：延迟执行，枚举时读取源数据产生分组序列，一轮消费。
- `ToLookup`：立即执行，构建可反复按键查询的 `ILookup<TKey, TElement>`——只读的多字典。

```csharp
var byLength = new[] { "a", "bb", "cc" }.ToLookup(s => s.Length);
// byLength[2] 包含 bb、cc；缺失键返回空序列而非抛异常
```

一次分组用完即弃选 `GroupBy`；需要多次按键取组（如索引结构）用 `ToLookup`。

### 100. `ToDictionary()` 遇到重复键怎么办？

**答案：**

直接抛 `ArgumentException`，不会静默保留最后一个。处理策略按业务定：

1. 键本应唯一：暴露数据错误，加日志与校验。
2. 允许多值：改用 `ToLookup`，或 `ToDictionary(k => k.Key, v => v.ToList())` 先分组。
3. 保留最后一个：显式 foreach 用索引器覆盖，或 `GroupBy(...).ToDictionary(g => g.Key, g => g.Last())`。

### 101. `Distinct()` 如何判断重复？

**答案：**

默认用 `EqualityComparer<T>.Default`（string 按内容，自定义类型按引用或其 `Equals` 实现），可传 `IEqualityComparer<T>` 定制。

```csharp
var names = new[] { "Alice", "ALICE", "Bob" }
    .Distinct(StringComparer.OrdinalIgnoreCase);
```

按业务键去重：投影后去重（`Select(x => x.Id).Distinct()`），或 .NET 6+ 用 `DistinctBy(x => x.Id)`。保持首次出现顺序在 LINQ to Objects 中成立，但不应作为跨实现的依赖。

### 102. `Concat`、`Union`、`Intersect`、`Except` 有什么区别？

**答案：**

- `Concat`：顺序连接，保留所有重复项。
- `Union`：并集，去重后合并。
- `Intersect`：交集，双方共有的不同元素。
- `Except`：差集，第一序列中有、第二序列中没有的元素。

例：`A = [1, 1, 2]`、`B = [2, 3]`：`Concat` 得 5 项；`Union` 得 1、2、3；`Intersect` 得 2；`Except` 得 1。后三者内部构建哈希集合，自定义类型需正确的相等性实现。

### 103. `Cast<T>()` 与 `OfType<T>()` 有什么区别？

**答案：**

- `Cast<T>()`：把每个元素转换为目标类型，遇到不兼容元素抛异常。
- `OfType<T>()`：过滤出兼容目标类型的元素（跳过其他），不抛异常。

```csharp
object[] data = { 1, "two", null, 3 };
var ints = data.OfType<int>(); // 1、3
// data.Cast<int>().ToList() 枚举到 "two" 时抛异常
```

注意 `Cast` 是引用转换与拆箱，不做数值转换：装箱的 `int` 用 `Cast<long>()` 会失败，需显式 `Select(x => (long)x)`。

### 104. Aggregate 如何使用？

**答案：**

`Aggregate` 是通用折叠：按累计函数把序列归约为单个值。带种子的重载支持空序列（返回种子），无种子重载要求序列非空（空序列抛异常）。

```csharp
int[] numbers = { 2, 3, 4 };
int product = numbers.Aggregate(1, (acc, n) => acc * n); // 24
```

求和、平均、最值优先用语义化的 `Sum`／`Average`／`Min`／`Max`；`Aggregate` 用于没有专用操作符的自定义累积。

### 105. 为什么重复枚举 `IEnumerable<T>` 可能有问题？

**答案：**

每次枚举都重新执行查询逻辑：`Count()` 之后再 `foreach` 是两遍完整计算——重复计算浪费、远程数据重复请求、源数据中途变化导致两次结果不一致，副作用型源（日志流）甚至第二次为空。

处理：需要复用结果时一次 `ToList()` 物化再多次读取；只需单遍处理时组织成一次遍历（自定义 foreach 累积），不为物化付出不必要的内存代价。

### 106. `Join()`、`GroupJoin()` 和左外连接如何理解？

**答案：**

- `Join`：等值内连接，匹配的元素两两组合成结果对。
- `GroupJoin`：分组连接，每个外部元素对应一组匹配的内部元素（可能为空组）。

```csharp
var leftJoin = outer.GroupJoin(
        inner, o => o.Key, i => i.Key,
        (o, matches) => new { o, matches = matches.DefaultIfEmpty() })
    .SelectMany(x => x.matches.Select(i => new { x.o, i }));
// 无匹配时 i 为 null —— 左外连接的惯用写法
```

`Join` 基于键哈希匹配（O(n+m)），不是按索引配对；按位置配对用 `Zip`。

### 107. LINQ 有哪些性能开销？如何优化？

**答案：**

开销来源：

1. 迭代器链：每个操作符一个状态机对象，多层管道多层分配与虚调用。
2. 闭包与委托：捕获变量的 lambda 产生闭包分配。
3. 重复枚举：见第 105 题。
4. 缓冲操作符：`OrderBy`／`GroupBy`／`ToList` 的中间集合分配。

优化次序：先消除重复枚举与无用排序（算法层面收益最大）→ 需要复用时物化一次 → 真正的热点改写显式 foreach + 复用容器 + struct 迭代。帧循环等分配敏感路径默认不用 LINQ 链，开发期便利代码在打包前替换。

### 108. PLINQ 的原理和适用场景是什么？

**答案：**

`AsParallel()` 把查询交给 PLINQ：数据分区 → 多线程并行执行 → 合并结果。适用条件：数据量大、每项计算重、元素间独立（无共享状态）、允许乱序或保序开销可接受。

不适用：小数据（分区调度开销大于收益）、IO 密集（应改异步）、每项工作极轻（并行退化）。保序（`AsOrdered()`）、共享状态写入都会显著增加同步成本。CPU 密集的批量数据变换是典型场景。

## 八、异步与并发

### 109. `async/await` 有什么作用？底层如何工作？

**答案：**

`async/await` 用同步的写法组织异步控制流，编译器把异步方法改写成状态机：

1. 方法先在调用线程同步执行，直到遇到第一个未完成的 `await`。
2. 挂起点：状态机注册后续逻辑（continuation）到被等待的 Task，方法立即返回——调用方不被阻塞。
3. 操作完成后，continuation 被调度执行（捕获的上下文或线程池），状态机从挂起点恢复，局部变量从状态机字段还原。
4. 已完成的 Task 直接同步继续，无挂起开销。

关键认知：`await` 不阻塞线程、`async` 不创建线程；异步 IO 完成靠 IO 完成端口等机制，不占用等待线程。

### 110. `Task` 和 `Thread` 有什么区别？

**答案：**

- `Thread`：一个真实的操作系统线程，1 MB 栈预留，创建销毁昂贵，直接控制线程生命周期。
- `Task`：一次异步操作的抽象——可能在线程池线程上执行（CPU 工作），也可能纯粹表示 IO 完成的通知（不占用任何线程）。

Task 支持组合（`WhenAll`／`WhenAny`）、延续、取消、异常传播、返回值。业务层一律优先 Task；需要独占线程、设置线程优先级或 STA 时才直接用 Thread。

### 111. `Task.Run()` 和直接调用 `async` 方法有什么区别？

**答案：**

```csharp
await DoAsync();                 // 在当前上下文开始执行，遇到真正的异步点让出
await Task.Run(() => Compute()); // 把委托提交到线程池队列执行
```

- 直接调用异步方法：方法体在当前同步上下文开始跑，异步等待点之后才可能切换。
- `Task.Run`：显式占用线程池线程，用于把 CPU 密集工作移出当前线程（如移出 UI 线程）。

误用：用 `Task.Run` 包装纯 IO 调用不带来任何收益，只是多一次线程调度；服务器高并发下大量 `Task.Run` 会耗尽线程池队列。

### 112. `SynchronizationContext` 是什么？

**答案：**

"把回调投递回特定环境执行"的调度抽象。UI 框架（WinForms／WPF／Unity 主线程）安装了 post 回 UI 线程的上下文。

`await` 默认捕获当前 `SynchronizationContext`，恢复执行时 Post 回原环境——这就是 `await` 后代码能继续操作 UI 控件的原因。没有同步上下文的环境（控制台、ASP.NET Core）恢复到线程池。`ConfigureAwait(false)` 声明不依赖原上下文，库代码应使用以提升性能并避免死锁。

### 113. 什么是线程安全？如何保证？

**答案：**

多线程并发访问共享数据时，不加额外同步也能得到正确结果。

保证手段（按优先级）：

1. 避免共享：线程本地数据、消息传递、不可变对象。
2. 共享但只读：初始化后不再修改的对象天然线程安全。
3. 必须共享可变状态：`lock` 互斥、`Interlocked` 原子操作、并发集合（`ConcurrentDictionary`）。

### 114. `lock` 的原理是什么？

**答案：**

编译器把 `lock (obj)` 展开为 `Monitor.Enter(obj)` + `try/finally { Monitor.Exit(obj) }`——异常路径也保证释放。Monitor 基于对象头的同步块索引（thin lock → 竞争时膨胀为同步块），竞争线程进入内核等待队列。

```csharp
private readonly object _lock = new();

void Add()
{
    lock (_lock)
    {
        count++;
    }
}
```

纪律：锁对象用 `private readonly object` 私有字段；不锁 `this`（外部可见，可能与其他代码死锁）、不锁 `Type`／字符串（全局共享，跨代码竞争）、锁粒度尽量小。

### 115. `lock` 为什么不能用于跨 `await` 的异步临界区？

**答案：**

`lock` 要求进入和退出在同一线程同步配对；`await` 挂起后恢复可能在另一个线程池线程上，且持有锁挂起期间其他等待该锁的线程全部阻塞——语法直接禁止 `lock` 块内 `await`。

异步互斥用 `SemaphoreSlim`：

```csharp
private readonly SemaphoreSlim semaphore = new(1, 1);

async Task ExecuteAsync()
{
    await semaphore.WaitAsync();
    try
    {
        await DoAsync();
    }
    finally
    {
        semaphore.Release();
    }
}
```

### 116. 什么是死锁？如何避免？

**答案：**

多个线程互相持有对方需要的资源并循环等待，全部无法推进。

四个必要条件：互斥、占有且等待、不可剥夺、循环等待——破坏任意一个即可预防。工程手段：

1. 统一锁获取顺序（按锁对象排序）。
2. 缩小锁范围、避免嵌套锁、不持锁调用外部回调。
3. 限时锁（`Monitor.TryEnter(timeout)`）打破"永久等待"。
4. 异步场景避免"同步等待异步"（见第 127 题）。

### 117. volatile 的作用是什么？与 Interlocked 有什么区别？

**答案：**

- `volatile`：保证读写的可见性与顺序性——读总是取最新值、不与相邻操作重排；但读—改—写复合操作（`count++`、先检查再赋值）仍然不是原子的。
- `Interlocked`：硬件级原子读改写（`Increment`、`Exchange`、`CompareExchange`），可见性 + 原子性兼得。

选择：标志位、单变量的发布读取用 volatile；计数器与状态更新用 Interlocked；多字段不变量用 lock。

### 118. 什么是原子操作？为什么 `count++` 不是原子操作？

**答案：**

原子操作对其他线程表现为不可分割的单步。`count++` 实际是三步：读 count 到寄存器、加一、写回——两个线程交错执行时丢失更新（双方都基于旧值加一，只体现一次）。

```csharp
Interlocked.Increment(ref count); // 原子递增
```

### 119. 什么是 CAS？

**答案：**

CAS（Compare-And-Swap）：仅当内存当前值等于预期值时才写入新值，返回是否成功（或观察到的旧值）。`Interlocked.CompareExchange` 是 .NET 的 CAS。

```csharp
int oldValue = Volatile.Read(ref value);

while (true)
{
    int newValue = oldValue + 1;
    int observed = Interlocked.CompareExchange(
        ref value, newValue, oldValue);

    if (observed == oldValue)
        break;

    oldValue = observed; // 失败，用观察到的最新值重试
}
```

CAS + 重试是无锁编程的基础；需要注意 ABA 问题（第 120 题）与高竞争下的重试风暴。

### 120. 什么是 ABA 问题？

**答案：**

线程读到值 A 后暂停；另一线程把它改成 B 又改回 A；第一个线程恢复后 CAS 发现值仍是 A，误判"期间没有变化"而成功交换——但对象的身份或中间状态已经变了。

解决：附加版本号／代数（值+版本组成复合比较）、.NET 引用类型可利用内存回收器保证引用不被复用（托管堆缓解了 ABA，但对象池复用场景仍会发生）。

### 121. ConcurrentDictionary 如何保证线程安全？复合操作怎么处理？

**答案：**

单次操作（读写键值）内部用细粒度锁（每桶一锁）保证线程安全，多线程并发读写不需要外部加锁。

复合操作必须用原子接口：`TryAdd`、`TryGetValue`、`TryUpdate`、`GetOrAdd`、`AddOrUpdate`——`ContainsKey` 再赋值是两个操作，中间可能被其他线程改变。

```csharp
dict.GetOrAdd(key, k => CreateValue(k));        // 不存在才创建
dict.AddOrUpdate(key, k => 1, (k, v) => v + 1); // 计数器
```

注意：`GetOrAdd`／`AddOrUpdate` 的工厂委托在竞争下可能被执行多次（值只用一个），不要在工厂里写副作用。

### 122. 无锁队列的原理和优缺点是什么？

**答案：**

原理：CAS 原子修改队头／队尾指针推进状态，失败则重试，全程不加锁。SPSC（单生产者单消费者）场景退化为读写两个索引的简单循环缓冲，无竞争 CAS，最快；MPMC 需要多指针协同与重试。

优缺点：低竞争下避免线程阻塞与上下文切换；但 CAS 重试、缓存一致性流量在高竞争下反而更慢，且实现极易出错（ABA、内存序、内存回收）。.NET 内置 `ConcurrentQueue<T>`（分段无锁）已经过验证，优先使用而非自写。

### 123. `Task.WhenAll()` 与 `Task.WhenAny()` 有什么区别？

**答案：**

- `WhenAll`：等待全部完成；任一失败则整体标记失败并聚合异常（遍历 `Exception.InnerExceptions`）；全部成功则结果数组按序对应。
- `WhenAny`：返回最先完成的那个 Task，用于超时竞速（任务 vs `Task.Delay`）、任意源可用即用。

```csharp
await Task.WhenAll(task1, task2);

Task completed = await Task.WhenAny(task1, task2);
await completed; // 观察获胜任务的结果或异常
```

`WhenAny` 不会取消或观察剩余任务——未处理的失败任务异常最终由 TaskScheduler 兜底，需自行处理。

### 124. CancellationToken 如何取消异步任务？

**答案：**

协作式取消：调用方 `Cancel()` 置位信号，执行方在检查点响应——轮询 `IsCancellationRequested`、`ThrowIfCancellationRequested()` 抛出、把 token 传给下游 IO/延迟调用。

```csharp
async Task WorkAsync(CancellationToken token)
{
    foreach (var item in items)
    {
        token.ThrowIfCancellationRequested();
        await ProcessAsync(item, token);
    }
}
```

要点：取消不会强杀线程，任务自己决定何时何地响应；注册 `Register()` 可挂清理回调；区分"被取消的任务"与"任务内部用了 token"——`TaskCanceledException` 表示该任务被取消。

### 125. 为什么通常不推荐使用 `async void`？

**答案：**

两个问题：

1. 调用方无法 await——完成时机不可观察，无法编排后续逻辑。
2. 异常无法被调用方捕获，直接打上线程池/SynchronizationContext，未处理即进程崩溃。

规则：异步方法返回 `Task`／`Task<T>`；`async void` 仅允许用于事件处理器签名（无法改签名时），且方法内必须自己兜住异常。`async void` 的单元测试、错误处理、组合全部失效。

### 126. ConfigureAwait(false) 有什么作用？

**答案：**

让后续代码不等待捕获的同步上下文，直接在线程池继续执行。

使用规则：

1. 通用库代码：所有 await 都加 `ConfigureAwait(false)`——不依赖调用方环境、避免 UI 死锁、减少 Post 调度开销。
2. 应用层代码（需要回到 UI／主线程更新界面）：不加。
3. 没有 SynchronizationContext 的环境（控制台、ASP.NET Core）加不加无行为差异。

Unity 场景特殊：Unity 的同步上下文保证 await 后回主线程，引擎 API 必须主线程调用——业务代码不加 `ConfigureAwait(false)`。

### 127. 为什么 `.Result`、`.Wait()` 可能造成死锁？

**答案：**

经典死锁链：UI 线程同步等待任务（阻塞且不泵消息）→ 任务完成回调需要 Post 回这个 UI 线程的上下文才能继续 → 上下文被自己阻塞，互相等待。

```csharp
// UI 线程上：死锁风险
var result = GetDataAsync().Result;
```

修复：整条调用链异步化，`await` 代替同步等待；库代码加 `ConfigureAwait(false)` 切断对上下文的依赖。没有同步上下文时不会死锁，但同步阻塞仍会造成线程池饥饿（大量线程在等任务，池线程耗尽）——同步等待异步在任何环境都是反模式。

### 128. `Task<T>` 与 `ValueTask<T>` 如何选择？

**答案：**

- `Task<T>`：引用类型，可任意存储、多次 await、组合；同步完成路径也要分配（或走缓存池）。
- `ValueTask<T>`：值类型包装，同步完成时零堆分配，适合"高频调用且大多数同步完成"的接口（如带缓存的读取）。

约束：`ValueTask<T>` 默认只应消费一次，不能多次 await、不能并发使用，需要复用时 `AsTask()` 转一次并保存 Task。默认选 `Task<T>`，有分配分析数据证明热点后再换。

### 129. `Thread.Sleep()` 与 `Task.Delay()` 有什么区别？

**答案：**

- `Thread.Sleep`：阻塞当前线程，期间什么都不做。
- `Task.Delay`：返回延迟后完成的 Task，`await` 挂起方法但线程被释放，可携带 CancellationToken 提前取消。

```csharp
await Task.Delay(500, cancellationToken);
```

异步代码中一律 `Task.Delay`；`Thread.Sleep` 只存在于不能 await 的极少数同步上下文。两者都不是高精度计时器（Windows 定时器精度约 15ms）。

## 九、异常处理

### 130. `throw;` 与 `throw ex;` 有什么区别？

**答案：**

`throw;` 保留原始异常的调用栈；`throw ex;` 把栈重置为当前抛出位置，丢失原始故障点。catch 后重新抛出一律用 `throw;`。

```csharp
try
{
    Execute();
}
catch (Exception ex)
{
    Log(ex);
    throw;
}
```

需要跨时间／跨线程传递异常时用 `ExceptionDispatchInfo.Capture(ex).Throw()` 保留原始栈。

### 131. finally 在什么时候执行？

**答案：**

正常离开 try、`return`、异常展开，三种路径都保证执行 finally——这是资源清理放 finally 的依据。

边界：进程强制终止（`Environment.FailFast`、`Process.Kill`、掉电）不会执行。finally 内抛新异常会掩盖原始异常（原异常丢失），清理代码要自行兜住异常。

### 132. 什么是异常过滤器？什么时候使用 `TryParse` 而不是异常？

**答案：**

`catch (Exception ex) when (condition)`：进入 catch 前先求值过滤器，不成立则继续匹配下一个 catch——不展开栈、不捕获再重抛。

```csharp
catch (IOException ex) when (ShouldRetry(ex))
{
    // 只处理满足重试条件的 I/O 错误
}
```

异常 vs 返回值：可预期的输入校验失败（字符串转数字、字典取值）用 `TryParse`／`TryGetValue`——异常构造与栈展开成本高，不应作为常规控制流；空 catch 吞掉无法处理的错误同样禁止，至少要记录。

## 十、设计与测试基础

### 133. 什么是里氏替换原则？

**答案：**

子类对象必须能替换父类对象使用，且不破坏父类承诺的行为契约。

契约要求：子类不得加强前置条件（对输入更挑剔）、不得削弱后置条件（返回更差的结果）、不得破坏不变量。反例：父类承诺"写操作总是成功"，子类重写后部分情况抛异常——语法上是合法重写，行为上违反 LSP。

判断标准是行为契约而非语法赋值：凡是"能赋值给父类引用但语义上不再成立"的继承，就是违反 LSP，通常意味着应该用组合或拆分类型。

### 134. Mock 与 Stub 有什么区别？

**答案：**

- Stub：替身，提供预设返回值，控制被测对象的依赖环境（喂固定数据）。
- Mock：验证交互，断言"是否以预期参数调用了预期方法"。

区别的本质：Stub 服务于输入安排（state），Mock 服务于行为验证（interaction）。优先验证可观察结果（状态断言）；只有"调用本身是业务契约"（如"下单成功必须发一次通知"）才断言调用次数——过度验证实现细节会让重构寸步难行。

## 十一、运行时与执行模型

### 135. C#、IL、CLR、JIT 与 AOT 分别是什么？

**答案：**

- C# 是语言；IL 是编译产物的中间指令；CLR 是运行时（类型系统、GC、异常、互操作）；JIT 在运行时把 IL 编译成本机机器码。

执行链路：C# 源码 → Roslyn 编译成含 IL 与元数据的程序集 → 运行时 JIT 按需编译执行。

JIT 的工作方式：方法首次调用时才编译成机器码并缓存，后续调用直接执行缓存；没有调用的方法永不编译。分层编译先快速生成弱优化代码保证启动，热点方法再由后台重编译做内联、去虚拟化、循环展开，还能利用当前 CPU 指令集（AVX 等）。收益是一份 IL 跨架构运行 + 运行时信息的深度优化。

代价：首次调用的编译开销（游戏主线程第一次走到复杂方法出现 JIT 卡顿）、IL 元数据常驻内存、平台限制——iOS 与游戏主机禁止运行时生成机器码，只能 AOT。游戏项目缓解 JIT 卡顿：加载阶段主动预热热点方法、IL2CPP 全程消除 JIT、控制泛型实例化数量。

托管 GC 只管托管内存，文件、连接、原生句柄仍需 Dispose／using。与 C++ 的对比见第 22 题和 [C++ 专题](C++高频题整理.md)。Mono JIT 与 IL2CPP 的执行流程对比见 [Unity 专题](Unity高频题整理.md)第 79 题。

### 136. IL2CPP 的工作原理是什么？

**答案：**

IL2CPP 是 Unity 的 AOT 编译后端，链路发生在打包阶段：C# → Roslyn 编译成 IL → IL2CPP 转换器翻译成等价 C++ 源码 → 目标平台本地工具链（clang／MSVC）编译成原生机器码 → 运行时由 libil2cpp 提供 GC、元数据、异常、线程等托管服务。运行时没有任何 JIT 环节。

与 Mono JIT 对比：

| 维度 | Mono JIT | IL2CPP |
| --- | --- | --- |
| 编译时机 | 运行时按需 | 打包时全部完成 |
| 首次调用 | 有 JIT 编译开销 | 无，性能稳定 |
| 构建时间 | 快 | 明显更长 |
| 平台 | 受限 | iOS／主机唯一选择 |
| 调试 | 托管调试友好 | 原生调试，符号复杂 |

AOT 特有的坑：

1. 托管代码裁剪（Stripping）会移除静态分析认为未使用的类型——反射、按字符串构造的序列化路径可能失效，用 `link.xml`／`[Preserve]` 保留。
2. 不支持 `Reflection.Emit`；静态分析覆盖不到的泛型值类型实例化会报 AOT 泛型缺失，需显式引用或生成代码。
3. 它仍是托管运行时：GC、元数据、托管异常都还在，不是"转成 C++ 就没有托管语义"。

追问题"怎么解决 JIT 卡顿"与本题直接相关：预热把编译开销挪到加载阶段，IL2CPP 从机制上消除 JIT，是 iOS 等平台唯一选择。

