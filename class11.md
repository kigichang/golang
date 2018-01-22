# Go Class 11 Build and Dependency Management

資料來源：[How To Build Go Executables for Multiple Platforms on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-build-go-executables-for-multiple-platforms-on-ubuntu-16-04)

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

作者的文章中，有提到 ubuntu 環境的 script，以此，修改 mac 的版本：

```bash {.line-numbers}
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