---
title: "安装Go Ethereum (geth)"
date: 2018-03-10T16:29:33+08:00
weight: 1002
menu:
    main: 
      identifier : ""      
      parent: "action"
    main: 
      identifier : ""    
      parent: "basic"  
---

## 简介 {#Introduction}
 
Go Ethereum 是以太坊区块链项目的三个原始实现（以及 c++ 和 python ）之一，使用 Go 语言实现，完全开源并使用开源协议 GNU LGPL v3 许可。 
**geth** 是简称，也是可执行程序的默认名称。

项目主页：https://ethereum.github.io/go-ethereum 。

## 安装 {#install}

可以使用多种途径安装 geth 可通过源代码或者二进制包安装。下面简要介绍安装，更多细节请浏览 geth 的英文版[安装指引](https://ethereum.github.io/go-ethereum/install/)。

### 独立安装包安装 {#standalone}

已提供 geth 各平台独立安装包软件，从官方[下载]((https://ethereum.github.io/go-ethereum/downloads/))安装。

### 从包管理器安装 {#install-from-a-package-manager}

在 mac 操作系统上使用 Homebrew 安装，Ubuntu 上使用 PPAs 安装。

#### 在 Ubuntu 上安装 {#on-ubuntu}

在 Ubuntu 上使用包管理器 PPAs 安装，先添加库：
```bash
sudo add-apt-repository -y ppa:ethereum/ethereum
```
此时便可安装最新稳定版的 Go Ethereum :
```bash
sudo apt-get update
sudo apt-get install ethereum
```
如果你想尝鲜使用开发版本，则只需执行：
```bash
sudo apt-get update
sudo apt-get install ethereum-unstable
```

#### 在 Mac 上安装 {#on-mac}

在 Mac 操作系统上使用包管理器 Homebrew 安装，如果无此工具，请先[安装](https://brew.sh/index_zh-cn)。因为 geth 涉及到多个依赖安装，故先 tap 再安装。
```shell 
brew tap ethereum/ethereum 
brew install ethereum 
``` 
可通过参数`--devel`直接安装开发版本(**可选**)
```shell 
brew install ethereum --devel 
``` 
安装完成后，可执行命令`geth version`查看版本信息，结果如下：
```text
Geth
Version: 1.6.7-stable
Git Commit: ab5646c532292b51e319f290afccf6a44f874372
Architecture: amd64
Protocol Versions: [63 62]
Network Id: 1
Go Version: go1.9
Operating System: darwin
GOPATH=/Users/one/Documents/dev/go
GOROOT=/usr/local/Cellar/go/1.9/libexec
``` 

### 下载 Docker 镜像 {#docker-image}

也可以直接使用以准备的 Go Ethereum 镜像环境，直接在 Docker 容器中运行 geth。

下面拉取 geth 镜像：
```bash
docker pull ethereum/client-go:latest
```  

## 运行以太坊私有链 {#private-chain-on-ethereum}

###  初始化创世区块 {#genesis} 

首先需要为私有链定义一个创世状态，使用 JSON 文件定义，例如取名为：`genesis.json`，内容如下：
```json
{
  "config": {
        "chainId": 88,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
  "alloc"      : {},
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x20000",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000000042",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}
```
保存到任意位置，例如 `$HOME/genesis.json`，
 
初始区块是区块链的起始 — 第一个区块，区块0，唯一没有指向前面区块的一个区块。协议确保其他连接你的节点必须和你的创世块一致，除非他们和你有相同的初始区块，这样你想创建多少私有测试网区块链，就可以创建多少！

执行初始化：
```shell 
geth  init $HOME/genesis.json
```
执行后，将提示成功初始化创世区。
```text
WARN [03-10|17:50:20] No etherbase set and no accounts found as default
INFO [03-10|17:50:20] Allocated cache and file handles         database=/Users/.../Library/Ethereum/geth/chaindata cache=16 handles=16
INFO [03-10|17:50:20] Writing custom genesis block
INFO [03-10|17:50:20] Successfully wrote genesis state         database=chaindata                                   hash=5e1fc7…d790e0
INFO [03-10|17:50:20] Allocated cache and file handles         database=/Users/.../Library/Ethereum/geth/lightchaindata cache=16 handles=16
INFO [03-10|17:50:20] Writing custom genesis block
INFO [03-10|17:50:20] Successfully wrote genesis state         database=lightchaindata                                   hash=5e1fc7…d790e0
```  

### 初始化账号   {#account}
运行命令创建账户，运行时将自动提示输入密码：
```
geth account new
WARN [03-10|17:53:03] No etherbase set and no accounts found as default
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {eac0e62d4faa273918770205a4b0e5f271e1ce23} 
``` 
创建成功后，将在末尾行显示账户地址`eac0e62d4faa273918770205a4b0e5f271e1ce23`。

虽然此账户可以在以太坊的JavaScript控制台中生成，但我一般直接通过命令工具生成，关于`geth account`命令还有更多信息：
```text
SAGE:
  geth account [options] command [command options] [arguments...] 

COMMANDS:
  list    汇总打印所有账号
  new     创建一个新账号
  update  更新已存在的账户
  import  导入私钥生成新的账号
  help, h 帮助
``` 
```shell
geth account list 
```
你应该可以看到上面所创建的账号。
```text 
Account #0: {eac0e62d4faa273918770205a4b0e5f271e1ce23} keystore:///Users/ysqi/Library/Ethereum/keystore/UTC--2018-03-10T09-53-08.639057000Z--eac0e62d4faa273918770205a4b0e5f271e1ce23
```
表明账号的私钥保存在keystore文件夹下。

### 进入控制台 {#console}
geth 提供了可交互的控制台，是一个 js 环境，可调用 geth 的一些 API。
```shell
geth  --networkid 9999 console
```
运行命令，将进入控制台，打印出如下信息。该信息非常重要。能告诉你运行时环境与配置信息。

+ Allocated cache and file handles 缓存存放目录
+ Disk storage enabled for ethash caches 数据存放目录
+ Disk storage enabled for ethash DAGs DAG数据存放目录
+ IPC endpoint opened IPC地址，**重要**，关系到后续以太坊钱包的链接

```text
INFO [09-09|10:40:36] Starting peer-to-peer node              instance=Geth/v1.6.7-stable-ab5646c5/darwin-amd64/go1.9
INFO [09-09|10:40:36] Allocated cache and file handles        database=/Users/one/Library/Ethereum/geth/chaindata cache=128 handles=1024
WARN [09-09|10:40:36] Upgrading chain database to use sequential keys
INFO [09-09|10:40:36] Database conversion successful
INFO [09-09|10:40:36] Initialised chain configuration          config="{ChainID: 15 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: <nil> EIP155: 0 EIP158: 0 Metropolis: <nil> Engine: unknown}"
INFO [09-09|10:40:36] Disk storage enabled for ethash caches  dir=/Users/one/Library/Ethereum/geth/ethash count=3
INFO [09-09|10:40:36] Disk storage enabled for ethash DAGs    dir=/Users/one/.ethash                      count=2
WARN [09-09|10:40:36] Upgrading db log bloom bins
INFO [09-09|10:40:36] Bloom-bin upgrade completed              elapsed=68.449µs
INFO [09-09|10:40:36] Initialising Ethereum protocol          versions="[63 62]" network=9999
INFO [09-09|10:40:36] Loaded most recent local header          number=0 hash=bd0e7a…9aadd0 td=131072
INFO [09-09|10:40:36] Loaded most recent local full block      number=0 hash=bd0e7a…9aadd0 td=131072
INFO [09-09|10:40:36] Loaded most recent local fast block      number=0 hash=bd0e7a…9aadd0 td=131072
INFO [09-09|10:40:36] Starting P2P networking
INFO [09-09|10:40:38] UDP listener up                          self=enode://f28939fbbd6038c074f322b656f314250bbf7372523ee6d7d2fd6b67f86dba3f41cdf394ab3c57bd105cb96bec70337c6c19a09ca794b6f0c36c3d04119c7c39@[::]:30303
INFO [09-09|10:40:38] RLPx listener up                        self=enode://f28939fbbd6038c074f322b656f314250bbf7372523ee6d7d2fd6b67f86dba3f41cdf394ab3c57bd105cb96bec70337c6c19a09ca794b6f0c36c3d04119c7c39@[::]:30303
INFO [09-09|10:40:38] IPC endpoint opened: /Users/one/Library/Ethereum/geth.ipc
Welcome to the Geth JavaScript console!

instance: Geth/v1.6.7-stable-ab5646c5/darwin-amd64/go1.9
coinbase: 0xe7a614776754b7c7ef3a1ef6430d29e90411fd75
at block: 0 (Thu, 01 Jan 1970 08:00:00 CST)
datadir: /Users/one/Library/Ethereum
modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0 
```

能在控制台中执行多种命令，具体见官方文档：[JavaScript-Console Wiki][JSCWiki]。
这是一个交互式的 JavaScript 执行环境，在这里面可以执行 JavaScript 代码，其中`>`是命令提示符。在这个环境里也内置了一些用来操作以太坊的Javascript对象，可以直接使用这些对象。这些对象主要包括：

+ eth：包含一些跟操作区块链相关的方法
+ net：包含以下查看p2p网络状态的方法
+ admin：包含一些与管理节点相关的方法
+ miner：包含启动&停止挖矿的一些方法
+ personal：主要包含一些管理账户的方法
+ txpool：包含一些查看交易内存池的方法
+ web3：包含了以上对象，还包含一些单位换算的方法


### 启动挖矿 {#miner}
如果你之前有部署运行过以太坊，请先将此目录下的DAG文件删除`rm -rf $HOME/.ethash/`。

参数说明

+ `--nodiscover `
使用这个命令可以确保你的节点不会被**非手动**添加你的人发现。否则，你的节点可能因为你与他有相同的创世文件和网络ID而被陌生人的区块链无意添加。

+ `--maxpeers 0` 
如果你不希望其他人连接到你的测试链，可以使用maxpeers 0。反之，如果你确切知道希望多少人连接到你的节点，你也可以通过调整数字来实现。 

+ `--rpc` 
可激活你节点的 rpc 服务。它在 geth 中通常被默认激活。 

+ `--rpcapi "db,eth,net,web3"` 
决定允许哪些 API 开放 rpc 服务。在默认情况下只激活了 web3 api。 

+ `--rpcport "8080"` 
将8000改变为你网络上开放的任何端口。Geth的默认设置是8080. 

+ `--rpccorsdomain "https://yushuangqi.com" `
这个可以指示什么URL能连接到你的节点来执行RPC定制端任务。务必谨慎，输入一个特定的URL而不是wildcard ( * )，后者会使所有的URL都能连接到你的RPC实例。

+ `--datadir "/home/TestChain1"` 
这是你的私有链数据所储存在的数据目录（在nubits下）。选择一个与你以太坊公有链文件夹分开的位置。 

+ `--identity "TestnetMainNode"` 
这会为你的节点设置一个身份，使之更容易在端点列表中被辨认出来。这个例子说明了这些身份如何在网络上出现。 

+ `--networkid 1999` 
数字类型，区分与其他的网络ID，以太坊公链的网络ID=1。必须区分，以放置钱包等误认为是以太坊公链。 2=Morden (disused), 3=Ropsten, 4=Rinkeby，默认为1。

+ `--port 30303` 
P2P网络监听端口，默认30303。 

+ `--fast ` 
这个命令是 Geth1.6.0之前的，只会被改成`--syncmode=fast`，但该命令继续有效。配置此命令能够快速的同步区块。 

+ `--cache=1024` 
程序内置的可用内存，单位MB。默认是16MB(最小值)。可以根据服务器能力配置到56, 512, 1024 (1GB), or 2048 (2GB)。

如下命令启动待遇自定义配置的私有链：
```bash
geth --identity "OneTestETH" --rpccorsdomain "*" --nodiscover --rpcapi "*"  --cache=1024 --networkid 9999  console
```
进入控制台后，可以启动矿工开始挖矿。
```bash
> miner.start(1)
```
这里的`1`表示只使用一个线程运行，如果配置过高，会导致电脑卡顿。
```text
INFO [09-09|11:17:36] Updated mining threads                  threads=1
INFO [09-09|11:17:36] Transaction pool price threshold updated price=18000000000
INFO [09-09|11:17:36] Starting mining operation
null
> INFO [09-09|11:17:36] Commit new mining work                  number=1 txs=0 uncles=0 elapsed=134.574µs
INFO [09-09|11:17:38] Generating DAG in progress              epoch=0 percentage=0 elapsed=1.051s
INFO [09-09|11:17:39] Generating DAG in progress              epoch=0 percentage=1 elapsed=2.087s
INFO [09-09|11:17:40] Generating DAG in progress              epoch=0 percentage=2 elapsed=3.129s
INFO [09-09|11:17:41] Generating DAG in progress              epoch=0 percentage=3 elapsed=4.229s
......
INFO [09-09|11:19:24] Generating DAG in progress              epoch=0 percentage=98 elapsed=1m47.287s
INFO [09-09|11:19:25] Generating DAG in progress              epoch=0 percentage=99 elapsed=1m48.790s
INFO [09-09|11:19:25] Generated ethash verification cache      epoch=0 elapsed=1m48.792s
INFO [09-09|11:19:29] Generating DAG in progress              epoch=1 percentage=0  elapsed=1.365s
INFO [09-09|11:19:30] Generating DAG in progress              epoch=1 percentage=1  elapsed=2.666s
INFO [09-09|11:19:31] Successfully sealed new block            number=1 hash=19b30c…c712b6
INFO [09-09|11:19:31] 🔨 mined potential block                  number=1 hash=19b30c…c712b6
INFO [09-09|11:19:31] Commit new mining work                  number=2 txs=0 uncles=0 elapsed=421.087µs

```
第一次运行时将开始创建 DAG 文件，只需等待进度条到100，则将开始挖矿。 实际你看到的挖矿速度很快，这是因为我们已经在初始化创世区块时配置为:`"nonce": "0x0000000000000042"`。
“0x42”难度能可使得私有链上快速挖出以太币，几分钟就会有上百个以太币。
 
挖矿时需要指定挖矿收取奖励的以太坊账户，而系统默认使用创建的第一个账号。

如果不想停止挖矿，则执行：
```shell
> miner.stop()
```
停止后，可以查看将挖出不少以太币。在控制台中可查询矿工余额。
```shell
> eth.accounts
```
这会返回到你拥有的账户地址排列。
```shell
> eth.getBalance(eth.accounts[0])
```
这个控制台指令会返回到你第一个以太坊地址。因为我们只创建了一个账号，也将是矿工的账号。
而` eth.getBalance()`返回的余额是以太币的最小面额 wei。

```shell
> primary = eth.accounts[0]
> balance = web3.fromWei(eth.getBalance(primary), "ether");
```
将返回矿工的以太币余额，将 wei 转换为以太币 ether。



[go-ethereum]: http://ethdocs.org/en/latest/ethereum-clients/go-ethereum/index.html#go-ethereum
[Parity]:http://ethdocs.org/en/latest/ethereum-clients/parity/index.html#parity
[cpp-ethereum]:http://ethdocs.org/en/latest/ethereum-clients/cpp-ethereum/index.html#cpp-ethereum
[pyethapp]:http://ethdocs.org/en/latest/ethereum-clients/pyethapp/index.html#pyethapp
[ethereumjs-lib]:http://ethdocs.org/en/latest/ethereum-clients/ethereumjs-lib/index.html#ethereumjs-lib
[Ethereum(J)]:http://ethdocs.org/en/latest/ethereum-clients/ethereumj/index.html#ethereum-j
[ruby-ethereum]:http://ethdocs.org/en/latest/ethereum-clients/ruby-ethereum/index.html#ruby-ethereum
[ethereumH]:http://ethdocs.org/en/latest/ethereum-clients/ethereumh/index.html#ethereumh
[Ethereum Foundation]: https://ethereum.org/foundation
[ether camp]: http://www.ether.camp
[BlockApps]: http://www.blockapps.net/
[Ethcore]: https://parity.io/
[Jan Xie]: https://github.com/janx/
[JSCWiki]: https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console
[ethereum org]:https://ethereum.org/
[mist wiki]:https://github.com/EthFans/wiki/wiki/以太坊钱包-Mist-使用教程
[ipcsrc]:https://github.com/ethereum/mist/blob/master/modules/settings.js#L248
[MinerIssue]:https://github.com/ethereum/go-ethereum/issues/2174 


## 使用 Mist 钱包

自行[下载](https://github.com/ethereum/mist/releases)以太坊钱包 Mist。