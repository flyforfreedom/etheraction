---
title: "单位和全局变量"
date: 2018-02-01T06:59:32+08:00
draft: false
weight: 3054
description: "Solidity文档中文版章节《单位和全局变量》"
menu:
    main:
      identifier : ""
      parent: "solidityindepth"
---
  
## 以太币单位 {#Ether-Units}

一个带有 `wei`，`finney`， `szabo` 或 `ether` 后缀的字面量数字，可在以太币各单位间转换。没有后缀的以太币数字假定为 Wei。例如 `2 ether == 2000 finney` 计算结果为 `true`. 

{{<adm type="tip">}}
+ Kwei(Babbage) = 10 ** 3 Wei
+ Mwei(Lovelace) = 10 ** 6 Wei
+ Gwei(Shannon) = 10 ** 9 Wei
+ Microether(Szabo) = 10 ** 12 Wei
+ Milliether(Finney) = 10 ** 15 Wei
+ Ether = 10 ** 18 Wei
{{</adm>}}
 
## 时间单位 {#Time-Units} 

有后缀 `seconds`，`minutes`，`hours`，`days`，`weeks` 或
`years` 的字面量数字，可在时间上互转。时间的基本单位是秒，如下时间为公理：

 * `1 == 1 seconds`
 * `1 minutes == 60 seconds`
 * `1 hours == 60 minutes`
 * `1 days == 24 hours`
 * `1 weeks == 7 days`
 * `1 years == 365 days`

