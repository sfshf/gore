# 工厂方法模式

工厂方法创建型设计模式允许创建对象，而无需指定要创建对象的确切类型。

## 实现

示例实现展示了如何为数据存储提供不同的后端，例如内存存储和磁盘存储。

### 定义

```go
package data

import "io"

type Store interface {
    Open(string) (io.ReadWriterCloser, error)
}

```

### 不同实现

```go
package data

type StorageType int

const (
    DiskStorage StorageType = 1 << iota
    TempStorage
    MemoryStorage
)

func NewStore(t StorageType) Store {
    switch t {
    case MemoryStorage:
        return newMemoryStorage( /*...*/ )
    case DiskStorage:
        return newDiskStorage( /*...*/ )
    default:
        return newTempStorage( /*...*/ )
    }
}

```

## 用法

使用工厂方法，用户可以指定他们想要的存储类型。

```go
s, _ := data.NewStore(data.MemoryStorage)
f, _ := s.Open("file")

n, _ := f.Write([]byte("data"))
defer f.Close()

```
