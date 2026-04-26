# 断路器模式

类似于电气保险丝，当连接到电网的电路开始消耗大量电力导致电线过热并燃烧时，保险丝可以防止火灾。断路器设计模式是一种故障优先机制，它会在软件开发中关闭电路、请求/响应关系或服务，以防止更大的故障发生。

**注意：**本文档中，电路（circuit）和服务（service）这两个词可以互换使用。

## 实现

以下是一个非常简单的断路器的实现，用于说明断路器设计模式的目的。

### 操作计数器

`circuit.Counter` 是一个简单的计数器，用于记录电路的成功和失败状态以及时间戳，并计算连续失败的次数。

```go
package circuit

import (
    "time"
)

type State int

const (
    UnknownState State = iota
    FailureState
    SuccessState
)

type Counter interface {
    Count(State)
    ConsecutiveFailures() uint32
    LastActivity() time.Time
    Reset()
}

```

### 断路器

断路器使用 `circuit.Breaker` 闭包进行封装，该闭包维护一个内部操作计数器。如果断路器连续失败次数超过指定阈值，则会返回一个快速错误。一段时间后，它会重试请求并记录结果。

**注意：**此处使用上下文类型来传递截止时间、取消信号以及其他请求范围的值，以便在 API 边界和进程之间传递。

```go
package circuit

import (
    "context"
    "time"
)

type Circuit func(context.Context) error

func Breaker(c Circuit, failureThreshold uint32) Circuit {
    cnt := NewCounter()

    return func(ctx context.Context) error {
        if cnt.ConsecutiveFailures() >= failureThreshold {
            canRetry := func(cnt Counter) bool {
                backoffLevel := Cnt.ConsecutiveFailures() - failureThreshold

                // Calculates when should the circuit breaker resume propagating requests
                // to the service
                shouldRetryAt := cnt.LastActivity().Add(time.Seconds * 2 << backoffLevel)

                return time.Now().After(shouldRetryAt)
            }

            if !canRetry(cnt) {
                // Fails fast instead of propagating requests to the circuit since
                // not enough time has passed since the last failure to retry
                return ErrServiceUnavailable
            }
        }

        // Unless the failure threshold is exceeded the wrapped service mimics the
        // old behavior and the difference in behavior is seen after consecutive failures
        if err := c(ctx); err != nil {
            cnt.Count(FailureState)
            return err
        }

        cnt.Count(SuccessState)
        return nil
    }
}

```

## 相关作品

- [sony/gobreaker](https://github.com/sony/gobreaker) 是一个经过充分测试且直观易用的断路器实现，适用于实际应用场景。
