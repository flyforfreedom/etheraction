---
title: "表达式与控制结构"
date: 2018-02-04T07:50:30+08:00
draft: false
weight: 3055
description: "Solidity文档中文版章节《表达式与控制结构》"
menu:
    main:
      identifier : ""
      parent: "solidityindepth"
---

## 入参与出参 {#input-parameters-and-output-parameters}

同 JavaScript 相同的是，函数可以把参数作为输入。不同于 JavaScript 和 C 的是也可返回任意数量的参数作为输出。

### 入参 {#input-parameters}

输入参数声明方式与定义变量相同。作为例外，未使用的参数可以省略变量名称。例如，假设希望合约接受具有两个整数的外部调用，会如下写：
```solidity
pragma solidity ^0.4.16;

contract Simple {
    function taker(uint _a, uint _b) public pure {
        // 用 _a 和 _b 做些事情
    }
}
```

### 出参  {#output-parameters}

在 `returns` 关键字后声明出参语法同入参。例如，假设希望返回两个结果：两个给定整数的总和和乘积，那么会写成：

```solidity
pragma solidity ^0.4.16;

contract Simple {
    function arithmetics(uint _a, uint _b)
        public
        pure
        returns (uint o_sum, uint o_product)
    {
        o_sum = _a + _b;
        o_product = _a * _b;
    }
}
```
可忽略出参参数名，输出值也可使用 `return` 语句指定。`return`语句也可返回多个值，见[多值返回]({{<ref "#mult-return">}})。返回参数被初始化为默认值零，如果没有明确赋值则保持为零。

入参和出参可以用在函数体中的表达式中。它们也可以在表达式的左边使用。

## 控制结构 {#control-structures}

在 Solidity 中除 `switch` 和 `goto`外，大部分来自 JavaScript 的控制结构都是可用的。有： `if`、`else`、`while`、`do`、`for`、`break`、`continue`、`return`、`?:`，语义与 C 或 JavaScript 相同。

条件语句中的括号不能省略，但在单条语句前后的花括号可以省略。

注意，Solidity中没有像 C 和 JavaScrip 那样 ，从非布尔类型类型到布尔类型的转换，所以在 Solidity 中`if (1){…}`是非法的。

### 多值返回 {#mult-return}

当函数有多个输出参数时，可用 `return (v0, v1, ...... , vn)`返回多个值。返回值的个数必须和函数出参数量一致。

## 函数调用 {#function-calls}

### 内部函数调用 {#internal-function-calls}
当前合约的函数可以直接（“内部”）调用，也可以递归地调用，如在这个荒谬的例子中所看到的：
```solidity
pragma solidity ^0.4.16;

contract C {
    function g(uint a) public pure returns (uint ret) {
       return f(); 
    }
    function f() internal pure returns (uint ret) { 
      return g(7) + f(); 
    }
}
```
这些函数调用被转换成 EVM 内部的简单跳转,这使得当前内存不会被清除。即将内存引用传递给内部调用的函数是非常有效的。只有同一合约的函数可以在内部调用。

### 外部函数调用 {#external-function-calls}

表达式 `this.g(8)` 和 `c.g(2)`(这里 `c` 指代合约实例)都是有效函数调用。但此时，g 函数被视为“外部(externally)”函数，通过消息调用而非直接跳转调用。 请注意，不能在合约构造器中使用`this`调用函数，因为此时合约还尚未创建。

调用其他合约的函数必须为外部调用，对于一个外部调用，所有函数参数必须被复制到内存中。

当调用其他合约函数时，可用特殊选项`.value()`和`.gas()`来分别指定调用其他合约的Value (单位 Wei) 和 Gas 量。
```solidity
pragma solidity ^0.4.0;

contract InfoFeed {
    function info() public payable returns (uint ret) { return 42; }
}

contract Consumer {
    InfoFeed feed;
    function setFeed(address addr) public { feed = InfoFeed(addr); }
    function callFeed() public { feed.info.value(10).gas(800)(); }
}
```
函数`info` 必须使用修饰符 `payable` ，否则 `.value()` 选项将不可用。

