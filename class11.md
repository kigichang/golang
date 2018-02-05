# Go Class 11 Build and Dependency Management

## Build

資料來源：[How To Build Go Executables for Multiple Platforms on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-build-go-executables-for-multiple-platforms-on-ubuntu-16-04)

執行指令：`env GOOS=target-OS GOARCH=target-architecture go build package-import-path`

平台列表

| GOOS - Target Operating System | GOARCH - Target Platform |
| - | -
| android | arm
| darwin | 386
| darwin | amd64
| darwin | arm
| darwin | arm64
| dragonfly | amd64
| freebsd | 386
| freebsd | amd64
| freebsd | arm
| linux | 386
| linux | amd64
| linux | arm
| linux | arm64
| linux | ppc64
| linux | ppc64le
| linux | mips
| linux | mipsle
| linux | mips64
| linux | mips64le
| netbsd | 386
| netbsd | amd64
| netbsd | arm
| openbsd | 386
| openbsd | amd64
| openbsd | arm
| plan9 | 386
| plan9 | amd64
| solaris | amd64
| windows | 386
| windows | amd64

**Warning: Cross-compiling executables for Android requires the Android NDK, and some additional setup which is beyond the scope of this tutorial.**

作者的文章中，有提到 ubuntu 環境的 script，以此，修改 mac 的版本：

```bash
#!/usr/bin/env bash

package=$1
if [[ -z "$package" ]]; then
  echo "usage: $0 <package-name>"
  exit 1
fi
package_split=(${package//\// })
package_split_len=${#package_split[@]}
package_split_last=`expr $package_split_len - 1`
package_name=${package_split[$package_split_last]}
platforms=("windows/amd64" "windows/386" "darwin/amd64")

for platform in "${platforms[@]}"
do
    platform_split=(${platform//\// })
    GOOS=${platform_split[0]}
    GOARCH=${platform_split[1]}
    output_name='../build/'$package_name'-'$GOOS'-'$GOARCH
    if [ $GOOS = "windows" ]; then
        output_name+='.exe'
    fi

    echo 'build '$output_name
    env GOOS=$GOOS GOARCH=$GOARCH go build -o $output_name $package
    if [ $? -ne 0 ]; then
        echo 'An error has occurred! Aborting the script execution...'
        exit 1
    fi
done
```

## Dependency Management

Go 原本是沒有 dependency management 工具，因此社群很多第三方的工具，後來官方才出自己的版本。[go dep](https://github.com/golang/dep)

原本 go 要用第三方套件時，都會使用 `go get -u 套件_URL` 的方式，將 source code 下載到 $GOPATH/src 下。如果不同專案，使用到相同的套件，但不同版本時，就會很麻煩。

大多數的管理工具，包含官方工具，會在專案的目錄下，產生 `vendor` 的目錄，將第三方套件的 source code 下載到這個目錄下，專案有使用到時，則來這個目錄找。

使用方式：

1. 一開始在專案的目錄下，執行 `dep init`，會產生 `Gopkg.toml`, `Gopkg.lock` 及 `vendor`
1. 產生一個 go 的程式檔案
1. 需要用到某個套件前(還沒開始寫 import), 執行 `dep ensure -add 套件_URI[@版本]`[^dep_version]。會發現在 `Gopkg.toml` 加入設定，以及下載套件到 `vendor` 目錄下。
1. 繼續寫程式

[^dep_version]: 如果沒有要指定版本，則會抓 master，如要指定版本，請在 URL 後, 加版本號碼, eg: `@v1.0.0`)

常用指令：

- `dep init`: 第一次使用管理工具時，請先執行，會自動產生 `Gopkg.toml` 檔案，裏面會去掃描有用到的第三方工具，並下載。
- `dep ensure`: 之後有異動 `Gopkg.toml`，再執行，會去更新 `vendor` 目錄

### Gopkg.toml 格式

```toml
# Gopkg.toml example
#
# Refer to https://github.com/golang/dep/blob/master/docs/Gopkg.toml.md
# for detailed Gopkg.toml documentation.
#
# required = ["github.com/user/thing/cmd/thing"]
# ignored = ["github.com/user/project/pkgX", "bitbucket.org/user/project/pkgA/pkgY"]
#
# [[constraint]]
#   name = "github.com/user/project"
#   version = "1.0.0"
#
# [[constraint]]
#   name = "github.com/user/project2"
#   branch = "dev"
#   source = "github.com/myfork/project2"
#
# [[override]]
#  name = "github.com/x/y"
#  version = "2.4.0"
```

最常用的是 **``[[constraint]]``**

eg:

```toml
[[constraint]]
  name = "google.golang.org/grpc"
  version = "1.7.1"

[[constraint]]
  name = "golang.org/x/net"
  branch = "master"
```

- version: 指定要用那個一版本。
- branch: 指定要用那一個分支，通常不指定 version 時，會直接用 master 分支。直接用 master 分支。