# 扇出消息传递模式

扇出是一种用于在工作节点（生产者：源节点，消费者：目标节点）之间分配任务的消息传递模式。

我们可以使用 Go 通道来模拟扇出。

```go
// Split a channel into n channels that receive messages in a round-robin fashion.
func Split(ch <-chan int, n int) []<-chan int {
    cs := make([]chan int)
    for i := 0; i < n; i++ {
        cs = append(cs, make(chan int))
    }

    // Distributes the work in a round robin fashion among the stated number
    // of channels until the main channel has been closed. In that case, close all channels and return.
    distributeToChannels := func(ch <-chan int, cs []chan<- int) {
        // Close every channel when the execution ends.
        defer func(cs []chan<- int) {
            for _, c := range cs {
                close(c)
            }
        }(cs)

        for {
            for _, c := range cs {
                select {
                case val, ok := <-ch:
                    if !ok {
                        return
                    }

                    c <- val
                }
            }
        }
    }

    go distributeToChannels(ch, cs)

    return cs
}

```

`Split` 函数使用 goroutine 将单个通道转换为通道列表，方法是：以轮询方式将接收到的值复制到列表中的各个通道。