> 注： payable 修饰符表示该函数可接受以太币。

注意，表达式 `InfoFeed（addr）`执行一个显式类型转换，说明“我们知道在给定地址的合约类型是 `InfoFeed”，可这不会执行 InfoFeed 的构造函数。必须谨慎处理显式类型转换，不要在不确定其类型的合约上调用函数。

也可直接使用`function setFeed(InfoFeed _feed) { feed = _feed; }`。请注意`feed.info.value(10).gas(800)`仅（本地）设置函数调用发送 value 和 gas，实际只有最后的括号才执行调用。

如果被调合约不存在（在某种意义上说该帐户不包含合约代码），或者被调合约本身抛出异常或 gas 耗尽，函数调用会导致异常。

{{<adm type="warning">}}
与其他合约的任何交互都会带来潜在的危险，特别是交互前不知合约源代码。当前合约将控制权移交给被调合约，这可能会做任何事情。即使被掉合约继承了已知的父合约，继承合约也只需要有一个正确的接口，而合约的执行却完全是任意的，因而构成危险。

此外，还要做好准备，以防在第一次调用返回之前调用你合约中的其他合约，甚至回到会到被调合约中。这意味着被调合约可以通过其函数改变主调合约的状态变量。编写函数的方式是，例如，在对合约中的状态变量进行任何更改之后再调用外部函数，这样合约就不会受到重入式攻击威胁。
{{</adm>}}



### 命名调用和匿名函数参数 {#named-calls-and-anonymous-function-parameters}
 
如果它们被封装在{}中 函数调用参数也可以按名称给出，可以在下面的例子中看到。参数列表名称必须同函数声明中的参数列表吻合，但可以按任意顺序排列。
```solidity
pragma solidity ^0.4.0;

contract C {
    function f(uint key, uint value) public {
        // ...
    }

    function g() public {
        // 命名参数
        f({value: 2, key: 3});
    }
}
```

### 省略函数参数名 {#omitted-function-parameter-names}

可以省略未使用的参数（尤其是返回参数）的名称。这些参数仍会出现在堆栈上，但是它们不可访问。

```solidity
pragma solidity ^0.4.16;

contract C {
    // 省略参数名
    function func(uint k, uint) public pure returns(uint) {
        return k;
    }
}
```

## 用 `new` 创建合约 {#creating-contracts-via-new}

合约可使用 `new` 关键字创建新合约。必须事先知道创建合约的完整代码，所以递归创建依赖是不可能的。

```solidity
pragma solidity ^0.4.0;

contract D {
    uint x;
    function D(uint a) public payable {
        x = a;
    }
}

contract C {
    D d = new D(4); // 将作为 C 的构造器中的一部分被执行。

    function createD(uint arg) public {
        D newD = new D(arg);
    }

    function createAndEndowD(uint arg, uint amount) public payable {
        // Send ether along with the creation
        D newD = (new D).value(amount)(arg);
    }
}
```
如示例中所见，可以在创建 `D` 合约实例时使用 `.value()`转移以太币，但不可以限制 gas 量。如果创建失败（由于堆栈外的余额不够或其他问题）会引发异常。


## 表达式计算次序 {#order-of-evaluation-of-expressions}
 
表达式的计算顺序是不是确定的（准确地说是, 顺序表达式树中的子节点表达式计算顺序是不确定的的, 但他们对节点本身，计算表达式顺序当然是确定的)。只保证语句执行顺序，以及布尔表达式的短路规则。更多信息见[操作符优先级](http://solidity.readthedocs.io/en/latest/miscellaneous.html#order)。

## 赋值 {#Assignment}

### 析构赋值并返回多个值 {#destructuring-assignments-and-returning-multiple-values}

Solidity 内部允许元组类型，即在编译时大小确定的可能有不同类型对象的列表。那些元组可以同时返回多个值，也可以同时将它们赋值给多个变量（或一般的左值）：
```solidity
pragma solidity ^0.4.16;

