# Go Class 03 程式結構與語法

## 關鍵字

```go
break    default     func   interface select
case     defer       go     map       struct
chan     else        goto   package   switch
const    fallthrough if     range     type
continue for         import return    var
```

## 內建常數

```go
true  false  iota  nil
```

## 資料型別

```go
int  int8  int16  int32  int64 uint  uint8  uint16  uint32  uint64 uintptr
float32  float64  complex64  complex128
bool  byte  rune  string  error
```

## 內建 Function

```go
make  len  cap  new  append  copy
delete
close
complex  real  imag
panic  recover
```

## Sample code

```go {.line-numbers}
// Boiling prints the boiling point of water.
package main

import "fmt"

const boilingF = 212.0

func main() {
    var f = boilingF
    var c = (f - 32) * 5 / 9
    fmt.Printf("boiling point = %g°F or %g°C\n", f, c)
}
```

output:
`boiling point = 212°F or 100°C`

## 宣告 Declaraion

### 完整寫法：

```go {.line-numbers}
var name type = expression
```

sample 1:

```go {.line-numbers}
var s string
fmt.Println(s) // ""
```

sample 2:

```go {.line-numbers}
var i, j, k int                 // int, int, int
var b, f, s = true, 2.3, "four" // bool, float64, string
```

sample 3:

```go {.line-numbers}
var f, err = os.Open(name) // os.Open returns a file and an error
```

#### Zero Values

```go {.line-numbers}
var s string    // ""
var a int       // 0
var f float32   // 0.0
var err error   // nil
```

### 簡寫：

```go {.line-numbers}
name := expression
```

sample 1:

```go {.line-numbers}
anim := gif.GIF{LoopCount: nframes}
freq := rand.Float64() * 3.0
t := 0.0
```

sample 2:

```go {.line-numbers}
i, j := 0, 1
```

sample 3:

```go {.line-numbers}
f, err := os.Open(name)
if err != nil {
    return err 
}

// ...use f...

f.Close()
```

#### Default Type

```go {.line-numbers}
i := 0      // int
f := 0.0    // float64
s := ""     // string
```

#### 注意事項

使用 `:=` 時，左邊的變數，至少要有一個是新的變數名稱。

sample 1:

```go {.line-numbers}
in, err := os.Open(infile)
// ...
out, err := os.Create(outfile)
```

以上，雖然 `err` 重覆，但 `out` 是新的變數名稱，所以 compile 會過。

sample 2:

```go {.line-numbers}
f, err := os.Open(infile)
// ...
f, err := os.Create(outfile) // compile error: no new variables
```

以上，`f` 與 `err` 都是舊的變數，所以在第二次，還是使用 `:=` 時，compile 會錯。

通常 compile 會報錯，都不是什麼大問題，修正就好了。

## 指標 (Pointer)

使用方法及觀念與 C 相同，要注意的是，C 可以對指標做位移，但 Go 不行。

eg:

sample 1:

```go {.line-numbers}
x := 1
p := &x         // p, of type *int, points to x
fmt.Println(*p) // "1"
*p = 2          // equivalent to x = 2
fmt.Println(x)  // "2"
```

不可做指標位移:

```go {.line-numbers}
a := 10
b := &a
b++     // invalid operation: b++ (non-numeric type *int)
```

## Tuple

### Tuple Assignment

```go {.line-numbers}
x, y = y, x
a[i], a[j] = a[j], a[i]
```

GCD sample:

```go {.line-numbers}
func gcd(x, y int) int {

    for y != 0 {
        x, y = y, x%y
    }

    return x
}
```

### Return Tuple

```go {.line-numbers}
func swap(x, y int) (int, int) {
    return y, x
}
```

## Type Declaration

Go 可以使用 `type`，利用原有的資料型別，宣告一個新的資料型別，最常用在 `struct` 宣告，使用 `type` 重新宣告資料型別，可以增加程式碼的可讀性。

eg:

sample 1:

```go {.line-numbers}
package tempconv

import "fmt"

type Celsius float64
type Fahrenheit float64
const (
    AbsoluteZeroC Celsius = -273.15
    FreezingC     Celsius = 0
    BoilingC      Celsius = 100
)

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }

func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }
```

sample 2:

```go {.line-numbers}
fmt.Printf("%g\n", BoilingC-FreezingC) // "100" °C
boilingF := CToF(BoilingC)
fmt.Printf("%g\n", boilingF-CToF(FreezingC)) // "180" °F
fmt.Printf("%g\n", boilingF-FreezingC)       // compile error: type mismatch
```

sample 3:

```go {.line-numbers}
var c Celsius
var f Fahrenheit
fmt.Println(c == 0)             // "true"
fmt.Println(f >= 0)             // "true"
fmt.Println(c == f)             // compile error: type mismatch
fmt.Println(c == Celsius(f))    // "true"!
```

以上，宣告兩種新的資料型別：`Celsius` 及 `Fahrenheit`，這兩個新的資料型別，底層都是 `float64`。

注意，雖然 `Celsius` 及 `Fahrenheit` 底層都是 `float64`，但都還是要視為不同的型別。

## Package and Imports

Go Package 概念跟 Java package, C/PHP namespace 類似。主要用意：

1. Modularity
1. Encapsulation
1. Reuse

package 命名時要注意，除了 `main` 之外，目錄的名稱要與 package 相同。

import 的路徑，由 `$GOPATH/src` 以下目錄開始。

比如有一個專案路徑 `gopl.io/ch1/`，package 命名 `helloworld`, 則開一個 `helloworld` 目錄。完整路徑 `$GOPATH/src/gopl.io/ch1/helloworld`

import 則用

```go {.line-numbers}
import "gopl.io/ch1/helloworld"
```

### Package Initialization

package 中，可以在某一個程式檔案，定義 `func init()`。當 package 被載入時，會先執行 `init` 的程式碼。

eg:

```go {.line-numbers}
func init() {

}
```

## Visible

Go 沒有 `private` and `public`，是利用字母的大、小寫來區分 `private` 及 `public`。如果變數是小寫開頭，則為 **private**，反之，大寫就是 **public**。

## Scope

local block -> function block -> global
## Scope

local block -> function block -> global
