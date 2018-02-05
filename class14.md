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
diviner/protos
├── common
│   ├── common.go
│   ├── common.pb.go
│   ├── common.proto
│   └── common_test.go
├── lmsr
│   ├── asset.go
│   ├── asset_test.go
│   ├── data.pb.go
│   ├── data.proto
│   ├── event.go
│   ├── event_test.go
│   ├── lmsr.go
│   ├── lmsr_test.go
│   ├── market.go
│   └── market_test.go
├── member
│   ├── member.go
│   ├── member.pb.go
│   ├── member.proto
│   └── member_test.go
└── service
    ├── service.go
    ├── service.pb.go
    └── service.proto
```

撰寫 .proto

eg: common/common.proto

```protobuf
syntax = "proto3";

package common;

import "google/protobuf/timestamp.proto";

message Verification {
  bytes public_key = 1;
  bytes hash = 2;
  bytes signature = 3;
  google.protobuf.Timestamp time = 4;
}
```

組成元素：

1. syntax: `syntax = "proto3";` 指定 protobuf 的版本，目前有 proto2 與 proto3。建議用 proto3.
1. package: 定義程式的 package, eg: `package prediction;`
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

#### message Verification

eg:

```protobuf
message Verification {
  bytes public_key = 1;
  bytes hash = 2;
  bytes signature = 3;
  google.protobuf.Timestamp time = 4;
}
```

欄位定義，每一組欄位定義後面都會有個數字。eg：`bytes public_key = 1;`。這個數字是指這個欄位的流水號，有點像資料庫的 primary key 的流水號。因此定義之後，不能再異動這個欄位的定義，否則會有相容性的問題。但可以移除這個欄位。如果有需要異動時，應該是再往下加新的欄位。

在相容性上，如果傳來的資料，缺少欄位的資料時，protobuf 會改成帶該欄位的 zero value。

### 轉成 Go 程式

1. 目錄切到 `$GOPATH/src`
1. 執行 `protoc --go_out=. diviner/protos/common/*.proto`

在 `diviner/protos/common` 的目錄下，會產生 `common.pb.go` 檔案。

eg:

```go { .line-numbers }
// Code generated by protoc-gen-go. DO NOT EDIT.
// source: diviner/protos/common/common.proto

/*
Package common is a generated protocol buffer package.

It is generated from these files:
    diviner/protos/common/common.proto

It has these top-level messages:
    Verification
*/
package common

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

type Verification struct {
    PublicKey []byte                     `protobuf:"bytes,1,opt,name=public_key,json=publicKey,proto3" json:"public_key,omitempty"`
    Hash      []byte                     `protobuf:"bytes,2,opt,name=hash,proto3" json:"hash,omitempty"`
    Signature []byte                     `protobuf:"bytes,3,opt,name=signature,proto3" json:"signature,omitempty"`
    Time      *google_protobuf.Timestamp `protobuf:"bytes,4,opt,name=time" json:"time,omitempty"`
}

func (m *Verification) Reset()                    { *m = Verification{} }
func (m *Verification) String() string            { return proto.CompactTextString(m) }
func (*Verification) ProtoMessage()               {}
func (*Verification) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{0} }

func (m *Verification) GetPublicKey() []byte {
    if m != nil {
        return m.PublicKey
    }
    return nil
}

func (m *Verification) GetHash() []byte {
    if m != nil {
        return m.Hash
    }
    return nil
}

func (m *Verification) GetSignature() []byte {
    if m != nil {
        return m.Signature
    }
    return nil
}

func (m *Verification) GetTime() *google_protobuf.Timestamp {
    if m != nil {
        return m.Time
    }
    return nil
}

func init() {
    proto.RegisterType((*Verification)(nil), "common.Verification")
}

func init() { proto.RegisterFile("diviner/protos/common/common.proto", fileDescriptor0) }

var fileDescriptor0 = []byte{
    // ...
}
```

如果要需要新增功能，不要修改在這個檔案。要另外開檔案來處理。如: `common.go`. 否則更新 protobuf 定義時，會重新產生新的檔案，會原本修改的內容去除。

### Marshal / Unmarshal

使用 protobuf 與 JSON 類似。

eg:

```go { .line-numbers }
import "github.com/golang/protobuf/proto"

