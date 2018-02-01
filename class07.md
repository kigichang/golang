# Go Class 07 Functions and Methods

## Functions

結構:

```go { .line-numbers }
func name(parameter-list) (result-list) {
    body
}
```

eg:

sample 1:

```go { .line-numbers }
func hypot(x, y float64) float64 {
      return math.Sqrt(x*x + y*y)
}

fmt.Println(hypot(3, 4)) // "5"
```

sample 2:

```go { .line-numbers }
func f(i, j, k int, s, t string)                { /* ... */ }
func f(i int, j int, k int, s string, t string) { /* ... */ }
```

sample 3:

```go { .line-numbers }
func add(x int, y int) int { return x+y }
func sub(x, y int) (z int) { z= x - y; return }
func first(x int, _ int) int { return x }
func zero(int, int) int { return 0 }
```

### Pass by Value (Call by Value)

Go 在傳遞參數時，是以 **by value** 的方式進行，也就是說在傳入 function 前，會產生一份新的資料，給 function 使用，也因此 function 修改時，也是修改此新的資料。

此時要特別注意傳入的資料型別：

- Aggregate Types (Array, Struct)，在 Java 的定義下，是屬於 Value Types，也就是說會產生一筆新的資料給 function，function 做任何修改，都**不會**異動到原本的資料，如果 array/struct 資料很龐大時，會造成記憶體的浪費。

- Reference Types (Pointer, Slice, Map, Function, Channel)，一樣在傳入 function 時，會複製新的值給 function，只是這新的值，只是 copy 原本的參照值(reference, 可以當作記憶體位址)，因此 function 做任何修改時，也都是透過原來的參照值在做資料異動，會修改到原本的資料，要特別小心。

struct sample:

```go { .line-numbers }
type Test struct {
    A int
    B string
}

func test(t Test) {
    t.A += 1
    t.B += " by test"
}

func testByPtr(t *Test) {
    t.A += 1
    t.B += " by test"
}

t := Test {
    A: 0,
    B: "Test",
}

fmt.Println(t)      // {0 Test}
test(t)             // 用原本的 struct
fmt.Println(t)      // {0 Test}

testByPtr(&t)       // 改用 pointer
fmt.Println(t)      // {1 Test by test}
```

array sample:

```go { .line-numbers }
func arrTest(a [3]int) {
    for i, x := range a {
        a[i] = x + 1
    }
}

func arrTestBySlice(a []int) {
    for i, x := range a {
        a[i] = x + 1
    }
}

a := [3]int{1, 2, 3}

fmt.Println(a);         // [1 2 3]
arrTest(a)              // 用原本的 array
fmt.Println(a);         // [1 2 3]

arrTestBySlice(a[:])    // 改用 Slice
fmt.Println(a);         // [2 3 4]
```

### 空白 Body

可以定義 function 但沒有 body。通常是用另一種程式語言來實作，比如 C。越是底層的工作越容易看到這樣子的做法。

### Recursion

遞迴

```go { .line-numbers }
func gcd(a, b int) int {
    if b == 0 {
        return a
    }

    return gcd(b, a % b)
}

fmt.Println(gcd(24, 128)) // 8
```

### Return Tuple

Go 的 function 可以一次回傳多個值 (tuple)

eg:

```go { .line-numbers }
func swap(x, y int) (int, int) {
    return y, x
}

a, b := 1, 2        // a = 1, b = 2
a, b = swap(a, b)   // a = 2, b = 1
```

### Variadic Functions

function 的參數個數是不固定的。

eg:

sample 1:

```go { .line-numbers }
func sum(vals ...int) int {
    total := 0
    for _, val := range vals {
        total += val
    }
    return total
}

fmt.Println(sum())           //  "0"
fmt.Println(sum(3))          //  "3"
fmt.Println(sum(1, 2, 3, 4)) //  "10"
```

如何將 slice 傳入:

```go { .line-numbers }
values := []int{1, 2, 3, 4}
fmt.Println(sum(values...)) // "10"
```

### Deferred function call

在 code block 或 function 結束前，一定要執行的程式碼。與 Java `finally` 很像。

```go { .line-numbers }
func double(x int) (result int) {
    defer func() { fmt.Printf("double(%d) = %d\n", x, result) }()
    return x + x
}

_ = double(4) // double(4) = 8
```

在有關 I/O 處理時，一定會用到。

```go { .line-numbers }
func ReadFile(filename string) ([]byte, error) {
    f, err := os.Open(filename)

    if err != nil {
        return nil, err
    }

    defer f.Close()
    return ReadAll(f)
}
```

**defer** 的呼叫順序是 **stack** 的LIFO (Last In First Out)，並且利用當下的變數值來執行。如下：

