# Go 语言中的 `nosplit` 函数详解

`nosplit` 是 Go runtime 中的一个关键底层机制，用于标记某些函数“不可触发栈检查与栈扩张”。这些函数在执行时 **不能发生 stack split（栈分裂）**，因此是 Go 调度、GC、汇编代码中非常重要的部分。

## 1. 什么是 `nosplit` 函数？

在普通 Go 函数中，编译器会自动插入：

- 栈空间检查
- 栈不足时执行栈扩张（stack growth）

但带有 `nosplit` 标记的函数：

- **执行前不会检查栈空间**
- **不能触发栈扩张**
- **不能进入 runtime 的扩栈逻辑**

也就是说，它们必须在一个已经足够大的栈上执行。

## 2. 为什么需要 `nosplit`？

某些函数处在 **Go runtime 的最底层**，执行环境极其敏感，如果触发扩栈，会导致：

- 调度器进入递归
- GC 扫描阶段栈结构被改变
- runtime 自己依赖自己的功能，造成死循环
- 崩溃或未定义行为

因此，这些函数必须保证：

❌ 不能检测栈空间
❌ 不能扩栈
❌ 不能抢占
❌ 不能阻塞

### 常见使用场景包括：

- Go runtime 的调度器代码
- GC 相关函数
- 系统调用前后的状态管理
- 汇编级别的基本操作
- `memmove`, `memclr`, `systemstack` 等底层实现

## 3. 如何标记 `nosplit`？

### 在汇编中：

```asm
TEXT runtime·memmove(SB), NOSPLIT, $0-24
```

关键词 `NOSPLIT` 对应 nosplit 属性。

### 在 Go 代码中：

```go
//go:nosplit
func f() {
    ...
}
```

编译器会给该函数施加 nosplit 标记。

## 4. nosplit 的限制

因为 nosplit 函数不能触发栈扩张，所以必须严格控制其内容。

### ❌ 不能做的事情

- 不能调用普通 Go 函数（除非对方也是 `nosplit`）
- 不能分配内存（可能触发 GC）
- 不能写复杂逻辑
- 不能执行可能阻塞或引发调度的操作
- 不能递归

### ✔️ 可以做的事情

- 进行简单的计算
- 操作指针、寄存器等底层内容
- 调用其他无栈检查的函数
- 汇编指令层面的计算

## 5. 在哪里可以看到 nosplit？

你可以在 Go 源码中搜索：

- `//go:nosplit`
- `NOSPLIT`

常见于：

- `runtime/*.s`
- `runtime/stubs.go`
- `runtime/memmove_*.s`
- 调度器与 GC 路径

例如：

```asm
TEXT runtime·memclrNoHeapPointers(SB), NOSPLIT, $0-32
```

和：

```go
//go:nosplit
func systemstack(fn func()) {
    ...
}
```

## 6. 为什么不建议普通用户使用 nosplit？

因为 nosplit 是极其危险的：

- 一旦在不足的栈空间调用 nosplit 函数，程序会直接崩溃或出现隐性错误
- 写错一次，会破坏 runtime 的核心逻辑
- 调度器、GC、系统调用交界处尤其敏感

**除非你在写 runtime 或汇编，否则不要随意使用 `//go:nosplit`。**

## 📌 总结

| 项目               | 说明                                        |
| ------------------ | ------------------------------------------- |
| **nosplit 的定义** | 禁止栈检查和栈扩展的函数                    |
| **典型使用位置**   | runtime 底层、GC、调度器、汇编              |
| **不能做的事情**   | 调用普通函数、分配内存、递归、阻塞          |
| **能做的事情**     | 简单底层操作、调用其他 nosplit 函数         |
| **为什么需要它**   | 避免 runtime 在自身关键路径中递归触发栈扩展 |

`nosplit` 是 Go runtime 最底层、最危险但最关键的机制之一，使得 Go 的协程、GC 和调度器可以稳定运行。
