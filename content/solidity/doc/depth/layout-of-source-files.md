---
title: "源文件布局"
date: 2018-01-21T06:29:18+08:00
weight: 3051
menu:
    main:  
      parent: "solidityindepth"
---

源文件中可以定义任意个合约(contract)、嵌入(inlucde)和杂注(pragma)指令。
 

## 版本杂注

源文件中能够(也应该)用所谓版本杂注，以便拒绝被含有不兼容性变更的高版本编译器所编译。 我们尽量保持最小变更，特别是语义变化而引起语法变更的，但这总不可能避免。因此，对于有重大变化的新版本，通读更新日志是一个不错的建议。大版本的版本号始终是 `0.x.0`或者 `x.0.0` 形式。  

版本杂注使用方式如下：
```js
pragma solidity ^0.4.0;
```
这样的源文件将不会使用低于版本 0.4.0 的编译器编译，也不适用于从 0.5.0 开始的编译器（被 `^` 限制）。 这做法背后的思想是，在版本 0.5.0 之前不会有任何重大改变，所以我们可确保代码始终按照我们的意愿进行编译。我们没有修复编译器的确切版本，因此补丁版仍然是有可能的。

可以为编译器指定更复杂的编译规则，指令表达式遵循 [npm](https://docs.npmjs.com/misc/semver>) 版本语义。
 

## 导入其他源文件
### 语法与语义 

Solidity 支持导入语句。尽管 Solidity 还不能知道 "default export" 的概念，但已经 非常类似 JavaScript 用法（从 ES6 起）。

在全局级上，你可使用下面格式使用导入语句：
```js
import "filename";
```
此语句将从 filename 中导入所有的全局符号到到当前全局域中（不同于 ES6 但 Solidity 是向后兼容）。
```js 
import * as symbolName from "filename";
```
......创建一个新全局符号 `symbolName`，成员均是来自 `"filename"` 中全局符号。

```js
import {symbol1 as alias, symbol2} from "filename";
```
......创建新全局符号 `alias` 和 `symbol2`。`symbol1`和`symbol2`分别从 `"filename"`引入。

另外一种语法不属于 ES6，但或许是便利的：
```js 
import "filename" as symbolName;
```
此等同于 `import * as symbolName from "filename";`.

### 路径 

在上面，`filename` 始终被视为以 `/` 作为目录分割符的文件路径。`.`表示当前目录，`..`表示父目录。 当 `.` 或 `..` 后面跟随的是非`/`字符时，不作为当前或父目录对待。除非以当前目录`.`或者父目录`..`开头，所有路径名均被视为绝对路径。

用 `import "./x" as x;` 语句导入当前源文件同目录下的文件`x`。 如果你使用`import "x" as x;`，相反，会有一个不同的文件(在全局 `include directory`范围内)被引入。

路径取决于编译器（见下）如何真实解析。通常，目录层次不必严格映射到你的本地文件系统，它也可以映射到资源发现，如 ipfs，http 或者 git。
   

## 使用真正的编译器

当编译器被调用后，它不仅能规定如何发现路径的第一个元素，还可规定路径前景重映射。例如 `github.com/ethereum/dapp-bin/library` 重映射到`/usr/local/dapp-bin/library` 且编译器也将从这里读取文件。如果多个重映射被规定，优先尝试 key 最长的一个。 还允许规定”回退映射“，例如，`""`映射到
`"/usr/local/include/solidity"`。此外，这些重映射可依赖于上下文，以允许你配置要所导入的包，例如，同一个库名的不同版本。 

**solc**: 

solc（命令行编译器），这些重映射以 `context:prefix=target` 参数形式提供。 `context`必须有，`=target`可选（此时 target 默认为 prefix）。所有重映射的常规文件都被编译（包括他们的依赖）。此机制完全向后兼容（只要没文件名包含 = 或 :），因此不会是一个破坏性变更。在 `content` 目录下或之下的，所有以 `prefix` 开头的文件都将被用 `target` 替换 `prefix` 来重定向。
 
举个例子，如果你克隆 `github.com/ethereum/dapp-bin/` 到本地 ``/usr/local/dapp-bin``，你可在源文件中如下使用： 
```js
import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;
```
再运行编译器:
```bash
solc github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ source.sol
```
作为一个更复杂的例子，假设你依赖一个非常旧版本的 dapp-bin 上的一些模块。 旧版本 dapp-bin 签出到`/usr/local/dapp-bin_old`, 那么你可使用：
```bash
solc module1:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ \
     module2:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin_old/ \
    source.sol
```   
以便 `module2` 下所有导入都指向旧版本，而 `module1` 指向新版本。

注意， solc 只允许你包含来自某些目录的文件：目录（或子目录）可以是明确指定源文件的目录之一，或重映射目标路径。如果你想指向绝对路径，只需重映射为 `=/`。

如果有多个重映射指向一个有效文件，那选择最长公共前缀的重映射。
 
**Remix**:

[Remix](https://remix.ethereum.org/) 提供了一个为 github 的自动重映射，将通过网络自动获取文件。你可从导入一个键迭代器，如：`import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;`。

可能在未来支持其他源码平台。
 

## 注释 

可单行注释(`//`)和多行注释(`/*...*/`)

```js
// This is a single-line comment.

/*
This is a
multi-line comment.
*/
```
此外，有另一种注释称为 natspec 注释，其文档尚未编写。 它们用三扛（`///`）或双星块(`/** ... */`)编辑，它应直接在方法声明或语句上使用。你可在注释中使用 [Doxygen](https://en.wikipedia.org/wiki/Doxygen)样式的标签来文档化方法，解释正常验证的条件。并提供一个 **确认信息**，可在用户尝试调用一个方法时提示。  


下面示例，我们记录合约标题和解释两个入参、两个返回值。

In the following example we document the title of the contract, the explanation
for the two input parameters and two returned values.

```js
pragma solidity ^0.4.0;

/** @title Shape calculator. */
contract shapeCalculator {
    
    /** @dev Calculates a rectangle's surface and perimeter.
      * @param w Width of the rectangle.
      * @param h Height of the rectangle.
      * @return s The calculated surface.
      * @return p The calculated perimeter.
      */
    function rectangle(uint w, uint h) returns (uint s, uint p) {
        s = w * h;
        p = 2 * (w + h);
    }
}
```    