```go { .line-numbers }
package main

import "fmt"

func a1() {
    for i := 0; i < 3; i++ {
        defer fmt.Print(i, " ")
    }
}

func a2() {
    for i := 0; i < 3; i++ {
        defer func() {
            fmt.Print(i, " ")
        }()
    }
}

func a3() {
    for i := 0; i < 3; i++ {
        defer func(n int) {
            fmt.Print(n, " ")
        }(i)
    }
}

func main() {
    a1()            // 2 1 0
    fmt.Println()
    a2()            // 3 3 3
    fmt.Println()
    a3()            // 2 1 0
    fmt.Println()
}
```

a1:
使用當下迴圈 i 的變數值。因此會是 **2 1 0**

a2:
每次迴圈完成時，會記錄要執行一個 **anonymous function**，當迴圈結束後，則開始執行 defer 記錄的 function，此時 i 的值已經是 **3**。

a3:
與 a2 類似，多傳入當下 i 的值，因此結果與會 a1 相同。

使用 **defer** 要特別小心當下操作的變數。

### Error Handling, Panic, Revcover

#### errors

在 Go 的 function 設計中，很多都會回傳包含 error 的 tuple。eg:

```go { .line-numbers }
resp, err := http.Get(url)

if err != nil {
    return nil, err
}
```

網路上戲稱是 Go 的 **error hell**。

eg:

```go { .line-numbers }
func WaitForServer(url string) error {
    const timeout = 1 * time.Minute
    deadline := time.Now().Add(timeout)

    for tries := 0; time.Now().Before(deadline); tries++ {
        _, err := http.Head(url)
        if err == nil {
            return nil // success
        }

        log.Printf("server not responding (%s); retrying...", err)
        time.Sleep(time.Second << uint(tries)) // exponential back-off
    }
    return fmt.Errorf("server %s failed to respond after %s", url, timeout)
}

if err := WaitForServer(url); err != nil {
    fmt.Fprintf(os.Stderr, "Site is down: %v\n", err)
    os.Exit(1)
}
```

#### Panic

與 Java Exception 類似，但是 `panic` 會導致程式中斷。在 Go 的設計中，除非是很嚴重的錯誤，才會使用 **panic**，如像 I/O, 設定檔錯誤，可以預期到，可以在撰寫程式時，可以檢查的，則儘量用 **error** 來處理。

##### Panic 現像：

```go { .line-numbers }
func f(x int) {
    fmt.Printf("f(%d)\n", x+0/x) // panics if x == 0
    defer fmt.Printf("defer %d\n", x)
    f(x - 1)
}

f(3)
```

Output:

```text
f(3)
f(2)
f(1)
defer 1
defer 2
defer 3
panic: runtime error: integer divide by zero

goroutine 1 [running]:
main.f(0x0, 0x11cfbc)
    /tmp/sandbox799571508/main.go:8 +0x220
main.f(0x1, 0x11cfbc)
    /tmp/sandbox799571508/main.go:10 +0x180
main.f(0x2, 0x11cfbc)
    /tmp/sandbox799571508/main.go:10 +0x180
main.f(0x3, 0xe19)
    /tmp/sandbox799571508/main.go:10 +0x180
main.main()
    /tmp/sandbox799571508/main.go:20 +0x20
```

不應該使用 Panic 的案例，請回傳 **error**:

```go { .line-numbers }
func Reset(x *Buffer) {
    if x == nil {
        panic("x is nil") // unnecessary!
    }
    x.elements = nil
}
```

#### Recover

用在取得 panic 發生的原因，通常與 **defer** 撘配使用，用在 debug 執行時期(runtime)的錯誤。

eg:

```go { .line-numbers }
package main

import "fmt"

func f(x int) {
    fmt.Printf("f(%d)\n", x+0/x) // panics if x == 0
    defer fmt.Printf("defer %d\n", x)
    f(x - 1)
}

func main() {
    defer func() {
        if p := recover(); p != nil {
            fmt.Printf("internal error: %v\n", p)
        }
    }()

    f(3)
}
```

output:

```text
f(3)
f(2)
f(1)
defer 1
defer 2
defer 3
internal error: runtime error: integer divide by zero
```

### Signature

一個 function 的型別，通常也稱做 **signature**。兩個 function 有相同的 signature，需滿足以下兩個條件：

1. 參數 (parameters) 資料型別與順序相同，與參數名稱無關。
1. 回傳的值的資料型別與順序相同

eg:

```go { .line-numbers }
func add(x int, y int) int { return x+y }
func sub(x, y int) (z int) { z= x - y; return }
func first(x int, _ int) int { return x }
func zero(int, int) int { return 0 }

fmt.Printf("%T\n", add)   // "func(int, int) int"
fmt.Printf("%T\n", sub)   // "func(int, int) int"
fmt.Printf("%T\n", first) // "func(int, int) int"
fmt.Printf("%T\n", zero)  // "func(int, int) int"
```

