---
title: "Solidity 示例"
date: 2018-01-20T06:30:54+08:00
draft: false
weight: 3004
menu:
    main:
      identifier : ""
      parent: "soliditydoc"
---

## 投票

下面合约十分复杂，但也展示了 Solidity 的许多特性。它实现的是一个投票合约。当然，电子投票的主要问题是如何确保合法投票人权利和避免暗箱操作。这里，我们先不需处理这些问题，先至少能看到如何委托投票，能**自动记票**，同时还是**完全透明**的。

想法是为每次选票创建一个合约，提供投票项简称。合约创建者充当主席，给每个地址分配投票权。地址背后的人类，就可以自行投票或者将投票权委托给他信任的第三方。在投票截止时，`winningProposal`方法返回投票数最多的提案。

```js
pragma solidity ^0.4.16;

/// @title 代表团投票
contract Ballot {
    // 定义一个新的复合类型，会作为合约变量。
    // 代表一名选民。
    struct Voter {
        uint weight; // 累计的投票权重
        bool voted;  // 为 true 说明选名已投票
        address delegate; // 选民委托给第三方
        uint vote;   // 投票提案项
    }

    // 投票单个提案项
    struct Proposal {
        bytes32 name;   // 简称(32字节)
        uint voteCount; // 获票累计数
    }

    address public chairperson;

    // 声明了一个状态变量来存储每个地址所代表的 `Voter` 信息。 
    mapping(address => Voter) public voters;

    // 声明一个 `Proposal` 动态数组.
    Proposal[] public proposals;

    /// 创建一组新的提案进行投票 
    function Ballot(bytes32[] proposalNames) public {
        chairperson = msg.sender;
        voters[chairperson].weight = 1;

        // 遍历每份提案，为其创建一个新的 proposal 对象添加到投票提案组末尾
        for (uint i = 0; i < proposalNames.length; i++) {
            // `Proposal({...})` 创建临时 Proposal 对象
            // `proposals.push(...)` 追加到 proposals 末尾。 
            proposals.push(Proposal({
                name: proposalNames[i],
                voteCount: 0
            }));
        }
    }

    // 给选民分配投票权，只有主席才可操作。
    function giveRightToVote(address voter) public {
        // 如果 `require` 参数计算结果为 `false`，将会终止并回复所有状态和余额变更。
        // 如果方法调用错误，这经常是一个好的处理方式。
        // 但需要小心，这也将消耗 gas (计划在未来调整)。
        require((msg.sender == chairperson) && !voters[voter].voted && (voters[voter].weight == 0));
        voters[voter].weight = 1;
    }

    /// 委托投票权给`to`
    function delegate(address to) public {
        // 获取引用
        Voter storage sender = voters[msg.sender];
        require(!sender.voted);

        // 不允许委托给自己
        require(to != msg.sender);

        // 当 `to` 也委托也委托时：
        // 一般来说，这样遍历是危险的，有可能需要执行很久，所需的 gas 超过区块所能提供的。使得委托不能被执行。
        // 另一种情况是，也许会导致合约执行完全卡住。
        while (voters[to].delegate != address(0)) {
            to = voters[to].delegate;

            // 不允许循环委托
            require(to != msg.sender);
        }

        // 标记为已投票
        sender.voted = true;
        sender.delegate = to;
        Voter storage delegate = voters[to];
        if (delegate.voted) {
            // 如果受托人已投票，则直接累加他的投票权重
            proposals[delegate.vote].voteCount += sender.weight;
        } else { 
            //  如果还没有，则转移受托人的投票权重
            delegate.weight += sender.weight;
        }
    }

    // 给指定提案投票（包括你是受托人）
    function vote(uint proposal) public {
        Voter storage sender = voters[msg.sender];
        require(!sender.voted);
        sender.voted = true;
        sender.vote = proposal;

        // 如果提案不在可选范围（超出索引），将自动抛错，并恢复变更
        proposals[proposal].voteCount += sender.weight;
    }

    /// @dev 计算胜出的提案，并返回所得票数
    function winningProposal() public view
            returns (uint winningProposal)
    {
        uint winningVoteCount = 0;
        for (uint p = 0; p < proposals.length; p++) {
            if (proposals[p].voteCount > winningVoteCount) {
                winningVoteCount = proposals[p].voteCount;
                winningProposal = p;
            }
        }
    }

    // 执行 winningProposal() 方法，获得胜出提案索引后，返回该提案简称。
    function winnerName() public view
            returns (bytes32 winnerName)
    {
        winnerName = proposals[winningProposal()].name;
    }
}
```

### 可改进项
当前，需要进行N多次交易才能将投票权分配给所有参与者。你能想出更好的办法吗？


## 秘密拍卖

## 公开拍卖

## 安全远程买卖

## 小额支付
