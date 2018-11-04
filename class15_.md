# 15 Fake Inheritance in Golang (待修正)
  
  
Golang 本身沒有 OOP[^non_oop]，但可以透過 Struct Embedding/Anonymous Fields 的機制來實作 OOP。也因此 Golang 可以模擬出多重繼承的效果。
  
Embedded struct 不要用 Pointer. ~~為了避免混餚，建議 embedded struct 不要用 pointer。~~
  
```go
package main
  
type animal struct {
}
  
type dog struct {
    animal
}
  
func test(x *animal) {
  
}
  
func main() {
  
    d := new(dog)
  
    test(d)             // cannot use d (type *dog) as type *animal in argument to test
    test(d.(animal))    // invalid type assertion: d.(animal) (non-interface type *dog on left)
  
    var a *animal = d   // cannot use d (type *dog) as type *animal in assignment
}
```
  
[^non_oop]: 因為 Golang 本身沒有繼承的機制，只是透過 Struct Embedding/Anonymous Fields 來達到繼承效果。
  
## 單一繼承與 Override
  
  
example:
  
```go
package main
  
import (
    "encoding/json"
    "log"
)
  
// Role ...
type Role struct {
    HP          float64
    MP          float64
    hiddenSkill string
}
  
func (r *Role) String() string {
    return "simple role"
}
  
// Move ...
func (r *Role) Move() (float64, float64) {
    return 1.0, 1.0
}
  
// Skill ...
func (r *Role) Skill() string {
    return r.hiddenSkill
}
  
// Dump ...
func (r *Role) Dump() string {
    bytes, _ := json.Marshal(r)
    return string(bytes)
}
  
// Magician ...
type Magician struct {
    Role
}
  
// NewMagician ...
func NewMagician() *Magician {
    return &Magician{Role{HP: 100.0, MP: 1000.0, hiddenSkill: "fireball"}}
}
  
// Move ...
func (m *Magician) Move() (float64, float64) {
    x, y := m.Role.Move()
    return 100 + x, 200 + y
}
  
func (m *Magician) String() string {
    return "magician"
}
  
func main() {
    role := &Role{}
    x, y := role.Move()
    log.Printf("%v: %s, skill: %q, move:(%g, %g)\n", role, role.Dump(), role.Skill(), x, y)
  
    magician := NewMagician()
    x, y = magician.Move()
    log.Printf("%v: %s, skill: %q, move:(%g, %g)\n", magician, magician.Dump(), magician.Skill(), x, y)
  
    magician.hiddenSkill += ";iceball"
    log.Printf("%v: %s, skill: %q, move:(%g, %g)\n", magician, magician.Dump(), magician.Skill(), x, y)
}
```
  
Output:
  
```text
2018/03/29 11:17:55 simple role: {"HP":0,"MP":0}, skill: "", move:(1, 1)
2018/03/29 11:17:55 magician: {"HP":100,"MP":1000}, skill: "fireball", move:(101, 201)
2018/03/29 11:17:55 magician: {"HP":100,"MP":1000}, skill: "fireball;iceball", move:(101, 201)
```
  
說明：
  
1. 同 package 下，是可以存取 private 變數，如: `magician.hiddenSkill += ";iceball"`
1. 可以各自 implements interface，如: `func (r *Role) String() string`, `func (m *Magician) String() string`。如果 `Magician` 沒有 implements `Stringer`，則會用 `Role` 的 `String()`
1. 可以透過 `m.Role` 來達到存取父類別的效果，如: `x, y := m.Role.Move()`
1. 重新宣告與 embed struct 的 method，可以達到 override 效果，如: `func (m *Magician) Move() (float64, float64)` override `func (r *Role) Move() (float64, float64)`
  
## 多重繼承與 Ambiguous
  
  
example:
  
```go
package main
  
import (
    "encoding/json"
    "log"
)
  
// Role ...
type Role struct {
    HP          float64
    MP          float64
    hiddenSkill string
}
  
func (r *Role) String() string {
    return "simple role"
}
  
// Move ...
func (r *Role) Move() (float64, float64) {
    return 1.0, 1.0
}
  
// Skill ...
func (r *Role) Skill() string {
    return r.hiddenSkill
}
  
// Dump ...
func (r *Role) Dump() string {
    bytes, _ := json.Marshal(r)
    return string(bytes)
}
  
// Flyer ...
type Flyer struct {
}
  
// Fly ...
func (f *Flyer) Fly() (float64, float64) {
    return 1000.0, 1000.0
}
  
// Skill ...
func (f *Flyer) Skill() string {
    return "fly"
}
  
// Bahamut ...
type Bahamut struct {
    Role
    Flyer
}
  
func (r *Bahamut) String() string {
    return "bahamut"
}
  
// NewBahamut ...
func NewBahamut() *Bahamut {
    return &Bahamut{Role{100000.0, 100000.0, "fireball"}, Flyer{}}
}
  
func main() {
    bahamut := NewBahamut()
    x, y := bahamut.Move()
    log.Printf("%v: %s, skill: %q, move:(%g, %g)\n", bahamut, bahamut.Dump(), bahamut.Skill(), x, y) // error: ambiguous selector bahamut.Skill
}
```
  
說明:
  
1. 因為 `Role`, `Flyer` 都有 `Skill` function，會不知道要取那一個. 因為必預要指定，如: `bahamut.Role.Skill()` or `bahamut.Flyer.Skill()`。或者 override `Skill` function. 如下：
  
    ```go
    // Skill ...
    func (b *Bahamut) Skill() string {
        return strings.Join([]string{b.Role.Skill(), b.Flyer.Skill()}, ";")
    }
    ```
  
