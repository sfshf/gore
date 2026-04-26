# 装饰器模式

装饰器结构模式允许在不改变对象内部实现的情况下，动态地扩展其功能。

装饰器提供了一种灵活的方法来扩展对象的功能。

## 实现

`LogDecorate` 装饰一个签名为 `func(int) int` 的函数，该函数用于操作整数并添加输入/输出日志记录功能。

```go
type Object func(int) int

func LogDecorate(fn Object) Object {
    return func(n int) int {
        log.Println("Starting the execution with the integer", n)

        result := fn(n)

        log.Println("Execution is completed with the result", result)

        return result
    }
}
```

## 用法

```go
func Double(n int) int {
    return n * 2
}

f := LogDecorate(Double)

f(5)
// Starting execution with the integer 5
// Execution is completed with the result 10
```

## 经验法则

- 与适配器模式不同，装饰器对象是通过**注入**方式获取的。
- 装饰器不应修改对象的接口。
