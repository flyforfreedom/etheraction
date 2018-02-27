---
title: "Soldity 汇编"
date: 2018-02-15T06:16:01+08:00
weight: 3057
description: "solidity文档中文版章节《汇编》"
menu:
    main:
      identifier : ""
      parent: "solidityindepth"
--- 

Solidity 定义了一种汇编语言，可不用Solidity 下使用。也可在 Solidity 源代码中作为“内联汇编”使用。下面开始介绍怎么使用内联汇编和与独立汇编有何不同，并讲解其特殊性。

## 内联汇编 {#inline-assembly}

在 Solidity 语句中交叉使用汇编，以更加接近虚拟机语言，可更细致控制，特别是为通过编写库来增强语言。因为实际上 EVM 为堆栈机，通常很难解决正确的堆栈插槽问题，并为堆栈上正确的操作码提供参数。 Solidity的内联汇编试图通过以下功能来简化手动汇编时出现的问题：

+ 函数式操作码：`mul(1, add(2, 3))`代替`push1 3 push1 2 add push1 1 mul`。
+ 汇编局部变量：`let x := add(2, 3)  let y := mload(0x40)  x := add(x, y)`。
+ 访问外部变量：`function f(uint x) public { assembly { x := sub(x, 1) } }`。
+ 标签：`let x := 10  repeat: x := sub(x, 1) jumpi(repeat, eq(x, 0))`
+ 循环：`for { let i := 0 } lt(i, x) { i := add(i, 1) } { y := mul(2, y) }`。
+ If 语句：`if slt(x, 0) { x := sub(0, x) }`。
+ Switch 语句：`switch x case 0 { y := mul(x, 2) } default { y := 0 }`。
+ 函数调用：`function f(x) -> y { switch x case 0 { y := 1 } default { y := mul(x, f(sub(x, 1))) }   }`。

下面将详细介绍内联汇编。

{{<adm type="warning">}}
内联汇编是低级别操作以太坊虚拟机的一种方式。这抛弃了几个重要的安全特性。
{{</adm>}}
{{<adm type="note">}}
TODO:Write about how scoping rules of inline assembly are a bit different and the complications that arise when for example using internal functions of libraries. Furthermore, write about the symbols defined by the compiler.
{{</adm>}}


### 示例
以下示例提供了库代码来访问另一个合约代码并将其加载到一个字节变量中。这在“原生 Solidity” 中是完全无法做到的，思路是以汇编库代码来增强语言。

```solidity
pragma solidity ^0.4.0;

library GetCode {
    function at(address _addr) public view returns (bytes o_code) {
        assembly {
            // 获取该地址合约代码大小 
            let size := extcodesize(_addr)
            // 分配字节数组，这也可使用 `o_code= new bytes(size)` 非汇编方式处理。 
            o_code := mload(0x40)
            // 存储到包括填充的新“内存末端”
            mstore(0x40, add(o_code, and(add(add(size, 0x20), 0x1f), not(0x1f))))
            // 存储长度到内存
            mstore(o_code, size)
            // 需要用汇编才能获取代码
            extcodecopy(_addr, add(o_code, 0x20), 0, size)
        }
    }
}
```
在优化器无法无法生成高效代码时，用内联汇编是合适的。请注意，由于编译器不执行汇编检查，因此汇编更难编写，因此只有在确实知道所做时才用汇编处理复杂情况。
```solidity
pragma solidity ^0.4.16;

library VectorSum {
    // 此函数性能低效，因为目前编译器无法删除数组边界访问检查。
    function sumSolidity(uint[] _data) public view returns (uint o_sum) {
        for (uint i = 0; i < _data.length; ++i)
            o_sum += _data[i];
    }

    // 应只能在边界内访问数组，可避免检查。
    // 因为数组第一个位置包含数组长度，因此需添加 0x20 到数组中。
    function sumAsm(uint[] _data) public view returns (uint o_sum) {
        for (uint i = 0; i < _data.length; ++i) {
            assembly {
                o_sum := add(o_sum, mload(add(add(_data, 0x20), mul(i, 0x20))))
            }
        }
    }

    // 同上，但完全使用汇编编写。
    function sumPureAsm(uint[] _data) public view returns (uint o_sum) {
        assembly {
           // 从_data中加载前32个字节，并获得值，其值为数组长度。 
           let len := mload(_data)

           // 跳过长度字段。
           //
           // 用临时变量，用于递增。
           //
           // 注意： 递增 _data 后将使得原始 _data 变量不可用。
           let data := add(_data, 0x20)

           // 迭代，直到超边界。
           for
               { let end := add(data, len) }
               lt(data, end)
               { data := add(data, 0x20) }
           {
               o_sum := add(o_sum, mload(data))
           }
        }
    }
}
```

