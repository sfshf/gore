# 单例模式

单例创建型设计模式将一个类型的实例化限制为单个对象。

## 实现

```go
package singleton

type singleton map[string]string

var (
    one sync.Once

    instance singleton
)

func New() singleton {
    once.Do(func() {
        instance = make(singleton)
    })

    return instance
}

```

## 用法

```go
s := singletone.New()

s["this"] = "that"

fmt.Println("This is ", s2["this"])
// This is that
```

## 经验法则

- 单例模式代表全局状态，并且大多数情况下会降低可测试性。