// UnmarshalMarket ...
func UnmarshalMarket(data []byte) (*Market, error) {
    mkt := &Market{}
    if err := proto.Unmarshal(data, mkt); err != nil {
        return nil, err
    }
    return mkt, nil
}

// MarshalMarket ...
func MarshalMarket(m *Market) ([]byte, error) {
    return proto.Marshal(m)
}
```

## gRPC

也是撰寫 .proto ，建議定義 gRPC service 要與資料 message 分開, 只放 service 會用到的 message，一來程式管理比較方便，二來也避免互相干擾。

eg: protos/service/service.proto

```protobuf
syntax = "proto3";

package service;

import "diviner/protos/common/common.proto";
import "diviner/protos/member/member.proto";
import "diviner/protos/lmsr/data.proto";
import "google/protobuf/timestamp.proto";

message QueryRequest {
  string id = 1;
  common.Verification check = 999;
}
message MemberCreateRequest {
  member.Member member = 1;
  common.Verification check = 999;
}

message MemberInfoResponse {
  member.Member member = 1;
  google.protobuf.Timestamp time = 999;
}

message EventCreateRequest {
  string user = 1;
  string title = 2;
  repeated string outcome = 3;
  common.Verification check = 999;
}

message EventInfoResponse {
  lmsr.Event event = 1;
  google.protobuf.Timestamp time = 999;
}

message MarketCreateRequest {
  string user = 1;
  string event = 2;
  double num = 3;
  bool isFund = 4;
  common.Verification check = 999;
}

message MarketInfoResponse {
  lmsr.Market market = 1;
  google.protobuf.Timestamp time = 999;
}

message TxRequest {
  string user = 1;
  bool isBuy = 2;
  string share = 3;
  double volume = 4;
  common.Verification check = 999;
}

message TxResponse {
  double price = 1;
  google.protobuf.Timestamp time = 999;
}

service DivinerSerivce {
  rpc QueryMember (QueryRequest) returns (MemberInfoResponse) {}
  rpc CreateMember (MemberCreateRequest) returns (MemberInfoResponse) {}

  rpc QueryEvent(QueryRequest) returns (EventInfoResponse) {}
  rpc CreateEvent(EventCreateRequest) returns (EventInfoResponse) {}

  rpc QueryMarket(QueryRequest) returns (MarketInfoResponse) {}
  rpc CreateMarket(MarketCreateRequest) returns (MarketInfoResponse) {}

  rpc Tx(TxRequest) returns (TxResponse) {}
}
```

主要 gRPC 的定義是這一段：

```go { .line-numbers }
service DivinerSerivce {
  rpc QueryMember (QueryRequest) returns (MemberInfoResponse) {}
  rpc CreateMember (MemberCreateRequest) returns (MemberInfoResponse) {}

  rpc QueryEvent(QueryRequest) returns (EventInfoResponse) {}
  rpc CreateEvent(EventCreateRequest) returns (EventInfoResponse) {}

  rpc QueryMarket(QueryRequest) returns (MarketInfoResponse) {}
  rpc CreateMarket(MarketCreateRequest) returns (MarketInfoResponse) {}

  rpc Tx(TxRequest) returns (TxResponse) {}
}
```

用 `rpc` 與 `returns` 這兩個關鍵字來定義 service.

與上述動作一樣，切換到 $GOPATH/src，執行 `protoc --go_out=plugins=grpc:. diviner/protos/service/*.proto`。與上述不一樣的地方，是在 `--go_out` 這個多了 `plugins=grpc` 設定。

在 `diviner/protos/service` 的目錄下，會產生 `service.pb.go`，一樣不建議直接修改 `service.pb.go`，有新加功能，都另開檔案來處理，eg: `service.go`

eg: service.pb.go

```go { .line-numbers }
// Code generated by protoc-gen-go. DO NOT EDIT.
// source: diviner/protos/service/service.proto

/*
Package service is a generated protocol buffer package.

It is generated from these files:
    diviner/protos/service/service.proto

It has these top-level messages:
    QueryRequest
    MemberCreateRequest
    MemberInfoResponse
    EventCreateRequest
    EventInfoResponse
    MarketCreateRequest
    MarketInfoResponse
    TxRequest
    TxResponse
*/
package service

