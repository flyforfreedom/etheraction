---
title: "简介"
date: 2018-01-20T06:10:54+08:00
draft: true 
weight: 3001 
menu:
    main:
      identifier : ""
      parent: "soliditydoc"
---

## Solidity
<div align="center" class="align-center"><img alt="Solidity logo" src="/images/soliditylogo.svg" width="120px"></div>
Solidity 是面向智能合约的高级编程语言，它的设计受 C++、Python和JavaScript影响，旨在针对以太坊虚拟机(EVM)。

Solidity 是静态类型语言，支持继承，库和复杂的自定义类型等功能。

正如你见，它可用来创建投票，众筹，盲目拍卖，多签名钱包等等合同。

{{< note title="Note" >}}
现在，学习 Solidity 的最佳途径是用在线IDE [Remix](https://remix.ethereum.org/)(可能需要加载一段时间完成准备，请耐心等待)。
{{< /note >}}

## 交易

下面文档已被社区志愿者翻译成几种语言，但是英文版可作为参考。

+ [Spanish](https://solidity-es.readthedocs.io/)
+ [Russian](https://github.com/ethereum/wiki/wiki/%5BRussian%5D-%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE-%D0%BF%D0%BE-Solidity)(已过时)

## 有用链接

* [以太坊官网](https://ethereum.org)
* [Solidity更新日志](https://github.com/ethereum/solidity/blob/develop/Changelog.md)
* [用例看板](https://www.pivotaltracker.com/n/projects/1189488)
* [Solidity开源代码](https://github.com/ethereum/solidity/)
* [以太坊 Stackexchange](https://ethereum.stackexchange.com/)
* [Gitter聊天](https://gitter.im/ethereum/solidity/)

## 可用的 Solidity 集成

+ [Remix](https://remix.ethereum.org)
基于浏览器的IDE，含编译器和运行环境，无需服务端组件。

+ [IntelliJ IDEA 插件](https://plugins.jetbrains.com/plugin/9475-intellij-solidity)
集成到Intillij IDEA 的Solidity插件（支持所有的 JetBrains IDE）。

+ [Visual Studio 扩展](https://visualstudiogallery.msdn.microsoft.com/96221853-33c4-4531-bdd5-d2ea5acc4799/)
含 Solidity 编译器的 微软 Visual Studio 插件。

+ [SublimeText包- Solidity语法高亮](https://packagecontrol.io/packages/Ethereum)
SublimeText编辑器中支持对 Solidty 语法高亮。

+ [Etheratom](https://github.com/0mkara/etheratom)
Atom 编辑器的 Solidity 插件，支持语法高亮，编译和一个运行时环境（后端节点与 VM 兼容）。

+ [Atom Solidity Linter](https://atom.io/packages/linter-solidity)
Atom 编辑下 Solidity 插件，可对 Solidity 代码检查分析。

+ [Atom Solium Linter] (https://atom.io/packages/linter-solium)

Atom 下，基于 Solium 的 Solidty 配置检查器。

+ [Solium](https://github.com/duaraghav8/Solium)
一个严格遵循[Solidity 代码风格指南] (http://solidity.readthedocs.io/en/latest/style-guide.html)规范的命令行 Solidity 代码检查分析器。

+ [Solhint](https://github.com/protofire/solhint)
Solidity 代码检查分析器，对智能合约提供安全、风格指南和最佳实践检查。

+ [Visual Studio Code 扩展](http://juan.blanco.ws/solidity-contracts-in-visual-studio-code)
微软 Visual Studio Code 下 Solidity 插件，含语法高亮和 编译器。

+ [Emacs Solidity](https://github.com/ethereum/emacs-solidity/)
Emacs 编辑器下 Solidity 插件，含语法高亮和编译错误提示。

+ [Vim Solidity](https://github.com/tomlion/vim-solidity)
Vim 编译器下 Solidity 插件，支持编译检查。

**停产**
+ [Mix IDE](https://github.com/ethereum/mix/)
+ [Ethereum Studio](https://live.ether.camp/)	

## Solidity 工具集


* [Dapp](https://dapp.readthedocs.io)
Solidity 的构建工具、包管理器和部署助手。

* [Solidity REPL])https://github.com/raineorshine/solidity-repl)
可立即在命令行尝试 Solidity。

* [solgraph](https://github.com/raineorshine/solgraph)
可视化 Solidity 控制流程并高亮突出显示潜在的安全漏洞。


* [evmdis](https://github.com/Arachnid/evmdis)
EVM 反编译，对字节码进行静态分析，为字节码提供比原始 EVM 操作更高的抽象级别。
  
* [Doxity](https://github.com/DigixGlobal/doxity)
Solidity 文档生成器。

## 第三方 Soldity 解析器和语法

* [solidity-parser](https://github.com/ConsenSys/solidity-parser)
JavaScript版 Solidity 解析器。

* [Solidity Grammar for ANTLR 4](https://github.com/federicobond/solidity-antlr4)
ANTLR4 解析器生成器的 Solidity 语法。

## 编程语言文档

接下几页，我们首先将看到一个 Solidity 所写简单的智能合约，其中包括 [区块链](content/soliditydoc/introduction-to-smart-contracts.md) 和 [以太坊虚拟机](content/soliditydoc/introduction-to-smart-contracts.md) 基础知识。

下一章节，将给出有用的示例合约来解释 Solidity 的一些特性。记得，你可随时在[浏览器](https://remix.ethereum.org/)中尝试他它们。

最后和更多的章节将全面深度覆盖 Solidity。

如果您仍有疑问，可尝试在 [以太坊Stackexchange](https://ethereum.stackexchange.com/) 上搜索或询问，或者进入我们的[ Gitter 聊天频道](https://gitter.im/ethereum/solidity/)。 永远欢迎改善 Solidity 或本文档的想法！









