---
title: "类型"
date: 2018-01-22T06:41:39+08:00
draft: true
weight: 3053
menu:
    main:
      identifier : ""
      parent: "solidityindepth"
---

Solidity 是一门静态类型语言，意味着每个变量（状态变量和本地变量）类型都需要在编译期确定（或至少是已知的——见下方的 [类型推导]({{<ref "#type-deduction">}})）。 Solidity 提供了几个基础类型，可以组合成复杂类型。

此外，类型可在含运算符的表达式中交互。快速查阅各种运算，请见[运算符优先级](http://solidity.readthedocs.io/en/latest/miscellaneous.html#order)

## 值类型

下面所列类型均可称为值类型，因为它们的变量总是值传递。即，当作为方法参数或用作赋值时，总是拷贝值。

### 布尔型 {#booleans}

布尔类型 ` bool `  ，其可能有的值是常量 ` true `  和 ` false ` 。

运算操作有：

|运算符|含义|
|---|---|
| **!** | 逻辑否 |
| **&&** | 逻辑且(and) |
| **\|\|** | 逻辑或(or) |
| **==** | 等于 |
| **!=** | 不等于|

 ` || ` 和 ` && ` 运算符常用短路求值(short-circuiting)策略。意思是，在表达式 ` f(x) || g(y) ` 中，如果 ` f(x) ` 运算结果是 ` true ` ，则 ` g(y) ` 不会被运算，哪怕它有副作用。

### 整数型 {#integers}

整数类型分 ` int ` 和 ` uint ` 。有符号和无符号各种不同位长度的整数，从 ` uint8 `  到 ` uint256 `  有8个（无符号 8 到 256 位），还有 ` int8 `  到 ` int256 ` 。 且 ` uint `  和 ` int `  分别是 ` uint256 `  和 ` int256 ` 的别名。

运算操作有：

+ 比较运算： ` <= ` 、 ` < ` 、 ` == ` 、 ` != ` 、 ` >= ` 、 ` > `  (结果为 ` bool `  )
+ 位运算： ` & ` 、 ` | ` 、 ` ^ ` (按位异或)、 ` ~ ` (按位取反)、` << ` （左移）、 ` >> ` （右移）
+ 算术运算： ` + ` 、 ` - ` 、一元 ` - `和` + ` 、 ` * ` 、 ` / ` 、 ` % ` (取余)、 ` ** ` (幂)

整数相除总是截断取整的（它只别编译到 EVM 的 `DIV`操作码），但如果两个数都是有小数（或是小数表达式）时则不会截断。

除以零和零取模会抛出运行时异常。

移位操作的结果是左操作数的类型。表达式 ` x << y ` 相当于 ` x * 2**y`。 ` x >> y`相当于 ` x / 2**y`。这意味着可延伸到负数的位移，位移一个负数金额会抛出运行时异常。


{{< warning title="负值右移" >}}
有符号整数类型的负值右移运算结果和其他编程语言运算结果是不一样的。

在 Solidity 中右移映射到除法，因此位移后的负值将趋向与零（被截断）。在其他编程语言中，右移负数形同相除再四舍五入（趋向负无穷大）。
{{< /warning >}}

## 定点数 {#fixed-point-numbers}

{{< warning title="未完整支持" >}}
在 Solidity 中定点数尚未完整支持。能够定义它们，但不能被分配或从。
{{< /warning >}}
 
定点数有 `fixed` 或 `ufixed`，有符合和无符号定点数各有不同位长度数字。`ufixedMxN` 和 `fixedMxN`，这里 `M` 表示该类型所代表的数字位长度，`N` 表示有多少小数点可用。 `M` 必须是从 8 位到 256 位间且能被 8 整除。`N` 必须是 `0` 到 `80` 之间。还包括，`ufixed` 和 `fixed` 分别是 `ufixed128x19` 和 `fixed128x19` 的别名。

运算符：
+ 比较运算： `<=`、`<`、`==`、`!=`、`>=`、`>` (结果为布尔 bool )
+ 算术元素：`+`、`-`、一元`-`和一元`+`、`*`、`、`、`%`（取余）


{{< note title="与浮点数" >}}
浮点数（在其他许多语言中的`float` 和 `double`，精度更高的 IEEE754 规范）和 定点数的主要区别是，前者是整数部分和小数部分的位长度是灵活的，而在后者中是严格界定的。通常，一般来说，在浮点数中，几乎整个空间都用来表示数字，而只有少量的位长定义了小数点的位置。
{{< /note >}}

## 地址 {#address}

地址 `address` 是保存 20 个字节的值（以太坊账户地址长度）。地址类型也有成员，是所有合约的基础。

运算符：
+ `<=`、`<`、`==`、`!=`、`>=`和`>` 

{{< note title="Note" >}}
从 0.5.0 版本开始，合约不是从地址类型派生的，但仍然可以明确地将合约转换为地址。
{{< /note >}}

### 地址成员 {#member-of-address}

+ **余额 `balance` 和 转账 (`transfer`)**

请参阅 [Address Related](http://solidity.readthedocs.io/en/latest/units-and-global-variables.html#address-related) ,以快速参考。

是可以使用属性 `balance` 查询地址余额的，且可使用 `transfer` 方法发送 Ether(以 wei 为单位)到一个账户：
```js
address x = 0x123;
address myAddress = this;
if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
```

{{< note title="与浮点数" >}}
如果 `x` 是合约地址，它的代码（更确切的说：它的回退方法，如果有）将被和 `transfer` 调用一起执行（这是 EVM 的一个特性，不能被绕开）。如果执行 gas 耗尽或者任何方式的失败，这 Ether 转账将被恢复（注：即回退交易），且当前合约而终止执行。
{{< /note >}}


+ **发送 (`send`)**

发送是转账的低级副本，如果执行失败，当前合约不会停止，但 `send` 将返回 `false`。


{{< warning title="与浮点数" >}}
使用 `send` 是有些危险的，如果执行堆栈深度到了 1024 （caller 强制限制） 则交易失败。同样 gas 耗尽也会失败。所以为了让转账安全，应总检查 `send` 返回值。 使用 `transfer` 或更好：使用一种收款人取款的模式。
{{< /warning >}}

+ **调用(call)**、**callcode**、**委托调用(delegatecall)**

此外，不遵循 ABI 合约接口，方法 `call` 提供任意类型的任意数量的参数。这些参数被填充到 32 字节并连接起来。一个例外是，第一个参数会被编码为四个字节。在此情况下，不允许
使用函数签名来填充。

```js
address nameReg = 0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2;
nameReg.call("register", "MyName");
nameReg.call(bytes4(keccak256("fun(uint256)")), a);
```
`call` 返回一个布尔值，来表明是否被调用方法被终止(`true`)还是引发了 EVM 异常 (false)。不可能访问返回的实际数据（这得需要我们事先知道编码和大小）。

可以用 `.gas()` 修饰符来调整供给的 gas :
```js
namReg.call.gas(1000000)("register", "MyName");
```
同样也可调整以太币 value：
```js
namReg.call.value(1 ether)("register", "MyName");
```
最后，这些修改可以连接在一起，顺序无关紧要：
nameReg.call.gas(1000000).value(1 ether)("register", "MyName");

{{< note title="Note" >}}
在重载方法上是不可能使用 gas 和 value 修改器的。

解决方案是引入一个 gas 和 value 特例，只需重新检查它们是否存在于重载分析点。
{{< /note >}}

类似地，`delegatecall` 方法能被使用。区别在于只用给定地址的代码，所有其他内容（存储，余额）都从当前合约中获取。`delegatecall`的用意是使用存储在其他合约中的库代码。用户必须确保两个合同中的存储布局适合委托使用。从 Prior 版 到 homestead 版，只有一个名为 `callcode` 的有限变天可用，它不提供对原始 `msg.sender` 和 `msg.value` 值得访问。

此三个方法 `call`、`delegatecall` 和 `callcode` 是非常低级别功能，只能做为最后手段，因为他们打破了 Solidity 的类型安全性。

`.gas()` 选项适用此三个方法，而已 `.value()`选项不支持 `delegatecall`

{{< note title="Note" >}} 
所有 contact 继承 address 成员，因此可以使用 `this.balance` 查询当前合约余额。
{{< /note >}}


{{< note title="Note" >}} 
不鼓励使用 `callcode` ，在未来，将移除它。
{{< /note >}}


{{< warning title="Note" >}} 
所有这些功能都是低级功能，应谨慎使用。具体而言，任何未知合约都可能是恶意的，如果你调用它，你把控制权移交给那个合约，然后再返回到你的合约。以便在调用返回时，准备好更改状态变量。
{{< /warning >}}

## 定长字节数组 {#fixed-size-byte-arrays}

固定大小有`bytes1`、`bytes2`、`bytes3`、...、`bytes32`，`byte` 是 `bytes1`别名。

运算符：

+ 比较运算：`<=`、`<`、`==`、`!=`、`>=`和`>` （结果为 `bool`）
+ 位运算： ` & ` 、 ` | ` 、 ` ^ ` (按位异或)、 ` ~ ` (按位取反)、` << ` （左移）、 ` >> ` （右移）
+ 索引访问： 如果 `x` 是 `bytesI` 类型，则 `x[k]`（0<= k <= I）返回第 `k` 个字节（只读）。 

位移运算符使用任何整数类型作为右操作数（但将返回左操作数的类型），这表示要移位的位数。位移负数将导致运行时异常。

### 成员

+ `length` 字段返回字节数组的固定长度（只读）。

{{< note title="Note" >}} 
可以使用字节数组作为 `byte[]`，但当调用传参时，它浪费了很多空间，每个元素占用 31 个字节。最好使用 `bytes`。
{{< /note >}}

## 动态大小字节数组 {#dynamically-sized-byte-array}

`bytes`:

 >  动态长度字节数组，见[数组]({{<ref "#arrays">}})，非值类型！

`string`:

>  动态长度的 UTF-8 编码字符串，见[数组]({{<ref "#arrays">}})，非值类型！

根据经验，任意长度的原始 byte 数据使用 `bytes`，任意长途的字符串(UTF-8)数据使用`string`。如果你能限制字节长度到一定数量，要总使用 `bytes1` 到 `bytes32`，因为它们便宜得多。

## 地址字面量 {#address-literals}

通过地址校验的十六进制字面量，例如 `0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF` 是一个 `address` 类型。长度在39到41位之间的十六进制字面值，不通过校验会产生警告，并被视为常规的有理数字面值。

{{< note title="Note" >}} 
[EIP-55](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md)定义了混合大小写的地址校验格式。
{{< /note >}}

## 有理数和整数字面量 {#rational-and-integer-literals}

整数字面量由 0 到 9 范围内的一系列数字组成，被视作十进制数。例如 `69` 意味着六十九。在 Solidity 中不存在八进制字面量，且高位零 (leading zeros)是无效的。

十进制小数字面量由 一个 `.` 组成，至少一边有数。例如：`1.`、`.1`和`1.3`。

科学记数法也支持，基数可以有小数，而指数不能。例子包括`2e10`，`-2e10`，`2e-10`，`2.5e1`。

例如，尽管中间结果不适应机器字长，可`(2**800 + 1) - 2**800` 返回常量 `1`（类型为 `uint8`）。此外，`.5 * 8` 返回整数`4`（虽然之间使用了非整数）。

只要操作数是整数，任何可以应用到整数的运算符也可以应用于数字字面量表达式中。如果两者中的任何一个是小数，则不允许位操作。如果指数是小数（因为这可能导致非有理数），则不能取幂。


{{< note title="Note" >}} 
每个有理数都有一个数字字面量类型。整数字面量和有理数字面量均属于数字字面量类型。此外，所有数字字面量表达式（即仅包含数字字面量和运算符的表达式）属于数字字面量类型。所以数字字面量表达式 `1 + 2` 和 `2 + 1` 对于有理数 3 都属于相同的数字字面量类型。
{{< /note >}}



{{< warning title="除法" >}} 
在早期版本中，数字字面量除法是截断的，但现在将转换为有理数，即 `5/2` 不等于 `2`，而是等于 `2.5`。
{{< /warning >}}


{{< note title="Note" >}} 
数字字面量表达式只要与非字面量表达式一起使用，就会转换为非字面量类型。即使我们知道在下面的例子中赋值给 `b` 的表达式值计算为一个整数，但是部分表达式 `2.5 + a`没有进行类型检查，所以代码不会编译。
{{< /note >}}

```js
uint128 a = 1;
uint128 b = 2.5 + a + 0.5;
```

## 字符串字面量 {#string-literal}

字符串字面量用双引号或单引号括起来（`"foo"`或`bar`）。这不意味着想C语言包含结束符。`"foo"`代表三个字节而非四个。与整数字面量一样，他们类型可变化。可隐式转换为 `bytes1`，...... `bytes32`。如果大小恰当，转换为 `bytes` 和 `string`。

字符串字面量支持转义字符，如`\n`、`\xNN`和`\uNNNN`。其中`\xNN`取十六进制值并插入大小合适字节，而 `\uNNNN` 采用 Unicode 码点值，插入一个 UTF-8 序列。


## 十六进制字面量 {#Hexadecimal-liternal}

十六进制字面量带有前缀 `hex` 关键字，并用双引号或单引号括起来(`hex"001122FF"`)。

其内容必须是一个十六进制字符串，其值将用二进制表示。

十六进制字面量行为像字符串字面量，并具有相同的可转换性限制。

## 枚举 {#enum}

枚举是在 Solidity 中创建用户自定义类型的一种方法。可显式地与整数类型互转，但是不允许隐式转换。 显式转换会在运行时检查值范围，失败会导致异常。 枚举需要至少一个成员。

```js
pragma solidity ^0.4.16;

contract test {
    enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
    ActionChoices choice;
    ActionChoices constant defaultChoice = ActionChoices.GoStraight;

    function setGoStraight() public {
        choice = ActionChoices.GoStraight;
    }

    // Since enum types are not part of the ABI, the signature of "getChoice"
    // will automatically be changed to "getChoice() returns (uint8)"
    // for all matters external to Solidity. The integer type used is just
    // large enough to hold all enum values, i.e. if you have more values,
    // `uint16` will be used and so on.
    function getChoice() public view returns (ActionChoices) {
        return choice;
    }

    function getDefaultChoice() public pure returns (uint) {
        return uint(defaultChoice);
    }
}
```

## 函数类型 {#function-type}

函数类型是函数的类型，函数类型的变量可以由函数赋值，函数类型的参数可传递函数，且函数调用可返回函数。 函数类型有两类：内部(internal)函数和外部(external)函数。

因为不能在当前合约上下文之外执行内部函数，内部函数只能在当前合约中调用（即在当前合约代码体中，还包括内部库函数和内部函数）。调用内部函数是通过跳到它的入口标签来实现的，就像在内部调用当前合同的函数一样。

外部函数由一个地址和一个函数签名组成，它们可以由外部函数调用或作为参数传递。


函数类型定义语法如下：
```js
function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]
```
与入参相反，返回值不能为空。 如果函数类型无返回值，则 `returns (<return types>)` 部分必须省去。

函数类型默认为内部函数，因此 关键字 `internal` 可省略。相反，合约函数本身默认是公开的，合约函数只有用作类型名时才默认是内部的。

在当前合约中有两种方式访问函数，直接用名称`f`或用`this.f`。前者将按内部函数处理，后者按外部函数处理。

如果一个函数类型变量尚未初始化，调用时将引起异常。同样地，在调用 `delete` 后调用函数也会引起异常。

如果在 Solidity 上下文外部使用外部函数类型，则将它们视为函数类型，将地址连接函数前面一起编码为单个 bytes24 字节。

注意，当前合约的公共函数可作为内部和外部函数使用。仅用`f`，则公共函数 `f` 当做内部函数使用，如果你想使用它的外部形式，则用`this.f`。

另外，公共（或外部）函数都有一个特殊的成员 `selector`，返回 [ABI函数 selector](http://solidity.readthedocs.io/en/latest/abi-spec.html#abi-function-selector)

```js
pragma solidity ^0.4.16;

contract Selector {
  function f() public view returns (bytes4) {
    return this.f.selector;
  }
}
```
如何使用内部函数类型的示例展示：
```js
pragma solidity ^0.4.16;

// 库
library ArrayUtils {
  // 因为在同一个合约代码上下文中，内部函数可作为内部库函数。 
  function map(uint[] memory self, function (uint) pure returns (uint) f)
    internal
    pure
    returns (uint[] memory r)
  {
    r = new uint[](self.length);
    for (uint i = 0; i < self.length; i++) {
      r[i] = f(self[i]);
    }
  }
  function reduce(
    uint[] memory self,
    function (uint, uint) pure returns (uint) f
  )
    internal
    pure
    returns (uint r)
  {
    r = self[0];
    for (uint i = 1; i < self.length; i++) {
      r = f(r, self[i]);
    }
  }
  function range(uint length) internal pure returns (uint[] memory r) {
    r = new uint[](length);
    for (uint i = 0; i < r.length; i++) {
      r[i] = i;
    }
  }
}

contract Pyramid {
  using ArrayUtils for *;
  function pyramid(uint l) public pure returns (uint) {
    return ArrayUtils.range(l).map(square).reduce(sum);
  }
  function square(uint x) internal pure returns (uint) {
    return x * x;
  }
  function sum(uint x, uint y) internal pure returns (uint) {
    return x + y;
  }
}
```

另外一个使用外部函数类型的示例：
```js
pragma solidity ^0.4.11;

contract Oracle {
  struct Request {
    bytes data;
    function(bytes memory) external callback;
  }
  Request[] requests;
  event NewRequest(uint);
  function query(bytes data, function(bytes memory) external callback) public {
    requests.push(Request(data, callback));
    NewRequest(requests.length - 1);
  }
  function reply(uint requestID, bytes response) public {
    // 检查 replay 来源可信源
    requests[requestID].callback(response);
  }
}

contract OracleUser {
  Oracle constant oracle = Oracle(0x1234567); // 已知合约
  function buySomething() {
    oracle.query("USD", this.oracleResponse);
  }
  function oracleResponse(bytes response) public {
    require(msg.sender == address(oracle));
    // 使用数据
  }
}
```

{{< note title="Note" >}} 
Lambda 或内联函数已计划但尚未支持。 
{{< /note >}}

## 引用类型 {#reference-types}

复杂类型，即不能总是适合 256 位的类型，必须比已见的值类型更仔细地处理。因为拷贝它们是十分昂贵的。因此我们不得不考虑是否要将它们存储在内存中（非持久）或存储器（状态变量保持）中。

### 数据位置 {#data-location}

每一种复杂类型，即数组和结构，都有一个额外的批注，即“数据位置（data location）”，关于它是存储在内存还是存储中。根据上下文，总有一个默认值，但是可以通过在类型中添加`存储 storage`或`内存 memory`来覆盖。函数参数（包括返回参数）的默认值在`内存`中，局部变量的默认值在`存储`中，并且状态变量位置被强制在存储中（显然）。

还有第三个数据位置 `calldata` ，它是存储函数参数的一个不可修改的非持久性区域。外部函数的函数入参（不是出参）被强制为 `calldata`，其行为主要与内存相似。

数据位置非常重要，因为它们会改变分配的行为：存储和内存之间的分配以及状态变量（甚至是来自其他状态变量）的分配始终会创建一个独立的副本。对本地存储变量的赋值只能指定一个引用，并且该引用总是指向状态变量，即使后者在此期间被更改。另一方面，从内存存储的引用类型到另一个内存存储的引用类型的分配不会创建副本。

```js
pragma solidity ^0.4.0;

contract C {
    uint[] x; // x 的数据位置在存储中

    // memoryArray 在内存中
    function f(uint[] memoryArray) public {
        x = memoryArray; // 正常，复制整个数组到存储
        var y = x; // 正常，分配指针，y 的数据位置在存储中 
        y[7]; // 返回第八个元素
        y.length = 2; // 通过 y 修改 x
        delete x; // 清空数组 x ，也会修改 y 
        
        // 下面无法工作，它需要在存储中创建一个新的临时/无名数组。
        // 但是存储是静态分配的。
        // y = memoryArray;


        // 这也不正常，因为他将重置一个指针，可没有明显的位置可指向
        // delete y;

        g(x); // 调用 g 函数，传递 x 指针 
        h(x); // 调用 h 函数，在内存中创建一个独立的临时副本
    }

    function g(uint[] storage storageArray) internal {}
    function h(uint[] memoryArray) public {}
}
```

**小结**

+ **强制数据位置:**
   + 外部函数入参： calldata
   + 状态变量： storage

+ **默认数据位置:**
    + 函数入参： memory
    + 所有其他局部变量： storage   

### 数组 {#array}

数组可以有一个编译期固定的大小，也可以是动态的。对于存储数组，元素类型可以是任意的（即其他数组，map 或结构）。对于内存数组，它不能是一个 map 。如果它是一个公共可见函数的参数，则它必须是一个 ABI 类型。

一个固定大小 `k` 和元素类型 `T` 的数组可写成 `T[k]`。动态大小的则写成`T[]`。例如，一个有5个动态`uint`数组的数组是`uint[][5]`(（请注意，与其他一些语言相比，表示方式是颠倒的)。访问第三个动态数组的第2个 uint，可用 `x[2][1]` (索引是基于零的，并且访问与声明方式相反，即 `x[2]` 在类型中从右边降一维)

字节`bytes`和字符串`string`类型的变量是特殊数组。一个 `bytes` 与 `byte[]` 相似，但是紧紧包裹在calldata 中。`string`等价`bytes`，但不允许长度或索引访问(当前)。

因为比较便宜，所以 `bytes` 总是比 `byte[]` 更受欢迎。

{{< note title="Note" >}} 
如果你想访问字符串`s`的字节形式，使用 `bytes(s).length` 或 `bytes(s)[7]='x'`。请记住，此时你访问的是 UTF-8 表示的低级字节，而不是单个字符！
{{< /note >}}

可将数组标记为`public`，Solidity 会创建一个 [getter](http://solidity.readthedocs.io/en/latest/contracts.html#visibility-and-getters)。数字索引将成为getter 函数的不可缺参数。

### 分配内存数组 {#allocatiing-memory-arrays}
在内存中创建可变长度的数组可以使用 `new` 关键字来完成。与存储数组相反，通过赋值给成员`.length` 来调整内存数组大小是不可能的。

```js
pragma solidity ^0.4.16;

contract C {
    function f(uint len) public pure {
        uint[] memory a = new uint[](7);
        bytes memory b = new bytes(len);
        // 这里，a.length 为7，b.length 为 len。

        a[6] = 8;
    }
}
```

### 数组字面量/内联数组 {#array-literanls-inline}

数组字面量是作为表达式写入的数组，且不会离开分配给一个变量。
```js
pragma solidity ^0.4.16;

contract C {
    function f() public pure {
        g([uint(1), 2, 3]);
    }
    function g(uint[3] _data) public pure {
        // ...
    }
}
```
数组字面量的类型是固定大小的内存数组，其元素类型是给定元素的公共类型。 数组`[1,2,3]`的类型是 `uint8[3] memory`，因为这三个常量元素类型均是`uint8`。 因而在示例中，我们有必要将第一个元素转换为 `uint` 类型。注意目前，固定大小的内存数组不能被分配给可变长度内存数组，即以下是不可能的：
```js
// 无法编译。

pragma solidity ^0.4.0;

contract C {
    function f() public {
        // 类型错误，因为 uint[3] 数组不能被转换为 uint[]
        uint[] x = [uint(1), 3, 4];
    }
}
```
计划在将来去除此限制，但是目前由于考虑数组在ABI中如何传递，会产生一些复杂性。

**成员**

+ **length**

数组有一个长度成员来保存元素的数量。动态数组可以通过改变.length成员在存储器中（而不是在内存中）调整大小。当尝试访问当前长度之外的元素时，这不会自动发生。存储器阵列的大小在创建之后是固定的（但是是动态的，即可以依赖运行时参数）。

+ **push**  

动态存储数组和 `bytes`（非`string`）有一个成员方法 `push`。可用于在数组末尾追加元素。函数返回新的数组长度。


{{< warning title="Warning" >}} 
无法在外部函数中使用数组的数组
{{< /warning >}}

{{< warning title="Warning" >}} 
由于 EVM 的局限性，调用外部函数无法返回动态内容，在合约`contract C { function f() returns (uint[]) { ... } }`中，如果 web3.js 调用方法 `f` 将返回些内容，但从 Solidity 调用则无。

现在唯一的解决方法是使用大型静态大小的数组。
{{< /warning >}}

```js
pragma solidity ^0.4.16;

contract ArrayContract {
    uint[2**20] m_aLotOfIntegers;
    // 注意，下面不是动态数组是元素，而是动态数组是元素（即，定长数组长度为2） 
    bool[2][] m_pairsOfFlags;
   
    // newPairs 存储在内存中 ———— 默认的函数入参 
    function setAllFlagPairs(bool[2][] newPairs) public { 
        // 分配一个存储型数组替换此完整数组
        m_pairsOfFlags = newPairs;
    }

    function setFlagPair(uint index, bool flagA, bool flagB) public {
        // 访问不存在的索引，将抛异常
        m_pairsOfFlags[index][0] = flagA;
        m_pairsOfFlags[index][1] = flagB;
    }

    function changeFlagArraySize(uint newSize) public {
        // 如果新长度过小，则将移除末尾元素
        m_pairsOfFlags.length = newSize;
    }

    function clear() public {
        // 这些完全清除数组
        delete m_pairsOfFlags;
        delete m_aLotOfIntegers;
        // 效果与上相同
        m_pairsOfFlags.length = 0;
    }

    bytes m_byteData;

    function byteArrays(bytes data) public {
        // 字节数组 ("bytes") 是不同的，因为他们没有追加存储。
        // 但可以背视为同等的 "uint8p[]"
        m_byteData = data;
        m_byteData.length += 7;
        m_byteData[3] = byte(8);
        delete m_byteData[2];
    }

    function addFlag(bool[2] flag) public returns (uint) {
        return m_pairsOfFlags.push(flag);
    }

    function createMemoryArray(uint size) public pure returns (bytes) {
        // 用 `new` 创建的动态内心型数组 
        uint[2][] memory arrayOfPairs = new uint[2][](size);
        // 创建动态 bytes 数组
        bytes memory b = new bytes(200);
        for (uint i = 0; i < b.length; i++)
            b[i] = byte(i);
        return b;
    }
}
```

### 结构 {#stucts}

Solidity 提供一种以结构形式定义新类型的方法。在下示例中展示：

```js
pragma solidity ^0.4.11;

// 众筹合约
contract CrowdFunding {
    // 定义新类型，有两个字段
    struct Funder {
        address addr;
        uint amount;
    }

    struct Campaign {
        address beneficiary; //发起者，受益人
        uint fundingGoal;//目标额
        uint numFunders;//支持者人数
        uint amount; // 累计募集资金
        mapping (uint => Funder) funders;//支持者具体信息
    }

    uint numCampaigns;
    mapping (uint => Campaign) campaigns;

    // 新建众筹项，指定受益人和目标数
    function newCampaign(address beneficiary, uint goal) public returns (uint campaignID) {
        // ID 递增，并存储众筹项

        campaignID = numCampaigns++; // campaignID  是函数返回变量 
        // 创建新的结构，并保存在存储中。忽略 mapping 类型
        campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
    }

    // 支持项目，将保存支持者信息(地址与捐赠额)
    function contribute(uint campaignID) public payable {
        Campaign storage c = campaigns[campaignID];
        // 创建一个临时内存结构变量，用给定值初始化，并复制到存储器。
        // 注意，也可以使用 Funder(msg.sender, msg.value) 初始化。
        c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
        c.amount += msg.value;
    }

    // 检查众筹项是否到达预期目标并将资金移交给受益人
    function checkGoalReached(uint campaignID) public returns (bool reached) {
        Campaign storage c = campaigns[campaignID];
        if (c.amount < c.fundingGoal)
            return false;
        uint amount = c.amount;
        c.amount = 0;
        c.beneficiary.transfer(amount);
        return true;
    }
}
```
合约没有提供众筹合约的全部功能，但它包含了理解结构所必需的基本概念。结构类型可以用在映射和数组中，它们本身可以包含映射和数组。

尽管结构类型本身可以映射成员的值类型，但结构不可能包含自身类型的成员。这个限制是必须的，因为必须限制结构类型大小。

请注意，在所有函数中，结构类型分配给本地变量（默认存储数据的位置）。这不会拷贝该结构，而只会存储一个引用，以便赋予局部变量的成员实际写入状态中。

### 映射 {#mapping}

用 `mapping(_KeyType => _ValueType)` 声明映射类型。这里 `_KeyType` 几乎可以是任何类型，除映射类型外，动态大小的数组，合约，枚举和结构。`_ValueType` 实际上可以是任何类型，包括映射类型。

映射类型可以看作[哈希表](https://en.wikipedia.org/wiki/Hash_table)，这哈希表被虚拟初始化，使得每个可能的 key 都存在并被映射到其字节表示全部为零的值：类型的默认值。但是，相似性就这些。key 数据实际上并不存储在映射中，只有它的 `keccak256`哈希用来查找值。

映射只能用于状态变量（或作为内部函数中的存储引用类型）。

可以标记映射类型为`public`，Solidity 会创建一个 [getter](http://solidity.readthedocs.io/en/latest/contracts.html#visibility-and-getters)。`_KeyType` 将成为getter 入参，它将返回`_ValueType`。

`_ValueType` 也可以是映射类型，getter 函数将为每个 `_KeyType` 递归地提供一个参数。

```js
pragma solidity ^0.4.0;

contract MappingExample {
    mapping(address => uint) public balances;

    function update(uint newBalance) public {
        balances[msg.sender] = newBalance;
    }
}

contract MappingUser {
    function f() public returns (uint) {
        MappingExample m = new MappingExample();
        m.update(100);
        return m.balances(this);
    }
}
```
{{< note title="Note" >}} 
不能迭代映射，但可以在其上实现一个数据结构，举个列子，见 [迭代 mapping](https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol)
{{< /note >}}


## 左值运算 {#Operators Involving LValues}

如果 `a` 是一个左值（即，变量或一些东西可以分配给它。）以下操作符可简写：

`a += e` 等同 `a = a + e`。相应的由操作符 `-=`、`*=`、`/=`、`%=`、` |=`、`&=`和`^=`。 `a++` 和 `a--` 分别等同 `a += 1` 和 `a -= 1`，但此表达式的计算结果仍然是 a 的旧值。相反，`--a` 和 ` ++ a` 对 a 有相同效果，但在更改后再返回值。

### delete 

`delete a`将类型的初始值赋给 a 。即对于整数，它相当于 `a = 0`，但它也可以用于数组，其中分配一个长度为零的动态数组或一个长度相同的静态数组，其中所有元素都被重置。对于结构体，它分配一个所有成员重置的结构体。

`delete` 对整个映射没影响（因为映射的键可能是任意的且通常是未知的）。如果你删除一个结构体，将重置所有不是映射类型的成员。但是可以删除个别密钥及其映射的内容。 

重要提醒，`delete a` 真实行为就想赋值给 `a`，即存储一个新对象到 `a`。

```js
pragma solidity ^0.4.0;

contract DeleteExample {
    uint data;
    uint[] dataArray;

    function f() public {
        uint x = data;
        delete x; // 设置 x 为 0，不影响 data  
        delete data; // 设置 data 为 0，不会影响仍然拥有副本的 x。
        uint[] storage y = dataArray;
        delete dataArray; // 设置 dataArray 长度为0
        //可 uint[] 作为复杂类型，y 作为存储对象的别名（引用）是受影响的。
        //另一方面 `delete y` 无效，作为局部变量的赋值，
        //只能从现有的存储对象中引用存储对象。 
    }
}
```

## 基础类型类型转换 {#Conversions-between-Elementary-Types}

### 隐式转换 {#Implicit-Conversions}

如果运算符应用于不同的类型，编译器会尝试隐式地将其中一个操作数转换为另一个的类型（对于赋值也是如此）。一般来说，如果语义上有意义并且没有信息丢失，那么值类型之间的隐式转换是可能的：`uint8`可转换为`uint16`，`int128`可转换为`int256`，但是`int8`不转换成`uint256`（因为`uint256`不能全部容纳，例如`-1`）。此外，无符号整数可以转换为相同或更大的 bytes ，反之亦然。任何可以转换为`uint160`的类型也可以转换为`address`。

### 显式转换 {#Explicit-Conversions}

如果编译器不允许隐式转换，但是你清楚自己在做什么，则有时可以使用显式类型转换。注意，这可能会给你一些意想不到的行为，所以一定要测试，以确保结果是你想要的！把下面的例子转换成一个负`int8`为`uint`：

```js
int8 y = -3;
uint x = uint(y);
```
在这个代码段末尾，`x`的值是`0xfffff..fd`（64个十六进制字符），在256位的二进制补码表示中是-3。

如果一个类型被显式地转换为一个较小的类型，那么高位被截断：
```
uint32 a = 0x12345678;
uint16 b = uint16(a); // b 将是 0x5678
```

## 类型断言 {#Type-Deduction}

为方便起见，并不总需要显式指定变量的类型，编译器会自动将其从赋值给变量的第一个表达式的类型中推断出来：

```solidity
uint24 x = 0x123;
var y = x;
```
这里，`y` 将是 `uint24` 类型，不能在函数入参或出参上使用 `var`。

{{< warning title="warning" >}} 
类型只是从第一个赋值中推导出来的，所以下面的代码段会是死循环。因为`i`是`uint8`类型，而此类型的最大值255小于`2000`。
`for (var i = 0; i < 2000; i++) { ... }`

{{< /warning >}}



{{< note title="Note" >}} 
{{< /note >}}















