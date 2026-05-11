# 函数式选项

函数式选项是 Go 语言中实现简洁/优雅 API 的一种方法。

以函数形式实现的选项用于设置该选项的状态。

## 实现

### 选项

```go
package file

type Options struct {
    UID int
    GID int
    Flags int
    Contents string
    Permissions os.FileMode
}

type Option func(*Options)

func UID(userID int) Option {
    return func(args *Options) {
        args.UID = userID
    }
}

func GID(groupID int) Option {
    return func(args *Options) {
        args.GID = groupID
    }
}

func Contents(c string) Option {
    return func(args *Options) {
        args.Contents = c
    }
}

func Permissions(perms os.FileMode) Option {
    return func(args *Options) {
        args.Permissions = perms
    }
}

```

### 构造器

```go
package file

func New(filepath string, setters ...Option) error {
    // Default Options
    args := &Options{
        UID: os.Getuid(),
        GID: os.Getgid(),
        Contents: "",
        Permissions: 0666,
        Flags: os.O_CREATE | os.O_EXCL | os.O_WRONLY,
    }

    for _, setter := range setters {
        setter(args)
    }

    f, err := os.OpenFile(filepath, args.Flags, args.Permissions)
    if err != nil {
        return err
    } else {
        defer f.Close()
    }

    if _, err := f.WriteString(args.Contents); err != nil {
        return err
    }

    return f.Chown(args.UID, args.GID)
}

```

## 用法

```go
emptyFile, err := file.New("/tmp/empty.txt")
if err != nil {
    panic(err)
}

fillerFile, err := file.New("/tmp/file.txt", file.UID(1000), file.Contents("Lorem Ipsum Dolor Amet"))
if err != nil {
    panic(err)
}
```
