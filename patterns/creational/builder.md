# 建造者模式

构建器模式将复杂对象的构造与其表示形式分离，使得相同的构造过程可以创建不同的表示形式。

在 Go 语言中，通常使用配置结构体来实现相同的功能，然而，将结构体传递给构建器方法会导致代码中出现大量样板代码，例如 `if cfg.Field != nil {...}` 检查。

## 实现

```go
package car

type Speed float64

const (
    MPH Speed = 1
    KPH = 1.60934
)

type Color string

const (
    BlueColor Color = "blue"
    GreenColor = "green"
    RedColor = "red"
)

type Wheels string

const (
    SportsWheels Wheels = "sports"
    SteelWheels = "steel"
)

type Builder interface {
    Color(Color) Builder
    Wheels(Wheels) Builder
    TopSpeed(Speed) Builder
    Build() Interface
}

type Interface interface {
    Drive() error
    Stop() error
}

```

## 用法

```go
assembly := car.NewBuilder().Paint(car.RedColor)

familyCar := assembly.Wheels(car.SportsWheels).TopSpeed(50 * car.MPH).Build()
familyCar.Drive()

sportsCar := assembly.Wheels(car.SteelWheels).TopSpeed(150 * car.MPH).Build()
sportsCar.Drive()
```