contract C {
    uint[] data;

    function f() public pure returns (uint, bool, uint) {
        return (7, true, 2);
    }

    function g() public {
        // 声明变量与赋值，明确指定类型是不可能的。
        var (x, b, y) = f();
        // 变量到已存在的变量。
        (x, y) = (2, 7);
        // 常见技巧交换两者值 -- 不适合非值类型的存储器类型。
        (x, y) = (y, x);
        // 组件可省略（也适合定义变量），
        // 如果元祖结束于空的组件，则其他值被丢弃。  
        (data.length,) = f(); // 丢弃其他组件，设置 data 长度为 7。
        // 同样可在左边，如果元祖以空组件开头，则开头值被丢弃。
        (,data[3]) = f(); // 设置 data[3] 为 2
        //组件只能被分配到左侧，有一个例外： 
        (x,) = (1,);
        // (1,) 是唯一的方式指定只有一个组件的元祖，因为`(1)`等于 1
    }
}
```

### 数组和结构的复杂性 {#complications-for-arrays-and-structs}

赋值的语义对于像数组和结构体这样的非值类型来说要复杂一些。状态变量赋值总是创建一个独立的副本，另一方面，对局部基本类型变量赋值需要创建独立的副本，即适合32字节的静态类型。如果从状态变量将结构或数组（包括字节和字符串）分配给局部变量，本地变量保存对原始状态变量的引用。对局部变量的第二次赋值不会修改状态变量，只会改变局部变量的引用。赋值给局部变量的成员（或元素）会改变原状态变量。


## 变量声明与作用域 {#scoping-and-declarations}

声明变量后将具有初始缺省值，其字节表示全部为零。变量的“默认值”是任何类型的典型“零状态”。比如，`bool`的默认值为 `false`，`unit`或`int`类型的默认值为`0`。对于定长数组和 bytes1 到 bytes32，每个单独的元素将被初始化为对应于其类型的默认值。最后，对于动态数组，bytes 和 string ，默认值是一个空数组或字符串。

一个在函数内部任意位置声明的变量的作用域将是整个函数范围内，而不管在哪声明。出现这种情况是因为 Solidity 从 JavaScript 继承了它的作用域规则。这与许多语言形成对比，在这些语言中，变量作用域只有在变量声明位置到语义块结尾处。结果是下面的代码非法，导致编译器抛出一个错误 `Identifier already declared`：
```solidity
// 这将无法编译

pragma solidity ^0.4.16;

contract ScopingErrors {
    function scoping() public {
        uint i = 0;

        while (i++ < 1) {
            uint same1 = 0;
        }

        while (i++ < 2) {
            uint same1 = 0;// 非法，第二次申明 same1
        }
    }

    function minimalScoping() public {
        {
            uint same2 = 0;
        }

        {
            uint same2 = 0; // 非法，第二次申明 same2
        }
    }

    function forLoopScoping() public {
        for (uint same3 = 0; same3 < 1; same3++) {
        }

        // 非法，第二次申明 same3
        for (uint same3 = 0; same3 < 1; same3++) {
        }
    }
}
```
除此之外，在函数的开头会将声明的变量初始化为默认值。下面的代码是合法的，尽管写得不好：
```solidity
pragma solidity ^0.4.0;

