---
title: "类型"
date: 2018-01-22T06:41:39+08:00
draft: false
weight: 3053
menu:
    main:
      identifier : ""
      parent: "solidityindepth"
---

Solidity 是一门静态类型语言，意味着每个变量（状态变量和局部变量）类型都需要在编译期确定（或至少是已知的见下方 [类型断言]({{<ref "#Type-Deduction">}})）。 Solidity 提供了几个基础类型，可以组合成复杂类型。

此外，类型可在含运算符的表达式中交互。快速查阅各种运算，请见[运算符优先级](http://solidity.readthedocs.io/en/latest/miscellaneous.html#order)

## 值类型

下面所列类型均可称为值类型，因为它们的变量总是值传递。即当作为方法参数或用作赋值时，总是值拷贝。

### 布尔型 {#booleans}

布尔类型 ` bool `  ，其只可能有的值是常量 ` true `  和 ` false ` 。

运算符有：

+ **!** ： 逻辑否 
+ **&&** ： 逻辑且(and) 
+ **\|\|** ： 逻辑或(or) 
+ **==** ： 等于 
+ **!=** ： 不等于

 ` || ` 和 ` && ` 运算符常用短路求值(short-circuiting)策略。意思是在表达式 ` f(x) || g(y) ` 中，如果 ` f(x) ` 运算结果是 ` true ` ，则 ` g(y) ` 不会被运算，哪怕它有副作用。

{{<adm type="quote" title="Wiki">}}
短路求值（Short-circuit evaluation，又称最小化求值），是一种逻辑运算符的求值策略。只有当第一个运算数的值无法确定逻辑运算的结果时，才对第二个运算数进行求值。例如，当 AND 的第一个运算数的值为 false 时，其结果必定为 false ；当 OR 的第一个运算数为 true 时，最后结果必定为 true ，在这种情况下，就不需要知道第二个运算数的具体值。在一些语言中（如Lisp），默认的逻辑运算符就是短路运算符，而在另一些语言中（如 Java，Ada ），短路和非短路的运算符都存在。对于一些逻辑运算，如 XOR，短路求值是不可能的 。

短路表达式` x AND y `，事实上等价于条件语句：`if x then y else false`。短路表达式`x OR y`，则等价于条件语句：`if x then true else y`。 
{{</adm>}}

### 整数型 {#integers}

整数类型分有符号`int`和无符号`uint`各种不同位大小的整数，从 ` uint8 `  到 ` uint256 `  有 8 个（无符号 8 到 256 位），还有 ` int8 `  到 ` int256 ` 。  ` uint `  和 ` int `  分别是 ` uint256 `  和 ` int256 ` 的别名。

运算操作有：

+ 比较运算： ` <= ` 、 ` < ` 、 ` == ` 、 ` != ` 、 ` >= ` 、 ` > `  (结果为 ` bool `  )
+ 位运算： ` & ` 、 ` | ` 、 ` ^ ` (按位异或)、 ` ~ ` (按位取反)、` << ` （左移）、 ` >> ` （右移）
+ 算术运算： ` + ` 、 ` - ` 、一元 ` - `和` + ` 、 ` * ` 、 ` / ` 、 ` % ` (取余)、 ` ** ` (幂)

整数相除总是截断取整的（它只被编译到 EVM 的 `DIV`操作码），但如果两个数都是有小数（或是小数表达式）时则不会截断。

除以零和零取模会抛出运行时异常。

移位操作的结果是左操作数的类型。表达式 ` x << y ` 相当于 ` x * 2**y`。 ` x >> y`相当于 ` x / 2**y`。这意味着可延伸到负数的位移，位移一个负数金额会抛出运行时异常。


{{< warning title="负值右移" >}}
有符号整数类型的负值右移运算结果和其他编程语言运算结果是不一样的。

在 Solidity 中右移映射到除法，因此位移后的负值将趋向与零（被截断）。在其他编程语言中，右移负数形同相除再四舍五入（趋向负无穷大）。
{{</adm>}}

### 定点数 {#fixed-point-numbers}

{{< adm type="bug" title="未完整支持" >}}
在 Solidity 中定点数尚未完整支持。能够定义它们，但不能被分配。
{{< /adm >}}
 
定点数有 `fixed` 或 `ufixed`，有符合和无符号定点数各有不同位长度数字。`ufixedMxN` 和 `fixedMxN`，这里 `M` 表示该类型所代表的数字位长度，`N` 表示有多少小数点可用。 `M` 必须是从 8 位到 256 位间且能被 8 整除。`N` 必须是 `0` 到 `80` 之间。还包括，`ufixed` 和 `fixed` 分别是 `ufixed128x19` 和 `fixed128x19` 的别名。

运算符：

+ 比较运算： `<=`、`<`、`==`、`!=`、`>=`、`>` (结果为布尔 bool )
+ 算术元素：`+`、`-`、一元`-`和`+`、`*`、`、`%`（取余）


{{< adm type="tip" title="vs 浮点数" >}}
浮点数（在其他许多语言中的`float` 和 `double`，精度更高的 IEEE754 规范）和 定点数的主要区别是，前者是整数部分和小数部分的位长度是灵活的，而在后者中是严格界定的。通常，一般来说，在浮点数中，几乎整个空间都用来表示数字，而只有少量的位长定义了小数点的位置。
{{< /adm >}}

### 地址 {#address}

地址 `address` 是保存 20 个字节的值（以太坊账户地址长度）。地址类型也有成员，是所有合约的基础。

运算符：
+ `<=`、`<`、`==`、`!=`、`>=`和`>` 

{{< adm type="info" >}}
从 0.5.0 版本开始，合约不是从地址类型派生的，但仍然可以明确地将合约转换为地址。
{{< /adm >}}

### 地址成员 {#member-of-address}

+ **余额 `balance` 和 转账 (`transfer`)**

详见 [Address Related](http://solidity.readthedocs.io/en/latest/units-and-global-variables.html#address-related)。是可以使用属性 `balance` 查询地址余额的，且可使用 `transfer` 方法发送 Ether(以 wei 为单位)到一个账户：
```solidity
address x = 0x123;
address myAddress = this;
if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
```

{{<adm type="info" >}}
如果 `x` 是合约地址，它的代码（更确切的说：它的回退方法，如果有）将被和 `transfer` 调用一起执行（这是 EVM 的一个特性，不能被绕开）。如果执行 gas 耗尽或者任何方式的失败，这 Ether 转账将被恢复（注：即回退交易），且合约终止执行。
{{</adm>}}


+ **发送 (`send`)**

发送是转账的低级副本，如果执行失败，当前合约不会停止，但 `send` 将返回 `false`。


{{<adm type="warning" >}}
使用 `send` 是有些危险的，如果执行堆栈深度到了 1024 （caller 强制限制） 则交易失败。同样 gas 耗尽也会失败。所以为了让转账安全，应总检查 `send` 返回值。 使用 `transfer` 也许更好：相当于一种收款人取款的模式。
{{</adm>}}

+ **调用(call)**、**callcode**、**委托调用(delegatecall)**

此外，不遵循 ABI 合约接口，函数 `call` 支持任意类型的任意数量的参数。这些参数被填充到 32 字节并连接起来。一个例外是，第一个参数总会被编码为四个字节。在此情况下，不允许使用函数签名来填充。

```solidity
address nameReg = 0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2;
nameReg.call("register", "MyName");
nameReg.call(bytes4(keccak256("fun(uint256)")), a);
```
`call` 返回一个布尔值，来表明是否被调用方法被终止(true)还是引发了 EVM 异常 (false)。是无法访问调用返回的实际数据（这得需要我们事先知道函数返回值的编码和大小）。

可以用 `.gas()` 修改符来调整所供给的 gas :
```solidity
namReg.call.gas(1000000)("register", "MyName");
```
同样也可调整以太币 value：
```solidity
namReg.call.value(1 ether)("register", "MyName");
```
最后这些修改符可以连接在一起，顺序无关紧要：
```solidity
nameReg.call.gas(1000000).value(1 ether)("register", "MyName");
```
{{<adm type="info" >}}
在重载方法上是无法使用 gas 和 value 修改符的。

解决方案是引入一个 gas 和 value 特例，只需重新检查它们是否存在于重载分析点。
{{</adm>}}

类似地，能使用 `delegatecall` 函数。区别在于只用给定地址的代码，所有其他内容（存储，余额）都从当前合约中获取。`delegatecall`的用意是使用存储在其他合约中的库代码。用户必须确保两个合同中的存储布局适合委托使用。从以太坊的 Prior 版 到 Homestead 版，只有一个名为 `callcode` 的有限变种可用，它不提供对原始 `msg.sender` 和 `msg.value` 值得访问。

这三个函数 `call`、`delegatecall` 和 `callcode` 是非常低级别功能，只能做为最后手段使用，因为他们打破了 Solidity 的类型安全性。

`.gas()` 适用此三个方法，而已 `.value()`不支持 `delegatecall`

{{<adm type="tip">}} 
所有 contact 继承 address 成员，因此可以使用 `this.balance` 查询当前合约余额。
{{</adm>}}


{{<adm type="warning">}} 
不鼓励使用 `callcode` ，将在未来移除它。
{{</adm>}}

{{<adm type="warning">}}
所有这些功能都是低级功能，应谨慎使用。具体而言，任何未知合约都可能是恶意的。如果你调用它，那么将把控制权移交给那个合约，然后再获得返回值到你的合约。以在调用返回时准备好更改状态变量。
{{</adm>}}

### 定长字节数组 {#fixed-size-byte-arrays}

固定长度有`bytes1`、`bytes2`、`bytes3`、...、`bytes32`，`byte` 是 `bytes1`别名。

运算符：

+ 比较运算：`<=`、`<`、`==`、`!=`、`>=`、`>` （结果为 `bool`）
+ 位运算： ` & ` 、 ` | ` 、 ` ^ ` (按位异或)、 ` ~ ` (按位取反)、` << ` （左移）、 ` >> ` （右移）
+ 索引访问： 如果 `x` 是 `bytesI` 类型，则 `x[k]`（0<= k <= I）返回第 `k` 个字节（只读）。 

位移运算符使用任何整数类型作为右操作数（但将返回左操作数的类型），这表示要移位的位数。位移负数将导致运行时异常。

#### 成员

+ `length` 字段返回字节数组的固定长度（只读）。

{{<adm type="tip">}}
可以使用字节数组作为 `byte[]`，但当调用传参时，每个元素占用 31 个字节，它浪费了很多空间。最好使用 `bytes`。
{{</adm>}}


### 可变字节数组 {#dynamically-sized-byte-array}

> 注：英文为：dynamically-sized byte array ，直译为动态字节数组。觉得非常别扭，因此这里翻译为：可变字节数组。

**bytes**：可变字节字节数组，见[数组]({{<ref "#array">}})，非值类型！

**string**：可变字节的 UTF-8 编码字符串，见[数组]({{<ref "#array">}})，非值类型！

根据经验，任意长度的原始 byte 数据使用 `bytes`，任意长途的字符串(UTF-8)数据使用`string`。如果你能限制字节长度到一定数量，优先使用 `bytes1` 到 `bytes32`，因为它们便宜得多。

### 地址字面量 {#address-literals}

通过地址类型校验的十六进制字面量，例如 `0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF` 是一个 `address` 类型。长度在39到41位之间的十六进制字面值，校验不通过会产生警告，并被视为常规的有理数字面值。

{{<adm type="tip">}}
[EIP-55](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md) 定义了混合大小写的地址校验格式。
{{</adm>}}

### 有理数和整数字面量 {#rational-and-integer-literals}

整数字面量由 0 到 9 范围内的一系列数字组成，被视作十进制数。例如 `69` 即六十九。在 Solidity 中不存在八进制字面量，且高位零 (leading zeros)是无效的。
十进制小数字面量由 一个 `.` 组成，至少一边有数。例如：`1.`，`.1`，`1.3`。
也支持科学记数法，基数可以有小数，而指数不能。例如：`2e10`，`-2e10`，`2e-10`，`2.5e1`。

即使中间量不适应机器字长，可`(2**800 + 1) - 2**800` 返回常量 `1`（类型为 `uint8`）。此外，`.5 * 8` 返回整数`4`（虽然之间使用了非整数）。

只要操作数是整数，任何操作整数的运算符也可以使用于数字字面量表达式中。如果两者中的任何一个是小数，则不允许位操作。如果指数是小数（因为这可能导致非有理数），则不能取幂。


{{<adm type="tip">}}
每个有理数都有一个数字字面量类型。整数字面量和有理数字面量均属于数字字面量类型。此外，所有数字字面量表达式（即仅包含数字字面量和运算符的表达式）属于数字字面量类型。所以数字字面量表达式 `1 + 2` 和 `2 + 1` 对于有理数 3 都属于相同的数字字面量类型。
{{</adm>}}


{{<adm type="warning" title="除法结果">}} 
在早期版本中，数字字面量除法是截断的，但现在将转换为**有理数**，即 `5/2` 不等于 `2`，而是等于 `2.5`。
{{</adm>}}


{{<adm type="tip">}}
数字字面量表达式只要与非字面量表达式一起使用，就会转换为非字面量类型。尽管知道下面示例中赋值给 `b` 的表达式值计算结果为一个整数 4，但是表达式 `2.5 + a` 没通过类型检查，所以编译失败。

```solidity
uint128 a = 1;
uint128 b = 2.5 + a + 0.5;
```
```text
TypeError: Operator + not compatible with types rational_const 5/2 and uint128

uint128 b = 2.5 + a + 0.5;
            ^-----^
```                
{{</adm>}}


### 字符串字面量 {#string-literal}

字符串字面量用双引号或单引号括起来（`"foo"`或`'bar'`）。与C语言字符串包含结束符不同，`"foo"`代表三个字节而非四个。与整数字面量一样，他们类型可变化。可隐式转换为 `bytes1`，...... `bytes32`，大小合适可转换为 `bytes` 和 `string`。

字符串字面量支持转义字符，如`\n`、`\xNN`和`\uNNNN`。其中`\xNN`取十六进制值并插入大小合适字节，而 `\uNNNN` 采用 Unicode 码点值，插入一个 UTF-8 序列。


### 十六进制字面量 {#Hexadecimal-liternal}

十六进制字面量带有前缀 `hex` 关键字，并用双引号或单引号括起来(`hex"001122FF"`)。
其内容必须是一个十六进制字符串，其值将用二进制表示。
十六进制字面量行为像字符串字面量，并具有相同的可转换性限制。

### 枚举 {#enum}

枚举是在 Solidity 中创建用户自定义类型的一种方法。可显式地与整数类型互转，但是不允许隐式转换。 显式转换会在运行时检查值范围，失败会导致异常。 

枚举至少需要含一个成员。

```solidity
pragma solidity ^0.4.16;

contract test {
    enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
    ActionChoices choice;
    ActionChoices constant defaultChoice = ActionChoices.GoStraight;

    function setGoStraight() public {
        choice = ActionChoices.GoStraight;
    }

    //因为枚举类型不属于 ABI ，因此对函数 "getChoice" 签名将自动
    //变成对 "getChoice() returns (uint8)" 的签名。
    //在 Soldity 中对于外部处理，返回的整数类型是满足枚举值得最小尺寸。
    //即如果枚举值够大，将被使用 `uint16`。
    function getChoice() public view returns (ActionChoices) {
        return choice;
    }

    function getDefaultChoice() public pure returns (uint) {
        return uint(defaultChoice);
    }
}
```

### 函数类型 {#function-type}

函数类型是函数的类型，函数类型的变量可以由函数赋值，函数类型的入参可以是函数，且函数调用可返回函数。 函数类型有两类：内部(internal)函数和外部(external)函数。

> 注：这里有点儿拗口，即可理解为函数也是一种数据类型，可以像其他数据一样被定义、被当为函数入参和返回值。

因为不能在当前合约上下文之外执行内部函数，内部函数只能在当前合约中调用（即在当前合约代码体中，还包括内部库函数和内部函数）。调用内部函数是通过跳到它的入口标签来实现的，就像在内部调用当前合同的函数一样。

外部函数由一个地址和一个函数签名组成，它们可以由外部函数调用或作为参数传递。


函数类型定义语法如下：
```solidity
function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]
```
与入参相反，返回值不能为空。 如果函数类型无返回值，则 `returns (<return types>)` 部分必须省去。

函数类型默认为内部函数，因此 关键字 `internal` 可省略。相反，合约函数本身默认是公开的，合约函数只有用作类型名时才默认是内部的。

```solidity
pragma solidity ^0.4.0; 
contract action {
    // 定义变量 set，属于函数类型，无返回值。
    function (uint) internal  set;
    // 定义变量 get ，属于函数类型，有返回值。
    function (uint) internal returns(uint)   get;  
}
```

在当前合约中有两种方式访问函数，直接用名称`f`或用`this.f`。前者将按内部函数处理，后者按外部函数处理。

如果一个函数类型变量尚未初始化，调用时将引起异常。同样地，在调用 `delete` 后调用函数也会引起异常。

如果在 Solidity 上下文外部使用外部函数类型，则将它们视为函数类型，将地址连接函数前面一起编码为单个 bytes24 字节。

注意，当前合约的公共函数可作为内部和外部函数使用。仅用`f`，则公共函数 `f` 当做内部函数使用，如果你想使用它的外部形式，则用`this.f`。

另外，公共（或外部）函数都有一个特殊的成员 `selector`，返回 [ABI函数 selector](http://solidity.readthedocs.io/en/latest/abi-spec.html#abi-function-selector)

```solidity
pragma solidity ^0.4.16;

