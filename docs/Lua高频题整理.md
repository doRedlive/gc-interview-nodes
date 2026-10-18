# Lua 面试题与答案

> 共 15 题。按主题分类，答案直接放在题目后。

[返回题库总目录](面试题库总目录.md)

## 目录

| 模块 | 题号 |
| --- | --- |
| [一、语言与对象模型](#%E4%B8%80%E3%80%81%E8%AF%AD%E8%A8%80%E4%B8%8E%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B) | 1–6 |
| [二、GC 与生命周期](#%E4%BA%8C%E3%80%81GC%20%E4%B8%8E%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F) | 7–10 |
| [三、常见函数与协程](#%E4%B8%89%E3%80%81%E5%B8%B8%E8%A7%81%E5%87%BD%E6%95%B0%E4%B8%8E%E5%8D%8F%E7%A8%8B) | 11–15 |

## 一、语言与对象模型

### 1. Lua 有哪些基本数据类型？哪些值为假？

**答案：**

八种基本类型：nil、boolean、number、string、function、table、userdata、thread。条件判断中只有 nil 和 false 为假——0 和空字符串都是真值（与 C 语言不同，是高频考点）。

补充：Lua 5.3／5.4 的 number 区分整数和浮点子类型（64 位）；thread 表示 Lua 协程，不是操作系统线程。

### 2. Lua table 的底层结构和特点是什么？

**答案：**

标准实现（PUC-Lua）是数组区 + 哈希区的混合结构：连续正整数键进入数组区直接索引，其他键进哈希区。table 一身兼任数组、字典、对象、命名空间。

要点：

1. 赋 nil 即删除键；`t[k] = nil` 后 k 不存在。
2. 序列中间有空洞（nil）时 `#t` 的结果不确定——数组部分与哈希部分的边界由实现决定，`#t` 只对"无空洞的连续序列"有保证。
3. 元素数量增长触发 rehash，按数组区与哈希区的实际分布重新分配两区大小——预先构造时避免逐个插入（`table.create`／构造器一次性填充可减少 rehash）。

### 3. pairs 和 ipairs 有什么区别？

**答案：**

- `ipairs`：从下标 1 开始逐个读 `t[1]`、`t[2]`……遇到第一个 nil 停止，只遍历连续数组部分。
- `pairs`：遍历整个 table 的所有键值对（经 next 函数），遍历顺序无保证，字典部分必须用它。

需要确定顺序时：把键收集到数组，排序后再按序取值。

### 4. 元表和元方法是什么？__index、__newindex 如何使用？

**答案：**

元表是控制一个值行为方式的 table，其中的元方法定义查找、运算、调用等钩子：`__index`（读缺失键）、`__newindex`（写缺失键）、`__add` 等算术、`__eq`／`__lt` 比较、`__call`（当函数调用）、`__tostring`、`__gc`。

- `__index` 可以是 table（缺键时转查该表——继承的实现基础）或 function（缺键时调用）。
- `__newindex` 可以是 table（写入转投到该表）或 function（拦截写入）——只对"不存在的键"触发，已有键直接读写。
- `rawget`／`rawset` 绕过元方法直接操作，常用于元方法内部避免无限递归。

### 5. Lua 如何实现面向对象？点号和冒号有什么区别？

**答案：**

基于 table + 元表的 `__index` 查找链：

```lua
local Actor = {}
Actor.__index = Actor
function Actor.new(hp)
    return setmetatable({hp = hp}, Actor)
end
function Actor:damage(amount)
    self.hp = self.hp - amount
end
local actor = Actor.new(100)
actor:damage(10) -- 等价于 actor.damage(actor, 10)
```

机制要点：

1. 实例是 table，元表的 `__index` 指向类表——方法查不到时去类表找，方法天然共享。
2. 继承：子类元表 `__index` 指向父类，形成查找链。
3. 点号 `.` 显式传 self，冒号 `:` 隐式传 self——定义与调用都要用冒号，混用导致 self 错位是最常见 bug。
4. 可变数据放实例 table，不要放类表（所有实例共享）。

### 6. 闭包和 upvalue 是什么？

**答案：**

闭包 = 函数 + 其捕获的环境。内部函数引用的外层局部变量称为 upvalue：外层函数返回后，只要闭包仍存活，upvalue 就被保留——Lua 用打开／关闭（open/closed upvalue）机制把栈上变量正确转移到堆上。

两个工程点：

1. upvalue 捕获的是变量本身（可读写），不是值快照——循环变量行为同 C# 闭包。
2. 事件、定时器长期持有闭包时，闭包连带持有其 upvalue 引用的所有对象——业务结束必须解除引用（置 nil／注销回调），这是 Lua 内存泄漏的常见来源。

## 二、GC 与生命周期

### 7. Lua 的 GC 是什么机制？

**答案：**

Lua GC 是标记—清扫（mark and sweep）：从根集合（全局表、registry、主线程栈）出发标记可达对象，清除不可达对象——孤立的循环引用可以被正确回收，无需手动打破。

版本演进：Lua 5.0 是全量 stop-the-world；Lua 5.1 起默认增量式（标记与清扫分步穿插运行，降低单次停顿）；Lua 5.4 新增分代模式（新对象按 minor GC 频繁回收，老对象偶发 major GC，适合对象多、多数朝生夕死的负载）。LuaJIT 有自己独立的增量实现。

调优：`collectgarbage("step")`／`setpause`／`setstepmul` 控制节奏；先观测 `count` 与分配速率再调参，游戏卡顿场景常用分帧手动 step 替代自动 GC。

### 8. Lua 如何检查和处理内存泄漏？

**答案：**

排查方法：

1. 反复执行"进入／退出同一业务流程"，用 `collectgarbage("count")` 对比每轮稳态内存——持续单调增长即有泄漏。
2. 检查全局表（`_G` 里本应为 local 的名字）、模块级缓存、事件／定时器回调表、未结束的协程、闭包 upvalue 链、C 侧 registry 引用。
3. 工具：打印 `_G` 新增键、`debug.getinfo` 定位分配源、xlua/sluabunlap 等桥接层的泄漏检测接口。

修复原则：业务结束解除强引用（回调注销、缓存键清除、协程 close）；桥接对象按"谁创建谁释放"明确契约。注意 `collectgarbage("count")` 不下降不代表泄漏——对象可能仍被合法引用，以"稳态是否增长"为准。

### 9. 弱表有什么作用？

**答案：**

元表设 `__mode` 为 "k"（弱键）、"v"（弱值）或 "kv"（双弱），表中对应方向的引用不阻止对象被 GC 回收，条目随对象消亡自动消失。

经典用途：对象 → 附加信息的映射（弱键：对象被回收，条目自动清理）、缓存（弱值：内存压力下自动淘汰）。

注意：条目随时可能消失，每次查询都要处理 nil；弱表只管 Lua 对象引用，不能替代文件句柄等外部资源的确定性释放。

### 10. Lua 与 C#／C++ 交互时如何管理对象生命周期？

**答案：**

桥接方式：userdata（C 对象包装）、registry 引用（`luaL_ref` 持有 Lua 对象）、托管侧句柄表（int 句柄映射到 C# 对象）。

核心问题：两侧 GC 互不知情——Lua 的 GC 看不见 C# 对象的引用关系，反之亦然，双向持有时单侧回收都可能悬垂。

工程契约：

1. 明确所有权方向：CS 持 Lua 用引用计数／句柄显式释放；Lua 持 CS 用 userdata 的 `__gc` 归还。
2. 生命周期绑定宿主：协程、事件回调随宿主对象销毁统一注销。
3. 异步回调返回时检查目标是否已销毁（句柄有效性检查），过期直接丢弃。
4. 退出顺序：先停回调与更新逻辑，再解除桥接引用，最后释放资源。

## 三、常见函数与协程

### 11. Lua 字符串如何拼接？大量拼接怎么优化？

**答案：**

Lua 字符串不可变、内部驻留（相同内容共享一份）。`..` 每次拼接都生成新串并复制全部内容——循环内 `..` 是 O(n²) 行为。

优化：片段收进数组，最后一次 `table.concat`（内部一次性分配、按序拷贝，O(n)）：

```lua
local parts = {}
for i = 1, 100 do parts[#parts + 1] = tostring(i) end
local text = table.concat(parts, ",")
```

补充：`string.format` 适合少量格式化；极大结果注意峰值内存，必要时分段输出。

### 12. table.sort 使用什么算法？比较函数怎么写？

**答案：**

标准 Lua 5.4 的 `table.sort` 是快速排序体系（含三数取中 pivot），不保证稳定；排序原地进行，不返回新表。

比较函数必须表达严格的"a 排在 b 前面"（`a.score < b.score`），用 `<=` 是无效排序器——排序过程可能触发 "invalid order function" 错误。需要稳定顺序时附加原始下标作次级关键字：

```lua
table.sort(list, function(a, b)
    if a.score ~= b.score then return a.score < b.score end
    return a.index < b.index
end)
```

### 13. Lua 协程的原理、状态和使用方法是什么？

**答案：**

协程是协作式的独立执行单元：`coroutine.resume` 启动／恢复，`coroutine.yield` 主动挂起并让出控制权，挂起期间的局部状态完整保留在协程栈中。

四种状态：suspended（挂起，可恢复）、running（正在运行）、normal（激活了别的协程）、dead（结束，不可恢复）。

```lua
local co = coroutine.create(function(x)
    local nextValue = coroutine.yield(x + 1)
    return nextValue * 2
end)
print(coroutine.resume(co, 10)) -- true, 11 —— yield 传出 x+1
print(coroutine.resume(co, 7))  -- true, 14 —— resume 传入 7 作为 yield 的返回值
```

要点：resume 首个返回值是成功标志（false 加错误对象表示出错）；yield 与 resume 是双向数据通道；协程不是线程——同一线程内死循环照样卡死所有逻辑，并行仍需多线程。

### 14. Lua 和 C 如何通过 C API 交互？

**答案：**

交互媒介是虚拟栈：C 与 Lua 之间所有数据经栈传递。

- C 调 Lua：`luaL_loadfile` 加载编译 → `lua_pushnumber` 等压入参数 → `lua_pcall` 保护调用 → 从栈顶取返回值。
- Lua 调 C：把 `lua_CFunction`（int (*)(lua_State*)）注册进全局表或字段，函数内用 `lua_tointeger`／`lua_tolstring` 读栈参数，压入结果并返回结果个数。

纪律：

1. 栈平衡：记录使用前栈深，返回前恢复，`lua_gettop` 校验。
2. 所有可以失败的调用（加载、pcall、tostring）都检查返回值。
3. userdata 的所有权：`__gc` 元方法统一释放，跨语言的异常（C++ 异常不可穿越 longjmp）用 pcall 包边界。

### 15. Lua 函数里一行报错，代码会继续执行吗？报错位置不确定时怎么定位？

**答案：**

不会继续：Lua 错误沿调用栈向上抛，中断当前函数及其所有调用者；未捕获时整个入口调用终止，回到宿主（C 侧 `lua_pcall` 返回错误码）。Lua 不存在"跳过出错行继续往下跑"。

防御与定位：

1. 保护调用：定时器回调、事件分发、单条消息处理用 `pcall`／`xpcall` 包裹，`xpcall` 附错误处理函数取 `debug.traceback`——防止一个 UI 回调炸掉整个分发循环。
2. 定位报错：`debug.traceback` 给完整调用栈；`debug.getinfo`／`debug.sethook`（行钩子）用于追踪偶发路径；线上用错误对象 + traceback 上报，避免重钩子拖慢运行。
3. 错误即值：`error({code = 1001, msg = "..."})` 抛 table，业务层按错误码分级处理，不靠字符串匹配。
