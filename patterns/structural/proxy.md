# 代理模式

[代理模式](https://en.wikipedia.org/wiki/Proxy_pattern)提供了一个对象，用于控制对另一个对象的访问，并拦截所有调用。

## 实现

代理可以与任何对象交互：网络连接、内存中的大型对象、文件，或其他成本高昂或难以复制的资源。

实现思路简述：

```go
// To use proxy and to object they must implement same methods
type IObject interface {
    ObjDo(action string)
}

// Object represents real objects which proxy will delegate data
type Object struct {
    action string
}

// ObjDo implements IObject interface and handles all logic
func (obj *Object) ObjDo(action string) {
    // Action behavior
    fmt.Printf("I can, %s", action)
}

// ProxyObject represents proxy object with intercepts actions
type ProxyObject struct {
    object *Object
}

// ObjDo are implemented IObject and intercept action before send in real Object
func (p *ProxyObject) ObjDo(action string) {
    if p.object == nil {
        p.object = new(Object)
    }
    if action == "Run" {
        p.object.ObjDo(action) // Prints: I can, Run
    }
}

```