import proto "github.com/golang/protobuf/proto"
import fmt "fmt"
import math "math"
import common "diviner/protos/common"
import member "diviner/protos/member"
import lmsr "diviner/protos/lmsr"
import google_protobuf "github.com/golang/protobuf/ptypes/timestamp"

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

type QueryRequest struct {
    Id    string               `protobuf:"bytes,1,opt,name=id" json:"id,omitempty"`
    Check *common.Verification `protobuf:"bytes,999,opt,name=check" json:"check,omitempty"`
}

func (m *QueryRequest) Reset()                    { *m = QueryRequest{} }
func (m *QueryRequest) String() string            { return proto.CompactTextString(m) }
func (*QueryRequest) ProtoMessage()               {}
func (*QueryRequest) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{0} }

func (m *QueryRequest) GetId() string {
    if m != nil {
        return m.Id
    }
    return ""
}

func (m *QueryRequest) GetCheck() *common.Verification {
    if m != nil {
        return m.Check
    }
    return nil
}

type MemberCreateRequest struct {
    Member *member.Member       `protobuf:"bytes,1,opt,name=member" json:"member,omitempty"`
    Check  *common.Verification `protobuf:"bytes,999,opt,name=check" json:"check,omitempty"`
}

func (m *MemberCreateRequest) Reset()                    { *m = MemberCreateRequest{} }
func (m *MemberCreateRequest) String() string            { return proto.CompactTextString(m) }
func (*MemberCreateRequest) ProtoMessage()               {}
func (*MemberCreateRequest) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{1} }

func (m *MemberCreateRequest) GetMember() *member.Member {
    if m != nil {
        return m.Member
    }
    return nil
}

func (m *MemberCreateRequest) GetCheck() *common.Verification {
    if m != nil {
        return m.Check
    }
    return nil
}

type MemberInfoResponse struct {
    Member *member.Member             `protobuf:"bytes,1,opt,name=member" json:"member,omitempty"`
    Time   *google_protobuf.Timestamp `protobuf:"bytes,999,opt,name=time" json:"time,omitempty"`
}

func (m *MemberInfoResponse) Reset()                    { *m = MemberInfoResponse{} }
func (m *MemberInfoResponse) String() string            { return proto.CompactTextString(m) }
func (*MemberInfoResponse) ProtoMessage()               {}
func (*MemberInfoResponse) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{2} }

func (m *MemberInfoResponse) GetMember() *member.Member {
    if m != nil {
        return m.Member
    }
    return nil
}

func (m *MemberInfoResponse) GetTime() *google_protobuf.Timestamp {
    if m != nil {
        return m.Time
    }
    return nil
}

type EventCreateRequest struct {
    User    string               `protobuf:"bytes,1,opt,name=user" json:"user,omitempty"`
    Title   string               `protobuf:"bytes,2,opt,name=title" json:"title,omitempty"`
    Outcome []string             `protobuf:"bytes,3,rep,name=outcome" json:"outcome,omitempty"`
    Check   *common.Verification `protobuf:"bytes,999,opt,name=check" json:"check,omitempty"`
}

func (m *EventCreateRequest) Reset()                    { *m = EventCreateRequest{} }
func (m *EventCreateRequest) String() string            { return proto.CompactTextString(m) }
func (*EventCreateRequest) ProtoMessage()               {}
func (*EventCreateRequest) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{3} }

func (m *EventCreateRequest) GetUser() string {
    if m != nil {
        return m.User
    }
    return ""
}

func (m *EventCreateRequest) GetTitle() string {
    if m != nil {
        return m.Title
    }
    return ""
}

func (m *EventCreateRequest) GetOutcome() []string {
    if m != nil {
        return m.Outcome
    }
    return nil
}

func (m *EventCreateRequest) GetCheck() *common.Verification {
    if m != nil {
        return m.Check
    }
    return nil
}

type EventInfoResponse struct {
    Event *lmsr.Event                `protobuf:"bytes,1,opt,name=event" json:"event,omitempty"`
    Time  *google_protobuf.Timestamp `protobuf:"bytes,999,opt,name=time" json:"time,omitempty"`
}

