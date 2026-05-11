# 扇入式消息传递模式

扇入式消息传递模式用于在工作者之间创建工作流（客户端：源，服务器：目标）。

我们可以使用 Go 通道来模拟扇入式消息传递。

```go
// Merge different channels in one channel
func Merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup

    out := make(chan int)

    // Start an send goroutine for each input channel in cs.
    // send copies values from c to out until c is closed, then calls wg.Done.
    send := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }

    wg.Add(len(cs))
    for _, c := range cs {
        go send(c)
    }

    // Start a goroutine to close out once all the send goroutines are done.
    // This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}

```

`Merge` 函数通过为每个入站通道启动一个 goroutine，将通道列表合并为一个出站通道，并将每个通道的值复制到该出站通道。

所有输出 goroutine 启动完毕后，`Merge` 函数会启动一个 goroutine 来关闭主通道。