在 Go 的 function 也可以當作參數與回傳值。也因此 Go 也算是一種 first-class lanaugage.

### First-Class

function 也有資料型別，可以當作變數，或當作另一個 function 的參數及回傳值。
以 Go 來說，**signature** 是 Function 的資料型別。當宣告 funcation 沒有指定 name 時，則稱為 **anonymous function**

Assignment:

```go { .line-numbers }
func square(n int) int { return n * n }
func negative(n int) int { return -n }
func product(m, n int) int { return m * n }

var f func(int) int     // signature
fmt.Printf("%T\n", f)   // "func(int) int"

f = square
fmt.Println(f(3))       // "9"

f = negative
fmt.Println(f(3))       // "-3"

f = product // cannot use product (type func(int, int) int) as type func(int) int in assignment
```

As parameter and return:

```go { .line-numbers }
func square(n int) int { return n * n }
func negative(n int) int { return -n }

func compose(f, g func(int) int) func(int) int {
    return func(a int) int {        // anonymous function
        return g(f(a))
    }
}

k1 := compose(square, negative)
fmt.Printf("%T\n", k1)              // func(int) int
fmt.Println(k1(10))                 // -100 negative(square(10))

k2 := compose(negative, square)
fmt.Printf("%T\n", k2)              // func(int) int
fmt.Println(k2(10))                 // 100 square(negative(10))
```

## Methods

在 OOP 中，會定義 Class 的 Method 來處理資料。在 Go 也有一樣的功能，主要是針對 struct 來定義 method.

eg:

```go { .line-numbers }
type Point struct{ X, Y float64 }

// traditional function
func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

// same thing, but as a method of the Point type
func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

p := Point{1, 2}
q := Point{4, 6}
fmt.Println(Distance(p, q)) // "5", function call
fmt.Println(p.Distance(q))  // "5", method call
```

### Declaration Method for Struct Pointer

在定義 method 也可以使用 struct pointer。

eg:

```go { .line-numbers }
package main

import "fmt"
import "math"

type Point struct{ X, Y float64 }

func (p *Point) Distance(q Point) float64 {
    if p == nil {
        return math.Hypot(q.X, q.Y)
    } else {
        return math.Hypot(q.X-p.X, q.Y-p.Y)
    }
}

func main() {
    p := Point{1, 2}
    q := Point{4, 6}
    fmt.Println(p.Distance(q))

    x := &p
    fmt.Println(x.Distance(q))

    var y *Point
    fmt.Println(y)
    fmt.Println(y.Distance(q))
}
```

用 struct pointer 定義 method 要特別注意會修改到原本的值。

sample 1, struct:

```go { .line-numbers }
package main

import "fmt"

type Point struct{ X, Y float64 }

func (p Point) ScaleBy(factor float64) {
    p.X *= factor
    p.Y *= factor
}

func main() {
    p := Point{1, 2}
    p.ScaleBy(10)
    fmt.Println(p) // {1 2}

    q := &Point{1, 2}
    q.ScaleBy(10)
    fmt.Println(q) // &{1 2}
}
```

sample 2, struct pointer:

```go { .line-numbers }
package main

import "fmt"

type Point struct{ X, Y float64 }

func (p *Point) ScaleBy(factor float64) {
    p.X *= factor
    p.Y *= factor
}

func main() {
    p := Point{1, 2}
    p.ScaleBy(10)
    fmt.Println(p) // {10 20}

    q := &Point{1, 2}
    q.ScaleBy(10)
    fmt.Println(q) // &{10 20}
}
```

**注意**: 與 slice 類似，但因為是 method 很難查覺是否有修改原本的資料。因此在實作上，儘量 method 都用 pointer 的方式。

1. 避免 pass by value 的記憶體浪費
1. 避免 golang 在 struct pointer 語法上的 puzzle (因為 struct 與 struct pointer 在 call method 的語法都一樣，不像 C 有分 `.` 與 `->`).

### Method Signature

method 本身就是 funcation，因此也有 signature.

```go { .line-numbers }
package main

import "fmt"

type Point struct{ X, Y float64 }

func (p *Point) ScaleBy(factor float64) {
    p.X *= factor
    p.Y *= factor
}

func main() {
    p := Point{1, 2}
    p.ScaleBy(10)
    fmt.Println(p)                          // {10 20}

    x := p.ScaleBy

    fmt.Printf("%T\n", x)                   // func(float64)

    x(100)
    fmt.Println(p)                          // {1000 2000}

    var xx func(float64) = p.ScaleBy
    xx(10)
    fmt.Println(p)                          // {10000 20000}
}
```