### 语法 {#syntax}

汇编同 Solidity 一样可解析注释、字面量和标识符，因此可用 `//`和`/**/`注释。内联汇编用`assembly { ...... }` 包住，内部可用如下：

+ 字面量，即 `0x123`、`42`或`"abc"`（字符串最多32个字符）。
+ 字节操作码，如： `mload`、`sload`、`dup1`、`sstore`，具体见下列表。
+ 函数指令，如：`add(1,mlod(0))`。
+ 标签，如：`name:`。
+ 定义变量，如：`let x := 7 `、`let x:= add(y, 3)`或`let x`(初始化赋空值(0))。
+ 标识符（标签或局部变量，以及作为内联汇编的外部变量），如：`jump(name)`、`3 x add`。
+ 指令式赋值，如：`3 =: x`。
+ 函数式赋值：如：`x := add(y, 3)`。
+ 定义局部变量的作用域块，如：`{ let x := 3 { let y := add(x, 1) } }`


### 操作码

本节不对以太坊虚拟机完整描述，但可在汇编中参考以下操作码列表。

如果操作码需要参数（总是取自栈顶），用括号给出。注意，参数顺序可看做是非功能样式中的颠倒顺序（解释见下）。操作码列表中用`-`表示不会有操作计算结果推到栈中，用`*`表示特殊的，将推所有项到栈中。

如下，`mem[a...b]`表示将从位置`a`开始到位置`b`(不包含)的内存字节，`storeage[p]`表示存储位置`p`的内容。操作码`pushi`和`jumpdest`不能直接使用。在语法中，操作码被标为预定义的标识符。

