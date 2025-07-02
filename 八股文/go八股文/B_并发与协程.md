[TOC]

## 一、 Goroutine (协程)

Goroutine 是 Go 语言并发设计的核心。它是一种轻量级的执行单元，常被称为“协程”。

### 1.1 什么是 Goroutine？

-   **轻量级**: 创建一个 Goroutine 的开销非常小（初始栈大小仅 2KB），因此可以轻松创建成千上万个 Goroutine。
-   **由 Go 运行时管理**: Goroutine 的调度由 Go 运行时（Go Scheduler）负责，而不是操作系统。它会将多个 Goroutine 调度到少数几个操作系统线程上执行，这大大减少了线程切换的开销。
-   **并发执行**: 使用 `go` 关键字可以启动一个新的 Goroutine，它会与主 Goroutine (main 函数) 以及其他 Goroutine 并发执行。

### 1.2 如何启动一个 Goroutine

只需在函数调用前加上 `go` 关键字即可。

```go
import (
    "fmt"
    "time"
)

func say(s string) {
    for i := 0; i < 3; i++ {
        fmt.Println(s)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    // 启动一个新的 Goroutine 来执行 say("World")
    go say("World")

    // main Goroutine 继续执行
    say("Hello")
}

/*
可能的输出 (顺序不固定):
Hello
World
Hello
World
Hello
World
*/
```

**注意**: `main` 函数本身就是一个 Goroutine。当 `main` 函数执行完毕时，整个程序就会退出，无论其他 Goroutine 是否执行完毕。为了等待 Goroutine 完成，我们需要使用 `sync.WaitGroup`。

### 1.3 使用 `sync.WaitGroup` 等待 Goroutine

`sync.WaitGroup` 是一个计数信号量，常用于等待一组 Goroutine 完成。

-   `Add(n)`: 增加计数器的值，通常在启动 Goroutine 前调用。
-   `Done()`: 减少计数器的值，通常在 Goroutine 的末尾使用 `defer` 调用。
-   `Wait()`: 阻塞，直到计数器归零。

```go
import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, wg *sync.WaitGroup) {
    // 在函数退出时，通知 WaitGroup 任务已完成
    defer wg.Done()

    fmt.Printf("Worker %d starting\n", id)
    time.Sleep(time.Second)
    fmt.Printf("Worker %d done\n", id)
}

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 3; i++ {
        // 计数器加 1
        wg.Add(1)
        // 启动 worker Goroutine
        go worker(i, &wg)
    }

    // 等待所有 Goroutine 完成
    fmt.Println("Waiting for workers to finish...")
    wg.Wait()
    fmt.Println("All workers done.")
}
```

## 二、 Channel (通道)

Channel 是 Go 中用于 Goroutine 之间通信的管道。它可以被看作是一个类型化的、线程安全的消息队列。

> **Go 的并发哲学**: "Do not communicate by sharing memory; instead, share memory by communicating." (不要通过共享内存来通信，而要通过通信来共享内存。)

### 2.1 创建 Channel

使用 `make` 函数创建 Channel。

```go
// 创建一个可以传输 int 类型的无缓冲 Channel
ch := make(chan int)

// 创建一个可以传输 string 类型、缓冲区大小为 10 的有缓冲 Channel
chBuffered := make(chan string, 10)
```

### 2.2 无缓冲 Channel

无缓冲 Channel 要求**发送方和接收方必须同时准备好**，否则就会阻塞。这是一种同步通信方式。

-   发送操作 `ch <- value` 会阻塞，直到有另一个 Goroutine 从 `ch` 接收。
-   接收操作 `value := <-ch` 会阻塞，直到有另一个 Goroutine 向 `ch` 发送。

```go
func main() {
    ch := make(chan string)

    go func() {
        fmt.Println("Goroutine is ready to send...")
        ch <- "Hello from Goroutine!" // 发送，会阻塞直到 main 接收
        fmt.Println("Goroutine finished sending.")
    }()

    fmt.Println("Main is waiting to receive...")
    msg := <-ch // 接收，会阻塞直到 Goroutine 发送
    fmt.Println("Main received:", msg)
}
```

### 2.3 有缓冲 Channel

有缓冲 Channel 是异步的。发送操作只有在缓冲区满时才会阻塞，接收操作只有在缓冲区空时才会阻塞。

