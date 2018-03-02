---
title: "合约结构"
date: 2018-01-21T16:54:56+08:00
draft: true
weight: 3052
menu:
    main:
      identifier : ""
      parent: "solidityindepth"
--- 

在 Solidity 中合约类似于面向对象编程语言中的类。每份合约中可定义 [状态变量 State Variables]({{< ref "#state-variables" >}})、[函数 Functions]({{<ref "#functions">}})、[Function Modifiers]({{<ref "#function-modifiers">}})、[事件 Events]({{<ref "#events">}})、[结构类型 Stuct Types]({{<ref "#struct-types">}}) 和 [枚举类型 Enum Types]({{<ref "#enum-types">}})，另外，合约可从其他合约继承。


## 状态变量 {#state-variables}

状态变量（state variables）是永久存储在合约存储器中的值。
```solidity
pragma solidity ^0.4.0;

contract SimpleStorage {
    uint storedData; // State variable
       // ...
}
```
在 [类型]({{<ref "#types.mdtypes">}}) 章节查看有效的状态变量类型。[可见性和可取性]({{<ref "#contract.md">}})供选择变量的可见性。


## 方法 {#functions}

方法是合约中的可执行单元。
```solidity
pragma solidity ^0.4.0;

contract SimpleAuction {
    function bid() public payable { // Function
        // ...
    }
}
```
[函数调用]({{<ref "#function-calls">}})可在内部或外部发生，且对不同其他合约有不同的可见级别( [Visibility and Getters)]({{<ref "#visibility-and-getters.md">}}) 


## 函数修饰符 {#function-modifiers}

函数修饰符可以用来以声明的方式修改方法的语义（详见合约章节的[Function Modifiers]({{<ref "#modifiers">}})）。

```solidity
pragma solidity ^0.4.11;

contract Purchase {
    address public seller;

    modifier onlySeller() { // Modifier
        require(msg.sender == seller);
        _;
    }

    function abort() public onlySeller { // Modifier usage
        // ...
    }
}
```

## 事件 {#events}
事件是记录日志到 EVM（太坊虚拟机）日志组件的便利入口。

```solidity
pragma solidity ^0.4.0;

contract SimpleAuction {
    event HighestBidIncreased(address bidder, uint amount); // Event

    function bid() public payable {
        // ...
        HighestBidIncreased(msg.sender, msg.value); // Triggering event
    }
}
```
查阅合约章节中[事件]({{<ref "#events">}})以了解如何定义事件和如何在 dapp 中使用事件。

## 结构类型 {#struct-types}

结构是自定义类型，能分组多个便利。（在类型章节查阅[结构]({{<ref "#structs">}})）。
```solidity
pragma solidity ^0.4.0;

contract Ballot {
    struct Voter { // Struct
        uint weight;
        bool voted;
        address delegate;
        uint vote;
    }
}
```

## 枚举类型 {#enum-types}

枚举可用来创建具有有限的一组值的自定义类型（在类型章节查阅[枚举]({{<ref "#enums">}})）。

```solidity
pragma solidity ^0.4.0;

contract Purchase {
    enum State { Created, Locked, Inactive } // Enum
}
```




