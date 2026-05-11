# go语言里常用的设计模式

## 一、创建型（Creational Patterns）

👉 关注：对象怎么创建

### 1. Abstract Factory（抽象工厂）

- 核心：创建“一组相关对象”的接口
- 理解：不是造一个对象，而是造一整套“产品族”
- 例子：数据库驱动（MySQL / PostgreSQL）统一接口
- Go特点：Go很少用复杂抽象工厂（更偏函数式）

### 2. Builder（建造者）✔

- 核心：一步步构建复杂对象
- 理解：把构造过程拆开（链式调用）
- 例子：

```go
user := NewUserBuilder().Name("Tom").Age(18).Build()
```

- 适用：参数很多 / 可选参数多

### 3. Factory Method（工厂方法）✔

- 核心：用函数代替 new
- 理解：把创建逻辑封装起来
- 例子：

```go
func NewLogger(type string) Logger
```

- 适用：Go中非常常见

### 4. Object Pool（对象池）✔

- 核心：复用对象，避免频繁创建
- 理解：像连接池
- Go典型：
  - `sync.Pool`
- 适用：高频创建/销毁对象

### 5. Singleton（单例）✔

- 核心：全局只有一个实例
- Go实现：

```go
var once sync.Once
```

- 典型场景：
  - 配置
  - Logger

## 二、结构型（Structural Patterns）

👉 关注：对象如何组合

### 6. Bridge（桥接）

- 核心：抽象和实现分离
- 理解：接口 + 多实现解耦
- Go：接口天然支持 → 很常见但不强调模式

### 7. Composite（组合）

- 核心：树形结构统一处理
- 例子：文件系统（文件 + 文件夹）
- 统一接口操作

### 8. Decorator（装饰器）✔

- 核心：动态增强功能
- Go典型：

```go
http.Handler -> middleware
```

- 理解：一层一层包

### 9. Facade（外观）

- 核心：对复杂系统提供简单接口
- 例子：SDK封装

### 10. Flyweight（享元）

- 核心：共享对象减少内存
- 例子：字符串池

### 11. Proxy（代理）✔

- 核心：控制访问
- 类型：
  - 远程代理（RPC）
  - 权限代理
- Go例子：client wrapper

## 三、行为型（Behavioral Patterns）

👉 关注：对象如何交互

### 12. Chain of Responsibility（责任链）

- 核心：请求沿链传递
- Go典型：HTTP middleware

### 13. Command（命令）

- 核心：把请求封装成对象
- 好处：可排队/撤销

### 14. Mediator（中介者）

- 核心：对象通过中介通信
- 避免：对象之间直接耦合

### 15. Memento（备忘录）

- 核心：保存对象状态（快照）
- 场景：撤销功能

### 16. Observer（观察者）✔

- 核心：发布-订阅
- Go实现：

```go
channel
```

- 例子：事件通知

### 17. Registry（注册表）

- 核心：全局管理对象
- 例子：插件系统

### 18. State（状态）

- 核心：状态驱动行为
- 例子：订单状态机

### 19. Strategy（策略）✔

- 核心：算法可替换
- Go实现：函数/接口注入
- 例子：

```go
sortFunc := quickSort or mergeSort
```

### 20. Template（模板方法）

- 核心：定义流程骨架
- Go：较少（因为没有继承）

### 21. Visitor（访问者）

- 核心：操作与结构分离
- 适用：AST、编译器

## 四、同步模式（Synchronization）

👉 关注：线程/协程协作

### 22. Condition Variable

- 等待条件满足再执行

### 23. Mutex（锁）

- 互斥访问

### 24. Monitor

- 锁 + 条件变量组合

### 25. Read-Write Lock

- 读多写少优化

### 26. Semaphore（信号量）✔

- 控制并发数量
- Go常用：

```go
chan struct{}
```

## 五、并发模式（Concurrency）

👉 Go的核心优势

### 27. N-Barrier

- 等待N个任务完成

### 28. Bounded Parallelism ✔

- 限制并发数量
- Go典型：

```go
worker pool
```

### 29. Broadcast

- 一对多通知

### 30. Coroutines

- 协程（Go自带 goroutine）

### 31. Generators ✔

- 生成器（channel实现）

### 32. Reactor

- IO多路复用（类似 epoll）

### 33. Parallelism ✔

- 并行处理任务

### 34. Producer-Consumer

- 生产者消费者模型

## 六、消息模式（Messaging）

### 35. Fan-In ✔

- 多输入 → 单输出

### 36. Fan-Out ✔

- 单输入 → 多输出

### 37. Futures & Promises

- 异步结果占位

### 38. Publish/Subscribe ✔

- 发布订阅
- Go：channel + goroutine

### 39. Push & Pull

- pipeline模式

## 七、稳定性模式（Stability）

👉 分布式系统核心

### 40. Bulkheads

- 隔离资源，防止雪崩

### 41. Circuit Breaker（熔断）✔

- 请求失败过多 → 直接拒绝
- 常见：Hystrix

### 42. Deadline

- 超时控制

### 43. Fail-Fast

- 快速失败

### 44. Handshaking

- 先问能不能处理再发请求

### 45. Steady-State

- 资源必须可回收

## 八、性能分析（Profiling）

### 46. Timing Functions ✔

- 给函数加耗时统计
- Go常见：

```go
defer timeTrack()
```

## 九、Go惯用法（Idioms）

### 47. Functional Options ✔

- Go最重要设计模式之一
- 核心：用函数配置对象
- 例子：

```go
NewServer(WithTimeout(10))
```

## ChatGPT总结