```go
func main() {
    ch := make(chan int, 2) // 缓冲区大小为 2

    ch <- 1 // 不会阻塞
    ch <- 2 // 不会阻塞

    // ch <- 3 // 会阻塞，因为缓冲区已满

    fmt.Println(<-ch) // 输出: 1
    fmt.Println(<-ch) // 输出: 2
}
```

### 2.4 关闭 Channel 和 `for-range` 遍历

-   `close(ch)`: 用于关闭一个 Channel。关闭后不能再发送，但仍可以接收已发送的数据。
-   接收方可以通过 `value, ok := <-ch` 的第二个返回值 `ok` 来判断 Channel 是否已关闭。如果 `ok` 为 `false`，表示 Channel 已关闭且无数据可读。
-   `for-range` 循环会自动处理这个过程，当 Channel 关闭并取完所有值后，循环会自动结束。

```go
func main() {
    jobs := make(chan int, 3)

    go func() {
        for i := 1; i <= 3; i++ {
            jobs <- i
            fmt.Println("Sent job", i)
        }
        close(jobs) // 完成所有发送后，关闭 Channel
        fmt.Println("Sent all jobs and closed channel.")
    }()

    // 使用 for-range 优雅地接收所有数据
    for job := range jobs {
        fmt.Println("Received job", job)
    }
}
```

## 三、 `select` 语句

`select` 语句让一个 Goroutine 可以同时等待多个 Channel 操作。它会阻塞，直到其中一个 `case` 可以执行，然后执行该 `case`。如果多个 `case` 同时就绪，它会随机选择一个。

### 3.1 基本用法

```go
func main() {
    c1 := make(chan string)
    c2 := make(chan string)

    go func() {
        time.Sleep(1 * time.Second)
        c1 <- "one"
    }()
    go func() {
        time.Sleep(2 * time.Second)
        c2 <- "two"
    }()

    // select 会等待 c1 和 c2
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-c1:
            fmt.Println("received", msg1)
        case msg2 := <-c2:
            fmt.Println("received", msg2)
        }
    }
}
```

### 3.2 超时处理

`select` 和 `time.After` 结合是实现超时的经典模式。

```go
func main() {
    c1 := make(chan string, 1)
    go func() {
        time.Sleep(2 * time.Second)
        c1 <- "result 1"
    }()

    select {
    case res := <-c1:
        fmt.Println(res)
    case <-time.After(1 * time.Second): // 等待 1 秒
        fmt.Println("timeout 1")
    }
}
```

### 3.3 非阻塞操作

`select` 的 `default` 分支使其变为非阻塞的。如果没有任何 Channel 操作就绪，它会立即执行 `default` 分支。

```go
func main() {
    messages := make(chan string)

    // 非阻塞发送
    select {
    case messages <- "hi":
        fmt.Println("sent message")
    default:
        fmt.Println("no message sent")
    }

    // 非阻塞接收
    select {
    case msg := <-messages:
        fmt.Println("received message", msg)
    default:
        fmt.Println("no message received")
    }
}
```

## 四、 并发安全与锁 (`sync` 包)

当多个 Goroutine 需要访问同一个共享资源（如一个变量）时，就会产生数据竞争 (Data Race)。我们需要使用锁来保护这些资源。

### 4.1 `sync.Mutex` (互斥锁)

`Mutex` 用于在代码的关键区域（临界区）提供互斥访问，确保同一时间只有一个 Goroutine 可以执行这段代码。

```go
import (
    "fmt"
    "sync"
)

// 一个被并发访问的计数器
type SafeCounter struct {
    mu sync.Mutex
    v  map[string]int
}

func (c *SafeCounter) Inc(key string) {
    c.mu.Lock()   // 加锁
    c.v[key]++
    c.mu.Unlock() // 解锁
}

func (c *SafeCounter) Value(key string) int {
    c.mu.Lock()
    defer c.mu.Unlock() // 使用 defer 保证解锁
    return c.v[key]
}

func main() {
    c := SafeCounter{v: make(map[string]int)}
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            c.Inc("somekey")
        }()
    }

    wg.Wait()
    fmt.Println(c.Value("somekey")) // 输出: 1000
}
```

### 4.2 `sync.RWMutex` (读写锁)

`RWMutex` 是一种更特殊的锁，它区分了“读”和“写”操作。

-   多个 Goroutine 可以**同时**获取读锁 (`RLock`)。
-   只有一个 Goroutine 可以获取写锁 (`Lock`)，且在写锁被持有时，其他任何读/写操作都会被阻塞。

**适用场景**: “读多写少”的场景。当数据不经常变化，但被频繁读取时，读写锁可以显著提高性能。
