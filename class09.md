# Go Class 09 Concurrency - Goroutine and Channel

此章節的資料，來自 [Go Systems Programming](https://www.packtpub.com/networking-and-servers/go-systems-programming)

## goroutine

1. A **goroutine** is the minimum Go entity that can be executed concurrently
1. goroutine is live in **Thread**, so it is not an autonomous entity
1. 當主程式結束時，goroutine 也一併會結束，即使還沒執行完畢。

使用 `go` 這個關鍵字來啟動一個 goroutine.

eg:

```go
package main

import (
    "fmt"
    "time"
)

func namedFunction() {
    time.Sleep(3 * time.Second)
    fmt.Println("Printing from namedFunction!")
}

func main() {
    go namedFunction()

    time.Sleep(5 * time.Second)
    fmt.Println("Exiting....")
}
```

說明：

1. 先定義一組 function，之後要用 goroutine 來執行。function 故意延遲 3 秒。

    ```go
    func namedFunction() {
        time.Sleep(3 * time.Second)
        fmt.Println("Printing from namedFunction!")
    }
    ```

1. `go namedFunction()`: 產生一個 goroutine 來執行 `namedFunction()`
1. 主程式故意延遲 5 秒。否則 goroutine 會來不及執行。
1. 也可用 anonymous function

```go
func main() {
    go func() {
        time.Sleep(3 * time.Second)
        fmt.Println("Printing from anonymous")
    }()

    time.Sleep(5 * time.Second)
    fmt.Println("Exiting....")
}
```

### Wait for goroutine

可以利用 `sync.WaitGroup` 來等待 goroutine 結束。

eg 1:

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    waitGroup := sync.WaitGroup{}

    waitGroup.Add(10)

    for i := 0; i < 10; i++ {
        go func(x int) {
            defer waitGroup.Done()
            time.Sleep(100 * time.Millisecond)
            fmt.Printf("%d ", x)
        }(i)
    }

    waitGroup.Wait()
    fmt.Println("Exit....")
}
```

說明：

1. `waitGroup := sync.WaitGroup{}`: 產生一個 wait group
1. `waitGroup.Add(10)`: 告知 wait group 要等幾個 goroutine。
1. 產生 10 個 goroutine，並 `defer waitGroup.Done()`，確保 function 結束後，會告知 wait group 有 goroutine 結束了。 

    ```go
    for i := 0; i < 10; i++ {
        go func(x int) {
            defer waitGroup.Done()
            time.Sleep(100 * time.Millisecond)
            fmt.Printf("%d ", x)
        }(i)
    }
    ```

1. `waitGroup.Wait()`: 主程序 wait

也可以每需要一個 goroutine 時，wait group 就加 1。記得 `waitGroup.Add(1)` 要在 `go nameFunction()` 之前。

eg 2:

```go
package main

import (
    "log"
    "sync"
    "time"
)

func test(x int, wait *sync.WaitGroup) {
    log.Println(x, "start")
    defer wait.Done()

    time.Sleep(5 * time.Second)
    log.Println(x, "end")

}

func main() {

    log.Println("Start...")
    waitGroup := sync.WaitGroup{}

    waitGroup.Add(1)

    go test(10, &waitGroup)

    waitGroup.Add(1)

    go test(11, &waitGroup)

    waitGroup.Add(1)

    go test(12, &waitGroup)

    waitGroup.Wait()
    log.Println("Exit....")

}
```

以上的 sample，有一點要特別注意，WaitGroup 傳給 function 時，一定要用 pointer。eg: `func test(x int, wait *sync.WaitGroup)` 及 `go test(10, &waitGroup)`，否則會出錯！(Why?? Ans: **Pass By Value**)

## channel and select

TO-DOO