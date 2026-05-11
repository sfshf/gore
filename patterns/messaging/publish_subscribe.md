# 发布/订阅消息传递模式

发布/订阅是一种消息传递模式，用于在不同组件之间传递消息，而这些组件彼此之间无需了解对方的身份。

它类似于观察者行为设计模式。

观察者和发布/订阅的基本设计原则是将对“事件消息”感兴趣的对象与消息传递者（观察者或发布者）解耦。这意味着您无需编写程序将消息直接发送给特定的接收者。

为了实现这一点，需要使用一个中间层，称为“消息代理”或“事件总线”，接收已发布的消息，然后将其路由给订阅者。

它包含三个组件：**消息**、**主题**和**用户**。

```go
type Message struct {
    // Contents
}

type Subscription struct {
    ch chan<- Message
    Inbox chan Message
}

func (s *Subscription) Publish(msg Message) error {
    if _, ok := <-s.ch; !ok {
        return errors.New("Topic has been closed")
    }

    s.ch <- msg

    return nil
}

```

```go
type Topic struct {
    Subscribers []Session
    MessageHistory []Message
}

func (t *Topic) Subscribe(uid uint64) (Subscription, error) {
    // Get session and create one if it's the first

    // Add session to the Topic & MessageHistory

    // Create a subscription
}

func (t *Topic) Unsubscribe(Subscription) error {
    // Implementation
}

func (t *Topic) Delete() error {
    // Implementation
}

```

```go
type User struct {
    ID uint64
    Name string
}

type Session struct {
    User User
    Timestamp time.Time
}

```

## 改进

利用无栈 goroutine，可以并行发布事件。

通过处理延迟订阅者，可以提高性能。使用缓冲收件箱，并在收件箱满时停止发送事件。