func (m *EventInfoResponse) Reset()                    { *m = EventInfoResponse{} }
func (m *EventInfoResponse) String() string            { return proto.CompactTextString(m) }
func (*EventInfoResponse) ProtoMessage()               {}
func (*EventInfoResponse) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{4} }

func (m *EventInfoResponse) GetEvent() *lmsr.Event {
    if m != nil {
        return m.Event
    }
    return nil
}

func (m *EventInfoResponse) GetTime() *google_protobuf.Timestamp {
    if m != nil {
        return m.Time
    }
    return nil
}

type MarketCreateRequest struct {
    User   string               `protobuf:"bytes,1,opt,name=user" json:"user,omitempty"`
    Event  string               `protobuf:"bytes,2,opt,name=event" json:"event,omitempty"`
    Num    float64              `protobuf:"fixed64,3,opt,name=num" json:"num,omitempty"`
    IsFund bool                 `protobuf:"varint,4,opt,name=isFund" json:"isFund,omitempty"`
    Check  *common.Verification `protobuf:"bytes,999,opt,name=check" json:"check,omitempty"`
}

func (m *MarketCreateRequest) Reset()                    { *m = MarketCreateRequest{} }
func (m *MarketCreateRequest) String() string            { return proto.CompactTextString(m) }
func (*MarketCreateRequest) ProtoMessage()               {}
func (*MarketCreateRequest) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{5} }

func (m *MarketCreateRequest) GetUser() string {
    if m != nil {
        return m.User
    }
    return ""
}

func (m *MarketCreateRequest) GetEvent() string {
    if m != nil {
        return m.Event
    }
    return ""
}

func (m *MarketCreateRequest) GetNum() float64 {
    if m != nil {
        return m.Num
    }
    return 0
}

func (m *MarketCreateRequest) GetIsFund() bool {
    if m != nil {
        return m.IsFund
    }
    return false
}

func (m *MarketCreateRequest) GetCheck() *common.Verification {
    if m != nil {
        return m.Check
    }
    return nil
}

type MarketInfoResponse struct {
    Market *lmsr.Market               `protobuf:"bytes,1,opt,name=market" json:"market,omitempty"`
    Time   *google_protobuf.Timestamp `protobuf:"bytes,999,opt,name=time" json:"time,omitempty"`
}

func (m *MarketInfoResponse) Reset()                    { *m = MarketInfoResponse{} }
func (m *MarketInfoResponse) String() string            { return proto.CompactTextString(m) }
func (*MarketInfoResponse) ProtoMessage()               {}
func (*MarketInfoResponse) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{6} }

func (m *MarketInfoResponse) GetMarket() *lmsr.Market {
    if m != nil {
        return m.Market
    }
    return nil
}

func (m *MarketInfoResponse) GetTime() *google_protobuf.Timestamp {
    if m != nil {
        return m.Time
    }
    return nil
}

type TxRequest struct {
    User   string               `protobuf:"bytes,1,opt,name=user" json:"user,omitempty"`
    IsBuy  bool                 `protobuf:"varint,2,opt,name=isBuy" json:"isBuy,omitempty"`
    Share  string               `protobuf:"bytes,3,opt,name=share" json:"share,omitempty"`
    Volume float64              `protobuf:"fixed64,4,opt,name=volume" json:"volume,omitempty"`
    Check  *common.Verification `protobuf:"bytes,999,opt,name=check" json:"check,omitempty"`
}

func (m *TxRequest) Reset()                    { *m = TxRequest{} }
func (m *TxRequest) String() string            { return proto.CompactTextString(m) }
func (*TxRequest) ProtoMessage()               {}
func (*TxRequest) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{7} }

func (m *TxRequest) GetUser() string {
    if m != nil {
        return m.User
    }
    return ""
}

func (m *TxRequest) GetIsBuy() bool {
    if m != nil {
        return m.IsBuy
    }
    return false
}

func (m *TxRequest) GetShare() string {
    if m != nil {
        return m.Share
    }
    return ""
}

func (m *TxRequest) GetVolume() float64 {
    if m != nil {
        return m.Volume
    }
    return 0
}

func (m *TxRequest) GetCheck() *common.Verification {
    if m != nil {
        return m.Check
    }
    return nil
}