| 字节码  | 特殊性 | 含义|
|-------|------|-------|
| stop                    | -    | 停止执行，等同 `return(0,0)`                                      |
| add(x, y)               |      | x + y                                                           |
| sub(x, y)               |      | x - y                                                           |
| mul(x, y)               |      | x * y                                                           |
| div(x, y)               |      | x / y                                                           |
| sdiv(x, y)              |      | x / y，针对二进制补码的有符号数                                     |
| mod(x, y)               |      | x % y                                                           |
| smod(x, y)              |      | x % y，针对二进制补码的有符号数                                     |
| exp(x, y)               |      | x 的 y 次方                                                      |
| not(x)                  |      | ~x, 对 x 的每一位取反                                             |
| lt(x, y)                |      | 如果 x <  y 则为 1, 否则为 0                                      |
| gt(x, y)                |      | 如果 x >  y 则为 1, 否则为 0                                      |
| slt(x, y)               |      | 如果 x <  y 则为 1, 否则为 0，针对二进制补码的有符号数                |
| sgt(x, y)               |      | 如果 x >  y 则为 1, 否则为 0，针对二进制补码的有符号数                 |
| eq(x, y)                |      | 如果 x == y 则为 1, 否则为 0                                       |
| iszero(x)               |      | 如果 x == 0 则为 1, 否则为 0                                           |
| and(x, y)               |      | x &  y 位与                                         |
| or(x, y)                |      | x |  y 位或                                        |
| xor(x, y)               |      | x ^| y 位异或                                       |
| byte(n, x)              |      | x 的第 n 个字节，从 0 开始                                                       |
| addmod(x, y, m)         |      | (x + y) % m 不丢失精确度                                            |
| mulmod(x, y, m)         |      | (x * y) % m 不丢失精确度                                               |
| signextend(i, x)        |      | 符号从（i * 8 + 7）位计数从最低有效位延伸                             |
| keccak256(p, n)         |      | keccak(mem[p...(p+n)))                                          |
| sha3(p, n)              |      | keccak(mem[p...(p+n)))                                          |
| jump(label)             | `-`  | 跳转到标签或代码位置                                                 |
| jumpi(label, cond)      | `-`  | 如果条件为非0，则跳转至标签                                          |
| pc                      |      | 代码当前位置                                                      |
| pop(x)                  | `-`  | 移除由 x 推送的元素                                                  |
| dup1 ... dup16          |      | 复制栈中第 i 个元素到栈顶（从栈顶计数）                                  |
| swap1 ... swap16        | `*`  | 交换栈中第 i 个和栈顶元素                                        |
| mload(p)                |      | mem[p..(p+32))                                                  |
| mstore(p, v)            | `-`  | mem[p..(p+32)) := v                                             |
| mstore8(p, v)           | `-`  | mem[p] := v & 0xff    - 只修改一个字节                            |
| sload(p)                |      | storage[p]                                                      |
| sstore(p, v)            | `-`  | storage[p] := v                                                 |
| msize                   |      | 内存大小，即可访问内存的最大索引                                     |
| gas                     |      | 当前可用 gas                                                     |
| address                 |      | 当前合约或执行上下文的地址                                          |
| balance(a)              |      | 地址 a 的余额，单位 wei                                           |
| caller                  |      | 调用者 (不包括委托调用)                                               |
| callvalue               |      | 发送到当前调用的以太币，单位 wei                                   |
| calldataload(p)         |      | 获取调用数据从位置 p 开始的 32 个字节数据                               |
| calldatasize            |      | 调用数据的大小，以字节表示                                            |
| calldatacopy(t, f, s)   | `-`  | 将调用数据 f 位置开始的 s  个字节复制到内存 t 位置                   |
| codesize                |      | 当前合约或执行上下文的代码长度                                       |
| codecopy(t, f, s)       | `-`  | 将代码 f 位置开始的 s 个字节复制到内存 t 位置                          |
| extcodesize(a)          |      | 地址 a  合约代码长度                                                 |
| extcodecopy(a, t, f, s) | `-`  | 类似 codecopy(t, f, s)，但取自地址 a 的合约代码                      |
| returndatasize          |      | 最后返回数据大小                                                   |
| returndatacopy(t, f, s) | `-`  | 将返回数据 f 位置开始的 s 个字节复制到内存 t 位置                        |
| create(v, p, s)         |      | 使用代码 `mem[p...(p+s)]`创建新合约且发送 v wei 资产，并返回合约代码     | 
| create2(v, n, p, s)     |      | 在指定地址`keccak256(<address> . n . keccak256(mem[p..(p+s)))` <br>上使用代码`mem[p...(p+s)]`创建新合约且发送 v wei 资产，并返回合约代码 |
| call(g, a, v, in,insize, out, outsize)       |      | 使用输入`mem[in..(in+insize))`调用合约 a，并指定 g 燃料量、v wei和 out 输出存放区域。调用成功时 mem[out..(out+outsize)) 为 1，否则为 0 （如 燃料耗尽）       |  
| callcode(g, a, v, in,insize, out, outsize)   |      | 同`call`，但只能使用 a 中的代码，否则将保留在当前合同的上下文中          | 
| delegatecall(g, a, in,insize, out, outsize)  |      | 同 `callcode` 但还保留 `caller`和 `callvalue`  |
| staticcall(g, a, in,insize, out, outsize)     |      | 形同 `call(g, a, 0, in, insize, out, outsize)` 但不允许修改状态 |
| return(p, s)            | `-`  | 结束执行，返回数据 `mem[p..(p+s))`                        |
| revert(p, s)            | `-`  | 结束执行并恢复状态修改，返回数据`mem[p..(p+s))`  |
| selfdestruct(a)         | `-`  | 结束执行并销毁当前合约，将合约资产转移给 a        |
| invalid                 | `-`  | 结束执行并反馈为无效指令                         |
| log0(p, s)              | `-`  | 记录无主题日志 `mem[p..(p+s))`                       |
| log1(p, s, t1)          | `-`  | 在主题 t1 下记录日志 `mem[p..(p+s))`                        |
| log2(p, s, t1, t2)      | `-`  | 在主题 t1、t2 下记录日志 `mem[p..(p+s))`                   |
| log3(p, s, t1, t2, t3)  | `-`  | 在主题 t1、t2、t3 下记录日志 `mem[p..(p+s))`                |
| log4(p, s, t1, t2, t3,t4)  | `-`  | 在主题 t1、t2、t3、t4 下记录日志 `mem[p..(p+s))`            | 
| origin                  |      | 获取交易发送者                                                    |
| gasprice                |      | 获取交易中所设置的燃油单价                                          |
| blockhash(b)            |      | 指定区块 b 的哈希值 - 只能处理除当前区块外的最后 256 个区块             |
| coinbase                |      | 当前挖矿受益人                                                    |
| timestamp               |      | 当前区块的秒级时间戳                                                 |
| number                  |      | 当前区块高度                                                      |
| difficulty              |      | 当前区块难度值                                                    |
| gaslimit                |      | 当前区块燃料限制                                                   |

### 字面量 {#Literals}

可使用十进制或十六进制符号写入整型常量，并自动生成相应的`PUSHI`指令。下面示例中将 2 和 3 加起来得到 5，然后与字符串“abc” 进行位与计算。字符串存储为左对齐，不能超过32个字节。
```solidity
assembly { 2 3 add "abc" and }
```

### 函数式 {#Functional-Style}