contract Selector {
  function f() public view returns (bytes4) {
    return this.f.selector;
  }
}
```
使用内部函数类型的示例：
```solidity
pragma solidity ^0.4.16;

// 库
library ArrayUtils {
  // 因为在同一个合约代码上下文中，内部函数可作为内部库函数。 
  function map(uint[] memory self, function (uint) pure returns (uint) f) internal pure returns (uint[] memory r)
  {
    // 创建同样长度的 uint 数组
    r = new uint[](self.length);
    // 遍历 self 数组元素，将元素值通过调用方法 `f` 后存入数组 r。
    for (uint i = 0; i < self.length; i++) {
      r[i] = f(self[i]);
    }
  }
  function reduce(uint[] memory self,function (uint, uint) pure returns (uint) f) internal pure returns (uint r)
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
  // 引入库函数
  using ArrayUtils for *;
  function pyramid(uint l) public pure returns (uint) {
    // 初始化一个 uint[] 数组，长度为 l，值为 0....l-1
    // 再进行使用map(square)加工使得值求平方，
    // 最后reduce(sum)获得累计求和值。
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
```solidity
pragma solidity ^0.4.11;

contract Oracle {
  struct Request {
    bytes data;
    // 外部函数类型的字段
    function(bytes memory) external callback;
  }
  Request[] requests;
  event NewRequest(uint);
  // external 标记外外部函数
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

{{<adm  type="info" >}} 
Lambda 表达式和内联函数已计划但尚未支持。 
{{</adm>}}

## 引用类型 {#reference-types}

复杂类型，即不能总是适合 256 位的类型，因为拷贝它们十分昂贵，必须比上面的值类型更仔细地处理。且我们必须得考虑是否要将它们存储在内存中（非持久）还是存储器（状态变量保持）中。

### 数据位置 {#data-location}

每一种复杂类型，即数组和结构，都有一个额外的批注，即“数据位置（data location）”，标记它是存储在内存 memory 还是存储器 storage 中。根据上下文总有一个默认值，但是可以通过在类型中添加`storage`或`memory` 修饰符来覆盖默认值。函数参数（包括返回参数）默认数据位置在内存中，局部变量默认在存储器中，而状态变量位置被强制在存储器中（显然）。

还有第三个数据位置 `calldata` ，它是存储函数参数的一个不可修改的非持久性区域。外部函数的函数入参（不是出参）被强制存储在 `calldata`，其行为主要与内存相似。

数据位置非常重要，因为它们会改变存储分配的行为：存储器与内存之间的分配以及状态变量（甚至是来自其他状态变量）的分配始终会创建一个独立的副本。对局部变量的赋值只能指定一个引用，并且引用总是指向状态变量，即使后者期间被更改。另一方面，从内存存储的引用类型到另一个内存存储的引用类型的分配不会创建副本。

```solidity
pragma solidity ^0.4.0;

contract C {
    uint[] x; // x 存储在存储器中

    // memoryArray 在内存中
    function f(uint[] memoryArray) public {
        x = memoryArray; // 正常，复制整个数组到存储器
        var y = x; // 正常，分配指针，y 存储在存储器中
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

#### 小结

+ **强制数据位置:**
   + 外部函数入参： calldata
   + 状态变量： storage

+ **默认数据位置:**
    + 函数入参： memory
    + 所有其他局部变量： storage   

### 数组 {#array}

数组可以有一个编译期固定的大小，也可以是动态的。对于存储型数组（ storage arrays），元素类型可以是任意的（如其他数组，映射或结构）。对于内存型数组（memory arrays），元素类型不能是[映射类型]({{<ref "#mapping">}}) 。如果它是一个公共可见函数的参数，则元素类型必须是一个 ABI 类型。

一个固定大小 `k` 和元素类型 `T` 的数组可写成 `T[k]`。动态大小的则写成`T[]`。例如，元素为动态 uint 数组的定长 5 的数组是`x uint[][5]`(（请注意，与其他一些语言相比，表示方式是颠倒的)。访问第三个动态数组的第二个 uint，可用 `x[2][1]` (索引是基于零的，并且访问与声明方式相反，即 `x[2]` 在类型中从右边降一维)

字节`bytes`和字符串`string`类型的变量是特殊数组。一个 `bytes` 与 `byte[]` 相似，但是紧紧包裹在calldata 中。`string`等价`bytes`，但不允许长度或索引访问(当前)。

因为比较便宜，所以 `bytes` 总是比 `byte[]` 更受欢迎。

{{<adm type="info" >}} 
如果你想访问字符串`s`的字节形式，使用 `bytes(s).length` 或 `bytes(s)[7]='x'`。请记住，此时你访问的是 UTF-8 表示的码值，而不是单个字符！
{{</adm>}}

可将数组标记为`public`，Solidity 会创建一个 [getter](http://solidity.readthedocs.io/en/latest/contracts.html#visibility-and-getters)。数组索引将成为 getter 函数的入参。

#### 分配内存型数组 {#allocatiing-memory-arrays}
在内存中创建可变长度的数组可以使用 `new` 关键字来完成。与存储型数组不同，不能通过给成员`.length` 赋值来调整内存型数组大小。

```solidity
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

#### 数组字面量/内联数组 {#array-literanls-inline}

数组字面量是作为表达式写入的数组，且不会立刻分配到一个变量。
```solidity
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
数组字面量的类型是固定大小的内存型数组，其元素类型是给定元素的公共类型。 数组`[1,2,3]`的类型是 `uint8[3] memory`，因为这三个常量元素类型均是`uint8`。 因而在示例中，我们有必要将第一个元素转换为 `uint` 类型。注意，目前固定大小的内存型数组不能被分配给动态内存型数组，即以下是不可能的：
```solidity
// 无法编译。

pragma solidity ^0.4.0;

contract C {
    function f() public {
        // 类型错误，因为 uint[3] 数组不能被转换为 uint[]
        uint[] x = [uint(1), 3, 4];
    }
}
```
计划在将来去除此限制，但是目前考虑数组在ABI中传输的情况，修改会产生一些复杂性。

#### 成员 {#memeber}

+ **length**

数组有一个长度成员来记录元素的数量。动态数组可以通过改变.length成员在存储器中（而不是在内存中）调整大小。当尝试访问当前长度之外的元素时，不会自动调整大小。存储型数组大小在创建之后是固定的（但是动态的，可以依赖运行时参数）。

+ **push**  

动态存储型数组和 `bytes`（非`string`）有一个成员方法 `push`。可用于在数组末尾追加元素。函数返回数组新长度。


{{<adm type="warning">}}
无法在外部函数中使用元素类型是数组的数组。
{{</adm>}}

{{<adm type="tip">}}
由于 EVM 的局限性，调用外部函数无法返回动态内容，在合约`contract C { function f() returns (uint[]) { ... } }`中，如果 web3.js 调用方法 `f` 将返回些内容，但从 Solidity 调用则没有。

现在唯一的解决方法是使用大型静态数组。

```solidity
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
{{</adm>}}

### 结构 {#stucts}

Solidity 提供一种以结构（structs）形式定义新类型的方法。在下示例中展示：

```solidity
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
上面合约没有提供众筹合约的全部功能，但它包含了理解结构所必需的基本概念。结构类型可以用在映射和数组中，它们本身可以包含映射和数组。

尽管结构本身可以映射成员的值类型，但结构成员不可能是自身类型。这个限制是必须的，因为必须限制结构大小。

请注意，在所有函数中，结构分配给局部变量（默认数据位置：存储器），这不会拷贝该结构。而只会指向一个引用，以便局部变量可写入状态中。

当然，你也可以直接访问结构的成员，而不必将其分配给局部变量。如：`campaigns[campaignID].amount = 0`

## 映射 {#mapping}

用 `mapping(_KeyType => _ValueType)` 声明映射类型。这里 `_KeyType` 几乎可以是任何类型，除映射类型外，可以是动态数组，合约，枚举和结构。`_ValueType` 实际上可以是任何类型，包括映射类型。

映射类型可以看作[哈希表](https://en.wikipedia.org/wiki/Hash_table)，这哈希表被虚拟初始化，使得每个可能的 key 都存在并被映射到其字节表示全部为零的值：类型的默认值。但相似性也就只有这些。key 数据实际上并不存储在映射中，只用它的 `keccak256`哈希用来查找值。

映射只能用于状态变量（或作为内部函数中的存储引用类型）。

可以标记映射类型为`public`，Solidity 会创建一个 [getter](http://solidity.readthedocs.io/en/latest/contracts.html#visibility-and-getters)。`_KeyType` 将成为 getter 入参，它将返回`_ValueType`。

`_ValueType` 也可以是映射类型，getter 函数将为每个 `_KeyType` 递归地提供一个参数。

```solidity
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
{{<adm type="info">}} 
不能迭代映射，但可以在其上实现一个数据结构，见 [迭代 mapping](https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol) 示例。
{{</adm >}}


## 左值运算 {#Operators Involving LValues}

如果 `a` 是一个左值（即变量或一些东西可以分配给它。）以下操作符可简写：

`a += e` 等同 `a = a + e`。相应的还有操作符 `-=`、`*=`、`/=`、`%=`、` |=`、`&=`和`^=`。 `a++` 和 `a--` 分别等同 `a += 1` 和 `a -= 1`，但此表达式的计算结果仍然是 a 的旧值。相反，`--a` 和 ` ++ a` 对 a 有相同效果，但是是在更改后再返回值（新值）。

### delete 

`delete a`将重置 a，并将类型的初始值赋给 a ，即对于整数它相当于 `a = 0`。它也可以用于数组，其中分配一个长度为零的动态数组或一个长度相同的静态数组，其中所有元素都被重置。对于结构体，它分配一个所有成员重置的结构体。

`delete` 对整个映射没影响（因为映射的键可能是任意的且通常是未知的）。如果你删除一个结构体，所有非映射类型的成员将被重置。但是可以删除个别密钥及其映射的内容。 

重要提醒，`delete a` 真实行为就向`a`赋值，即存储一个新对象到 `a`。

```solidity
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

## 基础类型的类型转换 {#Conversions-between-Elementary-Types}

### 隐式转换 {#Implicit-Conversions}

如果运算符应用于不同的类型，编译器会尝试隐式地将其中一个操作数转换为另一个的类型（对于赋值也是如此）。一般来说，如果语义上有合理且没有信息丢失，那么值类型之间的隐式转换是可能的：`uint8`可转换为`uint16`，`int128`可转换为`int256`，但是`int8`不转换成`uint256`（因为`uint256`不能全部容纳，例如`-1`）。此外，无符号整数可以转换为相同或更大的 bytes ，反之亦然。任何可以转换为`uint160`的类型也可以转换为`address`。

### 显式转换 {#Explicit-Conversions}

如果编译器不允许隐式转换，但是你又清楚自己在做什么，则有时可以使用显式类型转换。注意，这可能会给你一些意想不到的行为，所以一定要测试，以确保结果是你想要的！把下面的例子转换成一个负`int8`为`uint`：

```solidity
int8 y = -3;
uint x = uint(y);
```
在这个代码段末尾，`x`的值是`0xfffff..fd`（64个十六进制字符），在256位的二进制补码表示中是-3。

如果一个类型被显式地转换为一个较小的类型，那么高位被截断：
```
uint32 a = 0x12345678;
uint16 b = uint16(a); // b 将是 0x5678
```

## 类型推导 {#Type-Deduction}

为方便起见，并不总需要显式指定变量的类型，编译器会自动将其从赋值给变量的第一个表达式的类型中推断出来：

```solidity
uint24 x = 0x123;
var y = x;
```
这里 `y` 会是 `uint24` 类型，不能在函数入参或出参上使用 `var`。

{{<adm type="warning">}}
类型只是从第一个赋值中推导出来的，所以下面的代码段会是死循环。因为`i`是`uint8`类型，而此类型的最大值255小于`2000`。`for (var i = 0; i < 2000; i++) { ... }`
{{</adm>}}
 













