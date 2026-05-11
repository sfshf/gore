# 对象池模式

对象池创建型设计模式用于根据需求预期准备和维护多个实例。

## 实现

```go
package pool

type Pool chan *Object

func New(total int) *Pool {
    p := make(Pool, total)

    for i := 0; i < total; i++ {
        p <- new(Object)
    }

    return &p
}

```

## 用法

以下是一个简单的对象池的生命周期示例。

```go
p := pool.New(2)

select {
case obj := <- p:
    obj.Do( /*...*/ )
    p <- obj
default:
    // No more objects left - retry later or fail
    return
}
```

## 经验法则

- 对象池模式适用于对象初始化成本高于对象维护成本的情况。
- 如果需求呈峰值而非稳定，维护开销可能会超过对象池带来的好处。
- 由于对象会被预先初始化，因此对象池模式对性能有积极影响。
