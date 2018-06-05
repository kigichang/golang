# 12 MySQL and Web

## MySQL

與 Java JDBC 類似，Go 有定義一套 interface，所有要連 DB 的 driver，都需要實作這些 interface (["database/sql/driver"](https://golang.org/pkg/database/sql/driver/))。以下我是用 [go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)

**Test table schema** (MySQL):

```sql
CREATE TABLE `member` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) NOT NULL DEFAULT '',
  `info` text,
  `birthday` date DEFAULT NULL,
  `register` datetime NOT NULL,
  `login` datetime NOT NULL,
  `vip` char(1) NOT NULL DEFAULT 'A',
  `created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `id` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

**sample code**:

```go { .line-numbers }
package main

import (
    "database/sql"
    "encoding/json"
    "fmt"
    "log"
    "time"

    _ "github.com/go-sql-driver/mysql"
)

// Member ...
type Member struct {
    ID       int
    Name     string
    Info     string
    Birthday time.Time // MySQL min Date: 1000-01-01
    Register time.Time // MySQL min DateTime: 1000-01-01 00:00:00
    Login    time.Time // MySQL min DateTime: 1000-01-01 00:00:00
    VIP      string
    Created  time.Time
    Updated  time.Time
}

// Connect ...
func Connect() (*sql.DB, error) {
    db, err := sql.Open("mysql", "test:1234test@tcp(localhost)/test?charset=utf8mb4,utf8&parseTime=true")
    if err != nil {
        return nil, err
    }
    return db, nil
}

func main() {

    db, err := Connect()
    if err != nil {
        log.Fatal("connect:", err)
    }
    defer db.Close()

    birthday := time.Date(0, time.January, 1, 0, 0, 0, 0, time.UTC) //time.Parse("2006-01-02", "1000-01-01")
    register := time.Date(0, time.January, 1, 0, 0, 0, 0, time.UTC)
    login := time.Now()

    m1 := Member{
        Name:     "小明",
        Info:     "小明",
        Birthday: birthday,
        Register: register,
        Login:    login,
        VIP:      "A",
    }
    log.Println(m1)

    ins, err := db.Prepare("insert into member(name, info, birthday, register, login, vip) values(?, ?, ?, ?, ?, ?)")
    if err != nil {
        log.Println("prepare insert:", err)
        return
    }
    defer ins.Close()

    result, err := ins.Exec(m1.Name, m1.Info, m1.Birthday, m1.Register, m1.Login, m1.VIP)
    if err != nil {
        log.Println("insert:", err)
        return
    }

    id, err := result.LastInsertId()
    if err != nil {
        log.Println("last id:", err)
        return
    }

    sel, err := db.Prepare("select id, name, info, birthday, register, login, vip, created, updated from member where id = ?")
    if err != nil {
        log.Println("prepare select:", err)
        return
    }
    defer sel.Close()

    rows, err := sel.Query(id)
    if err != nil {
        log.Println("query:", err)
        return
    }

    defer rows.Close()

    if rows.Next() {
        m2 := Member{}
        err = rows.Scan(&m2.ID, &m2.Name, &m2.Info, &m2.Birthday, &m2.Register, &m2.Login, &m2.VIP, &m2.Created, &m2.Updated)
        if err != nil {
            log.Println("scan:", err)
            return
        }
        bytes, _ := json.Marshal(m2)
        fmt.Println(string(bytes))
    } else {
        log.Printf("data (%d) not found\n", id)
    }

    fmt.Println("end")
}
```

### 連線

1. import package.

    ```go { .line-numbers }
    import (
        "database/sql"
        _ "github.com/go-sql-driver/mysql"
    )
    ```

    1. `"database/sql"` 是 go 定義 sql interface 的 package
    1. `_ "github.com/go-sql-driver/mysql"` mysql driver package

1. 定義資料的 struct，類似要做 ORM 的動作，當然也可以不要這個定義，都用變數來存資料。

    ```go { .line-numbers }
    // Member ...
    type Member struct {
        ...
    }
    ```

1. 連線資料庫

    ```go { .line-numbers }
    func Connect() (*sql.DB, error) {
        db, err := sql.Open("mysql", "test:1234test@tcp(localhost)/test?charset=utf8mb4,utf8&parseTime=true")
        if err != nil {
            return nil, err
        }
        return db, nil
    }
    ```

    與 JDBC 連線類似，指定 driver 的種類，並傳入一組 url 的設定, 格式是：`[username[:password]@][protocol[(address)]]/dbname[?param1=value1&...&paramN=valueN]`。詳細的說明，請見：[DSN (Data Source Name)](https://github.com/go-sql-driver/mysql#dsn-data-source-name)。我在連線後，可以多做了 Ping 的動作，如下：

    ```go { .line-numbers }
    if err := db.Ping(); err != nil {
        return nil, err
    }
    ```

    記得取的 db 連線後，立即下 `defer db.Close()`，確保程式在結束後，會關閉連線。如下：

    ```go { .line-numbers }
    db, err := Connect()
    if err != nil {
        log.Fatal("connect:", err)
    }
    defer db.Close()
    ```

### Insert

```go { .line-numbers }
birthday := time.Date(0, time.January, 1, 0, 0, 0, 0, time.UTC)
register := time.Date(0, time.January, 1, 0, 0, 0, 0, time.UTC)
login := time.Now()

m1 := Member{
    Name:     "小明",
    Info:     "小明",
    Birthday: birthday,
    Register: register,
    Login:    login,
    VIP:      "A",
}
log.Println(m1)

ins, err := db.Prepare("insert into member(name, info, birthday, register, login, vip) values(?, ?, ?, ?, ?, ?)")
if err != nil {
    log.Println("prepare insert:", err)
    return
}
defer ins.Close()

result, err := ins.Exec(m1.Name, m1.Info, m1.Birthday, m1.Register, m1.Login, m1.VIP)
if err != nil {
    log.Println("insert:", err)
    return
}
defer ins.Close()

id, err := result.LastInsertId()
if err != nil {
    log.Println("last id:", err)
    return
}
```

說明：

1. 利用 DB.Pepare 建立一個 PreparedStatement 連線，記得下 `defer ins.Close()`

    ```go { .line-numbers }
    ins, err := db.Prepare("insert into member(name, info, birthday, register, login, vip) values(?, ?, ?, ?, ?, ?)")
    if err != nil {
        log.Println("prepare insert:", err)
        return
    }
    defer ins.Close()
    ```

1. 使用 Stmt.Exec 執行指 SQL 指令。

    ```go { .line-numbers }
    result, err := ins.Exec(m1.Name, m1.Info, m1.Birthday, m1.Register, m1.Login, m1.VIP)
    if err != nil {
        log.Println("insert:", err)
        return
    }
    defer ins.Close()

    id, err := result.LastInsertId()
    if err != nil {
        log.Println("last id:", err)
        return
    }
    ```

    Result 功能：

    1. LastInsertId(): 取得 **auto_increment** 的 id
    1. RowsAffected(): 取得異動的資料筆數，注意，[文件](https://golang.org/pkg/database/sql/#Result)上說並非所有的 driver 都會實作。

**Update/Delete** 與 Insert 類似。

### Select

1. 下 SQL，與 Java 的 PreparedStatement 類似。

    ```go { .line-numbers }
    sel, err := db.Prepare("select id, name, info, birthday, register, login, vip, created, updated from member where id = ?")
    if err != nil {
        log.Println("prepare select:", err)
        return
    }
    defer sel.Close()
    ```

    與上述 db 類似，取得連線後，記得下 `defer sel.Close()` 確保程式結束後，會關閉 statement 連線。

1. 使用 Stmt.Query 方式，取得 Rows

    ```go { .line-numbers }
    rows, err := sel.Query(id)
    if err != nil {
        log.Println("query:", err)
        return
    }

    defer rows.Close()
    ```

    與上述取得連線一樣，立即下 `defer rows.Close()` 確保程式結束後，會關閉 rows。(說明文件說，會[自動關閉](https://golang.org/pkg/database/sql/#Rows.Close)。這部分就看自己的習慣了。但 DB 與 Stmt 一定要記得關。)

1. 一定要先執行 **Next** 才能取資料。

    ```go { .line-numbers }
    if rows.Next() {
        m2 := Member{}
        err = rows.Scan(&m2.ID, &m2.Name, &m2.Info, &m2.Birthday, &m2.Register, &m2.Login, &m2.VIP, &m2.Created, &m2.Updated)
        if err != nil {
            log.Println("scan:", err)
            return
        }
        bytes, _ := json.Marshal(m2)
        fmt.Println(string(bytes))
    } else {
        log.Printf("data (%d) not found\n", id)
    }
    ```

1. 透過 Rows.Scan 取得資料。

    ```go { .line-numbers }
    err = rows.Scan(&m2.ID, &m2.Name, &m2.Info, &m2.Birthday, &m2.Register, &m2.Login, &m2.VIP, &m2.Created, &m2.Updated)
    if err != nil {
        log.Println("scan:", err)
        return
    }
    ```

### Connect, Prepared Statement, Rows, Cursor 關係

![DB](db.png)

## Web

Go 有內建撰寫 Web Server 的套件，可以自己實作一套 AP server。因為 web 還會用到版型與靜態資料，因此在專案目錄的配置建議如下：

```text
web
├── main.go
├── public
│   └── db.png
└── templates
    ├── layout.html
    ├── nav.html
    └── test.html
```

目錄說明

1. public: 放置靜態資料，實際運作上，AP server 可以不用處理靜態資料。
1. templates: 放置版型

**完整的程式碼**:

```go { .line-numbers }
package main

import (
    "fmt"
    "html/template"
    "log"
    "net/http"
)

// MyData ...
type MyData struct {
    Title string
    Nav   string
    Data  interface{}
}

func generateHTML(w http.ResponseWriter, data interface{}, files ...string) {
    var tmp []string
    for _, f := range files {
        tmp = append(tmp, fmt.Sprintf("templates/%s.html", f))
    }

    tmpl := template.Must(template.ParseFiles(tmp...))
    tmpl.ExecuteTemplate(w, "layout", data)
}

func test(w http.ResponseWriter, r *http.Request) {
    data := &MyData{
        Title: "測試",
        Nav:   "test",
    }

    data.Data = struct {
        TestString   string
        SimpleString string
        TestStruct   struct{ A, B string }
        TestArray    []string
        TestMap      map[string]string
        Num1, Num2   int
        EmptyArray   []string
        ZeroInt      int
    }{
        `O'Reilly: How are <i>you</i>?`,
        "中文測試",
        struct{ A, B string }{"foo", "boo"},
        []string{"Hello", "World", "Test"},
        map[string]string{"A": "B", "abc": "DEF"},
        10,
        101,
        []string{},
        0,
    }

    tmpl := template.Must(template.ParseFiles("templates/layout.html", "templates/nav.html", "templates/test.html"))
    tmpl.ExecuteTemplate(w, "layout", data)
}

func setCookie(w http.ResponseWriter, r *http.Request) {
    c1 := http.Cookie{
        Name:     "first_cookie",
        Value:    "Go Web Programming",
        HttpOnly: true,
    }
    c2 := http.Cookie{
        Name:     "second_cookie",
        Value:    "Manning Publications Co",
        HttpOnly: true,
    }
    http.SetCookie(w, &c1)
    http.SetCookie(w, &c2)
}

func getCookie(w http.ResponseWriter, r *http.Request) {
    h := r.Header["Cookie"]
    fmt.Fprintln(w, h)
}

func main() {
    mux := http.NewServeMux()
    files := http.FileServer(http.Dir("./public"))

    mux.Handle("/static/", http.StripPrefix("/static/", files))
    mux.HandleFunc("/", test)
    mux.HandleFunc("/set_cookie", setCookie)
    mux.HandleFunc("/get_cookie", getCookie)

    server := &http.Server{
        Addr:    "0.0.0.0:8080",
        Handler: mux,
    }

    err := server.ListenAndServe()
    if err != nil {
        log.Fatalln(err)
    }
}
```

程式說明:

1. import 必要的 package

    ```go { .line-numbers }
    import (
        // ...
        "html/template"
        "net/http"
    )
    ```

    1. `"html/template"`: Go 的 template engine。可以直接修改版型，不用重啟系統。
    1. `"net/http"`: Http 模組

1. 實作 routing 機制：

    ```go { .line-numbers }
    mux := http.NewServeMux()
    files := http.FileServer(http.Dir("./public"))

    mux.Handle("/static/", http.StripPrefix("/static/", files))
    mux.HandleFunc("/", test)
    mux.HandleFunc("/set_cookie", setCookie)
    mux.HandleFunc("/get_cookie", getCookie)
    ```

    1. http.NewServMux(): 產生 `ServMux` 物件，用來處理 url routing 的工作。
    1. 處理靜態資料:

        ```go { .line-numbers }
        files := http.FileServer(http.Dir("./public"))
        mux.Handle("/static/", http.StripPrefix("/static/", files))
        ```

        1. 指定檔案放的目錄：`http.FileServer(http.Dir("./public"))`
        1. 設定 靜態資料的 url，指到剛剛設定的 `FileServer`。

1. 其他 URL 的 routing: 利用 `HandleFunc` 來設定 URL 與處理 function 的關係。以下的 sample，`/` 會執行 `test`, `/set_cookie` 會執行 `setCookie`

    ```go { .line-numbers }
    mux.HandleFunc("/", test)
    mux.HandleFunc("/set_cookie", setCookie)
    mux.HandleFunc("/get_cookie", getCookie)
    ```

1. 綁定 port 並啟動 web server

    ```go { .line-numbers }
    server := &http.Server{
        Addr:    "0.0.0.0:8080",
        Handler: mux,
    }

    err := server.ListenAndServe()
    if err != nil {
        log.Fatalln(err)
    }
    ```

### Handler 與 Request Parameter

Handler function 的定義：

```go { .line-numbers }
func name(w http.ResponseWriter, r *http.Request) {
    body
}
```

eg:

```go { .line-numbers }
func test(w http.ResponseWriter, r *http.Request) {
    ...
}
```

1. 可透過 `r.Method` 來判斷是 GET or POST 等
1. 透過 `r.PostFormValue` 來取得 POST 值。Go 有內建多種取 request 值的方式，整理如下：

| Field | Should call method first | parameters in URL | Form | URL encoded | Multipart (upload file)
| - | - | - | - | - | -
| Form | ParseForm | ✓ | ✓ | ✓ | -
| PostForm | Form | - | ✓ | ✓ | -
| MultipartForm | ParseMultipartForm | - | ✓ | - | ✓ |
| FormValue | NA | ✓ | ✓ | ✓ | -
| PostFormValue | NA | - | ✓ | ✓ | -

from: [Go Web Programming](https://www.manning.com/books/go-web-programming)

### Response Header

預設 response 的 status code 是 **200(OK)**，如果要修改 header 值或 status code 時，要注意 `w.WriteHeader(code)` 要最後呼叫，因為呼叫完 `WriteHeader` 後，任何 header 的更動，都不會被接受。也就是改了也沒用。

eg:

```go {.line-numbers}
w.Header().Set("Location", "/")
w.WriteHeader(302)
```

### Cookie

Cookie struct

```go {.line-numbers}
type Cookie struct {
    Name       string
    Value      string
    Path       string
    Domain     string
    Expires    time.Time
    RawExpires string
    MaxAge     int
    Secure     bool
    HttpOnly   bool
    Raw        string
    Unparsed   []string
}
```

eg:

```go
func setCookie(w http.ResponseWriter, r *http.Request) {
    c1 := http.Cookie{
        Name:     "first_cookie",
        Value:    "Go Web Programming",
        HttpOnly: true,
    }
    c2 := http.Cookie{
        Name:     "second_cookie",
        Value:    "Manning Publications Co",
        HttpOnly: true,
    }
    //w.Header().Set("Set-Cookie", c1.String())
    //w.Header().Add("Set-Cookie", c2.String())

    http.SetCookie(w, &c1)
    http.SetCookie(w, &c2)
}

func getCookie(w http.ResponseWriter, r *http.Request) {
    h := r.Header["Cookie"]
    fmt.Fprintln(w, h)
}
```

### Templates

Go template engine 很好用，會自動依版型的內容，來自動做 escape 動作。使用 template engine 需要再學習它的語法。

Go html template 是利用 text template，因此相關的語法，要看 ["text/template"](https://golang.org/pkg/text/template/#pkg-index)

範例中，整理了我覺得常用的案例。

**程式碼**:

```go { .line-numbers }
func test(w http.ResponseWriter, r *http.Request) {
    data := &MyData{
        Title: "測試",
        Nav:   "test",
    }

    data.Data = struct {
        TestString   string
        SimpleString string
        TestStruct   struct{ A, B string }
        TestArray    []string
        TestMap      map[string]string
        Num1, Num2   int
        EmptyArray   []string
        ZeroInt      int
    }{
        `O'Reilly: How are <i>you</i>?`,
        "中文測試",
        struct{ A, B string }{"foo", "boo"},
        []string{"Hello", "World", "Test"},
        map[string]string{"A": "B", "abc": "DEF"},
        10,
        101,
        []string{},
        0,
    }

    tmpl := template.Must(template.ParseFiles("templates/layout.html", "templates/nav.html", "templates/test.html"))
    tmpl.ExecuteTemplate(w, "layout", data)
}
```

說明：

1. `template.ParseFiles(tmp...)`: 選擇會用到的版型檔案，要確認版型的路徑與檔案是否正確。
1. `template.Must(template.ParseFiles(tmp...))`: 使用 `template.Must` 產生版型物件。
1. `tmpl.ExecuteTemplate(w, "layout", data)`: 執行版型，並將版型會用的資料(`data`)帶入。其中 `"layout"` 是定義在版型內。

### 版型

範例的版型結構：

1. `layout.html`: 版型的主框。內含 `nav` 與  `content` 這個子版型。
    1. `{{ template "navbar" . }}`
    1. `{{ template "content" . }}`

    在 `layout.html` 定義了這個版型的名稱 **layout** `{{ define "layout" }}`，也就是程式碼 `tmpl.ExecuteTemplate(w, "layout", data)` 中的 `"layout"`。

    在 include 子版型的語法中，eg: `{{ template "navbar" . }}`，有 **`.`**，指的是傳進來的資料，在 ["text/template"](https://golang.org/pkg/text/template/#pkg-index) 有詳細的說明。

    ```html { .line-numbers}
    {{ define "layout" }}
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=9">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>OOXX - {{ .Title }}</title>
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta.3/css/bootstrap.min.css" integrity="sha384-Zug+QiDoJOrZ5t4lssLdxGhVrurbmBWopoEl+M6BdEfwnCJZtKxi1KgxUyJq13dy" crossorigin="anonymous">
    </head>
    <body>
        {{ template "navbar" . }}
        <div class="container">
        {{ template "content" . }}
        </div> <!-- /container -->
        <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta.3/js/bootstrap.min.js" integrity="sha384-a5N7Y/aK3qNeh15eJKGWxsqtnX/wWdSZSKp+81YjTmS15nvnvxKHuzaWwXHDli+4" crossorigin="anonymous"></script>
    </body>
    </html>
    {{ end }}
    ```

1. `nav.html`: Navigation bar。跟 `layout.html` 一樣，一開頭定義這個版型的名稱 `{{ define "navbar" }}`，也就是 `layout.html` 中 `{{ template "navbar" . }}` 的 `"navbar"`。

    ```html { .line-numbers }
    {{ define "navbar" }}
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
        <a class="navbar-brand" href="#">OOXX</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNavDropdown" aria-controls="navbarNavDropdown" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNavDropdown">
        <ul class="navbar-nav">
            <li class="nav-item {{ if eq .Nav "home" }}active{{ end }}">
            <a class="nav-link" href="/">首頁</a>
            </li>
            <li class="nav-item {{ if eq .Nav "add" }}active{{ end }}">
            <a class="nav-link" href="/add">新增</a>
            </li>
            <li class="nav-item {{ if eq .Nav "all" }}active{{ end }}">
                <a class="nav-link" href="/all">看全部</a>
            </li>
        </ul>
        </div>
    </nav>
    {{ end }}
    ```

1. `test.html`: 內容的子版型，開頭 `{{ define "content" }}` 與上述相同。

    ```html
    {{ define "content" }}

    <script languate="javascript">
        var pair = {{ .Data.TestStruct }};
        var array = {{ .Data.TestArray }};
        var str1 = "{{ .Data.TestString }}";
        var str2 = "{{ .Data.SimpleString }}";
    </script>

    <p> escape string <br />
    {{ .Data.TestString }} <br />
    <a title='{{ .Data.TestString }}'>{{ .Data.TestString }}</a> <br />
    <a href="/{{ .Data.TestString }}">{{ .Data.TestString }}</a> <br />
    <a href="?q={{ .Data.TestString }}">{{ .Data.TestString }}</a> <br />
    <a onx='f("{{ .Data.TestString }}")'>{{ .Data.TestString }}</a> <br />
    <a onx='f({{ .Data.TestString }})'>{{ .Data.TestString }}</a> <br />
    <a onx='pattern = /{{ .Data.TestString }}/;'>{{.Data.TestString }}</a> <br />
    </p>

    <p> non escape string <br />
    {{ .Data.SimpleString }} <br />
    <a title='{{ .Data.SimpleString }}'>{{ .Data.SimpleString }}</a> <br />
    <a href="/{{ .Data.SimpleString }}">{{ .Data.SimpleString }}</a> <br />
    <a href="?q={{ .Data.SimpleString }}">{{ .Data.SimpleString }}</a> <br />
    <a onx='f("{{ .Data.SimpleString }}")'>{{ .Data.SimpleString }}</a> <br />
    <a onx='f({{ .Data.SimpleString }})'>{{ .Data.SimpleString }}</a> <br />
    <a onx='pattern = /{{ .Data.SimpleString }}/;'>{{ .Data.SimpleString }}</a> <br />
    </p>

    <p>array index <br />
        {{ index .Data.TestArray 0 }}<br />
        {{ index .Data.TestArray 1 }}<br />
        {{ index .Data.TestArray 2 }}<br />
        len: {{ len .Data.TestArray }}<br />

    </p>

    <p>Map<br />
    abc : {{ index .Data.TestMap "abc"}} <br />
    A : {{ index .Data.TestMap "A"}} <br />
    B : {{ index .Data.TestMap "B"}} <br />
    </p>

    <p>Compare<br />
    {{ with .Data }}
    {{ if eq .Num1 .Num2}} eq {{ else }} ne {{end}}<br/>
    {{ if ne .Num1 .Num2}} ne {{ else }} eq {{end}}<br/>
    {{ if lt .Num1 .Num2}} lt {{ else if gt .Num1 .Num2 }} gt {{ else if le .Num1 .Num2 }} le {{ else }} ge {{end}}
    {{ end }}
    </p>

    <p>Range Array 1<br />
        {{ range .Data.TestArray}}
        {{.}} <br />
        {{else}}
        no data
        {{end}}
        <br />
    </p>

    <p>Range Array 2<br />
        {{ range $idx, $elm := .Data.TestArray }}
        {{ $idx }} : {{ $elm }} <br />
        {{else}}
        no data
        {{end}}
        <br />
    </p>

    <p>Range Map<br />
        {{ range $key, $elm := .Data.TestMap }}
        {{ $key }} : {{ $elm }} <br />
        {{else}}
        no data
        {{end}}
        <br />
    </p>

    <p>Range empty<br />
        {{ range .Data.EmptyArray}}
        {{ . }} <br />
        {{ else }}
        no data
        {{ end }}
        <br />
    </p>

    <p>with empty<br />
    {{with .Data.EmptyArray}}
        have data
        {{else}}
        nodata
        {{end}}
    </p>

    <p>with int zero value<br />
        {{with .Data.ZeroInt}}
        have data
        {{else}}
        nodata
        {{end}}
    </p>

    {{ end }}
    ```

### Template 重點語法說明

Go template engine 會依照版型的內容，自動做 escape。

1. 在 `<script></script>` 的效果

    語法：

    ```html
    <script languate="javascript">
        var pair = {{ .Data.TestStruct }};
        var array = {{ .Data.TestArray }};
        var str1 = "{{ .Data.TestString }}";
        var str2 = "{{ .Data.SimpleString }}";
    </script>
    ```

    結果：

    ```html
    <script languate="javascript">
        var pair = {"A":"foo","B":"boo"};
        var array = ["Hello","World","Test"];
        var str1 = "O\x27Reilly: How are \x3ci\x3eyou\x3c\/i\x3e?";
        var str2 = "中文測試";
    </script>
    ```

    如果 string 內容有需要做 escape 時，go template engine 會自動做，eg: `"O\x27Reilly: How are \x3ci\x3eyou\x3c\/i\x3e?"`, 比較特別的是如果資料是 struct 或 array，會自動轉成 javascript 的 data type 型態。eg: `var pair = {"A":"foo","B":"boo"};` 及 `var array = ["Hello","World","Test"];`。

1. string 自動 escape 效果

    語法：

    ```html
    <p> escape string <br />
    {{ .Data.TestString }} <br />
    <a title='{{ .Data.TestString }}'>{{ .Data.TestString }}</a> <br />
    <a href="/{{ .Data.TestString }}">{{ .Data.TestString }}</a> <br />
    <a href="?q={{ .Data.TestString }}">{{ .Data.TestString }}</a> <br />
    <a onx='f("{{ .Data.TestString }}")'>{{ .Data.TestString }}</a> <br />
    <a onx='f({{ .Data.TestString }})'>{{ .Data.TestString }}</a> <br />
    <a onx='pattern = /{{ .Data.TestString }}/;'>{{.Data.TestString }}</a> <br />
    </p>
    ```

    結果：

    ```html
    <p> escape string <br />
    O&#39;Reilly: How are &lt;i&gt;you&lt;/i&gt;? <br />
    <a title='O&#39;Reilly: How are &lt;i&gt;you&lt;/i&gt;?'>O&#39;Reilly: How are &lt;i&gt;you&lt;/i&gt;?</a> <br />
    <a href="/O%27Reilly:%20How%20are%20%3ci%3eyou%3c/i%3e?">O&#39;Reilly: How are &lt;i&gt;you&lt;/i&gt;?</a> <br />
    <a href="?q=O%27Reilly%3a%20How%20are%20%3ci%3eyou%3c%2fi%3e%3f">O&#39;Reilly: How are &lt;i&gt;you&lt;/i&gt;?</a> <br />
    <a onx='f("O\x27Reilly: How are \x3ci\x3eyou\x3c\/i\x3e?")'>O&#39;Reilly: How are &lt;i&gt;you&lt;/i&gt;?</a> <br />
    <a onx='f(&#34;O&#39;Reilly: How are \u003ci\u003eyou\u003c/i\u003e?&#34;)'>O&#39;Reilly: How are &lt;i&gt;you&lt;/i&gt;?</a> <br />
    <a onx='pattern = /O\x27Reilly: How are \x3ci\x3eyou\x3c\/i\x3e\?/;'>O&#39;Reilly: How are &lt;i&gt;you&lt;/i&gt;?</a> <br />
    </p>
    ```

1. string 沒有需要做 escape 時

    語法：

    ```html
    <p> non escape string <br />
    {{ .Data.SimpleString }} <br />
    <a title='{{ .Data.SimpleString }}'>{{ .Data.SimpleString }}</a> <br />
    <a href="/{{ .Data.SimpleString }}">{{ .Data.SimpleString }}</a> <br />
    <a href="?q={{ .Data.SimpleString }}">{{ .Data.SimpleString }}</a> <br />
    <a onx='f("{{ .Data.SimpleString }}")'>{{ .Data.SimpleString }}</a> <br />
    <a onx='f({{ .Data.SimpleString }})'>{{ .Data.SimpleString }}</a> <br />
    <a onx='pattern = /{{ .Data.SimpleString }}/;'>{{ .Data.SimpleString }}</a> <br />
    </p>
    ```

    效果：

    ```html
    <p> non escape string <br />
    中文測試 <br />
    <a title='中文測試'>中文測試</a> <br />
    <a href="/%e4%b8%ad%e6%96%87%e6%b8%ac%e8%a9%a6">中文測試</a> <br />
    <a href="?q=%e4%b8%ad%e6%96%87%e6%b8%ac%e8%a9%a6">中文測試</a> <br />
    <a onx='f("中文測試")'>中文測試</a> <br />
    <a onx='f(&#34;中文測試&#34;)'>中文測試</a> <br />
    <a onx='pattern = /中文測試/;'>中文測試</a> <br />
    </p>
    ```

1. Compare and if-else

    語法：

    ```html
    <p>Compare<br />
    {{ with .Data }}
    {{ if eq .Num1 .Num2}} eq {{ else }} ne {{end}}<br/>
    {{ if ne .Num1 .Num2}} ne {{ else }} eq {{end}}<br/>
    {{ if lt .Num1 .Num2}} lt {{ else if gt .Num1 .Num2 }} gt {{ else if le .Num1 .Num2 }} le {{ else }} ge {{end}}
    {{ end }}
    </p>
    ```

    結果：

    ```html
    <p>Compare<br />

    ne <br/>
    ne <br/>
    lt 

    </p>
    ```

1. 讀取 array 值

    語法：

    ```html
    <p>array index <br />
        {{ index .Data.TestArray 0 }}<br />
        {{ index .Data.TestArray 1 }}<br />
        {{ index .Data.TestArray 2 }}<br />
        len: {{ len .Data.TestArray }}<br />
    </p>
    ```

    結果：

    ```html
    <p>array index <br />
        Hello<br />
        World<br />
        Test<br />
        len: 3<br />
    </p>
    ```
1. array travel (range-else)

    語法：

    ```html
    <p>Range Array 1<br />
        {{ range .Data.TestArray}}
        {{.}} <br />
        {{else}}
        no data
        {{end}}
    </p>

    <p>Range Array 2<br />
        {{ range $idx, $elm := .Data.TestArray }}
        {{ $idx }} : {{ $elm }} <br />
        {{else}}
        no data
        {{end}}

    </p>

    <p>Range empty<br />
        {{ range .Data.EmptyArray}}
        {{ . }} <br />
        {{ else }}
        no data
        {{ end }}
    </p>
    ```

    結果：

    ```html
    <p>Range Array 1<br />

        Hello <br />

        World <br />

        Test <br />

    </p>

    <p>Range Array 2<br />

        0 : Hello <br />

        1 : World <br />

        2 : Test <br />

    </p>

    <p>Range empty<br />

        no data

    </p>
    ```
1. 讀取 map

    語法：

    ```html
    <p>Map<br />
    abc : {{ index .Data.TestMap "abc"}} <br />
    A : {{ index .Data.TestMap "A"}} <br />
    B : {{ index .Data.TestMap "B"}} <br />
    </p>
    ```

    結果：

    ```html
    <p>Map<br />
    abc : DEF <br />
    A : B <br />
    B :  <br />
    </p>
    ```

1. Map travel

    語法：

    ```html
    <p>Range Map<br />
        {{ range $key, $elm := .Data.TestMap }}
        {{ $key }} : {{ $elm }} <br />
        {{else}}
        no data
        {{end}}
    </p>
    ```

    結果：

    ```html
    <p>Range Map<br />

        A : B <br />

        abc : DEF <br />

    </p>
    ```

1. 確認值是否**不是 zero value**，要特別小心當值是 **zero value**，像 `int` 型別，值又是 **"0"** 時，會判定成沒有值，會進到 `else` 的區塊。

    語法：

    ```html
    <p>with empty<br />
    {{with .Data.EmptyArray}}
        have data
        {{else}}
        nodata
        {{end}}
    </p>

    <p>with int zero value<br />
        {{with .Data.ZeroInt}}
        have data
        {{else}}
        nodata
        {{end}}
    </p>
    ```

    結果：

    ```html
    <p>with empty<br />

        nodata

    </p>

    <p>with int zero value<br />

        nodata

    </p>
    ```

## Build Web with Gorilla Toolkit

- [mux](https://github.com/gorilla/mux): Mux Router，可以定義更彈性的 routing path.
- [securecookie](https://github.com/gorilla/securecookie): 加密 cookie
- [schema](https://github.com/gorilla/schema): 將 post form 的資料，轉成 struct
- [csrf](https://github.com/gorilla/csrf): 避免被 CSRF 功擊[^csrf]

[^csrf]: [讓我們來談談 CSRF](https://blog.techbridge.cc/2017/02/25/csrf-introduction/)

將綜合以上與 Gorilla Tool Kit，撰寫註冊與登入功能。

### Table Schema

**account**:

```sql
CREATE TABLE `member` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) NOT NULL,
  `created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

```sql
CREATE TABLE `account` (
  `id` int(11) NOT NULL,
  `email` varchar(100) NOT NULL,
  `password` char(100) NOT NULL,
  `created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `email` (`email`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

### mux

```go {.line-numbers}
r := mux.NewRouter()

r.HandleFunc("/", handler.Index)
r.HandleFunc("/login", handler.ShowLogin).Methods("GET")
r.HandleFunc("/login", handler.DoLogin).Methods("POST")
r.HandleFunc("/logout", handler.Logout)
r.HandleFunc("/register", handler.ShowRegister).Methods("GET")
r.HandleFunc("/register", handler.DoRegister).Methods("POST")

s := r.PathPrefix("/member").Subrouter()
s.HandleFunc("", handler.Member)
s.HandleFunc("/edit", handler.MemberShowEdit).Methods("GET")
s.HandleFunc("/edit", handler.MemberDoEdit).Methods("POST")

s.Use(handler.MemberAuthHandler)
```

- `Methods(xxx)`: 限定某種 Http Method
- `r.PathPrefix(xxx).Subrouter()`: 產生子 router, 方便管理
- `s.Use(xxx)`: 此 router 下的所有 request 都需要先經過某個 handler 處理，類似 fiter 功能。

### csrf

1. 設定:

    ```go {.line-numbers}
    CSRF := csrf.Protect(
        []byte(`1234567890abcdefghijklmnopqrstuvwsyz!@#$%^&*()_+~<>?:{}|,./;'[]\`),
        csrf.RequestHeader("X-ATUH-Token"),
        csrf.FieldName("auth_token"),
        csrf.Secure(false),
    )

    log.Fatal(http.ListenAndServe(":8000", CSRF(r)))
    ```

1. 用 `csrf.TemplateField(r)` 產生 token 並傳給版型

    ```go {.line-numbers}
    func ShowLogin(w http.ResponseWriter, r *http.Request) {
        tmpl.HTML(w, csrf.TemplateField(r), "layout", "login")
    }
    ```

1. 將 token 放在 html form 內

    ```html
    {{ define "content" }}
    <form method="post" action="/login">
    {{ . }}
        <div class="form-group row">
        <label for="email" class="col-sm-2 col-form-label">Email:</label>
        <div class="col-sm-10">
            <input type="email" class="form-control" id="email" name="email" required>
        </div>
        </div>
        <div class="form-group row">
            <label for="password" class="col-sm-2 col-form-label">Password</label>
            <div class="col-sm-10">
            <input type="password" class="form-control" id="password" name="password" required>
            </div>
        </div>
        <div class="form-group row">
        <div class="col-sm-10">
            <button type="submit" class="btn btn-primary">Submit</button>
        </div>
        </div>
    </form>
    {{ end }}
    ```

1. 用 Gorilla schema 處理時，記得要加一個 token 欄位，可以不處理

    ```go {.line-number}
    form := struct {
        Email    string `schema:"email"`
        Password string `schema:"password"`
        Token    string `schema:"auth_token"`
    }{}

    r.ParseForm()

    err := schema.NewDecoder().Decode(&form, r.PostForm)
    ```

### schema

1. form 版型

    ```html
    {{ define "content" }}
    <form method="post" action="/register">
    {{ . }}
        <div class="form-group row">
        <label for="name" class="col-sm-2 col-form-label">Name:</label>
        <div class="col-sm-10">
            <input type="text" class="form-control" id="name" name="name" required>
        </div>
        </div>
        <div class="form-group row">
        <label for="email" class="col-sm-2 col-form-label">Email:</label>
        <div class="col-sm-10">
            <input type="email" class="form-control" id="email" name="email" required>
        </div>
        </div>
        <div class="form-group row">
            <label for="password" class="col-sm-2 col-form-label">Password</label>
            <div class="col-sm-10">
            <input type="password" class="form-control" id="password" name="password" required>
            </div>
        </div>
        <div class="form-group row">
        <div class="col-sm-10">
            <button type="submit" class="btn btn-primary">Submit</button>
        </div>
        </div>
    </form>
    {{ end }}
    ```

1. 宣告相對應的 struct 並 decode

    ```go {.line-numbers}
    form := struct {
        Name     string `schema:"name"`
        Email    string `schema:"email"`
        Password string `schema:"password"`
        Token    string `schema:"auth_token"`
    }{}

    r.ParseForm()

    err := schema.NewDecoder().Decode(&form, r.PostForm)
    ```

### securecookie

1. Initialize

    ```go {.line-number}
    secureC = securecookie.New([]byte(hashKey), []byte(blockKey))
    ```

    - hashKey: 32 or 64 bytes
    - blockKey: 16 (AES-128), 24 (AES-192), 32 (AES-256) bytes

1. Encode and Set Cookie

    ```go {.line-numbers}
    tmp, err := secureC.Encode(key, value)

    if err != nil {
        return
    }

    c := &http.Cookie{
        Name:   key,
        Value:  tmp,
        MaxAge: 0,
        Path:   "/",
    }

    http.SetCookie(w, c)
    ```

    - key: cookie name
    - value: cookie value to encode

1. Read Cookie and Decode

    ```go {.line-numbers}
    c, err := r.Cookie(key)
    if err != nil {
        return
    }

    value := ""
    err = secureC.Decode(key, c.Value, &value)
    if err != nil {
        return
    }
    ```

    - key: cookie name
    - value: original cookie value