type TxResponse struct {
    Price float64                    `protobuf:"fixed64,1,opt,name=price" json:"price,omitempty"`
    Time  *google_protobuf.Timestamp `protobuf:"bytes,999,opt,name=time" json:"time,omitempty"`
}

func (m *TxResponse) Reset()                    { *m = TxResponse{} }
func (m *TxResponse) String() string            { return proto.CompactTextString(m) }
func (*TxResponse) ProtoMessage()               {}
func (*TxResponse) Descriptor() ([]byte, []int) { return fileDescriptor0, []int{8} }

func (m *TxResponse) GetPrice() float64 {
    if m != nil {
        return m.Price
    }
    return 0
}

func (m *TxResponse) GetTime() *google_protobuf.Timestamp {
    if m != nil {
        return m.Time
    }
    return nil
}

func init() {
    proto.RegisterType((*QueryRequest)(nil), "service.QueryRequest")
    proto.RegisterType((*MemberCreateRequest)(nil), "service.MemberCreateRequest")
    proto.RegisterType((*MemberInfoResponse)(nil), "service.MemberInfoResponse")
    proto.RegisterType((*EventCreateRequest)(nil), "service.EventCreateRequest")
    proto.RegisterType((*EventInfoResponse)(nil), "service.EventInfoResponse")
    proto.RegisterType((*MarketCreateRequest)(nil), "service.MarketCreateRequest")
    proto.RegisterType((*MarketInfoResponse)(nil), "service.MarketInfoResponse")
    proto.RegisterType((*TxRequest)(nil), "service.TxRequest")
    proto.RegisterType((*TxResponse)(nil), "service.TxResponse")
}

// Reference imports to suppress errors if they are not otherwise used.
var _ context.Context
var _ grpc.ClientConn

// This is a compile-time assertion to ensure that this generated file
// is compatible with the grpc package it is being compiled against.
const _ = grpc.SupportPackageIsVersion4

// Client API for DivinerSerivce service

type DivinerSerivceClient interface {
    QueryMember(ctx context.Context, in *QueryRequest, opts ...grpc.CallOption) (*MemberInfoResponse, error)
    CreateMember(ctx context.Context, in *MemberCreateRequest, opts ...grpc.CallOption) (*MemberInfoResponse, error)
    QueryEvent(ctx context.Context, in *QueryRequest, opts ...grpc.CallOption) (*EventInfoResponse, error)
    CreateEvent(ctx context.Context, in *EventCreateRequest, opts ...grpc.CallOption) (*EventInfoResponse, error)
    QueryMarket(ctx context.Context, in *QueryRequest, opts ...grpc.CallOption) (*MarketInfoResponse, error)
    CreateMarket(ctx context.Context, in *MarketCreateRequest, opts ...grpc.CallOption) (*MarketInfoResponse, error)
    Tx(ctx context.Context, in *TxRequest, opts ...grpc.CallOption) (*TxResponse, error)
}

type divinerSerivceClient struct {
    cc *grpc.ClientConn
}

func NewDivinerSerivceClient(cc *grpc.ClientConn) DivinerSerivceClient {
    return &divinerSerivceClient{cc}
}

