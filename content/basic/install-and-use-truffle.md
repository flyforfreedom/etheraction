---
title: "安装和使用 truffle"
date: 2018-03-10T09:43:29+08:00
weight: 1001
menu:
    main:
      identifier : ""
      parent: "action"
    main:
      identifier : ""
      parent: "basic"  
---

[Truffle](http://truffleframework.com/) 是一个以太坊智能合约开发框架，利用它可以方便地生成项目模板、编译合约、部署合约到区块链、测试合约等等。本篇文章介绍Truffle的安装过程以及基本使用。

Truffle安装，可直接看[官方安装说明文档](http://truffleframework.com/docs/getting_started/installation)：

## 简介

Truffle 是世界级的开发环境、测试框架、以太坊的资源管理通道，致力于让以太坊上的开发变得简单，使用 Truffle，可以：

1. 内置的智能合约编译，链接，部署和二进制文件的管理。
2. 快速开发下的自动合约测试。
3. 脚本化的，可扩展的部署与发布框架。
4. 部署到不管多少的公网或私网的网络环境管理功能
5. 使用EthPM&NPM提供的包管理，使用ERC190标准。
6. 与合约直接通信的直接交互控制台（写完合约就可以命令行里验证了）。
7. 可配的构建流程，支持紧密集成。
8. 在Truffle环境里支持执行外部的脚本。

{{<adm type="tip">}}
Truffle 是最流行的开发框架，使命是让开发更容易。
{{</adm>}}

## 安装依赖工具

### 安装 nodejs
Truffle 是一个 nodejs 包，因此依赖 nodejs 环境，前往[安装 nodejs](https://nodejs.org/zh-cn/download/) 。

#### 配置 npm 源(可选) 

npm 是 nodejs 的包管理器，nodejs 模块都是通过 npm 在线安装，npm 默认从国外的源（[https://registry.npmjs.org](https://registry.npmjs.org)）获取和下载包信息，国内访问速度很不理想。就像其他很多开源软件都有国内镜像源，npm 也不例外，所以我们可以利用国内镜像源来加速模块安装。使用如下命令切换到淘宝 npm 源：

```bash
# 配置 registry 
npm config set registry http://registry.npm.taobao.org
# 验证配置是否修改成功
npm config get registry
```

### 安装 git
git 为开源的源代码管理器，安装 Truffle 前需安装此工具。前往官网[安装 git](https://git-scm.com/downloads)

## 安装 truffle
通过 npm 安装最新版：
```bash 
npm install -g truffle 
# 检查是否安装成功
truffle version
```
{{<adm type="tip">}}
如果安装失败，也许是权限问题。尝试使用 sudo 安装。
```bash
sudo npm install -g truffle 
```
{{</adm>}}

更多细节，见官方文档： http://truffleframework.com/docs/getting_started/installation 。


## 使用 Truffle

### 创建项目
首先新建一个工程目录
```bash
mkdir myContract
cd myContract
```
接下来，通过下面的命令初始化一个 Truffle 工程：
```bash
truffle init
```
执行后，会在当前目录生成一个项目模板，生成的工程目录结构如下：
```text
myproject
├── contracts
│   └── Migrations.sol
├── migrations
│   └── 1_initial_migration.js
├── test
├── truffle-config.js
└── truffle.js

3 directories, 4 files
```
其中 contracts 文件夹是 Truffle 默认的合约文件存放地址、migrations 中存放的是发布合约的脚本、test 是用来测试应用和合约的测试脚本、truffle.js 是 Truffle 的配置文件。

### 编写合约
contracs 目录下的 Migrations.sol 是执行`init`时自动创建的默认合约，可用于熟悉 Truffle 使用。

```solidity
pragma solidity ^0.4.17;

contract Migrations {
  address public owner;
  uint public last_completed_migration;

  # 定义一个修改器，只有在执行方为 owner 时可执行。
  modifier restricted() {
    if (msg.sender == owner) _;
  }

  function Migrations() public {
    owner = msg.sender;
  }

  function setCompleted(uint completed) public restricted {
    last_completed_migration = completed;
  }

  function upgrade(address new_address) public restricted {
    Migrations upgraded = Migrations(new_address);
    upgraded.setCompleted(last_completed_migration);
  }
}

```

### 编译合约
要编译合约，使用：
```bash
truffle compile
```
Truffle 仅默认编译自上次编译后被修改过的文件，来减少不必要的编译。如果你想编译全部合约，可以使用 `--compile-all` 选项。
```bash
truffle compile --compile-all
```
{{<adm type="tip">}}
编译默认输出到 `./build/contracts` 下，如果目录不存在会自动创建。这些编译文件对于 Truffle 框架能否正常工作至关重要。你不应该在正常的编译或发布以外手动修改这些文件。
{{</adm>}}

{{<adm type="warning">}}
Truffle 规定定义的合约名称必须同文件名准确匹配。举例来说，如果文件名为 MyContract.sol，那么合约文件须为如下两者之一：
```solidity
contract MyContract {
  ...
}
// or
library MyContract {
  ...
}
```
这种匹配是**区分大小写的**，也就是说大小写也要一致。推荐大写每一个开头字母，如上述代码定义。
{{</adm>}}