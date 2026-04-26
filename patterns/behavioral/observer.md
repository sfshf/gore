# 观察者模式

观察者模式允许一个类型实例将事件“发布”给其他类型实例（“观察者”），以便它们在特定事件发生时收到更新通知。

## 实现

在长时间运行的应用程序（例如 Web 服务器）中，实例可以维护一个观察者集合，用于接收触发事件的通知。

实现方式多种多样，但可以使用接口来创建标准的观察者和通知者：

```go

type (
    // Event defines an indication of a point-in-time occurrence.
    Event struct {
        // Data in this case is a simple int, but the actual
        // implementation would depend on the application.
        Data int64
    }

    // Observer defines a standard interface for instances that wish to list
    // for the occurrence of a specific event.
    Observer interface {
        // OnNotify allows an event to be "published" to interface implementations.
        // In the "real world", error handling would likely be implemented.
        OnNotify(Event)
    }

    // Notifier is the instance being observed. Publisher is perhaps another decent
    // name, but naming things is hard.
    Notifier interface {
        // Register allows an instance to register itself to listen/observe events.
        Register(Observer)
        // Deregister allows an instance to remove itself from the collection of
        // observers/listeners.
        Deregister(Observer)
        // Notify publishes new events to listeners. The method is not absolutely
        // necessary, as each implementation could define this itself without
        // losing functionality.
        Notify(Event)
    }
)

```