如果你是 Go 开发者，优先掌握这些👇：

### ⭐ 最重要（高频）

- Factory Method
- Singleton
- Decorator（middleware）
- Observer（channel）
- Strategy
- Functional Options

### ⭐ Go并发核心

- Fan-in / Fan-out
- Bounded Parallelism
- Producer-Consumer
- Semaphore

### ⭐ 分布式必备

- Circuit Breaker
- Fail-Fast
- Timeout（Deadline）

## 参考

- [go-patterns](https://github.com/tmrts/go-patterns)
  - 这个仓库和传统《设计模式》最大的区别是：
    - 👉 它不是OO设计模式，而是“Go工程模式”
    - 少继承 → 多组合
    - 少类 → 多函数
    - 少抽象 → 多并发

# Anti-Patterns

## 一、什么是Anti-Patterns？

👉 定义很简单：

```txt
看起来像解决方案，但实际上会导致更糟糕结果的写法
```

可以理解为：

- 设计模式 = 最佳实践
- 反模式 = 常见错误套路

## 二、仓库里的核心 Anti-Patterns

### 1. 🔥 God Object（上帝对象）

#### 核心问题

一个对象做了“所有事情”

#### Go里的典型表现

```go
type Service struct {
    db *sql.DB
    cache *redis.Client
    logger Logger
    userLogic ...
    orderLogic ...
    paymentLogic ...
}
```

#### 问题

- 职责混乱
- 难维护
- 难测试

#### 正确做法

👉 拆分成多个小模块（组合优于继承）

### 2. 🔥 Spaghetti Code（面条代码）

#### 核心问题

代码流程混乱、跳来跳去

#### Go表现

- if / else 套娃
- goroutine乱飞
- channel关系不清

#### 问题

- 无法读
- 无法debug

#### 建议

👉 明确结构：

- pipeline
- 分层（handler / service / repo）

### 3. 🔥 Copy-Paste Programming（复制粘贴式编程）

#### 核心问题

到处复制代码

#### Go表现

```go
func GetUser() {}
func GetOrder() {}
func GetProduct() {}
```

逻辑几乎一样，只是换个名字

#### 问题

- 修改一处 → 忘记改其他
- bug成倍增长

#### 正解

👉 抽象公共逻辑（函数 / interface）

### 4. 🔥 Premature Optimization（过早优化）

#### 核心问题

还没性能问题就开始优化

#### Go表现

- 滥用 `sync.Pool`
- 手写内存管理
- 过度使用 unsafe

#### 问题

- 代码复杂度暴涨
- 收益很小

#### 正解

👉 原则：先跑，再测，再优化

### 5. 🔥 Reinventing the Wheel（重复造轮子）

#### 核心问题

明明有成熟库，还自己实现

#### Go表现

- 自己写 HTTP 框架
- 自己写 JSON parser
- 自己实现连接池

#### 问题

- Bug多
- 不稳定

#### 正解

👉 优先用标准库：

- net/http
- database/sql

### 6. 🔥 Golden Hammer（金锤子）

#### 核心问题

“我只会这一招，到处用”

#### Go表现

- 什么都用 channel
- 什么都 goroutine
- 什么都微服务

#### 问题

- 过度设计
- 性能反而更差

#### 正解

👉 工具匹配问题，而不是反过来

### 7. 🔥 Overengineering（过度设计）

#### 核心问题

把简单问题搞复杂

#### Go表现

- 小项目搞微服务
- 滥用接口
- 不必要的抽象层

#### 问题

- 开发成本高
- 可读性差

#### 正解

👉 Go哲学：简单优先（Simple > Clever）

### 8. 🔥 Not Invented Here（非我发明）

#### 核心问题

#### 拒绝使用外部方案

#### 表现

- 不用开源库
- 不信任第三方代码

#### 问题

- 重复劳动
- 技术封闭

### 9. 🔥 Vendor Lock-in（供应商锁定）

#### 核心问题

严重依赖某个服务/库

#### Go表现

- 强绑定某云厂商 SDK
- 无法迁移

#### 正解

👉 抽象接口层（adapter）

### 10. 🔥 Hard Coding（硬编码）

#### 核心问题

把配置写死

#### Go表现

```go
const DB_URL = "root:123456@tcp(127.0.0.1:3306)"
```

#### 问题

无法部署不同环境

#### 正解

👉 配置化（env / config）

### 11. 🔥 Global State Abuse（滥用全局变量）

#### 核心问题

到处用全局变量

#### Go表现

```go
var DB *sql.DB
```

#### 问题

- 难测试
- 并发问题

#### 正解

👉 依赖注入（DI）

### 12. 🔥 Tight Coupling（紧耦合）

#### 核心问题

模块之间强依赖

#### 表现

改一个地方 → 全部崩

#### 正解

👉 interface 解耦

## 三、给你一个关键总结

### ✅ 好代码 vs 坏代码

| 维度 | 好模式 | 反模式 |
| ---- | ------ | ------ |
| 结构 | 清晰   | 混乱   |
| 扩展 | 容易   | 困难   |
| 维护 | 简单   | 痛苦   |
| 并发 | 可控   | 混乱   |

## 四、最容易踩的几个坑（Go开发者必看）

如果你现在在写 Go 项目，最危险的是：

### ⚠️ 高频反模式

- God Object（一个 struct 做所有事）
- 滥用 goroutine（泄漏 / 不可控）
- channel乱用（死锁）
- 过度抽象 interface
- 复制粘贴代码

## 五、一个很现实的建议

👉 学设计模式：

看“该怎么做”

👉 学反模式：

看“千万别这么做”
