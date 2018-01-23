# Go Class 13 MySQL and Web

## MySQL

與 Java JDBC 類似，Go 有定義一套 interface，所有要連 DB 的 driver，都需要實作這個 interface。以下我是用 [go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)

**Select** sample code:

```go {.line-numbers}
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
)

// Schedule ...
type Schedule struct {
    ID      int64
    URL     string
    Referer string
    Count   int64
    Start   string
    Current int64
    Success int64
    Failed  int64
    Status  int32
}


// Connect ...
func Connect(driver, uri string) (*sql.DB, error) {
    db, err := sql.Open(driver, uri)
    if err != nil {
        return nil, err
    }

    if err := db.Ping(); err != nil {
        return nil, err
    }
    return db, nil
}

// RowScanSchedule ...
func RowScanSchedule(rows *sql.Rows) (*Schedule, error) {
    tmp := &Schedule{}
    if err := rows.Scan(&tmp.ID, &tmp.URL, &tmp.Referer, &tmp.Count, &tmp.Start, &tmp.Current, &tmp.Success, &tmp.Failed, &tmp.Status); err != nil {
        return nil, err
    }
    return tmp, nil
}

db, err := Connect("mysql", "crawler:1234@/crawler")

if err != nil {
    // ...
    return
}

defer db.Close()

sel, err := db.Prepare("select id, url, referer, count, start, current, success, failed, status from schedule where status = ? order by start desc")

if err != nil {
    // ...
    return
}
defer sel.Close()

rows, err := sel.Query(1)
if err != nil {
    // ...
    return
}
defer rows.Close()

for rows.Next() {
    tmp, err := RowScanSchedule(rows)
    if err != nil {
        // ...
        return
    }
    // ...
}
```

說明：

1. import package.

    ```go
    import (
        "database/sql"
        _ "github.com/go-sql-driver/mysql"
    )
    ```

    - `"database/sql"` 是 go 定義 sql interface 的 package
    - `_ "github.com/go-sql-driver/mysql"` mysql driver package

1. 定義資料的 struct，類似要做 ORM 的動作，當然也可以不要這個定義，先都用變數來存資料。

    ```go
    // Schedule ...
    type Schedule struct {
        // ...
    }
    ```

1. 連線資料庫

    ```go
    func Connect(driver, uri string) (*sql.DB, error) {
        db, err := sql.Open(driver, uri)
        if err != nil {
            return nil, err
        }

        if err := db.Ping(); err != nil {
            return nil, err
        }
        return db, nil
    }

    db, err := Connect("mysql", "crawler:1234@/crawler")

    if err != nil {
        // ...
        return
    }

    defer db.Close()
    ```

    與 JDBC 連線類似，會傳入一組類似 url 的設定, 格式是：`[username[:password]@][protocol[(address)]]/dbname[?param1=value1&...&paramN=valueN]`。詳細的說明，請見：[DSN (Data Source Name)](https://github.com/go-sql-driver/mysql#dsn-data-source-name)。我在連線後，多做了 Ping 的動作，如下：

    ```go
    if err := db.Ping(); err != nil {
        return nil, err
    }
    ```

    記得取的 db 連線後，立即下 `defer db.Close()`，確保程式在結束後，會關閉連線。