func (c *divinerSerivceClient) QueryMember(ctx context.Context, in *QueryRequest, opts ...grpc.CallOption) (*MemberInfoResponse, error) {
    out := new(MemberInfoResponse)
    err := grpc.Invoke(ctx, "/service.DivinerSerivce/QueryMember", in, out, c.cc, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

func (c *divinerSerivceClient) CreateMember(ctx context.Context, in *MemberCreateRequest, opts ...grpc.CallOption) (*MemberInfoResponse, error) {
    out := new(MemberInfoResponse)
    err := grpc.Invoke(ctx, "/service.DivinerSerivce/CreateMember", in, out, c.cc, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

func (c *divinerSerivceClient) QueryEvent(ctx context.Context, in *QueryRequest, opts ...grpc.CallOption) (*EventInfoResponse, error) {
    out := new(EventInfoResponse)
    err := grpc.Invoke(ctx, "/service.DivinerSerivce/QueryEvent", in, out, c.cc, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

func (c *divinerSerivceClient) CreateEvent(ctx context.Context, in *EventCreateRequest, opts ...grpc.CallOption) (*EventInfoResponse, error) {
    out := new(EventInfoResponse)
    err := grpc.Invoke(ctx, "/service.DivinerSerivce/CreateEvent", in, out, c.cc, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

func (c *divinerSerivceClient) QueryMarket(ctx context.Context, in *QueryRequest, opts ...grpc.CallOption) (*MarketInfoResponse, error) {
    out := new(MarketInfoResponse)
    err := grpc.Invoke(ctx, "/service.DivinerSerivce/QueryMarket", in, out, c.cc, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

func (c *divinerSerivceClient) CreateMarket(ctx context.Context, in *MarketCreateRequest, opts ...grpc.CallOption) (*MarketInfoResponse, error) {
    out := new(MarketInfoResponse)
    err := grpc.Invoke(ctx, "/service.DivinerSerivce/CreateMarket", in, out, c.cc, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

func (c *divinerSerivceClient) Tx(ctx context.Context, in *TxRequest, opts ...grpc.CallOption) (*TxResponse, error) {
    out := new(TxResponse)
    err := grpc.Invoke(ctx, "/service.DivinerSerivce/Tx", in, out, c.cc, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

// Server API for DivinerSerivce service

type DivinerSerivceServer interface {
    QueryMember(context.Context, *QueryRequest) (*MemberInfoResponse, error)
    CreateMember(context.Context, *MemberCreateRequest) (*MemberInfoResponse, error)
    QueryEvent(context.Context, *QueryRequest) (*EventInfoResponse, error)
    CreateEvent(context.Context, *EventCreateRequest) (*EventInfoResponse, error)
    QueryMarket(context.Context, *QueryRequest) (*MarketInfoResponse, error)
    CreateMarket(context.Context, *MarketCreateRequest) (*MarketInfoResponse, error)
    Tx(context.Context, *TxRequest) (*TxResponse, error)
}

func RegisterDivinerSerivceServer(s *grpc.Server, srv DivinerSerivceServer) {
    s.RegisterService(&_DivinerSerivce_serviceDesc, srv)
}

func _DivinerSerivce_QueryMember_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(QueryRequest)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(DivinerSerivceServer).QueryMember(ctx, in)
    }
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/service.DivinerSerivce/QueryMember",
    }
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(DivinerSerivceServer).QueryMember(ctx, req.(*QueryRequest))
    }
    return interceptor(ctx, in, info, handler)
}

func _DivinerSerivce_CreateMember_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(MemberCreateRequest)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(DivinerSerivceServer).CreateMember(ctx, in)
    }
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/service.DivinerSerivce/CreateMember",
    }
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(DivinerSerivceServer).CreateMember(ctx, req.(*MemberCreateRequest))
    }
    return interceptor(ctx, in, info, handler)
}

func _DivinerSerivce_QueryEvent_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(QueryRequest)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(DivinerSerivceServer).QueryEvent(ctx, in)
    }
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/service.DivinerSerivce/QueryEvent",
    }
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(DivinerSerivceServer).QueryEvent(ctx, req.(*QueryRequest))
    }
    return interceptor(ctx, in, info, handler)
}

func _DivinerSerivce_CreateEvent_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(EventCreateRequest)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(DivinerSerivceServer).CreateEvent(ctx, in)
    }
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/service.DivinerSerivce/CreateEvent",
    }
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(DivinerSerivceServer).CreateEvent(ctx, req.(*EventCreateRequest))
    }
    return interceptor(ctx, in, info, handler)
}

func _DivinerSerivce_QueryMarket_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(QueryRequest)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(DivinerSerivceServer).QueryMarket(ctx, in)
    }
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/service.DivinerSerivce/QueryMarket",
    }
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(DivinerSerivceServer).QueryMarket(ctx, req.(*QueryRequest))
    }
    return interceptor(ctx, in, info, handler)
}

func _DivinerSerivce_CreateMarket_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(MarketCreateRequest)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(DivinerSerivceServer).CreateMarket(ctx, in)
    }
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/service.DivinerSerivce/CreateMarket",
    }
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(DivinerSerivceServer).CreateMarket(ctx, req.(*MarketCreateRequest))
    }
    return interceptor(ctx, in, info, handler)
}