可在操作码后面接着输入操作码，同样地也会生成字节码。比如：添加内容`3`到内存`0x80`位置：
```solidity
3 0x80 mload add 0x80 mstore
```
可通常很难看清某些操作码的实际参数是什么，Solidity 内联汇编中提供了"函数式"表示法，上面指令式代码可写成：
```solidity
mstore(0x80, add(mload(0x80), 3))
```
函数式表达式不能用于指令式中，即：`1 2 mstore(0x80, add)` 非法，必须写成 `mstore(0x80, add(2, 1))`。 对于无参操作码，括号可以省略。

注意，参数的顺序在函数式上与指令式方式相反。如果使用函数式，第一个参数将会在栈顶。



### Access to External Variables and Functions 

Solidity variables and other identifiers can be accessed by simply using their name.
For memory variables, this will push the address and not the value onto the
stack. Storage variables are different: Values in storage might not occupy a
full storage slot, so their "address" is composed of a slot and a byte-offset
inside that slot. To retrieve the slot pointed to by the variable ``x``, you
used ``x_slot`` and to retrieve the byte-offset you used ``x_offset``.

In assignments (see below), we can even use local Solidity variables to assign to.

Functions external to inline assembly can also be accessed: The assembly will
push their entry label (with virtual function resolution applied). The calling semantics
in solidity are:

 - the caller pushes ``return label``, ``arg1``, ``arg2``, ..., ``argn``
 - the call returns with ``ret1``, ``ret2``, ..., ``retm``

This feature is still a bit cumbersome to use, because the stack offset essentially
changes during the call, and thus references to local variables will be wrong.

```solidity
pragma solidity ^0.4.11;
contract C {
    uint b;
    function f(uint x) public returns (uint r) {
        assembly {
            r := mul(x, sload(b_slot)) // ignore the offset, we know it is zero
        }
    }
}
```
### Labels 

Another problem in EVM assembly is that ``jump`` and ``jumpi`` use absolute addresses
which can change easily. Solidity inline assembly provides labels to make the use of
jumps easier. Note that labels are a low-level feature and it is possible to write
efficient assembly without labels, just using assembly functions, loops, if and switch instructions
(see below). The following code computes an element in the Fibonacci series.

```solidity
{
    let n := calldataload(4)
    let a := 1
    let b := a
loop:
    jumpi(loopend, eq(n, 0))
    a add swap1
    n := sub(n, 1)
    jump(loop)
loopend:
    mstore(0, a)
    return(0, 0x20)
}
```
Please note that automatically accessing stack variables can only work if the
assembler knows the current stack height. This fails to work if the jump source
and target have different stack heights. It is still fine to use such jumps, but
you should just not access any stack variables (even assembly variables) in that case.

Furthermore, the stack height analyser goes through the code opcode by opcode
(and not according to control flow), so in the following case, the assembler
will have a wrong impression about the stack height at label ``two``:

```solidity
{
    let x := 8
    jump(two)
    one:
        // Here the stack height is 2 (because we pushed x and 7),
        // but the assembler thinks it is 1 because it reads
        // from top to bottom.
        // Accessing the stack variable x here will lead to errors.
        x := 9
        jump(three)
    two:
        7 // push something onto the stack
        jump(one)
    three:
}
```

### Declaring Assembly-Local Variables 

You can use the ``let`` keyword to declare variables that are only visible in
inline assembly and actually only in the current ``{...}``-block. What happens
is that the ``let`` instruction will create a new stack slot that is reserved
for the variable and automatically removed again when the end of the block
is reached. You need to provide an initial value for the variable which can
be just ``0``, but it can also be a complex functional-style expression.

```solidity
pragma solidity ^0.4.16;
contract C {
    function f(uint x) public view returns (uint b) {
        assembly {
            let v := add(x, 1)
            mstore(0x80, v)
            {
                let y := add(sload(v), 1)
                b := y
            } // y is "deallocated" here
            b := add(b, v)
        } // v is "deallocated" here
    }
}
```

### Assignments 

Assignments are possible to assembly-local variables and to function-local
variables. Take care that when you assign to variables that point to
memory or storage, you will only change the pointer and not the data.

There are two kinds of assignments: functional-style and instruction-style.
For functional-style assignments (``variable := value``), you need to provide a value in a
functional-style expression that results in exactly one stack value
and for instruction-style (``=: variable``), the value is just taken from the stack top.
For both ways, the colon points to the name of the variable. The assignment
is performed by replacing the variable's value on the stack by the new value.

```solidity
{
    let v := 0 // functional-style assignment as part of variable declaration
    let g := add(v, 2)
    sload(10)
    =: v // instruction style assignment, puts the result of sload(10) into v
}
```

### If 

The if statement can be used for conditionally executing code.
There is no "else" part, consider using "switch" (see below) if
you need multiple alternatives.

