.. index:: voting, ballot

.. _voting:

********
投票合约
********

以下合约相当复杂，但展示了 Solidity 的许多功能。它实现了一个投票合约。当然，电子投票的主要问题是如何将投票权分配给正确的人以及如何防止操纵 我们不会在这里解决所有问题，但至少我们将展示如何进行委托投票，以便同时 **自动和完全透明** 的投票计数。

我们的想法是为每个选票创建一个合约，为每个选项提供一个简称。
然后担任主席的合约创建者将分别授予每个地址的投票权。

地址后面的人可以选择自己投票或将投票委托给他们信任的人。

在投票时间结束时， ``winingProposal()`` 将返回得票最多的提案。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    /// @title 委托投票
    contract Ballot {
        // 这声明了一个新的复杂类型，将用于稍后的变量。
        // 它将代表一个选民。
        struct Voter {
            uint weight; // 权重，通过委托累计
            bool voted;  // 如果为真，则表明已投票
            address delegate; // 被委托人
            uint vote;   // 投票提案索引
        }

        // 提案的接口类型
        struct Proposal {
            bytes32 name;   // 简称（最多 32 个字节）
            uint voteCount; // 累计票数
        }

        // 主席
        address public chairperson;

        // 这声明了一个状态变量，用于为每个可能的地址存储一个 `Voter`。
        mapping(address => Voter) public voters;

        // 动态大小的 `Proposal`（提案）结构数组。
        Proposal[] public proposals;

        // 创建一个新的投票来选择一个 `proposalNames`。
        constructor(bytes32[] memory proposalNames) {
            chairperson = msg.sender;
            voters[chairperson].weight = 1;

            // 为每个提供的提案名称，创建一个新的提案对象并将其添加到数组的末尾。
            for (uint i = 0; i < proposalNames.length; i++) {
                // `Proposal({...})` 创建了一个临时的 `Proposal` 对象，
                // 并且 `proposals.push(...)` 将它附加到 `proposals` 的末尾。
                proposals.push(Proposal({
                    name: proposalNames[i],
                    voteCount: 0
                }));
            }
        }

        // 给 `voter` (选民) 投票的权利.
        // 只能被 `chairperson`（主席）调用
        function giveRightToVote(address voter) external {
            // 如果 `require` 执行结果为 `false`，则终止执行，回滚对所以状态和以太币余额的变更。
            // 在旧版的EVM中会消耗的所有gas，但现在不会消耗了。
            // 使用 `require` 来检查方法是否被正确调用，通常是一个好主意。
            // 使用 `require` 的第二个参数，您还可以提供有关出错原因的解释。
            require(
                msg.sender == chairperson,
                "只有主席才可以分配投票权。"
            );
            require(
                !voters[voter].voted,
                "选民已经投票了。"
            );
            require(voters[voter].weight == 0);
            voters[voter].weight = 1;
        }

        // 委托你的投票权给投票者 `to`。
        function delegate(address to) external {
            // 分配引用
            Voter storage sender = voters[msg.sender];
            require(!sender.voted, "你已经投票了。");

            require(to != msg.sender, "不允许自行委托。");

            // 如果被委托者 `to` 也委托了，则转发委托。
            // 一般来说，这样的循环非常危险，因为如果它们运行时间过长，可能需要比区块中可用余额更多的费用。
            // 一旦发生币区块余额还多费用的情况，委托将不会被执行，在其他情况下，这样的循环可能会导致合约完全 "卡住"。
            while (voters[to].delegate != address(0)) {
                to = voters[to].delegate;

                // 我们在委托中发现了一个循环，中断执行。
                require(to != msg.sender, "在委托中找到循环。");
            }

            // 因为 `sender` 是一个引用，所以实际会修改 `voters[msg.sender].voted`
            sender.voted = true;
            sender.delegate = to;
            Voter storage delegate_ = voters[to];
            if (delegate_.voted) {
                // 如果被委托者已经投票，则直接增加票数
                proposals[delegate_.vote].voteCount += sender.weight;
            } else {
                // 如果被委托者还没投票，则增加他的权重
                delegate_.weight += sender.weight;
            }
        }

        // 给提案 `proposals[proposal].name` 投票（包括委托给你的投票）。
        function vote(uint proposal) external {
            Voter storage sender = voters[msg.sender];
            require(sender.weight != 0, "Has no right to vote");
            require(!sender.voted, "Already voted.");
            sender.voted = true;
            sender.vote = proposal;

            // 如果 `proposal` 超出数组的范围，这将自动抛出异常并回滚所有更改。
            proposals[proposal].voteCount += sender.weight;
        }

        // @dev 结合之前所有的投票，统计出最终胜出的提案
        function winningProposal() public view
                returns (uint winningProposal_)
        {
            uint winningVoteCount = 0;
            for (uint p = 0; p < proposals.length; p++) {
                if (proposals[p].voteCount > winningVoteCount) {
                    winningVoteCount = proposals[p].voteCount;
                    winningProposal_ = p;
                }
            }
        }

        // 调用 winProposal() 方法获取 proposals数组中获胜者的索引，然后返回获胜者的名字
        function winnerName() external view
                returns (bytes32 winnerName_)
        {
            winnerName_ = proposals[winningProposal()].name;
        }
    }


可能的改进
=====================

目前，需要进行很多交易才能将投票权分配给所有参与者。你能想到更好的方法吗？