func _DivinerSerivce_Tx_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
    in := new(TxRequest)
    if err := dec(in); err != nil {
        return nil, err
    }
    if interceptor == nil {
        return srv.(DivinerSerivceServer).Tx(ctx, in)
    }
    info := &grpc.UnaryServerInfo{
        Server:     srv,
        FullMethod: "/service.DivinerSerivce/Tx",
    }
    handler := func(ctx context.Context, req interface{}) (interface{}, error) {
        return srv.(DivinerSerivceServer).Tx(ctx, req.(*TxRequest))
    }
    return interceptor(ctx, in, info, handler)
}

var _DivinerSerivce_serviceDesc = grpc.ServiceDesc{
    ServiceName: "service.DivinerSerivce",
    HandlerType: (*DivinerSerivceServer)(nil),
    Methods: []grpc.MethodDesc{
        {
            MethodName: "QueryMember",
            Handler:    _DivinerSerivce_QueryMember_Handler,
        },
        {
            MethodName: "CreateMember",
            Handler:    _DivinerSerivce_CreateMember_Handler,
        },
        {
            MethodName: "QueryEvent",
            Handler:    _DivinerSerivce_QueryEvent_Handler,
        },
        {
            MethodName: "CreateEvent",
            Handler:    _DivinerSerivce_CreateEvent_Handler,
        },
        {
            MethodName: "QueryMarket",
            Handler:    _DivinerSerivce_QueryMarket_Handler,
        },
        {
            MethodName: "CreateMarket",
            Handler:    _DivinerSerivce_CreateMarket_Handler,
        },
        {
            MethodName: "Tx",
            Handler:    _DivinerSerivce_Tx_Handler,
        },
    },
    Streams:  []grpc.StreamDesc{},
    Metadata: "diviner/protos/service/service.proto",
}

func init() { proto.RegisterFile("diviner/protos/service/service.proto", fileDescriptor0) }

var fileDescriptor0 = []byte{
    // ...
}
```

主要會定義 server 與 client 的 interface。

eg:

```go { .line-numbers }
type divinerSerivceClient struct {
    cc *grpc.ClientConn
}

type DivinerSerivceServer interface {
    QueryMember(context.Context, *QueryRequest) (*MemberInfoResponse, error)
    CreateMember(context.Context, *MemberCreateRequest) (*MemberInfoResponse, error)
    QueryEvent(context.Context, *QueryRequest) (*EventInfoResponse, error)
    CreateEvent(context.Context, *EventCreateRequest) (*EventInfoResponse, error)
    QueryMarket(context.Context, *QueryRequest) (*MarketInfoResponse, error)
    CreateMarket(context.Context, *MarketCreateRequest) (*MarketInfoResponse, error)
    Tx(context.Context, *TxRequest) (*TxResponse, error)
}
```

### gRPC client

```go { .line-numbers }
import pbs "diviner/protos/service"

conn, err = grpc.Dial(viper.GetString("host"), grpc.WithInsecure())

if err != nil {
    panic(fmt.Sprintf("dial grpc server error: %v\n", err)
}


client = pbs.NewDivinerSerivceClient(conn)

req, err := pbs.NewMemberCreateRequest(priv)
resp, err := client.CreateMember(context.Background(), req)
```

### gRPC Service

```go { .line-numbers }
type divinerService struct {
    // ...
}

func (s *divinerService) QueryEvent(ctx context.Context, req *pbs.QueryRequest) (*pbs.EventInfoResponse, error) {
    // ...
}

func main() {
    // ...

    lis, err := net.Listen(conf.GetString("protocol"), conf.GetString("listen"))

    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }

    s := grpc.NewServer()

    service := &divinerService{
        // ...
    }

    pbs.RegisterDivinerSerivceServer(s, service)

    reflection.Register(s)

    log.Println("serving...")
    if err := s.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }

    log.Println("end")
}
```

程序：

1. listen port: `lis, err := net.Listen(conf.GetString("protocol"), conf.GetString("listen"))`
1. New gRPC Server: `s := grpc.NewServer()`
1. New Service: `service := &divinerService{ }`
1. register:

    ```go { .line-numbers }
    pbs.RegisterDivinerSerivceServer(s, service)

    reflection.Register(s)
    ```
1. Serv:

    ```go { .line-numbers }
    if err := s.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
    ```