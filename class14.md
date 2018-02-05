# Go Class 14 Protobuf and gRPC

[Protocol Buffers (ProtoBuf) 官網](https://developers.google.com/protocol-buffers/)

[Developer Guide](https://developers.google.com/protocol-buffers/docs/overview)

[Protocol Buffer Basics: Go](https://developers.google.com/protocol-buffers/docs/gotutorial)

ProtoBuf 是 Google 開發的工具，主要來取代 JSON, 與 XML，通常會用在 RPC (Remote Procedure Call) 上，也因此 ProtoBuf 會撘配 Google 開發的 gRPC 使用。

ProtoBuf 本身支援多種常用的程式語言，也因此可以利用 ProtoBuf 當作中介的橋樑，在不同的程式語言間，交換資料。

## protoc

**protoc** 是 Protobuf 的工具，主要是將 protobuf 的定義檔 (.proto) 轉成對應的程式語言。

1. 到 [protoc release](https://github.com/google/protobuf/releases) 下載對應平台的執行檔。
1. 執行 `go get -u github.com/golang/protobuf/protoc-gen-go` 下載 protoc 的 go plugin。

### .proto

使用 protobuf 前，我們需要先定義資料格式，寫起來有點像在寫 struct。首先在專案目錄下，開一個目錄，如: `protos`，在 `protos` 下還可以依功能再細分。

```text
go_test/class14
├── Gopkg.lock
├── Gopkg.toml
├── hello_client
│   └── main.go
├── hello_service
│   └── main.go
├── protos
│   ├── test.go
│   ├── test.pb.go
│   └── test.proto
└── service
    ├── service.pb.go
    └── service.proto
```

撰寫 .proto

eg: protos/test.proto

```protobuf
syntax = "proto3";

package protos;

import "google/protobuf/timestamp.proto";

message Hello {
  string name = 1;
  google.protobuf.Timestamp time = 99;
}
```

組成元素：

1. syntax: `syntax = "proto3";` 指定 protobuf 的版本，目前有 proto2 與 proto3。建議用 proto3.
1. package: 定義程式的 package, eg: `package protos;`
1. import: 如果有用到其他的 protobuf 資料型別，一樣需要 import, eg: `import "google/protobuf/timestamp.proto";`
1. message: 定義資料結構 `message 資料名稱`

### 資料型別

[proto3](https://developers.google.com/protocol-buffers/docs/proto3)

對應表：

.proto Type | Notes | C++ Type | Java Type | Python Type[^[2]^](#type_2) | Go Type | Ruby Type | C# Type | PHP Type
| - | - | - | - | - | - | - | - | - |
double | | double | double | float | float64 | Float | double | float
float |  | float | float | float | float32 | Float | float | float
int32 | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead. | int32 | int | int | int32 | Fixnum or Bignum (as required) | int | integer
int64 | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead. | int64 | long | int/long[^[3]^](#type_3) | int64 | Bignum | long | integer/string[^[5]^](#type_5)
uint32 | Uses variable-length encoding. | uint32 | int[^[1]^](#type_1) | int/long[^[3]^](#type_3) | uint32 | Fixnum or Bignum (as required) | uint | integer
uint64 | Uses variable-length encoding. | uint64 | long[^[1]^](#type_1) | int/long[^[3]^](#type_3) | uint64 | Bignum | ulong | integer/string[^[5]^](#type_5)
sint32 | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s. | int32 | int | int | int32 | Fixnum or Bignum (as required) | int | integer
sint64 | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s. | int64 | long | int/long[^[3]^](#type_3) | int64 | Bignum | long | integer/string[5]
fixed32 | Always four bytes. More efficient than uint32 if values are often greater than 228. | uint32 | int[^[1]^](#type_1) | int | uint32 | Fixnum or Bignum (as required) | uint | integer
fixed64 | Always eight bytes. More efficient than uint64 if values are often greater than 256. | uint64 | long[^[1]^](#type_1) | int/long[^[3]^](#type_3) | uint64 | Bignum | ulong | integer/string[^[5]^](#type_5)
sfixed32 | Always four bytes. | int32 | int | int | int32 | Fixnum or Bignum (as required) | int | integer
sfixed64 | Always eight bytes. | int64 | long | int/long[^[3]^](#type_3) | int64 | Bignum | long | integer/string[^[5]^](#type_5)
bool |  | bool | boolean | bool | bool | TrueClass/FalseClass | bool | boolean
string | A string must always contain UTF-8 encoded or 7-bit ASCII text. | string | String | str/unicode[^[4]^](#type_4) | string | String (UTF-8) | string | string
bytes | May contain any arbitrary sequence of bytes. | string | ByteString | str | []byte | String (ASCII-8BIT) | ByteString | string

<a name="type_1"></a>[1] In Java, unsigned 32-bit and 64-bit integers are represented using their signed counterparts, with the top bit simply being stored in the sign bit.
<a name="type_2"></a>[2] In all cases, setting values to a field will perform type checking to make sure it is valid.
<a name="type_3"></a>[3] 64-bit or unsigned 32-bit integers are always represented as long when decoded, but can be an int if an int is given when setting the field. In all cases, the value must fit in the type represented when set. See [[2]](#type_2).
<a name="type_4"></a>[4] Python strings are represented as unicode on decode but can be str if an ASCII string is given (this is subject to change).
<a name="type_5"></a>[5] Integer is used on 64-bit machines and string is used on 32-bit machines.

#### message Hello

eg:

```protobuf
message Hello {
  string name = 1;
  google.protobuf.Timestamp time = 99;
}
```

欄位定義，每一組欄位定義後面都會有個數字。eg：`string name = 1;`。這個數字是指這個欄位的流水號，有點像資料庫的 primary key 的流水號。因此定義之後，不能再異動這個欄位的定義，否則會有相容性的問題。但可以移除這個欄位。如果有需要異動時，應該是再往下加新的欄位。

在相容性上，如果傳來的資料，缺少欄位的資料時，protobuf 會改成帶該欄位的 zero value。

### 轉成 Go 程式

1. 目錄切到 `$GOPATH/src`
1. 執行 `protoc --go_out=. go_test/class14/protos/*.proto`

在 `go_test/class14/protos` 的目錄下，會產生 `test.pb.go` 檔案。

eg:

```go { .line-numbers }
// Code generated by protoc-gen-go. DO NOT EDIT.
// source: go_test/class14/protos/test.proto

/*
Package protos is a generated protocol buffer package.

It is generated from these files:
    go_test/class14/protos/test.proto

It has these top-level messages:
    Hello
*/
package protos

import proto "github.com/golang/protobuf/proto"
import fmt "fmt"
import math "math"
import google_protobuf "github.com/golang/protobuf/ptypes/timestamp"

// Reference imports to suppress errors if they are not otherwise used.
var _ = proto.Marshal
var _ = fmt.Errorf
var _ = math.Inf

// This is a compile-time assertion to ensure that this generated file
// is compatible with the proto package it is being compiled against.
// A compilation error at this line likely means your copy of the
// proto package needs to be updated.
const _ = proto.ProtoPackageIsVersion2 // please upgrade the proto package

type Hello struct {
    Name string                     `protobuf:"bytes,1,opt,name=name" json:"name,omitempty"`
    Time *google_protobuf.Timestamp `protobuf:"bytes,99,opt,name=time" json:"time,omitempty"`
}

func (m *Hello) Reset()                    { *m = Hello{} }
func (m *Hello) String() string            { return proto.CompactTextString(m) }
func (*Hello) ProtoMessage()               {}
func (*Hello) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{0} }

func (m *Hello) GetName() string {
    if m != nil {
        return m.Name
    }
    return ""
}

func (m *Hello) GetTime() *google_protobuf.Timestamp {
    if m != nil {
        return m.Time
    }
    return nil
}

func init() {
    proto.RegisterType((*Hello)(nil), "protos.Hello")
}

func init() { proto.RegisterFile("go_test/class14/protos/test.proto", fileDescriptor0) }

var fileDescriptor0 = []byte{
    // 140 bytes of a gzipped FileDescriptorProto
    // ....
}

```

如果要需要新增功能，不要修改在這個檔案。要另外開檔案來處理。如: `test.go`. 否則更新 protobuf 定義時，會重新產生新的檔案，會原本修改的內容去除。

eg: test.go

```go {.line-numbers}
package protos

import (
    proto "github.com/golang/protobuf/proto"
    "github.com/golang/protobuf/ptypes"
)

// CreateHello ...
func CreateHello(name string) *Hello {
    return &Hello{
        Name: name,
        Time: ptypes.TimestampNow(),
    }
}

// UnmarshalHello ...
func UnmarshalHello(data []byte) (*Hello, error) {
    ret := &Hello{}

    if err := proto.Unmarshal(data, ret); err != nil {
        return nil, err
    }

    return ret, nil
}

// MarshalHello ...
func MarshalHello(data *Hello) ([]byte, error) {
    return proto.Marshal(data)
}
```

### Marshal / Unmarshal

使用 protobuf 與 JSON 類似。

eg:

```go { .line-numbers }
import (
    proto "github.com/golang/protobuf/proto"
)
// UnmarshalHello ...
func UnmarshalHello(data []byte) (*Hello, error) {
    ret := &Hello{}

    if err := proto.Unmarshal(data, ret); err != nil {
        return nil, err
    }

    return ret, nil
}

// MarshalHello ...
func MarshalHello(data *Hello) ([]byte, error) {
    return proto.Marshal(data)
}
```

## gRPC

也是撰寫 .proto ，建議定義 gRPC service 要與資料 message 分開, 只放 service 會用到的 message，一來程式管理比較方便，二來也避免互相干擾。

eg: service/service.proto

```protobuf
syntax = "proto3";

package service;

import "go_test/class14/protos/test.proto";

message Request {
    string name = 1;
}

service HelloService {
    rpc Hello(Request) returns (protos.Hello) {}
}
```

主要 gRPC 的定義是這一段：

```go { .line-numbers }
service HelloService {
    rpc Hello(Request) returns (protos.Hello) {}
}
```

用 `rpc` 與 `returns` 這兩個關鍵字來定義 service.

與上述動作一樣，切換到 $GOPATH/src，執行 `protoc --go_out=plugins=grpc:. go_test/class14/service/*.proto`。與上述不一樣的地方，是在 `--go_out` 這個多了 `plugins=grpc` 設定。

在 `go_test/class14/service` 的目錄下，會產生 `service.pb.go`，一樣不建議直接修改 `service.pb.go`，有新加功能，都另開檔案來處理，eg: `service.go`

eg: service.pb.go

```go { .line-numbers }
// Code generated by protoc-gen-go. DO NOT EDIT.
// source: go_test/class14/service/service.proto

/*
Package service is a generated protocol buffer package.

It is generated from these files:
    go_test/class14/service/service.proto

It has these top-level messages:
    Request
*/
package service

import proto "github.com/golang/protobuf/proto"
import fmt "fmt"
import math "math"
import protos "go_test/class14/protos"

import (
    context "golang.org/x/net/context"
    grpc "google.golang.org/grpc"
)

// Reference imports to suppress errors if they are not otherwise used.
var _ = proto.Marshal
var _ = fmt.Errorf
var _ = math.Inf

// This is a compile-time assertion to ensure that this generated file
// is compatible with the proto package it is being compiled against.
// A compilation error at this line likely means your copy of the
// proto package needs to be updated.
const _ = proto.ProtoPackageIsVersion2 // please upgrade the proto package

type Request struct {
    Name string `protobuf:"bytes,1,opt,name=name" json:"name,omitempty"`
}

func (m *Request) Reset()                    { *m = Request{} }
func (m *Request) String() string            { return proto.CompactTextString(m) }
func (*Request) ProtoMessage()               {}
func (*Request) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{0} }

func (m *Request) GetName() string {
    if m != nil {
        return m.Name
    }
    return ""
}

func init() {
    proto.RegisterType((*Request)(nil), "service.Request")
}

// Reference imports to suppress errors if they are not otherwise used.
var _ context.Context
var _ grpc.ClientConn

// This is a compile-time assertion to ensure that this generated file
// is compatible with the grpc package it is being compiled against.
const _ = grpc.SupportPackageIsVersion4

// Client API for HelloService service

type HelloServiceClient interface {
    Hello(ctx context.Context, in *Request, opts ...grpc.CallOption) (*protos.Hello, error)
}

type helloServiceClient struct {
    cc *grpc.ClientConn
}

func NewHelloServiceClient(cc *grpc.ClientConn) HelloServiceClient {
    return &helloServiceClient{cc}
}

func (c *helloServiceClient) Hello(ctx context.Context, in *Request, opts ...grpc.CallOption) (*protos.Hello, error) {
    out := new(protos.Hello)
    err := grpc.Invoke(ctx, "/service.HelloService/Hello", in, out, c.cc, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

// Server API for HelloService service

type HelloServiceServer interface {
    Hello(context.Context, *Request) (*protos.Hello, error)
}

func RegisterHelloServiceServer(s *grpc.Server, srv HelloServiceServer) {
    s.RegisterService(&_HelloService_serviceDesc, srv)
}

func _HelloService_Hello_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(Request)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(HelloServiceServer).Hello(ctx, in)
    }
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/service.HelloService/Hello",
    }
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(HelloServiceServer).Hello(ctx, req.(*Request))
    }
    return interceptor(ctx, in, info, handler)
}

var _HelloService_serviceDesc = grpc.ServiceDesc{
    ServiceName: "service.HelloService",
    HandlerType: (*HelloServiceServer)(nil),
    Methods: []grpc.MethodDesc{
        {
            MethodName: "Hello",
            Handler:    _HelloService_Hello_Handler,
        },
    },
    Streams:  []grpc.StreamDesc{},
    Metadata: "go_test/class14/service/service.proto",
}

func init() { proto.RegisterFile("go_test/class14/service/service.proto", fileDescriptor0) }

var fileDescriptor0 = []byte{
    // 140 bytes of a gzipped FileDescriptorProto
    // ...
}
```

主要會定義 server 與 client 的 interface。

eg:

```go { .line-numbers }
type HelloServiceClient interface {
    Hello(ctx context.Context, in *Request, opts ...grpc.CallOption) (*protos.Hello, error)
}

type HelloServiceServer interface {
    Hello(context.Context, *Request) (*protos.Hello, error)
}
```

### gRPC Service

```go { .line-numbers }
package main

import (
    "context"
    "fmt"
    "go_test/class14/protos"
    "go_test/class14/service"
    "log"
    "net"

    "google.golang.org/grpc"
    "google.golang.org/grpc/reflection"
)

type helloService struct{}

func (h *helloService) Hello(ctx context.Context, req *service.Request) (*protos.Hello, error) {
    if req == nil || "" == req.Name {
        return nil, fmt.Errorf("request is not ok: %v", req)
    }

    return protos.CreateHello(req.Name), nil
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }

    s := grpc.NewServer()

    service.RegisterHelloServiceServer(s, &helloService{})

    reflection.Register(s)

    log.Println("serving...")
    if err := s.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }

    log.Println("start....")
}
```

說明：

1. listen port: `lis, err := net.Listen("tcp", ":50051")`
1. New gRPC Server: `s := grpc.NewServer()`
1. register:

    ```go { .line-numbers }
    service.RegisterHelloServiceServer(s, &helloService{})

    reflection.Register(s)
    ```
1. Serv:

    ```go { .line-numbers }
    if err := s.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
    ```

### gRPC client

```go { .line-numbers }
package main

import (
    "context"
    "fmt"
    "log"

    "go_test/class14/service"

    "google.golang.org/grpc"
)

func main() {

    conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
    if err != nil {
        panic(fmt.Sprintf("dial grpc server error: %v", err))
    }
    defer conn.Close()

    client := service.NewHelloServiceClient(conn)

    resp, err := client.Hello(context.Background(), &service.Request{Name: "Bob"})

    log.Println(resp)

    log.Println("end...")
}
```

說明：

1. connect to service: `conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())`, 因為沒有設定加密，因此要多一個 `grpc.WithInsecure()` 選項。(gRPC 預設是要用加密的，但我們沒有加密的相關設定，因此請用 *Insecure*)
1. 透過 connection 產生 client: `client := service.NewHelloServiceClient(conn)`
1. 呼叫 service 的 function: `resp, err := client.Hello(context.Background(), &service.Request{Name: "Bob"})`