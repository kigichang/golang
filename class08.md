# Go Class 08 Interface

Interface 宣告:

```go
type Name interface {
    FuncName(ParameterName DataType) DataType
}
```

eg:

```go
type Chaincode interface {
    Init(stub ChaincodeStubInterface) pb.Response
    Invoke(stub ChaincodeStubInterface) pb.Response
}
```

Interface 與 Java 類似，用 struct 的 method 來實作 interface 指定的 method。這邊需注意的是，在實作 interface 的 method 時，是用 struct 還是 pointer.

sample 1 用 **struct** 實作 interface method

```go
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
p.ScaleBy(10)
fmt.Println(p)  // {100 200}
```

sample 2 用 **pointer** 實作 interface method

```go
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
p.ScaleBy(10)
fmt.Println(p)  // {10000 20000}

Test(p, 10)     // cannot use p (type Point) as type Scale in argument to Test: Point does not implement Scale (ScaleBy method has pointer receiver)
```

## Stringer interface

**Stringer** interface 有一個 `String()`，功能類似 Java Object 的 **toString**.

```go
type Stringer interface {
    String() string
}
```

eg:

```go
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

```go
var any interface{}
fmt.Printf("%T\n", any)     // <nil>

a := 10
b := 100.0
c := "string"
d := struct {A, B string}{"foo", "boo"}
e := []string{}
f := map[string]int{}

any = a
fmt.Printf("%T\n", any)     // int

any = &a
fmt.Printf("%T\n", any)     // *int

any = b
fmt.Printf("%T\n", any)     // float64

any = &b
fmt.Printf("%T\n", any)     // *float64

any = c
fmt.Printf("%T\n", any)     // string

any = &c
fmt.Printf("%T\n", any)     // *string

any = d
fmt.Printf("%T\n", any)     // struct { A string; B string }

any = &d
fmt.Printf("%T\n", any)     // *struct { A string; B string }

any = e
fmt.Printf("%T\n", any)     // []string

any = &e
fmt.Printf("%T\n", any)     // *[]string

any = f
fmt.Printf("%T\n", any)     // map[string]int

any = &f
fmt.Printf("%T\n", any)     // *map[string]int
```
