# 16 Cowork with C/C++ (swig)

在 Golang 有 cgo 與 gccgo 可以與 C 的程式互動。在 compile Golang 的程式時，可以 link C 的 library。

但 Golang 無法直接使用 C++ 程式，因為 Golang 本身沒有 OOP 的設計，所以必須在 C++ 再封裝一層程式來使用。目前 golang 有支援 swig 套件，可以協助封裝 C/C++ 程式。

## Golang unsafe package

因為 Go 的 unsafe 套件使用到系統底層的屬性，所以會失去相容與移植。如非必要，儘可能不要使用。

unsafe package 主要有三個 function 與一個 data type

- functions:
  - func Alignof(x ArbitraryType) uintptr: memory alignment
    > The unsafe.Alignof function reports the required alignment of its argument’s type. Like Sizeof, it may be applied to an expression of any type, and it yields a constant. Typically, boolean and numeric types are aligned to their size (up to a maximum of 8 bytes) and all other types are word-aligned. (where a **word** is **4** bytes on a **32-bit** platform and **8** bytes on a **64-bit** platform)

  - func Offsetof(x ArbitraryType) uintptr: offset from start for struct and array
  - func Sizeof(x ArbitraryType) uintptr: size of for type
- data type:
  - type Pointer: like void* in c

### Sizeof and Alignof

```go {.line-numbers}
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    var x struct {
        a bool
        b float64
        c int16
    }

    fmt.Println("Sizeof x:", unsafe.Sizeof(x))
    fmt.Println("Alignof x:", unsafe.Alignof(x))

    fmt.Println("Sizeof x.a:", unsafe.Sizeof(x.a), "AlignOf x.a:", unsafe.Alignof(x.a), "Offsetof x.a:", unsafe.Offsetof(x.a))
    fmt.Println("Sizeof x.b:", unsafe.Sizeof(x.b), "AlignOf x.b:", unsafe.Alignof(x.b), "Offsetof x.b:", unsafe.Offsetof(x.b))
    fmt.Println("Sizeof x.c:", unsafe.Sizeof(x.c), "AlignOf x.c:", unsafe.Alignof(x.c), "Offsetof x.c:", unsafe.Offsetof(x.c))

    var y struct {
        a float64
        b int16
        c bool
    }

    fmt.Println("Sizeof y:", unsafe.Sizeof(y))
    fmt.Println("Alignof y:", unsafe.Alignof(y))

    fmt.Println("Sizeof y.a:", unsafe.Sizeof(y.a), "AlignOf y.a:", unsafe.Alignof(y.a), "Offsetof y.a:", unsafe.Offsetof(y.a))
    fmt.Println("Sizeof y.b:", unsafe.Sizeof(y.b), "AlignOf y.b:", unsafe.Alignof(y.b), "Offsetof y.b:", unsafe.Offsetof(y.b))
    fmt.Println("Sizeof y.c:", unsafe.Sizeof(y.c), "AlignOf y.c:", unsafe.Alignof(y.c), "Offsetof y.c:", unsafe.Offsetof(y.c))

    var z struct {
        a bool
        b int16
        c float64
    }

    fmt.Println("Sizeof z:", unsafe.Sizeof(z))
    fmt.Println("Alignof z:", unsafe.Alignof(z))

    fmt.Println("Sizeof z.a:", unsafe.Sizeof(z.a), "AlignOf y.a:", unsafe.Alignof(z.a), "Offsetof z.a:", unsafe.Offsetof(z.a))
    fmt.Println("Sizeof z.b:", unsafe.Sizeof(z.b), "AlignOf y.b:", unsafe.Alignof(z.b), "Offsetof z.b:", unsafe.Offsetof(z.b))
    fmt.Println("Sizeof z.c:", unsafe.Sizeof(z.c), "AlignOf y.c:", unsafe.Alignof(z.c), "Offsetof z.c:", unsafe.Offsetof(z.c))
}
```

Output:

```text
Sizeof x: 24
Alignof x: 8
Sizeof x.a: 1 AlignOf x.a: 1 Offsetof x.a: 0
Sizeof x.b: 8 AlignOf x.b: 8 Offsetof x.b: 8
Sizeof x.c: 2 AlignOf x.c: 2 Offsetof x.c: 16
Sizeof y: 16
Alignof y: 8
Sizeof y.a: 8 AlignOf y.a: 8 Offsetof y.a: 0
Sizeof y.b: 2 AlignOf y.b: 2 Offsetof y.b: 8
Sizeof y.c: 1 AlignOf y.c: 1 Offsetof y.c: 10
Sizeof z: 16
Alignof z: 8
Sizeof z.a: 1 AlignOf y.a: 1 Offsetof z.a: 0
Sizeof z.b: 2 AlignOf y.b: 2 Offsetof z.b: 2
Sizeof z.c: 8 AlignOf y.c: 8 Offsetof z.c: 8
```

x, y, z 在 64-bit 系統下， alignment 都是 8 bytes. 但 x 的 size 是 24 bytes (3 words[^word])，而其他 y, z 都是 16 bytes (2 words)。
Ｖ主要因為 x 的 bool (x.a) 與 int16 (x.c)中間是 float64 (x.b)，佔了 8 bytes (1 word)，bool 雖只佔 1 byte，但要補足成 8 bytes (1 word), 同理 x.c 只佔 2 bytes，也要補足成 8 bytes (1 word)。
而 y, z 因為 bool, int16 是相連，因此在 bool 後面補 1 bytes, int16 補 4 bytes，補足成 8 bytes(1 word)。所以 x 是 24 bytes，而 y, z 是 16 bytes。