```solidity
{
    if eq(value, 0) { revert(0, 0) }
}
```
The curly braces for the body are required.

### Switch 

You can use a switch statement as a very basic version of "if/else".
It takes the value of an expression and compares it to several constants.
The branch corresponding to the matching constant is taken. Contrary to the
error-prone behaviour of some programming languages, control flow does
not continue from one case to the next. There can be a fallback or default
case called ``default``.

```solidity
{
    let x := 0
    switch calldataload(4)
    case 0 {
        x := calldataload(0x24)
    }
    default {
        x := calldataload(0x44)
    }
    sstore(0, div(x, 2))
}
```
The list of cases does not require curly braces, but the body of a
case does require them.

### Loops 

Assembly supports a simple for-style loop. For-style loops have
a header containing an initializing part, a condition and a post-iteration
part. The condition has to be a functional-style expression, while
the other two are blocks. If the initializing part
declares any variables, the scope of these variables is extended into the
body (including the condition and the post-iteration part).

The following example computes the sum of an area in memory.

```solidity
{
    let x := 0
    for { let i := 0 } lt(i, 0x100) { i := add(i, 0x20) } {
        x := add(x, mload(i))
    }
}
```
For loops can also be written so that they behave like while loops:
Simply leave the initialization and post-iteration parts empty.

```solidity
{
    let x := 0
    let i := 0
    for { } lt(i, 0x100) { } {     // while(i < 0x100)
        x := add(x, mload(i))
        i := add(i, 0x20)
    }
}
```
### Functions 

Assembly allows the definition of low-level functions. These take their
arguments (and a return PC) from the stack and also put the results onto the
stack. Calling a function looks the same way as executing a functional-style
opcode.

Functions can be defined anywhere and are visible in the block they are
declared in. Inside a function, you cannot access local variables
defined outside of that function. There is no explicit ``return``
statement.

If you call a function that returns multiple values, you have to assign
them to a tuple using ``a, b := f(x)`` or ``let a, b := f(x)``.

The following example implements the power function by square-and-multiply.

```solidity
{
    function power(base, exponent) -> result {
        switch exponent
        case 0 { result := 1 }
        case 1 { result := base }
        default {
            result := power(mul(base, base), div(exponent, 2))
            switch mod(exponent, 2)
                case 1 { result := mul(base, result) }
        }
    }
}
```
### Things to Avoid 

Inline assembly might have a quite high-level look, but it actually is extremely
low-level. Function calls, loops, ifs and switches are converted by simple
rewriting rules and after that, the only thing the assembler does for you is re-arranging
functional-style opcodes, managing jump labels, counting stack height for
variable access and removing stack slots for assembly-local variables when the end
of their block is reached. Especially for those two last cases, it is important
to know that the assembler only counts stack height from top to bottom, not
necessarily following control flow. Furthermore, operations like swap will only
swap the contents of the stack but not the location of variables.

### Conventions in Solidity 

In contrast to EVM assembly, Solidity knows types which are narrower than 256 bits,
e.g. ``uint24``. In order to make them more efficient, most arithmetic operations just
treat them as 256-bit numbers and the higher-order bits are only cleaned at the
point where it is necessary, i.e. just shortly before they are written to memory
or before comparisons are performed. This means that if you access such a variable
from within inline assembly, you might have to manually clean the higher order bits
first.

Solidity manages memory in a very simple way: There is a "free memory pointer"
at position ``0x40`` in memory. If you want to allocate memory, just use the memory
from that point on and update the pointer accordingly.

Elements in memory arrays in Solidity always occupy multiples of 32 bytes (yes, this is
even true for ``byte[]``, but not for ``bytes`` and ``string``). Multi-dimensional memory
arrays are pointers to memory arrays. The length of a dynamic array is stored at the
first slot of the array and then only the array elements follow.

{{<adm type="waring">}}
    Statically-sized memory arrays do not have a length field, but it will be added soon
    to allow better convertibility between statically- and dynamically-sized arrays, so
    please do not rely on that.
{{</adm>}}

## Standalone Assembly 

The assembly language described as inline assembly above can also be used
standalone and in fact, the plan is to use it as an intermediate language
for the Solidity compiler. In this form, it tries to achieve several goals:

1. Programs written in it should be readable, even if the code is generated by a compiler from Solidity.
2. The translation from assembly to bytecode should contain as few "surprises" as possible.
3. Control flow should be easy to detect to help in formal verification and optimization.

