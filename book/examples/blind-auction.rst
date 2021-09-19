.. index:: auction;blind, auction;open, blind auction, open auction

*************
盲拍
*************

在本节中，我们将展示在以太坊上创建一个完全盲目的拍卖合约是多么容易。我们将从公开拍卖开始，每个人都可以看到出价，然后将此合同扩展为盲拍卖，在竞标期结束之前无法看到实际出价。

**关键要素**

- 受益人，最终的钱款接收方
- 竞拍者，任意持有合法账户的人都可以参与竞拍
- 竞拍时间，只能在指定时间内完成出价
- 展示时间，参与者亮出出价

.. _simple_auction:

简单的公开拍卖
===================

以下简单拍卖合约的大体思路：每个人都可以在投标期间发送他们的投标。 出价已经包括资金/以太币以将投标人绑定到他们的出价中。如果拍卖最高价格被提高，则之前出价最高的人会收回他们的钱。投标期结束后，必须手动调用合同，受益人才能收到钱 - 合同无法自行激活。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;
    contract SimpleAuction {
        // 拍卖收益人
        address payable public beneficiary;
        // 结束时间，UTC时间戳（秒单位）或以秒为单位的时间段。
        uint public auctionEndTime;

        // 当前出价最高者
        address public highestBidder;
        uint public highestBid;

        // 运行撤回之前的出价
        mapping(address => uint) pendingReturns;

        // 最后设置为 true，不允许任何更改。
        // 默认初始化为 `false`。
        bool ended;

        // 最高价更新时触发事件
        event HighestBidIncreased(address bidder, uint amount);
        // 拍卖结束时触发事件
        event AuctionEnded(address winner, uint amount);

        // 描述失败的错误。

        // 三斜线注释是被称为 **natspec** 的注释格式。当要求用户确认交易或显示错误时，它们将显示。

        /// 拍卖已经结束。
        error AuctionAlreadyEnded();
        /// 已经有更高或相等的出价。
        error BidNotHighEnough(uint highestBid);
        /// 拍卖还没有结束。
        error AuctionNotYetEnded();
        /// auctionEnd方法已经被调用。
        error AuctionEndAlreadyCalled();

        /// 代表受益人地址 `beneficiaryAddress` 创建一个带有 `biddingTime` 秒出价时间的简单拍卖。
        constructor(
            uint biddingTime,
            address payable beneficiaryAddress
        ) {
            beneficiary = beneficiaryAddress;
            auctionEndTime = block.timestamp + biddingTime;
        }

        /// 对拍卖进行出价，出价将与此次交易一起发送。
        /// 只有在没有赢得拍卖的情况下才会退还本次出价。
        function bid() external payable {
            // 不需要任何参数，所有信息都已经是交易的一部分。
            // 该方法需要关键字payable才能接收Ether。

            // 如果投标期结束，则取消并回滚此方法的执行。
            if (block.timestamp > auctionEndTime)
                revert AuctionAlreadyEnded();

            // 如果出价 <= 现有最高价，则退回资产（revert 语句将回滚此方法执行中的所有更改，包括它已收到的资产）。
            if (msg.value <= highestBid)
                revert BidNotHighEnough(highestBid);

            if (highestBid != 0) {
                // 直接使用 highestBidder.send(highestBid) 退回资金会有安全风险，
                // 因为它可能会执行一个不受信任的合约。
                // 让收款人自己取款会比较安全的。
                pendingReturns[highestBidder] += highestBid;
            }
            highestBidder = msg.sender;
            highestBid = msg.value;
            emit HighestBidIncreased(msg.sender, msg.value);
        }

        /// 由投标者自己取回出价
        function withdraw() external returns (bool) {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // 将此设置为零很重要，
                // 因为作为接收调用的一部分，
                // 投标者可以在 `send` 返回之前再次调用此方法。
                pendingReturns[msg.sender] = 0;

                if (!payable(msg.sender).send(amount)) {
                    // 无需在此处调用 throw，只需重置未成功发送的资产
                    pendingReturns[msg.sender] = amount;
                    return false;
                }
            }
            return true;
        }

        /// 结束拍卖并将最高出价发送给受益人。
        function auctionEnd() external {
            // 将与其他合约通信的方法（即它们会调用此方法或发送以太币）
            // 将此步骤分为三个阶段是一个很好的指导方针：
            // 1. 检查条件
            // 2. 执行操作（可能改变条件）
            // 3. 与其他合约通信
            // 如果这些阶段混合在一起，另一个合约可能会回调到当前合约，
            // 并修改状态或导致多次执行（以太币支付）。
            // 如果内部调用的方法包含与外部合约的交互，则它们也必须被视为与外部合约的交互。

            // 1. 条件
            if (block.timestamp < auctionEndTime)
                revert AuctionNotYetEnded();
            if (ended)
                revert AuctionEndAlreadyCalled();

            // 2. 生效
            ended = true;
            emit AuctionEnded(highestBidder, highestBid);

            // 3. 交互
            beneficiary.transfer(highestBid);
        }
    }

盲拍
=============

之前的公开拍卖在下文中扩展为盲拍。盲拍的优点是在投标期结束时没有时间压力。在透明的计算平台上创建盲拍听起来似乎很矛盾，但密码学可以解决此问题。

