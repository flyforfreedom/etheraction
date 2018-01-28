---
title: "安装 Solidity"
date: 2018-01-20T06:16:54+08:00
draft: false
weight: 3003
menu:
    main:
      identifier : ""
      parent: "soliditydoc"
---

## 版本

不但发行版版本名遵循[语意化版本](https://semver.org/lang/zh-CN/)原则，**每晚自动构建版**也遵循原则。尽管我们努力确保其正常，但有些非正式或破坏性变更会被包含，而使得每晚自动构建的版本也许无法正常使用。

推荐使用最新发行版，下面使用最新发行版进行安装说明。

## Remix

> 推荐使用 Remix 来开发简单合约和快速学习 Solidity。

无需安装任何东西，就可[在线使用Reminx](https://remix.ethereum.org/)。如果你想离线使用，可按 https://github.com/ethereum/browser-solidity/tree/gh-pages 页面说明下载 zip 文件使用。

该页面有进一步详细说明如何安装 solidity 命令行编译器到你计算机上。如果你刚好要处理大型合约，或者需要更多的编译选项，那么选择使用命令行编译器 solc。

## npm/ Node.js

使用 npm 安装 Solidity 编译器 solcjs 小菜一碟。solcjs 程序的功能比本页下面的所有选项都要少。在[编译器](http://solidity.readthedocs.io/en/latest/using-the-compiler.html) 文档中，我们假定你所使用完整功能的编译器。 所以如果你是从 npm 安装 solcjs ，就此打住，直接跳到 [solc-js](https://github.com/ethereum/solc-js) 去了解。

{{<note title="Note" >}}
[solcjs](https://github.com/ethereum/solc-js) 是利用 Emscripten 从 C++ 版的 solc 跨平台编译为 JavaScript 的。可在 JavaScript 项目中使用 solcjs (同 Remix)。 具体介绍请查看 [solcjs](https://github.com/ethereum/solc-js) 代码库。
{{</note>}}
```bash
npm install -g solc
```
{{<note title="Note" >}}
命令行工具名称为 solcjs

solcjs 的命令行选项同 solc 和一些工具(如 geth )是不兼容的。solcjs 是阉割版的 solc。
{{</note>}}

## Docker

也提供最新版的 solc docker。 `stable` 库含发行版，而`nightly` 库含开发分支上的不稳定更改。

```bash
docker run ethereum/solc:stable solc --version
```
目前，docker 镜像只含有 solc 的可执行程序，因此你必须另外再处理链接合约源代码和输出目录的配置。

## 二进制安装包

可在 [solidity/releases](https://github.com/ethereum/solidity/releases)下载 Solidity 的二进制安装包。 

也有 Unbuntu 下的软件包，如下安装最新稳定版：
```bash
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc
```
也可安装揪心的开发版：
```bash
sudo add-apt-repository ppa:ethereum/ethereum
sudo add-apt-repository ppa:ethereum/ethereum-dev
sudo apt-get update
sudo apt-get install solc
```

同时也提供 snap 包，可在所有 [Linux 发行版](https://snapcraft.io/docs/core/install)中安装。安装最新稳定版命令如下：
```bash
sudo snap install solc
```
或者，如果你想测试 develop 分支下的变更，可如下安装：
```bash
sudo snap install solc --edge
```

同样，Arch Linux 也有提供安装包，仅限于最新开发版：
```bash
pacman -S solidity
```

在写本文时，从 Jenkins 迁移到 TravisCI 后还没有准备好 Homebrew 构建所需的二进制包。 现在，还得从源码安装，我们将尽快提供 homebrew 下二进制安装包。

```bash
brew update
brew upgrade
brew tap ethereum/ethereum
brew install solidity
brew linkapps solidity
```

如果你需要特定版本的 Solidity ，你需要从 Github 上安装一个 Homebrew formula ，见 Github 上 solidity.rb d的[提交记录](https://github.com/ethereum/homebrew-ethereum/commits/master/solidity.rb)的

如下，仍可从指定的`solidity.rb` commit 来下载安装历史版本：
```bash
brew unlink solidity
# Install 0.4.8
brew install https://raw.githubusercontent.com/ethereum/homebrew-ethereum/77cce03da9f289e5a3ffe579840d3c5dc0a62717/solidity.rb
```

Gentoo Linux 下也提供了安装包，可使用 `emerge` 安装：
```bash
emerge dev-lang/solidity
```


## 从源码安装

### 克隆代码库
执行以下命令，克隆源代码：
```bash
git clone --recursive https://github.com/ethereum/solidity.git
cd solidity
```
如果你想参与开发 Solidity, 你可 Fork Solidity 后，用你自个人的 Fork 库作为第二远程源。
```bash
cd solidity
git remote add personal git`@`github.com:[username]/solidity.git
```
Solidity 有 Git 子模块，需确保完全加载它们：
```bash
git submodule update --init --recursive
```

### 先决条件 - macOS
在 macOS 中，需确保有安装最新版的 [Xcode](https://developer.apple.com/xcode/download/)。 Xcode 包含[Clang C++ 编译器](https://en.wikipedia.org/wiki/Clang)。而 Xcode IDE 和其他苹果开发工具是 OS X 下编译 C++ 应用所必须的。如果你是第一次安装 Xcode 或者刚好更新了 Xcode 新版本，则在使用命令行构建前，需同意 Xcode 的使用协议：
```bash
sudo xcodebuild -license accept
```

Solidity 在 OS X 下构建，必须[安装 Homebrew](http://brew.sh/)包管理器来安装依赖。如果你想从头开始，这里有告诉你[如何卸载 Homebrew](https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/FAQ.md#how-do-i-uninstall-homebrew)。

### 先决条件 - Windows
需下载的依赖软件包：

|  软件  | 备注    |
| --- | --- |
| [Git](https://git-scm.com/download/win)   | 命令行从 Github 获取源码    |
| [CMake](https://cmake.org/download/)   | 跨平台构建文件生成器   |
| [Visual Studio 2015](https://www.visualstudio.com/products/vs-2015-product-editions) | C++ 编译开发环境|

### 外部依赖

在 macOS、Windows和其他 Linux 发行版上，有一个一键脚本可安装所需的外部依赖库。一键执行:
```bash
./scripts/install_deps.sh
```
Windows 下执行：
```bash
scripts\install_deps.bat
```

### 命令行构建
**确保你已安装外部依赖（见上面）**

Solidity 使用 CMake 来配置构建。在 Linux、macOS 和其他 Unix系统安装都差不多：
```bash
mkdir build
cd build
cmake .. && make
```
也有更简单的：
```bash
#note: 将安装 solc 和 soltest 到 usr/local/bin
./scripts/build.sh
```

对于 Windows 执行：
```bash
mkdir build
cd build
cmake -G "Visual Studio 14 2015 Win64" ..
```
命令尾行会在 build 目录下创建一个 solidity.sln 文件，双击使用 Visual Studio 打开。我们仅仅只要创建 **RelWithDebugInfo** 配置文件。

或者用命令创建：
```sh
cmake --build . --config RelWithDebInfo
```

## CMake 选项
如果你对 CMake 命令选项有兴趣，可执行 `cmake .. -LH`查看。

## 版本名说明

Solidity 版本名含有四部分

1. 版本号
2. 先行版本号，通常为 `develop.YYYY.MM.DD`或者`nightly.YYYY.MM.DD`
3. commit 和 格式化内容 `commit.GITHASH`
4. 含平台和编译器的详细信息的多条目内容

如果本地有修改，则 commit 部分有后缀 `.mod `。

此四部分按照 Semver 要求组成，第2部分等同 Semver 先行版本号，第三和四部分组成 Semver 版本编译信息。

发行版样例：`0.4.8+commit.60cc1668.Emscripten.clang`。

先行版样例：`0.4.9-nightly.2017.1.17+commit.6ecb4aa3.Emscripten.clang`。

## 版本重要说明
在版本发布之后，补丁版会跳跃。假设只有补丁变更，当变更被合并后，版本号根据变更和变更的严重程度来调整。最终是每晚构建版本来发布，而没有 `先行版`部分。

例如：

0. 0.4.0 版本发布
1. 从现在开始，每晚构建为 0.4.1 版本
2. 引入非破坏性变更 - 不改变版本号
3. 引入破坏性变更   -  版本跳跃到 0.5.0
4. 0.5.0 版本发布

此情况在[version pragma](http://solidity.readthedocs.io/en/latest/layout-of-source-files.html#version-pragma)下运行良好。

