In order to achieve the first and last goal, assembly provides high-level constructs
like ``for`` loops, ``if`` and ``switch`` statements and function calls. It should be possible
to write assembly programs that do not make use of explicit ``SWAP``, ``DUP``,
``JUMP`` and ``JUMPI`` statements, because the first two obfuscate the data flow
and the last two obfuscate control flow. Furthermore, functional statements of
the form ``mul(add(x, y), 7)`` are preferred over pure opcode statements like
``7 y x add mul`` because in the first form, it is much easier to see which
operand is used for which opcode.

The second goal is achieved by introducing a desugaring phase that only removes
the higher level constructs in a very regular way and still allows inspecting
the generated low-level assembly code. The only non-local operation performed
by the assembler is name lookup of user-defined identifiers (functions, variables, ...),
which follow very simple and regular scoping rules and cleanup of local variables from the stack.

Scoping: An identifier that is declared (label, variable, function, assembly)
is only visible in the block where it was declared (including nested blocks
inside the current block). It is not legal to access local variables across
function borders, even if they would be in scope. Shadowing is not allowed.
Local variables cannot be accessed before they were declared, but labels,
functions and assemblies can. Assemblies are special blocks that are used
for e.g. returning runtime code or creating contracts. No identifier from an
outer assembly is visible in a sub-assembly.

If control flow passes over the end of a block, pop instructions are inserted
that match the number of local variables declared in that block.
Whenever a local variable is referenced, the code generator needs
to know its current relative position in the stack and thus it needs to
keep track of the current so-called stack height. Since all local variables
are removed at the end of a block, the stack height before and after the block
should be the same. If this is not the case, a warning is issued.

Why do we use higher-level constructs like ``switch``, ``for`` and functions:

Using ``switch``, ``for`` and functions, it should be possible to write
complex code without using ``jump`` or ``jumpi`` manually. This makes it much
easier to analyze the control flow, which allows for improved formal
verification and optimization.

Furthermore, if manual jumps are allowed, computing the stack height is rather complicated.
The position of all local variables on the stack needs to be known, otherwise
neither references to local variables nor removing local variables automatically
from the stack at the end of a block will work properly. The desugaring
mechanism correctly inserts operations at unreachable blocks that adjust the
stack height properly in case of jumps that do not have a continuing control flow.

Example:

We will follow an example compilation from Solidity to desugared assembly.
We consider the runtime bytecode of the following Solidity program::
```solidity
pragma solidity ^0.4.16;
contract C {
  function f(uint x) public pure returns (uint y) {
    y = 1;
    for (uint i = 0; i < x; i++)
      y = 2 * y;
  }
}
```
The following assembly will be generated::
```solidity
{
  mstore(0x40, 0x60) // store the "free memory pointer"
  // function dispatcher
  switch div(calldataload(0), exp(2, 226))
  case 0xb3de648b {
    let r := f(calldataload(4))
    let ret := $allocate(0x20)
    mstore(ret, r)
    return(ret, 0x20)
  }
  default { revert(0, 0) }
  // memory allocator
  function $allocate(size) -> pos {
    pos := mload(0x40)
    mstore(0x40, add(pos, size))
  }
  // the contract function
  function f(x) -> y {
    y := 1
    for { let i := 0 } lt(i, x) { i := add(i, 1) } {
      y := mul(2, y)
    }
  }
}
```
After the desugaring phase it looks as follows::
```solidity
{
  mstore(0x40, 0x60)
  {
    let $0 := div(calldataload(0), exp(2, 226))
    jumpi($case1, eq($0, 0xb3de648b))
    jump($caseDefault)
    $case1:
    {
      // the function call - we put return label and arguments on the stack
      $ret1 calldataload(4) jump(f)
      // This is unreachable code. Opcodes are added that mirror the
      // effect of the function on the stack height: Arguments are
      // removed and return values are introduced.
      pop pop
      let r := 0
      $ret1: // the actual return point
      $ret2 0x20 jump($allocate)
      pop pop let ret := 0
      $ret2:
      mstore(ret, r)
      return(ret, 0x20)
      // although it is useless, the jump is automatically inserted,
      // since the desugaring process is a purely syntactic operation that
      // does not analyze control-flow
      jump($endswitch)
    }
    $caseDefault:
    {
      revert(0, 0)
      jump($endswitch)
    }
    $endswitch:
  }
  jump($afterFunction)
  allocate:
  {
    // we jump over the unreachable code that introduces the function arguments
    jump($start)
    let $retpos := 0 let size := 0
    $start:
    // output variables live in the same scope as the arguments and is
    // actually allocated.
    let pos := 0
    {
      pos := mload(0x40)
      mstore(0x40, add(pos, size))
    }
    // This code replaces the arguments by the return values and jumps back.
    swap1 pop swap1 jump
    // Again unreachable code that corrects stack height.
    0 0
  }
  f:
  {
    jump($start)
    let $retpos := 0 let x := 0
    $start:
    let y := 0
    {
      let i := 0
      $for_begin:
      jumpi($for_end, iszero(lt(i, x)))
      {
        y := mul(2, y)
      }
      $for_continue:
      { i := add(i, 1) }
      jump($for_begin)
      $for_end:
    } // Here, a pop instruction will be inserted for i
    swap1 pop swap1 jump
    0 0
  }
  $afterFunction:
  stop
}
```

