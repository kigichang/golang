# Go Class 02 Hello World

## Hello World

1. 在 src 下開一個目錄
1. 產生一個檔案 `main.go` 內容如下：

    ```go { .line-numbers }
    package main

    import "fmt"

    func main() {
        fmt.Println("Hello, 世界")
    }
    ```
1. 在目錄下，可執行 `go run main.go`，可以看到結果。
1. 在目錄下，可執行 `go build`，編繹成執行檔。

### 說明

1. 寫執行檔的程式，檔名不一定要命名成 `main.go`，但程式碼的 package 宣告一定要是 **main**。
1. 經過 build 之後，產生的執行檔名，會是目錄的名稱。
1. 可以使用 `go run main.go` 直接執行程式，如果程式是拆分成多個 .go 的檔案，則需要將每個檔名也加入。eg: `go run main.go a.go b.go`
1. `import` 是將會用到的 package 加入，跟 Java 一樣，有用到的 package 用 import 加入。Go 的工具，會幫忙找內建的 package ，自動加入到程式碼中，很方便；但如果是第三方的套件，就要自己寫。第三方套件 import 路徑，是從 `$GOPATH/src` 以下。eg: 程式放在 `$GOPATH/src/a/b/c` 則 import 路徑是 `import "a/b/c"`。
1. 程式的進入點 (Entry point): `func main()`，跟大多數的程式語言一樣，寫執行檔都會需要有一個主函式 **main**

## Arguemnts

重覆上述的動作，sample code 如下：

```go { .line-numbers }
package main

import (
    "fmt"
    "os"
)

func main() {
    if len(os.Args) > 1 {
        fmt.Println("hi, ", os.Args[1])
    }
    fmt.Println("hello world")
}
```

利用 `os.Args` 來接 command line 傳進來的參數。`os.Args[0]` 是執行檔的完整檔名，所以傳入的參數值要從 `os.Args[1]` 開始。如果要寫較複雜的 command line 程式，建議用 [spf13/cobra](https://github.com/spf13/cobra) 這個套件來管理參數。