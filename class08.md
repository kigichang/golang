# Go Class 08 Interface

Interface 與 Java 類似，用 struct 的 method 來實作 interface 指定的 method。這邊需注意的是，在實作 interface 的 method 時，是用 struct 還是 pointer.

sample 1 用 **struct** 實作 interface method

```go {.line-numbers}
type Scale interface {
    ScaleBy(float64)
}

func Test(s Scale, f float64) {
    s.ScaleBy(f)
}

type Point struct {
    X float64
    Y float64
}

func (p Point) ScaleBy(a float64) {
    p.X *= a
    p.Y *= a
}

p := Point{ 100.0, 200.0 }

fmt.Println(p)  // {100 200}
Test(p, 10)
fmt.Println(p)  // {100 200}
```

sample 2 用 **pointer** 實作 interface method

```go {.line-numbers}
type Scale interface {
    ScaleBy(float64)
}

func Test(s Scale, f float64) {
    s.ScaleBy(f)
}

type Point struct {
    X float64
    Y float64
}

func (p *Point) ScaleBy(a float64) {
    p.X *= a
    p.Y *= a
}

p := Point{ 100.0, 200.0 }

fmt.Println(p)  // {100 200}
Test(&p, 10)
fmt.Println(p)  // {1000 2000}

Test(p, 10)     // cannot use p (type Point) as type Scale in argument to Test: Point does not implement Scale (ScaleBy method has pointer receiver)
```

## Stringer interface

**Stringer** interface 有一個 `String()`，功能類似 Java Object 的 **toString**. 

```go {.line-numbers}
type Stringer interface {
    String() string
}
```

eg:

```go {.line-numbers}
type Point struct {
    X float64
    Y float64
}

func (p Point) String() string {
    return fmt.Sprintf("(%f, %f)", p.X, p.Y)
}

p := Point{ 100.0, 200.0 }
fmt.Println(p)              // (100.000000, 200.000000)
```

## interface{}

An interface value (**interface{}**) can hold arbitrarily large dynamic values

```go {.line-numbers}
var x interface{} = time.Now()
```