请小心使用这些单位进行日历计算，因为不是每年都有 365 天，甚至因为[闰秒](https：//en.wikipedia.org/wiki/Leap_second)每天不都有 24 小时。由于闰秒无法预测，一个确切的日历库必须由外部预言（external oracle）进行更新。

给变量赋值时，不能使用这些后缀。如果想表达一些输入变量，可如下处理：
```solidity
function f(uint start, uint daysAfter) public {
    if (now >= start + daysAfter * 1 days) {
      // ...
    }
}
```
## 特殊变量和函数 {#Special-Variables-and-Functions} 

在全局空间中的始终存在特殊变量和函数，主要用于提供区块信息。
 

### 区块和交易属性 {#Block-and-Transaction-Properties} 

- `block.blockhash(uint blockNumber) returns (bytes32)`： 给定区块的哈希值—— 只适用于除当前区块外的256个最新的区块。
- `block.coinbase` (`address`)： 当前区块矿工地址 
- `block.difficulty` (`uint`)： 当前区块难度值
- `block.gaslimit` (`uint`)： 当前区块 gas 限制
- `block.number` (`uint`)： 当前区块高度
- `block.timestamp` (`uint`)： 当前区块时间戳，从UNIX新纪元以来的秒数。
- `msg.data` (`bytes`)： 完整的 calldata
- `msg.gas` (`uint`)： 剩余 gas
- `msg.sender` (`address`)： 消息发送者（当前调用者） 
- `msg.sig` (`bytes4`)： calldata 前四个字节（即函数签名）
- `msg.value` (`uint`)： 消息所发送的资产value，单位 wei。
- `now` (`uint`)： 当前区块时间戳(`block.timestamp`别名)
- `tx.gasprice` (`uint`)： 此交易 gas 单价
- `tx.origin` (`address`)： 此交易发送者

{{<adm type="info">}} 
`msg`的所有成员值，包括`msg.sender`和`msg.value` 每次**外部**函数调用时改变，这包括对库函数的调用。 
{{</adm>}}

{{<adm type="info">}} 
除非你知道自己行为，否则不要依赖 `block.timestamp`、`now`和`block.blockhash` 做为随机源。

在一定程度上，时间戳和区块哈希都会受到矿工的影响。例如，执行赌博支付合约，依赖区块哈希时，矿工中的不良派如果他们没有收到任何钱，只需重试不同的散列，已选出符合要求的区块哈希值即可。

而当前区块的时间戳必须严格大于最后一个块的时间戳，但也仅保证是在合法链中的两个连续块的时间戳之间。

{{</adm>}}

{{<adm type="info">}} 
由于所有区块的可伸缩性，不可取所有区块的区块哈希。只能访问最近 256 个区块的哈希值，其他都将为零。 
{{</adm>}}

### 错误处理 {#Error-Handling} 

+ `assert(bool condition)`：如果条件不满足则抛出 - 用于内部错误。 
+ `require(bool condition)`：如果条件不满足则抛出 - 用于输入或外部组件的错误。
+ `revert()`：中止执行并恢复状态。

### 数学和密码学函数 {#Mathematical-and-Cryptographic-Functions} 

#### addmod
`addmod(uint x, uint y, uint k) returns (uint)`：计算 `(x + y) % k` ，其中加法是任意精度，不局限在 `2**256` 内。

#### mulmod
`mulmod(uint x, uint y, uint k) returns (uint)`：计算 `(x * y) % k`，其中乘法是任意精度，不局限在 `2**256` 内。 

#### keccak256
`keccak256(...) returns (bytes32)`：计算[（紧凑）排列](http：//solidity.readthedocs.io/en/latest/abi-spec.html#abi-packed-mode)参数的 Ethereum-SHA-3 (Keccak-256) 哈希值 。

> 注：以太坊使用 keccak-256。它不遵循基于 Keccak 规范的 FIPS-202 标准（于2015年8月完成。）
> 对字符串“testing”进行hash的不同结果：
>
> + Solidity 的 keccak256= 5f16f4c7f149ac4f9510d9cf8cf384038ad348b3bcdc01915f95de12df9d1b02
> + Keccak-256 (原始填充)= 5f16f4c7f149ac4f9510d9cf8cf384038ad348b3bcdc01915f95de12df9d1b02
> + SHA3-256 (标准) = 7f5979fb78f082e8b1c676635db8795c4ac6faba03525fb708cb5fd68fd40c5e
>
> 见 [EIP 59](https：//github.com/ethereum/EIPs/issues/59)和   [stackexchange](http：//ethereum.stackexchange.com/questions/550/which-cryptographic-hash-function-does-ethereum-use)

#### sha256
`sha256(...) returns (bytes32)`：计算（紧凑排列）参数 的 SHA-256。

#### sha3
`sha3(...) returns (bytes32)`： [keccak256]({{<ref "#keccak256">}}) 别名。

#### ripemd160
`ripemd160(...) returns (bytes20)`：计算（紧凑排列）参数 的 RIPEMD-160。

### ecrecover

`ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)`：从椭圆曲线签名中提取与公钥有关的地址，在出错时返回零（[用法示例](https：//ethereum.stackexchange.com/q/1777/222)）。

上面的“紧凑排列(tightly packed)”意思是参数间没有填充连接符的连续排列。意味着以下几个结果完全相同：
```solidity
keccak256("ab", "c")
keccak256("abc")
keccak256(0x616263)
keccak256(6382179)
keccak256(97, 98, 99)
```
如果需要填充则可以使用显式类型转换：`keccak256("\x00\x12")` 同`keccak256(uint16(0x12))`结果一样。
```solidity
function Get() returns(bytes32 a,bytes32 b){
    a = keccak256("\x00\x12");
    b =  keccak256(uint16(0x12));  
}
// output：
// 0x677034980f47f6cb0a55e7d8674ba838c39165afe34da2fc538f695d4950b38e
```
> 注：0x12=18，默认是按 uint8 类型打包。转换为 uint16 打包时前面一个字节使用零填充，同`keccak256(bytes2(0x12))`。

注意，常量将使用能存储他们的最小字节数打包，意味着`keccak256(0) == keccak256(uint8(0))`，
`keccak256(0x12345678) == keccak256(uint32(0x12345678))`.

在**私有 blockchain** 里，你可能在使用 `sha256`、`ripemd160` 或 `ecrecover` 时碰到 "Out-of-Gas" 错误。原因在于这个仅仅是预编译的合约，合约要在他们接到的第一个消息以后才真正的生成（虽然他们的合约代码是硬编码的）。向没有真正生成的合约发送消息是非常昂贵的，这时就会碰到 "Out-of-Gas" 错误。 解决方法是事先把 1wei 发送到各个你当前使用的各个合约上。在官方(主网)或测试网上无此问题。


### 地址相关 {#Address-Related} 

#### balance
`<address>.balance` (`uint256`)： 该[地址]({{<ref "types.md#address" >}})余额，单位 Wei。

#### transfer
`<address>.transfer(uint256 amount)`：发送资金到该地址，单位 Wei。未来将遗弃它。
 
#### send 
`<address>.send(uint256 amount) returns (bool)`：发送资金到该地址，单位 Wei。 

#### call {#Call}
`<address>.call(...) returns (bool)`：调用低级的 `call` ，失败时返回。 false。  

#### callcode {#Callcode}
`<address>.callcode(...) returns (bool)`：调用低级的`callcode`，失败时返回。 

#### delegatecall {#DelegateCall}
`<address>.delegatecall(...) returns (bool)`： 调用低级的`delegatecall`，失败时返回。  

更多信息见[地址类型]({{<ref "types.md#address" >}}。

{{<adm type="warning">}}
使用`send`有些危险：如果调用栈深度到 1024 （强制的） 则转账失败。如果接受者运行时耗尽了 gas ，它也会失败。所以安全起见，与总是检查结果的 `send` 相比，用 `transfer` 更好：收款人收款模式。 
{{</adm>}}

{{<adm type="info">}}
不鼓励使用`callcode`，将来会删除。 
{{</adm>}}


### 合约相关 {#Contract-Related} 

#### this {#This}

`this` (当前合约的类型)：目前的合同，显示转换为[地址]({{<ref "types.md#address" >}})。

#### selfdestruct
`selfdestruct(address recipient)`：销毁当前合约，把资金发送给指定地址。 

#### suicide
`suicide(address recipient)`： 函数 [selfdestruct]({{<ref "#selfdestruct">}}) 别名。 

此外，当前合同的所有函数均可直接调用，包含上面函数。 