contract C {
    function foo() public pure returns (uint) {
        // foo 隐式地初始化为默认值 0 
        uint bar = 5;
        if (true) {
            bar += foo;
        } else {
            uint foo = 10;// 永远不会执行
        }
        return bar;// 返回 5
    }
}
```

## 错误处理（Assert, Require, Revert 和 Exceptions） {#error-handling-assert-require-revert-and-exceptions}

Solidity 使用状态恢复异常来处理错误。这样的异常将撤消对当前调用（及其所有子调用）中的状态所做的所有更改，并且还向调用者标记错误。便捷函数 `assert`和 `require`可用于条件检查并在条件不满足时抛出异常。函数 `assert` 应只用于测试内部错误和检查不变量。函数 `rqquire` 应该被用来确保有效条件，例如输入或合约状态变量被满足，或者验证调用外部合同返回值。如果使用得当，分析工具可以评估合约以识别达到失败 `assert` 的条件和函数调用。正确运行的代码不应该达到失败的断言语句，如果发生这种情况，应该修复合同中的错误。

还有其他两种方法可以触发异常情况：`revert` 函数可用来标记错误并恢复当前调用。将来还可能在调用 `rever`中包含有关错误的详细信息。关键字 `throw`也可用作`revert()`替代方式。

{{<adm type="info">}}
从 0.4.13 版本开始，`throw` 关键字被弃用，将来会被淘汰。
{{</adm>}}

当子调用发生异常时，会自动“冒泡”（即重抛出异常）。这条规则的例外是 `send` 和低级函数 `call`、`delegatecall`和`callcode`，他们在发送异常而非冒泡情况下返回`false`。

{{<adm type="warning">}}
如同调用的账户不存在，作为 EVM 设计的一部分，低级函数 `call`、`delegatecall`和`callcode` 将返回成功。 如果需要，必须在调用前检查账户的存在性。
{{</adm>}}

现在还不可以捕获异常。

下面示例中，可以看到如何使用`require`来轻松检查输入的条件以及`assert`如何用于内部错误检查：
```solidity
pragma solidity ^0.4.0;

contract Sharer {
    function sendHalf(address addr) public payable returns (uint balance) {
        require(msg.value % 2 == 0); // 只允许偶数 
        uint balanceBeforeTransfer = this.balance;
        addr.transfer(msg.value / 2);
        // Since transfer throws an exception on failure and
        // cannot call back here, there should be no way for us to
        // still have half of the money.
        assert(this.balance == balanceBeforeTransfer - msg.value / 2);
        return this.balance;
    }
}
```
以下情况下会生成`assert`型异常：

1. 访问一个数组的索引太大或为负数（即 `x[i]`中 `i >= x.length`或`i < 0 `）。
2. 访问定常的`bytesN`索引太大或为负数。
3. 除以零或零取模（即：` 5 / 0`或`23 % 0`）。
4. 转移一个负数金额。
5. 转换为枚举时值太大或为负数。
6. 调用内部函数类型的零初始化变量。
7. 调用`assert`参数计算结果为 false。

下面情况会生成`require`型异常：

1. 调用 `throw`
2. 调用 `require` 参数计算结果为 false。
3. 除非使用低级函数 `call`、`delegatecall`和`callcode`，通过消息调用来调用一个函数，但不能正确完成（即 gas 耗尽、无匹配函数、被调用函数抛出异常）。低级操作不会抛出异常，而是通过返回 false 来指示失败。
4. 使用`new`创建合约，但是合约没有正确创建完成（见下“not finish properly”的定义）。
5. 对不包含代码的合约进行外部函数调用。
6. 没有`payable`修饰符的合约公共函数由接收到以太币（包括构造函数和回退函数）。
7. 合约通过公共的 `getter` 函数接受到以太币。
8. `.transfer()`失败

Solidity 内部为require式异常执行恢复操作（指令0xfd），并执行一个无效操作（指令0xfe）来引发一个assert式异常。在这两种情况下，都会导致 EVM 恢复对状态所做的所有更改。恢复的原因是因为预期效果没有发生，没有安全的方法来继续执行。因为我们想要保持交易的原子性，最安全的做法就是恢复所有的变更，使整个交易（至少是调用）无效。

注意，从Metropolis 版本开始，assert 型异常消耗所有可用于调用的气体，而 require 型异常不会消耗任何气体。











