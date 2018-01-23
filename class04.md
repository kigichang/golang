# Go Class 04 Data Type - Basic Types

Go 的 Data Type 分成四個類別：

- Basic Type
  - numbers
  - strings
  - booleans
- Aggregate Types
  - arrays
  - structs
- Reference Types
  - pointers
  - slices
  - maps
  - functions
  - channels
- Interface Types
  - interface

## Numbers

### Integers

跟 C 一樣，有分 signed 及 unsigned，可以指定變數的 bit 數。

- int8, uint8
- int16, uint16
- int32, uint32
- int64, uint64
- int, uint: 會依作業系統(32bit, or 64bit)，變成 int32/int64 or uint32/uint64

eg:

```go
var a int32   // zero value: 0
b := 10       // type: int
```

### Float-Point Numbers

- float32: C 的 float
- float64: C 的 double

eg:

```go
var f float32 // zero value: 0.0
d := 0.0      // type: float64
```

### Complex Numbers

數學上的複數

- complex64: 由兩個 float32 組成
- complex128: 由兩個 float64 組成

eg:

sample 1:

```go
x := 1 + 2i
y := 3 + 4i
```

```go
var x complex128 = complex(1, 2)    // 1+2i
var y complex128 = complex(3, 4)    // 3+4i
fmt.Println(x*y)                    // "(-5+10i)"
fmt.Println(real(x*y))              // "-5"
fmt.Println(imag(x*y))              // "10"
```

## Booleans

只有 `true` 及 `false`，不用能 integers 來當 boolean 使用

eg:

```go
var b boolean   // zero value: false
ok := true
```

## Strings

- **Immutable** sequence of bytes
- **UTF-8** encoded

eg:

```go
var str string    // zero value: "" (empty string)
str2 := "hello world"
```

topic 1: get length

```go
len := len(str2)    // use len() to get length of bytes in string
```

topic 2: substring

使用 `str[i:j]` 取得 substring. 會從第 i 個開始，取到第 j-1 個為止。可以省略 i 及 j。

```go
substr2 := s[0:5]    // hello
substr3 := s[1:]     // 從第 1 個開始，取到最後
substr4 := s[:5]     // 從 0 開始，取到第 4 個
substr5 := s[:]      // 全取
```

topic 3: concate

```go
a := "hello"
b := " world"
c := a + b    // "hello world"
```

套件: **strings**

常用 functions:

```go
func Contains(s, substr string) bool
func Count(s, sep string) int
func Fields(s string) []string
func HasPrefix(s, prefix string) bool
func Index(s, sep string) int
func Join(a []string, sep string) string
```

### Rune

`rune` 是 unicode chacter 的概念，它的底層型別是 **int32** 也就是 4 bytes. 一般 string 操作單位是 **byte**。

eg:

```go
package main

import "fmt"

func main() {
  const nihongo = "日本語"
  for i := 0; i < len(nihongo); i++ {
    fmt.Printf("%d: %x\n", i, nihongo[i])
  }

  for index, runeValue := range nihongo {
    fmt.Printf("%#U starts at byte position %d\n", runeValue, index)
  }
}
```

## Conversions between Strings and Numbers

使用 `fmt.Sprintf()` 與 `strconv` 這個套件。

eg 1:

```go
x := 123
y := fmt.Sprintf("%d", x)
fmt.Println(y, strconv.Itoa(x)) // "123 123"
```

eg 2:

```go
fmt.Println(strconv.FormatInt(int64(x), 2)) // "1111011"
s := fmt.Sprintf("x=%b", x)                 // "x=1111011"
```

eg 3:

```go
x, err := strconv.Atoi("123")             // x is an int
y, err := strconv.ParseInt("123", 10, 64) // base 10, up to 64 bits
```

## Constants

與 C 相同，利用 `const` 這個關鍵字來宣告常數。

```go
const pi = 3.14159 // approximately; math.Pi is a better approximation
```

```go
const (
    e = 2.71828182845904523536028747135266249775724709369995957496696763
    pi = 3.14159265358979323846264338327950288419716939937510582097494459
)
```

## Zero Value

每一種資料型別在宣告時，沒有給定值的話，則 Go 會給予一個預設值，這個預設值則稱為該型別的 **zero value**

- int: 0
- float: 0.0
- string: ""
- boolean: falsealse
- struct: struct that field with zero value
- array: 指定長度，內含 zero value.
- reference type: nil