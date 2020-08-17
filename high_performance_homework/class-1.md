# 题目
分值：200

**题目描述**

本地下载TiDB，TiKV，PD源代码，改写源码并编译部署以下环境：
+ 1 TiDB
+ 1 PD
+ 3 TiKV

改写后
+ 使得TiDB启动事务时，会打一个“hello transaction”的日志

**输出： 一篇文章介绍以上过程**

截止时间：2020-08-16 24:00:00（逾期不给分）

作业提交方式：提交至个人github，将链接发送给talent-plan@tidb.io


# 答案

## 思路

很明显，整体可以拆解为两个问题：

1. 如何在本地编译并部署一个TiDB集群？
2. TiDB启动事务的代码在哪儿？

下面介绍每个问题的解决过程。

## 编译

### 准备环境

操作系统：
```bash
$ uname -a
Linux 4.14.81.bm.15-amd64 #1 SMP Debian 4.14.81.bm.15 Sun Sep 8 05:02:31 UTC 2019 x86_64 GNU/Linux
```

已有`go`的开发环境：
```bash
$ go version
go version go1.15 linux/amd64
$ go env
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/home/yangsongbao/.cache/go-build"
GOENV="/home/yangsongbao/.config/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOINSECURE=""
GOMODCACHE="/home/yangsongbao/go/pkg/mod"
GONOPROXY=""
GONOSUMDB=""
GOOS="linux"
GOPATH="/home/yangsongbao/go"
GOPRIVATE=""
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
GCCGO="gccgo"
AR="ar"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build576785736=/tmp/go-build -gno-record-gcc-switches"
```
知道`tikv`是用`rust`开发的，本地没有`rust`的开发环境，要准备。google一下"rust install"，找到[安装教程](https://www.rust-lang.org/tools/install)，其实就下面一个命令，很方便
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
安装过程很快，安装完成会有如下提示
```bash
Rust is installed now. Great!

To get started you need Cargo's bin directory ($HOME/.cargo/bin) in your PATH
environment variable. Next time you log in this will be done
automatically.

To configure your current shell run source $HOME/.cargo/env
```
很明显是要配置PATH，不着急，先`source $HOME/.cargo/env`。

`rust`环境安装完毕：
```bash
$ rustc --version
rustc 1.45.2 (d3fb005a3 2020-07-31)
```

### 下载代码
```bash
$ pwd
/home/yangsongbao
$ mkdir tidb
$ cd tidb
$ git clone https://github.com/pingcap/pd.git
$ git clone https://github.com/tikv/tikv.git
$ git clone https://github.com/pingcap/tidb.git
$ ls
pd  tidb  tikv
```

### 编译代码
看了下三个仓库，都有makefile文件，那应该可以直接`make`吧。
#### 编译pd
```bash
$ pwd
/home/yangsongbao/tidb/pd
$ make
....
CGO_ENABLED=0 go build -gcflags '' -ldflags '-X "github.com/pingcap/pd/v4/server/versioninfo.PDReleaseVersion=v4.0.0-rc.2-138-gbde92c6b" -X "github.com/pingcap/pd/v4/server/versioninfo.PDBuildTS=2020-08-16 09:22:22" -X "github.com/pingcap/pd/v4/server/versioninfo.PDGitHash=bde92c6b6d094fa26b84481a36949faf8eae94dc" -X "github.com/pingcap/pd/v4/server/versioninfo.PDGitBranch=master" -X "github.com/pingcap/pd/v4/server/versioninfo.PDEdition=Community" -X "github.com/pingcap-incubator/tidb-dashboard/pkg/utils/version.InternalVersion=2020.08.07.1" -X "github.com/pingcap-incubator/tidb-dashboard/pkg/utils/version.Standalone=No" -X "github.com/pingcap-incubator/tidb-dashboard/pkg/utils/version.PDVersion=v4.0.0-rc.2-138-gbde92c6b" -X "github.com/pingcap-incubator/tidb-dashboard/pkg/utils/version.BuildTime=2020-08-16 09:22:22" -X "github.com/pingcap-incubator/tidb-dashboard/pkg/utils/version.BuildGitHash=01f0abe88e93"' -o bin/pd-recover tools/pd-recover/main.go
```
编译成功
```bash
$ ls bin
pd-ctl	pd-recover  pd-server
```

#### 编译tidb

```bash
$ pwd
/home/yangsongbao/tidb/tidb
$ make
CGO_ENABLED=1 GO111MODULE=on go build  -tags codes  -ldflags '-X "github.com/pingcap/parser/mysql.TiDBReleaseVersion=v4.0.0-beta.2-953-gbaedc336a-dirty" -X "github.com/pingcap/tidb/util/versioninfo.TiDBBuildTS=2020-08-16 09:27:32" -X "github.com/pingcap/tidb/util/versioninfo.TiDBGitHash=baedc336afa26c85bbc9c865547789cce31597b0" -X "github.com/pingcap/tidb/util/versioninfo.TiDBGitBranch=master" -X "github.com/pingcap/tidb/util/versioninfo.TiDBEdition=Community" ' -o bin/tidb-server tidb-server/main.go
Build TiDB Server successfully!
```
很顺利
```bash
$ ls bin
tidb-server
```

#### 编译tikv
```bash
$ pwd
/home/yangsongbao/tidb/tikv
$ make
cargo build --release --no-default-features --features " jemalloc portable sse protobuf-codec"
   Compiling libc v0.2.66
   Compiling cfg-if v0.1.10
...
Building [==================================>                  ] 58/531
```
看着长长的进度条，等吧，直到出现如下提示，表示编译成功。
```bash
Finished release [optimized] target(s) in 12m 59s
```
## 部署
### 启动方式
因为已经都编译成二进制了，所以很自然地想到能不能直接通过二进制起一个集群。看了下三个仓库的README，在`pd`的[README](https://github.com/pingcap/pd/blob/master/README.md)中看到了二进制启动的方式，挺简单，我想其他两个应该也类似，于是决定直接用二进制启动试试看。

开始动手之前也在网上搜了下是否有这方面的介绍，看到一篇文章，做参考吧：[TiKV 源码解析 —— 调试环境搭建（二）之运行服务实例](http://www.iocoder.cn/TiKV/build-debugging-environment-second/)

### 启动顺序

之前了解过tidb的架构，大概直到三者之间的关系，pd管理元信息，tikv是kv层，tidb是sql层，依赖顺序应该是pd<-tikv<-tidb；我想这个顺序起应该没错。

### 启动pd
按照`pd`的[README](https://github.com/pingcap/pd/blob/master/README.md)的描述执行:
```bash
$ pwd
/home/yangsongbao/tidb/pd
$ nohup ./bin/pd-server --name=pd \
>                 --data-dir=pd \
>                 --client-urls="http://127.0.0.1:2379" \
>                 --peer-urls="http://127.0.0.1:2380" \
>                 --log-file=pd.log &
[1] 1025697
$ nohup: ignoring input and appending output to 'nohup.out'

$ tailf pd.log
[2020/08/16 09:52:24.145 +00:00] [WARN] [proxy.go:181] ["fail to recv activity from remote, stay inactive and wait to next checking round"] [remote=0.0.0.0:4000] [interval=2s] [error="dial tcp 0.0.0.0:4000: connect: connection refused"]
```
日志中看到很多WARN，应该是因为其他组件还没启动吧，先忽略。执行下检查，看到如下结果，应该是启动成功了。

```bash
$ curl http://127.0.0.1:2379/pd/api/v1/members
{
  "header": {
    "cluster_id": 6860867483441441832
  },
  "members": [
    {
      "name": "pd",
      "member_id": 3474484975246189105,
      "peer_urls": [
        "http://127.0.0.1:2380"
      ],
      "client_urls": [
        "http://127.0.0.1:2379"
      ],
      "deploy_path": "/data00/home/yangsongbao/tidb/pd/bin",
      "binary_version": "v4.0.0-rc.2-138-gbde92c6b",
      "git_hash": "bde92c6b6d094fa26b84481a36949faf8eae94dc"
    }
  ],
  "leader": {
    "name": "pd",
    "member_id": 3474484975246189105,
    "peer_urls": [
      "http://127.0.0.1:2380"
    ],
    "client_urls": [
      "http://127.0.0.1:2379"
    ]
  },
  "etcd_leader": {
    "name": "pd",
    "member_id": 3474484975246189105,
    "peer_urls": [
      "http://127.0.0.1:2380"
    ],
    "client_urls": [
      "http://127.0.0.1:2379"
    ],
    "deploy_path": "/data00/home/yangsongbao/tidb/pd/bin",
    "binary_version": "v4.0.0-rc.2-138-gbde92c6b",
    "git_hash": "bde92c6b6d094fa26b84481a36949faf8eae94dc"
  }
}
```

### 启动tikv

按照题目要求，需要部署3个TiKV实例，按照[参考文章](http://www.iocoder.cn/TiKV/build-debugging-environment-second/)，TiKV启动时可以指定启动配置，那么起三个实例应该需要三份配置，也需要有自己的存储。看了下TiKV的[配置模板](https://github.com/tikv/tikv/blob/master/etc/config-template.toml)，不同的实例最关键的应该是[server]配置中的[addr]和[status-addr]不要冲突，于是我从copy了三份tikv代码，修改了各自的配置。
```bash
$ pwd
/home/yangsongbao/tidb
$ ls
pd  tidb  tikv1  tikv2	tikv3
//tikv1/etc/config.toml
[server]
addr = "127.0.0.1:20161"
status-addr = "127.0.0.1:20181"
//tikv2/etc/config.toml
[server]
addr = "127.0.0.1:20162"
status-addr = "127.0.0.1:20182"
//tikv3/etc/config.toml
[server]
addr = "127.0.0.1:20163"
status-addr = "127.0.0.1:20183"
```
修改完配置就可以启动了，分别用下面三个命令启动了三个实例
```bash
//tikv1
nohup ./target/debug/tikv-server --config=./etc/config.toml --log-file=tikv.log &
//tikv2
nohup ./target/debug/tikv-server --config=./etc/config.toml --log-file=tikv.log &
//tikv3
nohup ./target/debug/tikv-server --config=./etc/config.toml --log-file=tikv.log &
```
在第三个实例启动后，分别在三个实例的日志文件里看到了如下日志，明显是raft在重新选主，TiKV3成为了leader

```bash
//tikv1
[2020/08/16 13:24:55.275 +00:00] [INFO] [raft.rs:894] ["became follower at term 12"] [term=12] [raft_id=49] [region_id=48]
//tikv2
[2020/08/16 13:24:55.275 +00:00] [INFO] [raft.rs:894] ["became follower at term 12"] [term=12] [raft_id=50] [region_id=48]
//tikv3
[2020/08/16 13:24:55.276 +00:00] [INFO] [raft.rs:985] ["became leader at term 12"] [term=12] [raft_id=51] [region_id=48]
```

### 启动tidb

根据[参考文章](http://www.iocoder.cn/TiKV/build-debugging-environment-second/)，启动`tidb`时指定了`--store=tikv`，不知道是什么意思，于是去[源码](https://github.com/pingcap/tidb/blob/master/tidb-server/main.go)里看了下注释
```bash
store            = flag.String(nmStore, "mocktikv", "registered store name, [tikv, mocktikv]")
```
嗯，那指定`--store=tikv`应该没啥毛病，于是用以下指令启动了`tidb`
```bash
nohup ./bin/tidb-server --store=tikv  --path="127.0.0.1:2379"   --log-file=tidb.log &
```
看到如下日志，应该是启动成功了
```bash
[2020/08/16 13:32:24.091 +00:00] [INFO] [printer.go:33] ["Welcome to TiDB."] ["Release Version"=v4.0.0-beta.2-953-gbaedc336a-dirty] [Edition=Community] ["Git Commit Hash"=baedc336afa26c85bbc9c865547789cce31597b0] ["Git Branch"=master] ["UTC Build Time"="2020-08-16 09:27:32"] [GoVersion=go1.15] ["Race Enabled"=false] ["Check Table Before Drop"=false] ["TiKV Min Version"=v3.0.0-60965b006877ca7234adaced7890d7b029ed1306]

...

[2020/08/16 13:30:21.274 +00:00] [INFO] [server.go:235] ["server is running MySQL protocol"] [addr=0.0.0.0:4000]
```

那么是不是可以用起来试试了呢？激动啊！！！于是在终端敲下命令：
```bash
mysql --host 127.0.0.1 --port 4000 -u root
```
Enter：
```bash
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.7.25-TiDB-v4.0.0-beta.2-953-gbaedc336a-dirty TiDB Server (Apache License 2.0) Community Edition, MySQL 5.7 compatible

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]>
```

没毛病。


## 改写源码

主要是要找到事务启动的代码在哪里。根据[TiDB 悲观事务模型](https://docs.pingcap.com/zh/tidb/stable/pessimistic-transaction)的描述：自 v3.0.8 开始，新创建的 TiDB 集群默认使用悲观事务模型。又在文章[TiDB 乐观事务模型](https://docs.pingcap.com/zh/tidb/stable/optimistic-transaction)中看到了乐观事务两阶段提交协议流程，在文章[TiDB 源码阅读系列文章（十九）tikv-client（下）](https://pingcap.com/blog-cn/tidb-source-code-reading-19/)中介绍了2PC算法的执行流程。

然后去`tidb`代码里搜了下关键字，找到了如下接口
```bash
// Storage defines the interface for storage.
// Isolation should be at least SI(SNAPSHOT ISOLATION)
type Storage interface {
	// Begin transaction
	Begin() (Transaction, error)
	// BeginWithStartTS begins transaction with startTS.
	BeginWithStartTS(startTS uint64) (Transaction, error)
```

这不就是开启事务的地方吗，看了下有多个实现，应该对应了不同的底层存储。那么日志可以加在接口实现的`Begin`方法中，也可以加在上层调用`Begin`的地方。因为在`tidb`启动时指定了`--store=tikv`，在`Storage`接口的实现中也有个`tikvStore`，于是我在`tikvStore`的实现中加入了日志。
```bash
func (s *tikvStore) Begin() (kv.Transaction, error) {
	logutil.BgLogger().Info("tikv store begin a transaction")
    //...
}

// BeginWithStartTS begins a transaction with startTS.
func (s *tikvStore) BeginWithStartTS(startTS uint64) (kv.Transaction, error) {
	logutil.BgLogger().Info("tikv store begin a transaction with start ts")
    //...
}
```
重新编译，启动`tidb`，发现在持续打印如下日志：
```bash
[2020/08/16 14:24:35.195 +00:00] [INFO] [kv.go:282] ["tikv store begin a transaction"]
[2020/08/16 14:24:35.247 +00:00] [INFO] [kv.go:292] ["tikv store begin a transaction with start ts"]
```
看到这里，我想应该是加对地方了，至于为什么一直打，我猜是`tidb`本身有些后台任务执行时也开启了事务，先不深究。

至此作业布置的任务算是基本完成了。

# 后记
虽然通过二进制启动了集群，但是整个过程还是有点麻烦的，是否有更容易的启动方式了，找了下文档，看到了[TiDB 数据库快速上手指南](https://docs.pingcap.com/zh/tidb/stable/quick-start-with-tidb)，看来官方推荐用TiUP来部署集群，正好看到学习交流群里有人提到：“我看到 tiup 支持指定 binary 参数，应该可以指定自己的编译文件”。看来是时候研究下TiUP了。