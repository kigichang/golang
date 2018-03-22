# 16 Cowork with C/C++ (swig)

在 Golang 有 cgo 與 gccgo 可以與 C 的程式互動。在 compile Golang 的程式時，可以 link C 的 library。

但 Golang 無法直接 link C++ 程式，因為 Golang 本身沒有 OOP 的設計，所以必須在 C++ 再封裝一層程式來使用。目前 golang 有支援 swig 套件，可以協助封裝 C/C++ 程式。

## Swig Introduction

## Wrap C++ Class with C