在 **投标期间**，投标人实际上不会发送他们的投标，而只是它的哈希版本。由于目前认为实际上不可能找到两个（足够长的）哈希值相等的值，因此投标人承诺投标。投标期结束后，投标人必须公开他们的投标：他们发送未加密的出价，合同检查哈希值是否与投标期间提供的值相同。

另一个挑战是如何让拍卖同时 **绑定和隐秘**：防止投标人赢得拍卖后不发送资金的唯一方法是让他们将出价资产与投标一起发送。由于以太坊中的资产转移无法被隐藏，因此任何人都可以看到转移的资产。

下面的合约通过接受任何大于最高出价的值来解决这个问题。由于这当然只能在展示阶段进行检查，因此某些出价可能是 **无效**的，这是故意的（它甚至提供了一个明确的标志来放置具有高价值转移的无效出价）：投标人可以通过以下方式混淆出价 放置几个高的或低的无效出价。

.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;
    contract BlindAuction {
        struct Bid {
            bytes32 blindedBid;
            uint deposit;
        }

        address payable public beneficiary;
        uint public biddingEnd;
        uint public revealEnd;
        bool public ended;

        mapping(address => Bid[]) public bids;

        address public highestBidder;
        uint public highestBid;

        // 允许撤回以前的出价
        mapping(address => uint) pendingReturns;

        event AuctionEnded(address winner, uint highestBid);

        // 描述失败的错误信息。

        /// 该方法调用过早。在 `time` 后重试。
        error TooEarly(uint time);
        /// 该方法调用过晚。 它不能在 `time` 之后调用。
        error TooLate(uint time);
        /// 方法auctionEnd 已经被调用。
        error AuctionEndAlreadyCalled();

        // modifier是验证方法输入参数的便捷方式。
        // `onlyBefore` 应用于下面的 `bid`：
        // 新方法内容是modifier的内容，其中`_` 被旧方法内容替换。
        modifier onlyBefore(uint time) {
            if (block.timestamp >= time) revert TooLate(time);
            _;
        }
        modifier onlyAfter(uint time) {
            if (block.timestamp <= time) revert TooEarly(time);
            _;
        }

        constructor(
            uint biddingTime,
            uint revealTime,
            address payable beneficiaryAddress
        ) {
            beneficiary = beneficiaryAddress;
            biddingEnd = block.timestamp + biddingTime;
            revealEnd = biddingEnd + revealTime;
        }

        /// 使用 `blindedBid` = keccak256(abi.encodePacked(value, fake, secret)) 进行盲拍。
        // 只有在展示阶段正确展示出价时，才会退还发送的以太币。如果与出价一起发送的以太币至少是 "value" 且 "fake" 不为真，则出价有效。将 "fake"设置为真并且不发送确切金额是隐藏真实出价但仍进行所需存款的方法。 同一个地址可以多次出价。
        function bid(bytes32 blindedBid)
            external
            payable
            onlyBefore(biddingEnd)
        {
            bids[msg.sender].push(Bid({
                blindedBid: blindedBid,
                deposit: msg.value
            }));
        }

        /// 显示您的盲拍出价。您将获得所有正确盲法无效投标和除最高出价之外的所有投标的退款。
        function reveal(
            uint[] calldata values,
            bool[] calldata fakes,
            bytes32[] calldata secrets
        )
            external
            onlyAfter(biddingEnd)
            onlyBefore(revealEnd)
        {
            uint length = bids[msg.sender].length;
            require(values.length == length);
            require(fakes.length == length);
            require(secrets.length == length);

            uint refund;
            for (uint i = 0; i < length; i++) {
                Bid storage bidToCheck = bids[msg.sender][i];
                (uint value, bool fake, bytes32 secret) =
                        (values[i], fakes[i], secrets[i]);
                if (bidToCheck.blindedBid != keccak256(abi.encodePacked(value, fake, secret))) {
                    // 出价实际上并未透露。
                    // 不退还押金。
                    continue;
                }
                refund += bidToCheck.deposit;
                if (!fake && bidToCheck.deposit >= value) {
                    if (placeBid(msg.sender, value))
                        refund -= value;
                }
                // Make it impossible for the sender to re-claim
                // the same deposit.
                bidToCheck.blindedBid = bytes32(0);
            }
            payable(msg.sender).transfer(refund);
        }

        /// 撤回出价过高的出价。
        function withdraw() external {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // 将其设置为零很重要，因为接收者可以在 `transfer` 返回之前作为接收调用的一部分再次调用此方法（请参阅上面关于条件 -> 效果 -> 交互的备注）。
                pendingReturns[msg.sender] = 0;

                payable(msg.sender).transfer(amount);
            }
        }

        /// 结束拍卖并将最高出价发送给受益人。
        function auctionEnd()
            external
            onlyAfter(revealEnd)
        {
            if (ended) revert AuctionEndAlreadyCalled();
            emit AuctionEnded(highestBidder, highestBid);
            ended = true;
            beneficiary.transfer(highestBid);
        }

        // 这是一个 "internal" 方法，这意味着它只能从合约本身（或派生合约）调用。
        function placeBid(address bidder, uint value) internal
                returns (bool success)
        {
            if (value <= highestBid) {
                return false;
            }
            if (highestBidder != address(0)) {
                // 退还先前最高出价者。
                pendingReturns[highestBidder] += highestBid;
            }
            highestBid = value;
            highestBidder = bidder;
            return true;
        }
    }
