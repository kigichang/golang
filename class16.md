# 16 Cowork with C/C++ (swig)

在 Golang 有 cgo 與 gccgo 可以與 C 的程式互動。在 compile Golang 的程式時，可以 link C 的 library。

但 Golang 無法直接使用 C++ 程式，因為 Golang 本身沒有 OOP 的設計，所以必須在 C++ 再封裝一層程式來使用。目前 golang 有支援 swig 套件，可以協助封裝 C/C++ 程式。

## Golang unsafe package

如果使用 Go 的 unsafe 套件，會失去相容與移植。因此如非必要，儘可能不要使用。

unsafe package 主要有三個 function 與一個 data type

- functions:
  - func Alignof(x ArbitraryType) uintptr: memory alignment
    > The unsafe.Alignof function reports the required alignment of its argument’s type. Like Sizeof, it may be applied to an expression of any type, and it yields a constant. Typically, boolean and numeric types are aligned to their size (up to a maximum of 8 bytes) and all other types are word-aligned. (where a **word** is **4** bytes on a **32-bit** platform and **8** bytes on a **64-bit** platform)

  - func Offsetof(x ArbitraryType) uintptr: offset from start for struct and array
  - func Sizeof(x ArbitraryType) uintptr: size of for type
- data type:
  - type Pointer: like void* in c

### sizeof and alignof

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

x, y, z 在 64-bit 系統下， alignment 都是 8 bytes. 但 x 的 size 是 24 bytes (3 words)，而其他 y, z 都是 16 bytes (2 words)。
Ｖ主要因為 x 的 bool (x.a) 與 int16 (x.c)中間是 float64 (x.b)，佔了 8 bytes (1 word)，bool 雖只佔 1 byte，但要補足成 8 bytes (1 word), 同理 x.c 只佔 2 bytes，也要補足成 8 bytes (1 word)。
而 y, z 因為 bool, int16 是相連，因此在 bool 後面補 1 bytes, int16 補 4 bytes，補足成 8 bytes(1 word)。所以 x 是 24 bytes，而 y, z 是 16 bytes。

### unsafe.Pointer

> unsafe.Pointer 可以是任意型別的指標。在 Golang 的 strong type 安全機制下，不同的資料型別與指標都不可以直接轉換，如：
>
>> - 两个不同指针类型的值，例如 int64和 float64。
>> - 指针类型和uintptr的值。
>
>但是借助unsafe.Pointer，我们可以打破Go类型和内存安全性，并使上面的转换成为可能。这怎么可能发生？让我们阅读unsafe包文档中列出的规则：
>
>任何类型的指针值都可以转换为unsafe.Pointer。
unsafe.Pointer可以转换为任何类型的指针值。
uintptr可以转换为unsafe.Pointer。
unsafe.Pointer可以转换为uintptr。
>
>The following patterns involving Pointer are valid. Code not using these patterns is likely to be invalid today or to become invalid in the future. Even the valid patterns below come with important caveats.

1. Conversion of a *T1 to Pointer to *T2.
1. Conversion of a Pointer to a uintptr (but not back to Pointer)
    Even if a uintptr holds the address of some object, the garbage collector will not update that uintptr's value if the object moves, nor will that uintptr keep the object from being reclaimed.
1. Conversion of a Pointer to a uintptr and back, with arithmetic.
1. Conversion of a Pointer to a uintptr when calling syscall.Syscall.
1. Conversion of the result of reflect.Value.Pointer or reflect.Value.UnsafeAddr from uintptr to Pointer.
1. Conversion of a reflect.SliceHeader or reflect.StringHeader Data field to or from Pointer.

## Swig Introduction

## Wrap C++ Class with C

## reference

1. [[译]Go里面的unsafe包详解Ｖ](https://gocn.io/question/371)
1. [Go 1 and the Future of Go Programs#Expections](https://golang.org/doc/go1compat#expectations)