Assembly happens in four stages:

1. Parsing
2. Desugaring (removes switch, for and functions)
3. Opcode stream generation
4. Bytecode generation

We will specify steps one to three in a pseudo-formal way. More formal
specifications will follow.


### Parsing / Grammar 

The tasks of the parser are the following:

- Turn the byte stream into a token stream, discarding C++-style comments
  (a special comment exists for source references, but we will not explain it here).
- Turn the token stream into an AST according to the grammar below
- Register identifiers with the block they are defined in (annotation to the
  AST node) and note from which point on, variables can be accessed.

The assembly lexer follows the one defined by Solidity itself.

Whitespace is used to delimit tokens and it consists of the characters
Space, Tab and Linefeed. Comments are regular JavaScript/C++ comments and
are interpreted in the same way as Whitespace.

Grammar::
```solidity
    AssemblyBlock = '{' AssemblyItem* '}'
    AssemblyItem =
        Identifier |
        AssemblyBlock |
        AssemblyExpression |
        AssemblyLocalDefinition |
        AssemblyAssignment |
        AssemblyStackAssignment |
        LabelDefinition |
        AssemblyIf |
        AssemblySwitch |
        AssemblyFunctionDefinition |
        AssemblyFor |
        'break' |
        'continue' |
        SubAssembly
    AssemblyExpression = AssemblyCall | Identifier | AssemblyLiteral
    AssemblyLiteral = NumberLiteral | StringLiteral | HexLiteral
    Identifier = [a-zA-Z_$] [a-zA-Z_0-9]*
    AssemblyCall = Identifier '(' ( AssemblyExpression ( ',' AssemblyExpression )* )? ')'
    AssemblyLocalDefinition = 'let' IdentifierOrList ( ':=' AssemblyExpression )?
    AssemblyAssignment = IdentifierOrList ':=' AssemblyExpression
    IdentifierOrList = Identifier | '(' IdentifierList ')'
    IdentifierList = Identifier ( ',' Identifier)*
    AssemblyStackAssignment = '=:' Identifier
    LabelDefinition = Identifier ':'
    AssemblyIf = 'if' AssemblyExpression AssemblyBlock
    AssemblySwitch = 'switch' AssemblyExpression AssemblyCase*
        ( 'default' AssemblyBlock )?
    AssemblyCase = 'case' AssemblyExpression AssemblyBlock
    AssemblyFunctionDefinition = 'function' Identifier '(' IdentifierList? ')'
        ( '->' '(' IdentifierList ')' )? AssemblyBlock
    AssemblyFor = 'for' ( AssemblyBlock | AssemblyExpression )
        AssemblyExpression ( AssemblyBlock | AssemblyExpression ) AssemblyBlock
    SubAssembly = 'assembly' Identifier AssemblyBlock
    NumberLiteral = HexNumber | DecimalNumber
    HexLiteral = 'hex' ('"' ([0-9a-fA-F]{2})* '"' | '\'' ([0-9a-fA-F]{2})* '\'')
    StringLiteral = '"' ([^"\r\n\\] | '\\' .)* '"'
    HexNumber = '0x' [0-9a-fA-F]+
    DecimalNumber = [0-9]+
```

### Desugaring 

An AST transformation removes for, switch and function constructs. The result
is still parseable by the same parser, but it will not use certain constructs.
If jumpdests are added that are only jumped to and not continued at, information
about the stack content is added, unless no local variables of outer scopes are
accessed or the stack height is the same as for the previous instruction.