[^word]: 在 32 bit 系統下，1 word = 4 bytes (32bit), 64 bit 是 8 bytes (64bit)

### unsafe.Pointer

unsafe.Pointer 可以是任意型別的指標。在 Golang 的 strong type 安全機制下，不同的資料型別與指標都不可以直接轉換，如：

- 不同指標的值，即使是相同 bit 數，如 int64 和 float64。
- 指標 與 uintptr 的值。

unsafe.Pointer 可以破壞 Go 的安全機制；unsafe.Pointer 的功能有：

1. 任何資料型別的指標，都可以轉成 unsafe.Pointer。
1. unsafe.Pointer 也可轉成任何資料型別的指標。
1. unsafe.Pointer 可以轉成 uintptr，之後可以利用 uintptr 做指標位移。
1. uintptr 可以轉成 unsafe.Pointer，但有風險。

unsafe.Pointer 一定要遵守[官網](https://golang.org/pkg/unsafe/#Pointer)提到的使用模式，官網有提到，如果沒有遵守的話，現在雖然沒問題，但在未來不一定正確。

1. Conversion of a *T1 to Pointer to *T2，前提 T2 的記憶體空間，要小於或等於 T1。如果 T2 > T1 的話，有可能在操作 T2 時，會造成溢位。

    ```go {.line-numbers}
    package main

    import (
        "fmt"
        "unsafe"
    )

    func main() {

        f := 3.1415926
        t1 := *(*uint64)(unsafe.Pointer(&f))
        fmt.Printf("%v, %T\n", t1, t1)

        t2 := *(*uint32)(unsafe.Pointer(&f))
        fmt.Printf("%v, %T\n", t2, t2)

        var f1 float32 = 3.1415926
        t3 := (*uint64)(unsafe.Pointer(&f1))
        fmt.Printf("%v, %v, %T\n", t3, *t3, t3)
        fmt.Printf("%v, %v, %T\n", &f1, f1, f1)

        *t3 = 1<<63 - 1
        fmt.Printf("%v, %T\n", *t3, t3)
        fmt.Printf("%v, %T\n", f1, f1)
    }
    ```

    (以上 t3 是 *uint64，大於 float32，在最後造成 float32 溢位)

1. Conversion of a Pointer to a uintptr (but not back to Pointer)

    >Even if a uintptr holds the address of some object, the garbage collector will not update that uintptr's value if the object moves, nor will that uintptr keep the object from being reclaimed.

    官網文件提到，Pointer 可以轉 uintptr，但反之 uintptr 轉 Pointer 不行。因為 uintptr 只是存當時的記憶體位址，但實體 (instance) 有可能已經被 GC (Garbage collection)，因此轉成 Pointer 操作後，會有問題。下面的例子中，Go 的工具會幫忙檢查。(go vet)

    ```go {.line-numbers}
    package main

    import (
        "fmt"
        "unsafe"
    )

    func main() {
        a := 10

        ap1 := uintptr(unsafe.Pointer(&a))

        fmt.Printf("%x, %T\n", ap1, ap1)

        ap2 := (*int)(unsafe.Pointer(ap1)) // warning: possible misuse of unsafe.Pointer

        fmt.Printf("%x, %v, %T\n", ap2, *ap2, ap2)
    }
    ```

1. Conversion of a Pointer to a uintptr and back, with arithmetic.

    指標位移

    ```go {.line-numbers}
    package main

    import (
        "fmt"
        "unsafe"
    )

    type test struct {
        name string
        age  int
        addr string
    }

    func main() {

        slice := []int{1, 2, 3, 4, 5}

        snd := *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&slice[0])) + 1*unsafe.Sizeof(slice[0])))
        fmt.Printf("%v, %T\n", snd, snd)

        t := test{"A", 10, "ABC"}

        addr := *(*string)(unsafe.Pointer((uintptr(unsafe.Pointer(&t)) + unsafe.Offsetof(t.addr))))
        fmt.Printf("%v, %T\n", addr, addr)
    }
    ```

    以下的操作方式是**不正確**的，雖然是可以執行。

    1. 取資料的結束位址
    1. 先用變數保留 uintptr 再轉回 Pointer

    ```go {.line-numbers}
    package main

    import (
        "fmt"
        "unsafe"
    )

    type test struct {
        name string
        age  int
        addr string
    }

    func main() {
        slice := []int{1, 2, 3, 4, 5}

        out := unsafe.Pointer(uintptr(unsafe.Pointer(&slice[0])) + uintptr(len(slice))) // 取結束位址
        fmt.Println(out)

        // 暫存 uintptr, 再轉成 unsafe.Pointer()
        firstp := uintptr(unsafe.Pointer(&slice[0]))
        sndp := firstp + unsafe.Sizeof(slice[0])

        snd = *(*int)(unsafe.Pointer(sndp)) // warning: possible misuse of unsafe.Pointer

        t := test{"A", 10, "ABC"}

        end := unsafe.Pointer(uintptr(unsafe.Pointer(&t)) + unsafe.Sizeof(t)) // 取結束位址
        fmt.Println(end)
    }
    ```

1. Conversion of a Pointer to a uintptr when calling syscall.Syscall.
1. Conversion of the result of reflect.Value.Pointer or reflect.Value.UnsafeAddr from uintptr to Pointer.
1. Conversion of a reflect.SliceHeader or reflect.StringHeader Data field to or from Pointer.

## Swig Introduction

## Wrap C++ Class with C

## reference

1. [[译]Go里面的unsafe包详解Ｖ](https://gocn.io/question/371)
1. [Go 1 and the Future of Go Programs#Expections](https://golang.org/doc/go1compat#expectations)
