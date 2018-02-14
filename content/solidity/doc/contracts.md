---
title: "合约"
date: 2018-02-06T06:48:06+08:00
draft: false
weight: 3056
description: "solidity文档中文版章节《合约》"
menu:
    main:
      identifier : ""
      parent: "solidityindepth"
---

Solidity 中的合约类似面向对象编程语言中的类，包含持久化数据（状态变量）和可修改这些变量的函数。通过 EVM 方法来调用另一个合约中函数，同时切换上下文使得原合约状态变量不可访问。

## 创建合约 {#creating-contracts}

合同可以通过以太坊交易从“外部”或从合约中创建。IDE 如 [Remix](https://remix.ethereum.org/) 界面操作全流程创建合约。

以太坊中，最好通过 JavaScript API [web3.js](https://github.com/ethereum/web3.js) 编写创建智能合约。当前有 `web3.eth.Contract` API 可创建合约。

当创建合约时，它的构造器（一个名称和合约名相同的函数）会执行一次。构造函数可选，但只允许有一个且不支持重载。

在 EVM 内部，构造函数入参在合约自身代码之后通过 [ABI](http://solidity.readthedocs.io/en/latest/abi-spec.html#abi) 编码，如果使用 [web3.js](https://github.com/ethereum/web3.js) 则不需要关心这些细节。

当合约创建合约时，创建者必须事先清楚被创建合约源代码（和字节码），这意味着不可能循环创建合约。

```solidity
pragma solidity ^0.4.16;

contract OwnedToken {
    // TokenCreator 是下方定义的合约类型 
    // 只要不用于创建新合约，引用
    TokenCreator creator;
    address owner;
    bytes32 name;

    // 此合约构造器将注册创建者和分配名称
    function OwnedToken(bytes32 _name) public {
        // 用状态变量名访问状态变量，而不是用 this.owner，这也适合在函数中 。
        // 也只能调用这些状态变量，因为合约自身尚未创建。 
        owner = msg.sender;
        // We do an explicit type conversion from `address`
        // to `TokenCreator` and assume that the type of
        // the calling contract is TokenCreator, there is
        // no real way to check that.
        creator = TokenCreator(msg.sender);
        name = _name;
    }

    function changeName(bytes32 newName) public {
        // 只有创建者可修改名称，
        // 合约可显示转换为地址，进行地址比较。
        if (msg.sender == address(creator))
            name = newName;
    }

    function transfer(address newOwner) public {
        // 只有当前拥有者可转账到 Token 
        if (msg.sender != owner) return;
        // 询问创建者是否可转账 ，此函数在下方已定义。
        // 如果调用失败（如 gas 耗尽），执行将立即停止。
        if (creator.isTokenTransferOK(owner, newOwner))
            owner = newOwner;
    }
}

contract TokenCreator {
    function createToken(bytes32 name)
       public
       returns (OwnedToken tokenAddress)
    {
        // 创建一个 Token 合约，并返回其地址。 
        // 从 JavaScript 方面来说，返回“地址”类型很简单，
        // 因为这是在 ABI 中可用的常见类型 
        return new OwnedToken(name);
    }

    function changeName(OwnedToken tokenAddress, bytes32 name)  public {
        // 外部类型  `tokenAddress` 就是一个 `address`.
        tokenAddress.changeName(name);
    }

    function isTokenTransferOK(address currentOwner, address newOwner)
        public
        view
        returns (bool ok)
    {
        // 条件检查
        address tokenAddress = msg.sender;
        return (keccak256(newOwner) & 0xff) == (bytes20(tokenAddress) & 0xff);
    }
}
```

## 可见性和可取性 {#Visibility-and-Getters}

因为 Solidity 有两种函数调用：消息调用（不发生 EVM 调用）和外部函数调用。函数和状态变量有四种可见性类型。函数可被指定为 `external`、`public`、`internal`或 `private`，默认为 `public`。 状态变量不能是`external`，默认为 `internal`。

+ **external 外部的**

外部函数是合约接口的一部分，可从其他合约和交易中执行。外部函数`f`不能当做内部函数调用（即调用`f()`错误，`this.f()`正确）。外部函数在接收大量数据时有时更有效率。

+ **internal 内部的**

这些函数和状态变量只能被内部访问（即从当前合约或派生合约），访问无需使用`this`。

+ **public 公共的**

公共函数作为合约接口的一部分，即可被内部调用也可通过消息调用。对于公共状态变量，会自动生成一个取值函数（[见下]("#getter-functions")）。

+ **private 私有的**

私有函数和状态变量只能在当前定义他们的合约中可见，不包括派生合约。

{{<adm type="tip">}}
合约外界都可以看到合约中的所有内容，使用 `private` 只会阻止其他合约访问和修改信息，但在区块链之外仍然可见。
{{</adm>}}

状态变量的可见性标识符在其类型之后给出，函数的可见性标识符在函数入参列表和出参列表之间给出。
```solidity
pragma solidity ^0.4.16;

contract C {

    uint public data;

    function f(uint a) private pure returns (uint b) { return a + 1; }
    function setData(uint a) internal { data = a; }
}
```

下面示例中，`D` 能通过调用`c.getData()`从状态存储中获取`data`值，但无法调用`f`。合约`E`继承自`C`从而可调用`compute()`
```solidity
// 将编译失败

pragma solidity ^0.4.0;

contract C {
    uint private data;

    function f(uint a) private returns(uint b) { return a + 1; }
    function setData(uint a) public { data = a; }
    function getData() public returns(uint) { return data; }
    function compute(uint a, uint b) internal returns (uint) { return a+b; }
}

contract D {
    function readData() public {
        C c = new C();
        uint local = c.f(7); // 错误： member `f` is not visible
        c.setData(3);
        local = c.getData();
        local = c.compute(3, 5); // 错误: member `compute` is not visible
    }
}

contract E is C {
    function g() public {
        C c = new C();
        uint val = compute(3, 5); // 访问内部成员（派生自父合约）
    }
}
```

### 取值函数 Getter {#getter-functions}

编译器自动为所有的 **public** 状态变量分别生成取值函数。如下合约，编译器自动生成一个名为`data`的函数，其函数无入参并返回类型为`uint`的状态变量`data`的值。状态变量的初始化可以在声明时完成。
```solidity
pragma solidity ^0.4.0;

contract C {
    uint public data = 42;
}

contract Caller {
    C c = new C();
    function f() public {
        uint local = c.data();
    }
}
```
取值函数是外部可见的。如果表达式是内部访问（不带`this.`），则按一个状态变量处理。如果是外部访问(带`this.`)则当做一个函数处理。
```solidity
pragma solidity ^0.4.0;

contract C {
    uint public data;
    function x() public {
        data = 3; // 内部访问
        uint val = this.data(); // 外部访问
    }
}
```

来个复杂点的示例：

```solidity
pragma solidity ^0.4.0;

contract Complex {
    struct Data {
        uint a;
        bytes3 b;
        mapping (uint => uint) map;
    }
    mapping (uint => mapping(bool => Data[])) public data;
}
```
它将生成如下形式的取值函数：
```solidity
function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
    a = data[arg1][arg2][arg3].a;
    b = data[arg1][arg2][arg3].b;
}
```
请注意，结构中的映射字段会被省略，因为无法处理映射类型的键。

## 函数修改器 Modifier {#function-modifiers}

很容易使用修改器改变函数行为。例如可在执行该函数之前自动执行一个条件检查。修改器是合约的可继承属性，可被派生合约重写。

```solidity
pragma solidity ^0.4.11;

contract owned {
    function owned() public { owner = msg.sender; }
    address owner;

    // 此合约有定义一个修改器，但未使用。将在派生合约中使用。
    // 使用修改器的函数的函数体会被插入到特殊符号`_;`处。 
    // 
    // 此修饰器意思是只能是合约拥有者才能调用此函数。
    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }
}

contract mortal is owned {
    // 此合约继承自`owned`， `onlyOwner`修饰器用于`close`函数中，
    // 结果是只有合约中存储的 owner 才可调用`close`。
    function close() public onlyOwner {
        selfdestruct(owner);
    }
}

contract priced {
    // 修改器可接受参数
    modifier costs(uint price) {
        if (msg.value >= price) {
            _;
        }
    }
}

contract Register is priced, owned {
    mapping (address => bool) registeredAddresses;
    uint price;

    function Register(uint initialPrice) public { price = initialPrice; }

    // 这里要点是 `payable`关键字，否则该函数将自动拒绝发送给它的以太币。 
    function register() public payable costs(price) {
        registeredAddresses[msg.sender] = true;
    }

    function changePrice(uint _price) public onlyOwner {
        price = _price;
    }
}

contract Mutex {
    bool locked;
    modifier noReentrancy() {
        require(!locked);
        locked = true;
        _;
        locked = false;
    }

    // 此函数被一个锁保护，意思是来自
    // `msg.sender.call`的递归调用不能再次调用`f`。
    function f() public noReentrancy returns (uint) {
        require(msg.sender.call());
        return 7; // 返回 7 但仍然会调用修改器中的`locked = false`
    }
}
```
一个函数上的多个修改器间用逗号分隔，将按给出的顺序依次执行。

{{<adm type="warning">}}
Solidity的较早版本中，有修改器的函数的 `return` 语句的行为不同。
{{</adm>}}

修改器或函数体中显式使用`return`，仅仅是退出当前修改器或函数体，并将返回结果赋值，而执行流程会在修改器中的“_”之后继续执行。

修改器入参可以是任意表达式。函数中引入的所有符号，修改器中均可见。但修改器中引入的符号在函数中不可见（因为它们有可能被重载）。

## 状态常量 {#constant-state-variables}

可用`constant`定义状态变量，此时必须在编译期确定将常量表达式值赋值给此状态变量。不允许表达式访问存储、区块信息（如：`now`、`this.balance`或`block.number`）、执行信息（`msg.gas`）或者调用外部合约。允许表达式改变内存分配，但不允许影响其他内存对象。允许使用内置函数`keccak256`、`sha256`、`ripemd160`、`ecrecover`、`addmod`和`mulmod`（即使它们调用合约外部）。

允许改变内存分配是因为应可构造复杂的对象，例如，查找表。此功能尚未完全可用。

编译器不会为这些常量保留一个存储槽位置，而是都被相应的常量表达式替换（也许会被编译器优化为单个值）。

并非所有类型都可以定义为常量，只支持值类型和字符串。

```solidity
pragma solidity ^0.4.0;

contract C {
    uint constant x = 32**22 + 8;
    string constant text = "abc";
    bytes32 constant myHash = keccak256("abc");
}
```

## 函数 {#functions}

### 视图函数(View Function) {#view-funcation}

函数可被定义为`view`，则说明此函数承诺不会修改合约状态。

以下语句被视为修改合约状态：

1. 写入状态变量。
2. [发送事件](#events)。
3. [创建其他合约]({{<ref "control-structures.md#creating-contracts-via-new">}})。
4. 使用 `selfdestruct`。
5. 通过调用发送以太币。
6. 调用其他未标记为`view`或`pure`的函数。
7. 使用低级别调用。
8. 使用包含某些操作码的内联汇编。

```solidity
pragma solidity ^0.4.16;

contract C {
    function f(uint a, uint b) public view returns (uint) {
        return a * (b + 42) + now;
    }
}
```

{{<adm type="tip">}}
constant 是 view 的一个别名。
{{</adm>}}

{{<adm type="tip">}}
取值函数被标记为 `view`。
{{</adm>}}

{{<adm type="warning">}}
编译器不会强制要求视图函数不修改状态。
{{</adm>}}

### 纯函数(Pure Function) {#pure-function}

函数可被定义为 `pure`，则说明此函数承诺不读取或修改合约状态。

除上面列出的状态修改语句之外，以下内容被认为读取合约状态：

1. 读取状态变量。
2. 访问`this.balance`或`<address>.balance`。
3. 访问`block`、`tx`、`msg`（除`msg.sig`和`msg.data`）的任何成员信息。
4. 调用任何未标记为`pure`的函数。
5. 使用包含某些操作码的内联汇编。

```solidity
pragma solidity ^0.4.16;

contract C {
    function f(uint a, uint b) public pure returns (uint) {
        return a * (b + 42);
    }
}
```

{{<adm type="warning">}}
编译器不会强制要求纯函数不读取状态。
{{</adm>}}

### 后备函数(Fallback Function) {#Fallback-function}

合约有且仅有一个无名函数，此函数无入参和返回值。如果没有任何其他函数与给定的函数标识符匹配（或者根本没有提供数据），它将在对该合约的调用中执行。

另外，此函数在合约收到以太币（且无数据）时执行。为了接收以太币，后备函数必须标记为`payable`。如果不标记，合约不能通过正常交易方式接受以太币。

此情况下，通常只有很少的瓦斯可用于此函数调用 (准确的是 2300 瓦斯)，所以要点是使得后备函数尽可能便宜。请注意，因为每次交易都会收取额外的 21000 瓦斯或更多的费用来进行签名检查，因此调用后备函数的交易比内部调用所需的气体要高得多。

特别是下面的操作会比提供给后备函数可用瓦斯量消耗更多的燃料：

+ 写存储。
+ 创建合约。
+ 调用消耗大量瓦斯的外部函数。
+ 发送以太币。

请确保在部署合同之前彻底测试后备函数，以确保执行成本低于 2300 瓦斯。


{{<adm type="tip">}}
尽管后备函数无入参，但然可以使用`msg.data`来获取随调用提供的 payload 信息。
{{</adm>}}

{{<adm type="warning">}}
合约可直接接收以太币（不需要调用函数，即用`send`或`transfer`），但没有定义后备函数会抛出异常并退还以太币（ Solidiy v0.4.0 开始）。因此你如果希望合约接收以太币，必须实现后备函数。
{{</adm>}}

{{<adm type="warning">}}
没有 payable 后备函数的合约可以接收以太币作为 coinbase 交易的收款人（区块奖励）或作为其他合约销毁时的资产转移目的地。

合约不能对上述这种以太币转移作出反应，因此也不能拒绝。这是因 EVM 和 solidity 设计模式决定了无法解决的问题。

这意味着，合约的 `this.balance` 可以比手工帐总和多（即在后备函数中有更新计数器）。

{{</adm>}}

```solidity
pragma solidity ^0.4.0;

contract Test {
    // 所有发送到此合约的消息都会调用此函数（因为没有其他函数）。
    // 发送以太币到此函合约会出现异常，
    // 因为此后备函数未使用`payable`修饰器 。
    function() public { x = 1; }
    uint x;
}


// 此合约可以接受所有以太币，别人无法追回。
contract Sink {
    function() public payable { }
}

contract Caller {
    function callTest(Test test) public {
        test.call(0xabcdef01); // 哈希不存在，
        // 结果是 test.x 变成 1。

        // 如下不能通过编译
        // 但是一旦有人发送以太币到 test 合约，
        // 交易将失败并被拒绝转移以太币。
        //test.send(2 ether);
    }
}
```

### 函数重载 {#function-overloading}

合约可以有多个同名不同参的函数，包括继承的函数。下面示例展示合约`A`有重载函数`f`。
```solidity
pragma solidity ^0.4.16;

contract A {
    function f(uint _in) public pure returns (uint out) {
        out = 1;
    }

    function f(uint _in, bytes32 _key) public pure returns (uint out) {
        out = 2;
    }
}
```

合约的外部函数也可重载，但只允许重载的外部函数的参数是外部类型。

```solidity
// 无法编译
pragma solidity ^0.4.16;

contract A {
    function f(B _in) public pure returns (B out) {
        out = _in;
    }

    function f(address _in) public pure returns (address out) {
        out = _in;
    }
}

contract B {
}
```
尽管上述两个`f`函数重载在 Solidity 内部认为不同，但最终都接受的是 ABI 的地址类型。

{{<adm type="info">}}
返回参数不作为函数重载标识的一部分。
{{</adm>}}

```solidity
pragma solidity ^0.4.16;

contract A {
    function f(uint8 _in) public pure returns (uint8 out) {
        out = _in;
    }

    function f(uint256 _in) public pure returns (uint256 out) {
        out = _in;
    }
}
```
因为 `250` 即可转换为 `uint8`，也可是`uint256`，所以调用`f(50)`将出现类型错误。另一面，因为 256 无法隐式转换为 uint8 ，调用`f(256)`可转换为`f(uint256)`重载调用。

## 事件 {#events}

事件可方便使用 EVM 日志记录工具，又可在 dapp 的用户界面中“调用”JavaScript 回调监听事件。

事件可继承，当它们被调用时，它们使得入参被存储在事务日志中 —— 区块链上的特殊结构数据。这些日志与合约地址相关联，被整合到区块链中，只要区块可访问就可访问此区块中的日志（ Frontier 版和 Homestead 版永久存储日志，但也许会在 Serenity 版中改变）。但不能在合约中直接访问日志和事件数据（即便是创建日志的合约）。

日志是可以通过简单支付验证（SPV）的，如果外部实体提供证明，它便验证日志确实存在于区块链中。但区块头必须提供，因为合约只能看到最后的 256 个区块。

事件参数可以标记为`indexed`（索引），有且最多三个参数可标记为索引，用做检索过滤主题：可在用户界面中利用索引参数筛选出特定值事件。其他所有非索引参数将被作为日志数据的一部分存储。

如果索引参数是数组（包括字符串和字节类型），则用它的 keccak-256 哈希值作为主题。除非定义事件为 `anonymous`（匿名），否则事件签名的哈希值也是主题之一，这表示无法通过事件名称筛选特定的匿名事件。



{{<adm type="info">}}
索引参数将不会被存储。只能搜索这些值，但不可能自行检索这些值。
{{</adm>}}
```solidity
pragma solidity ^0.4.0;

contract ClientReceipt {
    event Deposit(
        address indexed _from,
        bytes32 indexed _id,
        uint _value
    );

    function deposit(bytes32 _id) public payable {
        // 对此函数的任何调用（包括嵌套循环）都可直接使用 
        // JavaScript API 过滤查询到`Deposit`事件被调用 
        Deposit(msg.sender, _id, msg.value);
    }
}
```
如下所示，在 JavaScript API 中的使用：
```js
var abi = /* 编译器生成的 ABI */;
var ClientReceipt = web3.eth.contract(abi);
var clientReceipt = ClientReceipt.at("0x1234...ab67" /* 地址 */);

var event = clientReceipt.Deposit();

// 监听变化
event.watch(function(error, result){
    // 结果包含各种信息，包括`Deposit`事件传入的参数值。
    if (!error)
        console.log(result);
});

// 或者通过回拨立刻查看
var event = clientReceipt.Deposit(function(error, result) {
    if (!error)
        console.log(result);
});
```

### 日志低阶接口 {#low-level-interface-to-logs}

也可通过函数`log0`、`log1`、`log2`、`log3`和`log4`访问日志机制的底层接口。`logi`需要 i+1 个 `bytes32` 类型的入参，其中第一个参数值作为日志数据的一部分存储，其他参数值作为主题。下面示例和上面示例中的事件调用效果一样：
```solidity
pragma solidity ^0.4.10;

contract C {
    function f() public payable {
        bytes32 _id = 0x420042;
        log3(
            bytes32(msg.value),
            bytes32(0x50cb9fe53daa9737b786ab3646f04d0150dc50ef4e75f59509d83667ad5adb20),
            bytes32(msg.sender),
            _id
        );
    }
}
```
示例中很长的那串十六进制数等于`keccak256("Deposit(address,hash256,uint256)")`，即事件签名的哈希值。

### 理解事件的额外资源 {#additional-resources-for-understanding-events}

+ [JavaScript 文档](https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events)
+ [事件示例](https://github.com/debris/smart-exchange/blob/master/lib/contracts/SmartExchange.sol)
+ [在JavaScript中如何访问日志](https://github.com/debris/smart-exchange/blob/master/lib/exchange_transactions.js)



## 继承(Inheritance) {#Inheritance}

Solidity 通过复制含多态性代码来支持多继承。除非明确给出合约代码，否则所有函数调用都是虚拟的，这表示最远派生方式会被调用。

> 注：如同面向对象编程中继承时，函数调用时从最低层的派生类向上搜索定位所需执行的函数。

当合约从多个合约继承时，区块链上只创建一个合约，并将所有基类合约中的代码复制到当前创建的合约代码中。

继承模式和 [Python](https://docs.python.org/3/tutorial/classes.html#inheritance) 很相似，特别是在多重继承上。

如下示例详细展示使用：

```Solidity
pragma solidity ^0.4.16;

contract owned {
    function owned() { owner = msg.sender; }
    address owner;
}

// 使用 `is` 从其他合约派生，
//派生合约能访问所有非私有成员，包括内部函数和状态变量。
// 但不能使用`this`按外部访问方式调用它们。
contract mortal is owned {
    function kill() {
        if (msg.sender == owner) selfdestruct(owner);
    }
}

// 抽象合约只为了让编译器知晓而提供接口，注意函数是无函数体的。
// 如果派生合约未实现所有接口函数，那么合约只能当做抽象合约接口。 
contract Config {
    function lookup(uint id) public returns (address adr);
}

contract NameReg {
    function register(bytes32 name) public;
    function unregister() public;
 }

// 可多继承，注意 `owned` 也是 `mortal` 的基类，
// 但这里只有单个`owner`实例（像 C++ 的虚拟接口）。 
contract named is owned, mortal {
    function named(bytes32 name) {
        Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
        NameReg(config.lookup(1)).register(name);
    }

    // 函数可以被其他拥有相同函数名和相同数量/类型的入参的其他函数重写。
    // 如果重写函数有不同类型的出参，则会报错。
    // 局部和消息调用均会执行重写函数。
    function kill() public {
        if (msg.sender == owner) {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).unregister();
            // 仍可调用被重写的函数
            mortal.kill();
        }
    }
}

// 如果父合约构造器有入参，则需在头部提供，
// 或者在派生合约的构造器中使用修饰符调用方式提供
// （见下节：基类构造函数参数）
contract PriceFeed is owned, mortal, named("GoldFeed") {
   function updateInfo(uint newInfo) public {
      if (msg.sender == owner) info = newInfo;
   }

   function get() public view returns(uint r) { return info; }

   uint info;
}
```

注意，上面示例中调用 `mortal.kill()` 来“转发”销毁合约请求，这样写是**有问题**的。见如下示例说明：
```solidity
pragma solidity ^0.4.0;

contract owned {
    function owned() public { owner = msg.sender; }
    address owner;
}

contract mortal is owned {
    function kill() public {
        if (msg.sender == owner) selfdestruct(owner);
    }
}

contract Base1 is mortal {
    function kill() public { /* do cleanup 1 */ mortal.kill(); }
}

contract Base2 is mortal {
    function kill() public { /* do cleanup 2 */ mortal.kill(); }
}

contract Final is Base1, Base2 {
}
```
调用`Final.kill()`时，将调用被派生重写的`Base2.kill()`，而绕过 `Base1.kill()`。是因为编译器不知道有`Base1` ，解决办法是使用`super` ：

```solidity
pragma solidity ^0.4.0;

contract owned {
    function owned() public { owner = msg.sender; }
    address owner;
}

contract mortal is owned {
    function kill() public {
        if (msg.sender == owner) selfdestruct(owner);
    }
}

contract Base1 is mortal {
    function kill() public { /* do cleanup 1 */ super.kill(); }
}


contract Base2 is mortal {
    function kill() public { /* do cleanup 2 */ super.kill(); }
}

contract Final is Base1, Base2 {
}
```
如果`Base2`用`super`调用函数，它不再是简单地对一个基类合约函数调用，而是在最终的继承图中依次向上调用每个基类上的这个函数。最终 `Base.kill()` 被调用（注意，最终的继承顺序是 —— 从继承图底层合约开始：Final，Base2，Base1，mortal，owned）。当使用`super`时,被调实际函数即使类型已知但在使用它的类的上下文中是未知的。

## 基类构造函数参数 {#arguments-for-base-constructors}

派生合约给基类合约提供所需参数，有两种方式：

```solidity
pragma solidity ^0.4.0;

contract Base {
    uint x;
    function Base(uint _x) public { x = _x; }
}

contract Derived is Base(7) {
    function Derived(uint _y) Base(_y * _y) public {
    }
}
```
一种是直接在继承列表上（`is Base(7)`），另一种是作为派生合约的构造函数头部修饰符的一部分（`Base(_y * _y)`）。如果构造函数的参数是常量来定义合约行为或描述用的，第一种方式更简单。如果基类的构造函数参数取决于派生合约的构造函数参数，则必须使用第二种方法。如果在上面这个愚蠢的例子中同时使用了这两种方式，那么第二种修饰符风格的优先。

## 多重继承与线性 {#multiple-inheritance-and-linearization}

允许多重继承的编程语言必须处理几个问题。一是[钻石问题](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem)，Solidity 沿用 Python 方式，使用 [C3 线性化](https://zh.wikipedia.org/wiki/C3线性化)，在基类有向无环图中强制使用特定顺序。坏处是过于单一，不允许一些继承关系，而特别关键是`is`后基类书写顺序是直接影响继承顺序的。在下面的代码中，Solidity 会报错：“继承关系不能线性化(Linearization of inheritance graph impossible)”。

```solidity
// 无法编译
pragma solidity ^0.4.0;

contract X {}
contract A is X {}
contract C is A, X {}
```
原因是 `C` 请求 `X` 重写 `A`（通过`A，X`的特定顺序），但是 `A` 自身请求重写 `X`，就形成了无法解决的循环矛盾。

>注：  X -> A ， A -> X 形成环路。必须是有向无环的线性顺序。

书写的简单原则是：从“最基类”到“最多派生”的顺序​​指定基类。

### 多继承后同名处理 {#inheriting-different-kinds-of-members-of-the-same-name}

除状态变量的get函数能重写公共函数外，当继承使得函数和修饰器有相同名称时，视为错误。还包括事件和修饰器同名、函数和事件同名。

## 抽象函数 {#abstract-contracts}

合约函数可缺少函数实现（请注意函数声明头以`;`结束），如下示例：
```solidity
pragma solidity ^0.4.0;

contract Feline {
    function utterance() public returns (bytes32);
}
```
此时合约不能被编译（即使它们包含已实现函数），但它可用于做基类合约：
```solidity
pragma solidity ^0.4.0;

contract Feline {
    function utterance() public returns (bytes32);
}

contract Cat is Feline {
    function utterance() public returns (bytes32) { return "miaow"; }
}
```

注意：如果继承自抽象合约的合约没有通过实现所有函数接口，它本身仍然是抽象合约，无法编译。

## 接口 {#interfaces}

接口同抽象合约类似，除他们不能有任何函数实现，还有如下限制：

+ 不能继承自其它合约和接口。
+ 不能定义构造器。
+ 不能定义变量。
+ 不能定义结构。
+ 不能定义枚举。

在未来，可能会取消某些限制。

接口基本上仅限于合约  ABI 可代表的内容，且要求在 ABI 和界面之间转换时不会丢失任何信息。

接口有自己的关键字`interface`：
```
pragma solidity ^0.4.11;

interface Token {
    function transfer(address recipient, uint amount) public;
}
```
合约能继承其他合约，也可继承接口。

## 库(Library) {#libraries}

库同合约类似，其用途是在特殊地址上部署一次，用`DELEGATECALL`（Homestead版是`CALLCODE`）调用库代码。使得调用库函数时，库代码将在调用合约的上下文中执行，即库代码中的 `this` 指向当前调用合约，尤其是可以访问来自调用合约的存储。

由于库是一个独立的源代码，因此它只能访问调用合约的状态变量（需明确定义状态变量，否则无法命名访问）。库函数只能在不修改状态（即库函数是 `view` 或者 `pure`）情况下被直接调用（不使用`DELEGATECALL`调用），因为库是无状态的。尤其是不可能销毁一个库，除非 Solidity 现有数据类型被丢弃。

库可被视为使用它们的合约的隐含基类合约。它们不会在继承层次结构中显式可见，但对库函数的调用看起来就像调用显式基类的函数（`L.f()`，`L`是库名）。另外，库的`internal`函数对所有合约是可见的，就像库是基类合约一样。当然，对内部函数的调用使用内部调用约定，这意味着所有内部类型都可传递，并且内存类型将通过引用传递并且不会被复制。为了在 EVM 中实现这一点，内部库函数的代码和从其中调用的所有函数将在编译时被拉入到调用合约中，用常规的`JUMP`调用，而非`DELEGATECALL`。

如下示例说明了如何使用库（务必查看[using for](http://solidity.readthedocs.io/en/latest/contracts.html#using-for)获取更高级的示例实现 Set）。

```solidity
pragma solidity ^0.4.16;

library Set {
  // 定义新数据结构，用于在调用合约中存储
  struct Data { mapping(uint => bool) flags; }

  //注意，第一个入参是 `storage` 引用，因此只是一个存储位置指针，无内容传递。
  //是库的一个特殊功能。 
  // 如果函数能看到对象成员，则惯用`self`命名第一个入参。
  function insert(Data storage self, uint value)
      public
      returns (bool)
  {
      if (self.flags[value])
          return false; // 已存在
      self.flags[value] = true;
      return true;
  }

  function remove(Data storage self, uint value)
      public
      returns (bool)
  {
      if (!self.flags[value])
          return false; // 不存在
      self.flags[value] = false;
      return true;
  }

  function contains(Data storage self, uint value)
      public
      view
      returns (bool)
  {
      return self.flags[value];
  }
}

contract C {
    Set.Data knownValues;

    function register(uint value) public {
        // 因为当前合约就是实例，因此不需要特定的实例调用库函数
        require(Set.insert(knownValues, value));
    }
    // 此合约中，可直接访问 knownValues.flags 
}
```

当然，不必遵循这种方式来使用库：它们也可以在不定义结构数据类型的情况下使用。库函数也可以在没有任何存储参数的情况下工作，并且它们可以具有多个存储参考参数并且可在任何位置。

调用 `Set.contains`、`Set.insert`和`Set.remove`都被编译为调用（`DELEGATECALL`）外部合约/库。如果使用库，留意执行的是实际外部函数。`msg.sender`、`msg.value`和`this`仍然在调用中保留，（但是在 Homestead 版本中，因为使用`CALLCODE`，`msg.sender`和`msg.value`值已改变。）

下面示例展示的是如何在库中使用内存类型和内部函数，以实现自定义类型而无外部函数调用开销。

<a id="set-example"></a>
```solidity
pragma solidity ^0.4.16;

library BigInt {
    struct bigint {
        uint[] limbs;
    }

    function fromUint(uint x) internal pure returns (bigint r) {
        r.limbs = new uint[](1);
        r.limbs[0] = x;
    }

    function add(bigint _a, bigint _b) internal pure returns (bigint r) {
        r.limbs = new uint[](max(_a.limbs.length, _b.limbs.length));
        uint carry = 0;
        for (uint i = 0; i < r.limbs.length; ++i) {
            uint a = limb(_a, i);
            uint b = limb(_b, i);
            r.limbs[i] = a + b + carry;
            if (a + b < a || (a + b == uint(-1) && carry > 0))
                carry = 1;
            else
                carry = 0;
        }
        if (carry > 0) {
            // too bad, we have to add a limb
            uint[] memory newLimbs = new uint[](r.limbs.length + 1);
            for (i = 0; i < r.limbs.length; ++i)
                newLimbs[i] = r.limbs[i];
            newLimbs[i] = carry;
            r.limbs = newLimbs;
        }
    }

    function limb(bigint _a, uint _limb) internal pure returns (uint) {
        return _limb < _a.limbs.length ? _a.limbs[_limb] : 0;
    }

    function max(uint a, uint b) private pure returns (uint) {
        return a > b ? a : b;
    }
}

contract C {
    using BigInt for BigInt.bigint;

    function f() public pure {
        var x = BigInt.fromUint(7);
        var y = BigInt.fromUint(uint(-1));
        var z = x.add(y);
    }
}
```
由于编译器无法知道库将在哪里部署，因此这些地址必须由链接器填充到最终的字节码中（见 [使用命令行编译器](http://solidity.readthedocs.io/en/latest/using-the-compiler.html#commandline-compiler)章节如何编译链接器）。如果地址未作为编译器的参数给出，编译后的十六进制代码将包含`__set______`形式的占位符（其中set是库的名称），这40个符合可手工使用库合同地址的十六进制编码替换填充。

与合约相比，库的限制：

+ 无状态变量。
+ 不能继承或被继承。
+ 不能接受以太币。

> 可能会在以后解除这些限制。

## 安全调用库 {#call-protection-for-libraries}

如前所述，如果调用库代码用`CALL`而不是`DELEGATECALL`或`CALLCODE`，则它将会恢复，除非调用函数是视图或纯函数。

EVM 没有提供直接的方式让合约检测它是否使用`CALL`来调用库，但合约可以使用`ADDRESS`指令来找出它当前正在运行的“哪里”。生成的代码将此地址与构造时使用的地址进行比较，以确定调用方式。

更具体地说，库的运行时代码始终以一个推送指令开始，该指令在编译时为20个字节的零。当部署代码运行时，此常量在内存中被当前地址替换，并且此修改后的代码存储在合同中。在运行时，这会导致部署实时地址成为第一个被推入堆栈的常量，调度程序代码会将当前地址与此常量进行比较，以查看任何非视图和非纯函数。

## 使用 For {#using-for}

指令 `using A for B;` 能附加库函数（从库 `A`）到任意类型(`B`)。这些函数将接受对象作为第一个入参。被调用对象将作为这些函数的第一个入参接收（像 Pythone 中的 `self`变量）。

`using A for *`的效果是库`A`的函数被附加到任何类型。

在这两种情况下，都附加了所有函数，即使是第一个参数的类型与对象类型不匹配的函数。将在函数被调用的地方检查类型并执行函数重载解析。

`using A for B` 指令只对于当前作用域有效，该作用域现在仅限于一个合约，但后续将被提升到全局作用域，这样通过包含一个模块使其数据类型（包括库函数）均可用，而无需额外增加代码。

使用 For 重写前面 [set 示例](#set-example)，可用这种方式扩展基本类型：

```solidity
pragma solidity ^0.4.16;

library Search {
    function indexOf(uint[] storage self, uint value)
        public
        view
        returns (uint)
    {
        for (uint i = 0; i < self.length; i++)
            if (self[i] == value) return i;
        return uint(-1);
    }
}

contract C {
    using Search for uint[];
    uint[] data;

    function append(uint value) public {
        data.push(value);
    }

    function replace(uint _old, uint _new) public {
        // This performs the library function call
        uint index = data.indexOf(_old);
        if (index == uint(-1))
            data.push(_new);
        else
            data[index] = _new;
    }
}
```

注意，所有库的调用实际是 EVM 函数调用。意味着如果你传递内存类型或值类型，将会发生拷贝。甚至是`self`变量。仅有的解决办法是使用存储引用变量。











