# 策略模式

策略行为设计模式允许在运行时选择算法的行为。

它定义算法，封装算法，并允许它们互换使用。

## 实现

实现一个可互换的运算符对象，该对象可对整数进行操作。

```go
type Operator interface {
    Apply(int, int) int
}

type Operation struct {
    Operator Operator
}

func (o *Operation) Operate(leftValue, rightValue int) int {
    return o.Operator.Apply(leftValue, rightValue)
}
```

## 用法

### 加法操作

```go
type Addition struct {}

func (Addition) Apply(lval, rval int) int {
    return lval + rval
}
```

```go
add := Operation{Addition{}}
add.Operate(3, 5) //
```

### 乘法操作

```go
type Multiplication struct {}

func (Multiplication) Apply(lval, rval int) int {
    return lval * rval
}

```

```go
mult := Operation{ Multiplication{} }

mult.Operate(3, 5) // 15
```

## 经验法则

- 策略模式与模板模式类似，区别在于其粒度。
- 策略模式允许你修改对象的内部结构，而装饰器模式则允许你修改对象的外部外观。
