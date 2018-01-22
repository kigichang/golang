# Go Class 12 spf13 Cobra and Viper

- [Viper](https://github.com/spf13/viper)
- [Cobra](https://github.com/spf13/cobra)

這兩個套件，很多 Go 的第三方套件都有用到，也很好用。

## Viper

設定檔套件。

- 支援 JSON, TOML, YAML, HCL, and Java properties 等格式
- 可設定預設值
- 自動載入
- 自動重載

在 import viper 後，會有一個 global 變數 **viper**，可以直接用。在使用前，需先設定要載入的檔名以及設定檔放的路徑，最後再執行讀取。

eg:

```go
viper.SetConfigName("config") // name of config file (without extension)
viper.AddConfigPath(".")      // path to look for the config file in

err := viper.ReadInConfig()

if err != nil {
    fmt.Println("Config not found...")
} else {
    name := viper.GetString("name")
    fmt.Println("Config found, name = ", name)
}
```

### 自定 Viper, 不使用預設的 viper

Viper 也允許自己產生一個全新的 viper，方便管理不同的設定檔。

eg:

```go
package config

import "github.com/spf13/viper"

var config *viper.Viper

func init() {
    config = viper.New()
    config.AddConfigPath(".")
    config.SetConfigName("config")
}

// Load load a default configuration file
func Load() (*viper.Viper, error) {
    if err := config.ReadInConfig(); err != nil {
        return nil, err
    }

    return config, nil
}

// LoadFile create a new Viper to load specific configuration file
func LoadFile(config string) (*viper.Viper, error) {
    v := viper.New()

    v.SetConfigFile(config)
    if err := v.ReadInConfig(); err != nil {
        return nil, err
    }
    return v, nil
}
```

## Cobra

管理 Command line 程式參數的套件，雖然 Go 已有內建 Flag 套件，但很多第三方套件還是用 Cobra。

### Cobra 參數管理方式

以 docker 為例，當執行 `docker` 時：

```text
Usage:  docker COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/Users/kigi/.docker")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/Users/kigi/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/Users/kigi/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/Users/kigi/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  checkpoint  Manage checkpoints
  config      Manage Docker configs
  container   Manage containers
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  deploy      Deploy a new stack or update an existing stack
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command
```

`docker` 是主程式，之後會再接 command，在 cobra 的設計中，`docker` 是 root command，以下的 command 稱做 sub command。

`docker run --help` 為例:

```text
Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

Options:
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cidfile string                 Write the container ID to the file
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
  -c, --cpu-shares int                 CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
  -d, --detach                         Run container in background and print container ID
      --detach-keys string             Override the key sequence for detaching a container
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
  -e, --env list                       Set environment variables
      --env-file list                  Read in a file of environment variables
      --expose list                    Expose a port or a range of ports
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to initialize before starting health-retries countdown (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage
  -h, --hostname string                Container host name
      --init                           Run an init inside the container that forwards signals and reaps processes
  -i, --interactive                    Keep STDIN open even if not attached
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g., 92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    Assign a name to the container
      --network string                 Connect a container to a network (default "default")
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --platform string                Set platform if server is multi-platform capable
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
      --read-only                      Mount the container's root filesystem as read only
      --restart string                 Restart policy to apply when a container exits (default "no")
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process (default true)
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container
```

`--xxx` 與 `-x` 是指傳入的 flag，一個 flag 可以有 longterm (`--xxx` 表示)，與 shortterm (`-x` 表示)，變數可以是 string, numbers, boolean 等資料型別。

以下，是模擬以上的效果。

### Step 1 Sub commands

先產生 root 及 sub commands

eg:

```go
package main

import (
    "github.com/spf13/cobra"
)

func main() {
    rootCmd := &cobra.Command{Use: "myapp"}

    createCmd := &cobra.Command{Use: "create"}

    updateCmd := &cobra.Command{Use: "update"}

    rootCmd.AddCommand(createCmd, updateCmd)

    rootCmd.Execute()
}
```

在專案目錄下，執行 `go run main.go` 結果是：

```text
Usage:

Flags:
  -h, --help   help for myapp

Additional help topics:
  myapp create
  myapp update
```

執行 `go run main.go create` or `go run main.go update` 會立即執行完畢，因為我們還沒定義 sub command 要做什麼事情。

### 定義 flag，參數與工作

接下來定義每個 sub command 需要的 flag, 參數與工作。

```go
package main

import (
    "fmt"

    "github.com/spf13/cobra"
)

var (
    name  string
    proxy bool
)

func main() {
    rootCmd := &cobra.Command{Use: "myapp"}

    createCmd := &cobra.Command{Use: "create"}

    updateCmd := &cobra.Command{Use: "update"}

    createCmd.Flags().StringVarP(&name, "name", "n", "myname", "assign a name")
    createCmd.Flags().BoolVarP(&proxy, "proxy", "p", false, "use proxy to connect")

    createCmd.Args = cobra.ExactArgs(1)

    createCmd.Run = func(cmd *cobra.Command, args []string) {
        fmt.Println("creating")
        fmt.Println("name:", name)
        fmt.Println("proxy:", proxy)
        fmt.Println("args:", args)
    }

    rootCmd.AddCommand(createCmd, updateCmd)

    rootCmd.Execute()
}
```

1. 定義兩個 flag，`name` 及 `proxy`

    ```go
    createCmd.Flags().StringVarP(&name, "name", "n", "myname", "assign a name")
    createCmd.Flags().BoolVarP(&proxy, "proxy", "p", false, "use proxy to connect")
    ```

1. 設定只能有一個參數。詳細設定，請見：[cobra#Positional and Custom Arguments](https://github.com/spf13/cobra#positional-and-custom-arguments)

    ```go
    createCmd.Args = cobra.ExactArgs(1)
    ```

1. 設定執行動作

    ```go
    createCmd.Run = func(cmd *cobra.Command, args []string) {
        fmt.Println("creating")
        fmt.Println("name:", name)
        fmt.Println("proxy:", proxy)
        fmt.Println("args:", args)
    }
    ```

### 測試

1. `go run main.go create`

    ```text
    Error: accepts 1 arg(s), received 0
    Usage:
        myapp create [flags]

    Flags:
        -h, --help          help for create
        -n, --name string   assign a name (default "myname")
        -p, --proxy         use proxy to connect
    ```

    會發現回傳錯誤，並沒有執行動作。因為我們並沒有傳入任何參數。

1. `go run main.go create abc`

    ```text
    creating
    name: myname
    proxy: false
    args: [abc]
    ```

    執行成功，並使用預設值

1. `go run main.go create abc def`

    ```text
    Error: accepts 1 arg(s), received 2
    Usage:
        myapp create [flags]

    Flags:
        -h, --help          help for create
        -n, --name string   assign a name (default "myname")
        -p, --proxy         use proxy to connect
    ```

    執行失敗，因為多傳了一個參數。

1. `go run main.go create --name=bob --proxy abc` or `go run main.go create -n bob -p abc`

    ```text
    creating
    name: bob
    proxy: true
    args: [abc]
    ```

### 與 Viper 結合

可以將 flag 當作設定檔的資料。如此一來，在大型的程式中，就可以統一都使用 Viper 來當共用設定，而這些設定可以是來自設定檔或者是 command line 的 flag。

eg:

```go
var author string

func init() {
  rootCmd.PersistentFlags().StringVar(&author, "author", "YOUR NAME", "Author name for copyright attribution")
  viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
}
```