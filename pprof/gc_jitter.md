# GC 抖动（GC jitter / GC pause jitter）

GC 抖动指的是：

```txt
垃圾回收导致程序延迟突然不稳定
```

或者更直白：

```txt
程序本来跑得很稳，
GC一来，
延迟突然尖刺。
```

这是高并发系统里非常核心的问题。

## 一、什么叫"抖动"

例如你的接口：

平时：

```txt
1ms
2ms
1ms
2ms
```

很稳定。

但某一瞬间：

```txt
1ms
2ms
80ms   ← GC
1ms
2ms
```

这：

```txt
80ms 的尖刺
```

就是：

```txt
GC 抖动
```

## 二、为什么 GC 会导致抖动

因为 GC 不是"完全免费"的。

GC 要做：

- 扫描对象
- 标记对象
- 清理对象
- 修改指针
- 协调 goroutine
- 与 runtime 调度配合

即使 Go 已经是：

```txt
低停顿 GC
```

仍然会产生：

```txt
短暂暂停
CPU抢占
cache miss
协程调度延迟
```

## 三、Go 的 GC 抖动来源

Go 里主要有几种。

### 1. STW（Stop The World）

最经典。

GC 某些阶段必须：

```txt
暂停所有 goroutine
```

虽然 Go 现在已经很短：

通常：

```txt
几十微秒 ~ 几毫秒
```

但高 QPS 服务：

```txt
1ms 都可能是事故
```

### 2. Assist（GC Assist）

这是 Go 特有的重要机制。

Go GC 不是后台线程全干。

而是：

```txt
你分配内存，
你也得帮忙GC。
```

即：

```txt
malloc 太快
→ 你必须协助 mark
```

于是：

某个业务 goroutine：

本来在处理请求。

突然：

```txt
被runtime拉去帮GC
```

于是延迟突然上升。

这也是 GC 抖动。

### 3. 大量对象分配

例如：

```go
fmt.Sprintf(...)
bytes.Buffer{}
map[string]interface{}{}
json.Marshal(...)
```

疯狂产生：

```txt
临时对象
```

结果：

```txt
GC频率暴增
```

于是：

```txt
GC一会儿来一次
```

延迟就开始抖。

### 4. 大堆扫描

heap 太大。

GC 要扫描：

```txt
几十GB对象
```

即使：

对象没死。

扫描成本仍然巨大。

## 四、GC 抖动最明显的现象

### 1. P99/P999 突然升高

例如：

```txt
P50: 1ms
P99: 120ms
```

典型 GC。

### 2. CPU 周期性尖刺

你会看到：

```txt
CPU突然飙高
```

因为 GC worker 在跑。

### 3. QPS 周期性下降

GC 时：

业务 goroutine 被抢占。

### 4. goroutine latency 增加

特别：

```txt
channel
mutex
网络poller
```

都可能被影响。

### 五、如何观察 GC 抖动

#### 方法1：GODEBUG

最经典。

```sh
GODEBUG=gctrace=1
```

输出：

```txt
gc 12 @5.682s 1%: 0.10+2.3+0.15 ms clock
```

这里：

```txt
0.10 ms STW
2.3 ms 并发mark
0.15 ms STW
```

#### 方法2：pprof

看：

```sh
go tool pprof
```

会看到：

```txt
runtime.gcBgMarkWorker
runtime.scanobject
```

占用很多 CPU。

#### 方法3：trace（最强）

```sh
go test -trace trace.out
go tool trace trace.out
```

直接看到：

- STW
- mark
- assist
- goroutine阻塞

这是最专业方式。

### 六、Go 为什么会有 GC Assist

因为 Go 选择了：

```txt
低延迟GC
```

而不是：

```txt
高吞吐GC
```

所以：

Go runtime：

```txt
让业务线程一起参与GC
```

避免：

```txt
后台GC线程来不及
导致巨大STW
```

这是 Go GC 的核心设计。

### 七、真正导致 GC 抖动的根源

核心只有一句：

```txt
对象分配速度 > GC处理速度
```

于是：

- assist增加
- GC频率增加
- heap增长
- scan增长
- pause增长

最终：

```txt
延迟抖动
```

### 八、如何减少 GC 抖动（Go核心优化）

#### 1. 少分配对象（最重要）

这是第一原则。

例如：

避免：

```go
fmt.Sprintf
[]byte(string)
map[string]interface{}
```

多用：

```go
append
strconv.AppendInt
对象复用
```

#### 2. 对象池 sync.Pool

例如：

```go
var pool = sync.Pool{
    New: func() any {
        return new(bytes.Buffer)
    },
}
```

减少短生命周期对象。

#### 3. 减少指针对象

Go GC：

```txt
扫描指针
```

不是扫描 int。

所以：

```go
[]int
```

比：

```go
[]*int
```

GC友好很多。

#### 4. 控制 heap 增长

Go 默认：

```sh
GOGC=100
```

意思：

```txt
heap翻倍才GC
```

可调整：

```sh
GOGC=50
```

更频繁 GC。

或者：

```sh
GOGC=200
```

减少 GC 次数。

#### 5. Go 1.19+ 的内存限制

```sh
GOMEMLIMIT
```

非常重要。

现代 Go 服务推荐配置。

### 九、真正高性能 Go 服务最关注什么

不是：

```txt
平均延迟
```

而是：

```txt
P99/P999
```

因为：

GC 抖动通常只影响尾延迟。

### 十、经典例子

例如：

Gin 服务：

平时：

```txt
2ms
```

突然：

```txt
80ms
```

查看：

```txt
GODEBUG=gctrace=1
```

发现：

```txt
GC assist很高
```

最后定位：

```go
fmt.Sprintf
json.Marshal
map[string]interface{}
```

大量临时对象。

优化后：

P99：

```txt
80ms → 5ms
```

这是 Go 服务优化里最典型的故事。

[goleak](https://github.com/uber-go/goleak)
[https://cs.opensource.google/go](https://cs.opensource.google/go)

真正企业级压测工具（函数级）-- benchstat（官方神器）
```sh
go install golang.org/x/perf/cmd/benchstat@latest
```