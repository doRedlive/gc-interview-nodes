# Android 与 JVM 客户端面试题

> 共 7 题。按主题分类，答案直接放在题目后。

[返回题库总目录](面试题库总目录.md)

## 目录

| 模块 | 题号 |
| --- | --- |
| [一、代码生成、反射与类加载](#%E4%B8%80%E3%80%81%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%E3%80%81%E5%8F%8D%E5%B0%84%E4%B8%8E%E7%B1%BB%E5%8A%A0%E8%BD%BD) | 1–3 |
| [二、协程与内存诊断](#%E4%BA%8C%E3%80%81%E5%8D%8F%E7%A8%8B%E4%B8%8E%E5%86%85%E5%AD%98%E8%AF%8A%E6%96%AD) | 4–5 |
| [三、UI](#%E4%B8%89%E3%80%81UI) | 6–7 |

## 一、代码生成、反射与类加载

### 1. KSP 在什么时候工作？与运行时反射有什么区别？

**答案：**

KSP（Kotlin Symbol Processing）在编译期工作：读取源码的声明、类型、注解等符号信息，生成新的源码或资源，生成物参与后续编译——零运行时开销、类型安全（Room、Hilt、Moshi 都基于此类方案）。限制：只能读不能改原源码。

运行时反射：运行期通过 Class 元数据查询类型、构造器、字段、方法与 RUNTIME 注解——灵活（插件、序列化、调试工具），但慢（查找与动态调用）、影响裁剪与混淆。

选择：结构已知用编译期生成（快且安全）；高度动态的插件化、运行时注入才用反射。

### 2. 运行时反射读取注解需要满足什么条件？

**答案：**

两个必要条件：

1. 保留策略为 RUNTIME：SOURCE 只在源码处理阶段可见，CLASS 写入字节码但 JVM 不暴露给反射。
2. 注解的目标元素允许（@Target 匹配读取的位置）。

工程注意：`@Inherited` 决定子类能否读到父类注解；可重复注解（`@Repeatable`）需按容器读取；混淆与代码裁剪可能删掉仅被反射访问的类和成员——需 keep 规则与裁剪白名单。

### 3. JVM／Android 类加载过程是什么？自定义 String 会覆盖系统 String 吗？

**答案：**

类加载五个阶段：加载（按全限定名读字节码／DEX，生成 Class 对象）→ 验证（字节码合法性与安全）→ 准备（静态字段分配并置默认值）→ 解析（符号引用转直接引用）→ 初始化（执行静态初始化块，惰性触发）。

双亲委派：类加载器先委托父加载器加载，父加载失败才自己加载——核心类库由 Bootstrap 加载器定义，保证 `java.lang.String` 永远来自平台而不是应用。

自定义 String 覆盖不了系统 String：

1. 全限定名必须完全相同才会冲突，而应用无法在 `java.lang` 包定义受保护的核心类。
2. 即使同名类存在，双亲委派让加载请求先到 Bootstrap，拿到的仍是系统版本。
3. 类的身份 = 全限定名 + 定义它的 ClassLoader——不同加载器定义的同名类是两个不同类型，互相赋值抛 ClassCastException。

## 二、协程与内存诊断

### 4. Kotlin 协程异常如何传播和处理？

**答案：**

结构化并发的传播规则：

1. `launch` 的子协程抛非取消异常 → 取消父 Job → 父取消连带取消所有兄弟——一个失败全军覆没。
2. `async` 的异常存进 Deferred，`await()` 时重新抛出；不 await 就泄漏成未处理异常。
3. 根作用域的未处理异常交给 `CoroutineExceptionHandler` 或平台默认处理（Android 上默认崩溃）。

处理手段：

1. `SupervisorJob`／`supervisorScope`：子任务独立失败不传染兄弟，适合"并行任务互不影响"的场景。
2. 可恢复的业务异常在协程内部 try/catch。
3. 不要捕获吞掉 `CancellationException`——它是取消机制的信号，捕获后要重新抛出。
4. 作用域绑定生命周期（Activity、ViewModel），销毁时 cancel。

### 5. LeakCanary 如何检测内存泄漏？

**答案：**

原理：对象销毁钩子（Activity onDestroy、Fragment 观察器）把"本应被回收的对象"放进弱引用观察；触发 GC 检查后若弱引用仍被保留（对象仍可达），说明有强引用链泄漏 → 生成 heap dump → Shark 引擎从 GC Root 遍历对象图，找出阻止回收的最短强引用路径（leak trace）。

结果解读：retained 不一定都是业务泄漏——缓存、单例持有的"设计内常驻"也会出现，按引用链是否合理判断；heap dump 会冻结进程数秒，只在 debug 构建启用。

### 6. Compose 与 XML View 系统的渲染模型有什么区别？

**答案：**

| 维度 | XML View | Compose |
| --- | --- | --- |
| UI 描述 | 命令式：inflate 出有状态 View 树，setter 修改节点 | 声明式：Composable 函数描述 UI，状态驱动 |
| 更新 | 手动同步（findView + setText） | State 变化自动触发重组（recposition） |
| 状态 | View 内部持有，易散落 | 单一数据源 + 单向数据流 |
| 性能 | 常规优化成熟 | 跳过未变化子树；依赖稳定性与重组范围控制 |

Compose 纪律：Composable 可能乱序、多次执行——保持快速、无副作用，副作用放 `LaunchedEffect` 等 Effect API；不稳定参数（List）会扩大重组范围。两套系统通过 AndroidView／ComposeView 互操作，支持渐进迁移。

### 7. Android 自定义 View 的 measure、layout、draw 流程是什么？

**答案：**

三阶段遍历（自顶向下 measure，再自顶向下 layout/draw）：

1. `onMeasure`：根据父级 MeasureSpec（模式 EXACTLY／AT_MOST／UNSPECIFIED + 尺寸）和自身内容算出测量尺寸，必须调用 `setMeasuredDimension` 收尾。
2. `onLayout`：ViewGroup 遍历子 View，用 `child.layout(l, t, r, b)` 分配实际位置与大小。
3. `onDraw`：在 Canvas 上绘制自身内容；ViewGroup 的 dispatchDraw 负责绘制子节点。

失效接口：`requestLayout()` 触发重新 measure + layout（布局变了）；`invalidate()` 只触发重绘（外观变了）——只改颜色用 invalidate，改尺寸用 requestLayout。

性能纪律：onDraw 每帧执行，不在其中创建 Paint／Path 等对象（提为字段）；自定义控件同步处理触摸事件与无障碍支持。
