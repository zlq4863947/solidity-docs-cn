.. index:: voting, ballot

.. _voting:

******
投票合约
******

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

            // Forward the delegation as long as
            // `to` also delegated.
            // In general, such loops are very dangerous,
            // because if they run too long, they might
            // need more gas than is available in a block.
            // In this case, the delegation will not be executed,
            // but in other situations, such loops might
            // cause a contract to get "stuck" completely.
            while (voters[to].delegate != address(0)) {
                to = voters[to].delegate;

                // We found a loop in the delegation, not allowed.
                require(to != msg.sender, "Found loop in delegation.");
            }

            // Since `sender` is a reference, this
            // modifies `voters[msg.sender].voted`
            sender.voted = true;
            sender.delegate = to;
            Voter storage delegate_ = voters[to];
            if (delegate_.voted) {
                // If the delegate already voted,
                // directly add to the number of votes
                proposals[delegate_.vote].voteCount += sender.weight;
            } else {
                // If the delegate did not vote yet,
                // add to her weight.
                delegate_.weight += sender.weight;
            }
        }

        /// Give your vote (including votes delegated to you)
        /// to proposal `proposals[proposal].name`.
        function vote(uint proposal) external {
            Voter storage sender = voters[msg.sender];
            require(sender.weight != 0, "Has no right to vote");
            require(!sender.voted, "Already voted.");
            sender.voted = true;
            sender.vote = proposal;

            // If `proposal` is out of the range of the array,
            // this will throw automatically and revert all
            // changes.
            proposals[proposal].voteCount += sender.weight;
        }

        /// @dev Computes the winning proposal taking all
        /// previous votes into account.
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

        // Calls winningProposal() function to get the index
        // of the winner contained in the proposals array and then
        // returns the name of the winner
        function winnerName() external view
                returns (bytes32 winnerName_)
        {
            winnerName_ = proposals[winningProposal()].name;
        }
    }


Possible Improvements
=====================

Currently, many transactions are needed to assign the rights
to vote to all participants. Can you think of a better way?