Pseudocode::
```solidity
    desugar item: AST -> AST =
    match item {
    AssemblyFunctionDefinition('function' name '(' arg1, ..., argn ')' '->' ( '(' ret1, ..., retm ')' body) ->
      <name>:
      {
        jump($<name>_start)
        let $retPC := 0 let argn := 0 ... let arg1 := 0
        $<name>_start:
        let ret1 := 0 ... let retm := 0
        { desugar(body) }
        swap and pop items so that only ret1, ... retm, $retPC are left on the stack
        jump
        0 (1 + n times) to compensate removal of arg1, ..., argn and $retPC
      }
    AssemblyFor('for' { init } condition post body) ->
      {
        init // cannot be its own block because we want variable scope to extend into the body
        // find I such that there are no labels $forI_*
        $forI_begin:
        jumpi($forI_end, iszero(condition))
        { body }
        $forI_continue:
        { post }
        jump($forI_begin)
        $forI_end:
      }
    'break' ->
      {
        // find nearest enclosing scope with label $forI_end
        pop all local variables that are defined at the current point
        but not at $forI_end
        jump($forI_end)
        0 (as many as variables were removed above)
      }
    'continue' ->
      {
        // find nearest enclosing scope with label $forI_continue
        pop all local variables that are defined at the current point
        but not at $forI_continue
        jump($forI_continue)
        0 (as many as variables were removed above)
      }
    AssemblySwitch(switch condition cases ( default: defaultBlock )? ) ->
      {
        // find I such that there is no $switchI* label or variable
        let $switchI_value := condition
        for each of cases match {
          case val: -> jumpi($switchI_caseJ, eq($switchI_value, val))
        }
        if default block present: ->
          { defaultBlock jump($switchI_end) }
        for each of cases match {
          case val: { body } -> $switchI_caseJ: { body jump($switchI_end) }
        }
        $switchI_end:
      }
    FunctionalAssemblyExpression( identifier(arg1, arg2, ..., argn) ) ->
      {
        if identifier is function <name> with n args and m ret values ->
          {
            // find I such that $funcallI_* does not exist
            $funcallI_return argn  ... arg2 arg1 jump(<name>)
            pop (n + 1 times)
            if the current context is `let (id1, ..., idm) := f(...)` ->
              let id1 := 0 ... let idm := 0
              $funcallI_return:
            else ->
              0 (m times)
              $funcallI_return:
              turn the functional expression that leads to the function call
              into a statement stream
          }
        else -> desugar(children of node)
      }
    default node ->
      desugar(children of node)
    }
```
### Opcode Stream Generation 

During opcode stream generation, we keep track of the current stack height
in a counter,
so that accessing stack variables by name is possible. The stack height is modified with every opcode
that modifies the stack and with every label that is annotated with a stack
adjustment. Every time a new
local variable is introduced, it is registered together with the current
stack height. If a variable is accessed (either for copying its value or for
assignment), the appropriate ``DUP`` or ``SWAP`` instruction is selected depending
on the difference between the current stack height and the
stack height at the point the variable was introduced.

Pseudocode::
```solidity
    codegen item: AST -> opcode_stream =
    match item {
    AssemblyBlock({ items }) ->
      join(codegen(item) for item in items)
      if last generated opcode has continuing control flow:
        POP for all local variables registered at the block (including variables
        introduced by labels)
        warn if the stack height at this point is not the same as at the start of the block
    Identifier(id) ->
      lookup id in the syntactic stack of blocks
      match type of id
        Local Variable ->
          DUPi where i = 1 + stack_height - stack_height_of_identifier(id)
        Label ->
          // reference to be resolved during bytecode generation
          PUSH<bytecode position of label>
        SubAssembly ->
          PUSH<bytecode position of subassembly data>
    FunctionalAssemblyExpression(id ( arguments ) ) ->
      join(codegen(arg) for arg in arguments.reversed())
      id (which has to be an opcode, might be a function name later)
    AssemblyLocalDefinition(let (id1, ..., idn) := expr) ->
      register identifiers id1, ..., idn as locals in current block at current stack height
      codegen(expr) - assert that expr returns n items to the stack
    FunctionalAssemblyAssignment((id1, ..., idn) := expr) ->
      lookup id1, ..., idn in the syntactic stack of blocks, assert that they are variables
      codegen(expr)
      for j = n, ..., i:
      SWAPi where i = 1 + stack_height - stack_height_of_identifier(idj)
      POP
    AssemblyAssignment(=: id) ->
      look up id in the syntactic stack of blocks, assert that it is a variable
      SWAPi where i = 1 + stack_height - stack_height_of_identifier(id)
      POP
    LabelDefinition(name:) ->
      JUMPDEST
    NumberLiteral(num) ->
      PUSH<num interpreted as decimal and right-aligned>
    HexLiteral(lit) ->
      PUSH32<lit interpreted as hex and left-aligned>
    StringLiteral(lit) ->
      PUSH32<lit utf-8 encoded and left-aligned>
    SubAssembly(assembly <name> block) ->
      append codegen(block) at the end of the code
    dataSize(<name>) ->
      assert that <name> is a subassembly ->
      PUSH32<size of code generated from subassembly <name>>
    linkerSymbol(<lit>) ->
      PUSH32<zeros> and append position to linker table
    }
```

