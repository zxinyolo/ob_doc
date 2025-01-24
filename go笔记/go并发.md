#### chan 通道示例

```go
package main

import (
	"fmt"
	"time"
)

var strChan = make(chan string, 3)

func main() {
	syncChan1 := make(chan struct{}, 1)
	syncChan2 := make(chan struct{}, 2)
    // 接收的goroutine
	go func() {
        // 由于syncChan1通道的缓冲值是1,当其中没有值的时候就是阻塞的,相当于一个可以开始执行的信号
		<-syncChan1
		fmt.Println("Received a sync signal and wait a second...[receiver]")
		time.Sleep(time.Second)
		for {
            // 接收到可以开始接收消息的信号,就从strChan中接收消息, elem, ok := <-strChan; ok可以判断strChan是否关闭
			if elem, ok := <-strChan; ok {
				fmt.Println("Received:", elem, "[receiver]")
			} else {
				break
			}
		}
		fmt.Println("Stopped. [receiver]")
        // 发送的goroutine的关闭消息,是为了告诉主gorouter,本goroutine执行完成
		syncChan2 <- struct{}{}
	}()
    // 发送的goroutine
	go func() {
		for _, elem := range []string{"a", "b", "c", "d"} {
            // 发送消息到strChan通道
			strChan <- elem
			fmt.Println("Sent:", elem, "[sender]")
			if elem == "c" {
                // 当elem为"c"的时候, 就往syncChan1发送一个消息,由于充当信号,发送空struct也没关系,通知接收方开始接收消息
				syncChan1 <- struct{}{}
				fmt.Println("Sent a sync signal. [sender]")
			}
		}
		fmt.Println("Wait 2 seconds... [sender]")
		time.Sleep(time.Second * 2)
        // 关闭通通道strChan
		close(strChan)
        // 发送的goroutine的关闭消息,是为了告诉主gorouter,本goroutine执行完成
		syncChan2 <- struct{}{}
	}()
    // 由于是两个gorouter,接收到两个消息就可以结束主gorouter了
	<-syncChan2
	<-syncChan2
}
//  map 是引用类型，通过通道传递时，不会复制数据，所有 goroutine 对它的修改会影响同一个实例。
var mapChan = make(chan map[string]int, 1)

func main() {
    syncChan := make(chan struct{}, 2)
    go func() {
        for {
            // 接收到mapChan中有值就加1
            if elem, ok := <-mapChan; ok {
                elem["count"]++
            } else {
                break
            }
        }
        fmt.Println("Stopped. [receiver]")
        syncChan <- struct{}{}
    }()
    go func() {
        countMap := make(map[string]int)
        for i := 0; i < 5; i++ {
            // 发送map到mapChan, 只传递了countMap的引用,所以任何地方修改, 其它的访问的都是修改后的值
            mapChan <- countMap
            time.Sleep(time.Millisecond)
            fmt.Printf("The count map: %v. [sender]\n", countMap)
        }
        close(mapChan)
        syncChan <- struct{}{}
    }()
    <-syncChan
    <-syncChan
}
// for语句来接收通道内容
var strChan = make(chan string, 3)

func receive(strChan <-chan string, syncChan1 <-chan struct{}, syncChan2 chan<- struct{}) {
	<-syncChan1
	fmt.Println("Received a sync signal and wait a second... [receiver]")
	for elem := range strChan {
		fmt.Println("Received:", elem, "[receiver]")
	}
	fmt.Println("Stopped. [receiver]")
	syncChan2 <- struct{}{}

}

func send(strChan chan<- string, syncChan1 chan<- struct{}, syncChan2 chan<- struct{}) {
	for _, elem := range []string{"a", "b", "c", "d"} {
		strChan <- elem
		fmt.Println("Send:", elem, "[sender]")
		if elem == "c" {
			syncChan1 <- struct{}{}
			fmt.Println("Sent a sync signal. [sender]")
		}
	}
	fmt.Println("Wait 2 seconds... [sender]")
	time.Sleep(time.Second * 2)
	close(strChan)
	syncChan2 <- struct{}{}
}

func main() {
	syncChan1 := make(chan struct{}, 1)
	syncChan2 := make(chan struct{}, 2)
	go receive(strChan, syncChan1, syncChan2)
	go send(strChan, syncChan1, syncChan2)
	<-syncChan2
	<-syncChan2
}
```

#### time 和 channel 的使用

```go
func main() {
    // time.NewTimer会创建一个定时器类型
	timer := time.NewTimer(2 * time.Second)
	fmt.Printf("Present time: %v.\n", time.Now())
    // 定时器时间到了之后, timer.C通道中就会有定时器结束那一刻的时间信息
	expirationTime := <-timer.C
	fmt.Printf("Expiration time: %v.\n", expirationTime)
	fmt.Printf("Stop timer: %v.\n", timer.Stop())
}
func main() {
	intChan := make(chan int, 1)
	go func() {
        // 等待1秒
		time.Sleep(time.Second)
		intChan <- 1
	}()
    // 会执行第二个case, 第一个case中intChan运行开始后1秒才有值,第二个case0.5秒就有值了
	select {
	case e := <-intChan:
		fmt.Println(e)
	case <-time.NewTimer(time.Millisecond * 500).C:
		fmt.Println("Timeout!")
	}
```