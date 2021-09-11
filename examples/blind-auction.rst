.. index:: auction;blind, auction;open, blind auction, open auction

*************
盲拍
*************

在本节中，我们将展示在以太坊上创建一个完全盲目的拍卖合约是多么容易。我们将从公开拍卖开始，每个人都可以看到出价，然后将此合同扩展为盲拍卖，在竞标期结束之前无法看到实际出价。

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

        /// 使用与此交易一起发送的价格在拍卖中出价。
        /// 只有在没有赢得拍卖的情况下才会退还该价值。
        function bid() external payable {
            // 不需要任何参数，所有信息都已经是交易的一部分。
            // 该方法需要关键字payable才能接收Ether。

            // 如果投标期结束，则取消回滚此方法的执行。
            if (block.timestamp > auctionEndTime)
                revert AuctionAlreadyEnded();

            // 如果出价 <= 现有最高加，则退回资产（revert 语句将回滚此方法执行中的所有更改，包括它已收到的资产）。
            if (msg.value <= highestBid)
                revert BidNotHighEnough(highestBid);

            if (highestBid != 0) {
                // Sending back the money by simply using
                // highestBidder.send(highestBid) is a security risk
                // because it could execute an untrusted contract.
                // It is always safer to let the recipients
                // withdraw their money themselves.
                pendingReturns[highestBidder] += highestBid;
            }
            highestBidder = msg.sender;
            highestBid = msg.value;
            emit HighestBidIncreased(msg.sender, msg.value);
        }

        /// Withdraw a bid that was overbid.
        function withdraw() external returns (bool) {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // It is important to set this to zero because the recipient
                // can call this function again as part of the receiving call
                // before `send` returns.
                pendingReturns[msg.sender] = 0;

                if (!payable(msg.sender).send(amount)) {
                    // No need to call throw here, just reset the amount owing
                    pendingReturns[msg.sender] = amount;
                    return false;
                }
            }
            return true;
        }

        /// End the auction and send the highest bid
        /// to the beneficiary.
        function auctionEnd() external {
            // It is a good guideline to structure functions that interact
            // with other contracts (i.e. they call functions or send Ether)
            // into three phases:
            // 1. checking conditions
            // 2. performing actions (potentially changing conditions)
            // 3. interacting with other contracts
            // If these phases are mixed up, the other contract could call
            // back into the current contract and modify the state or cause
            // effects (ether payout) to be performed multiple times.
            // If functions called internally include interaction with external
            // contracts, they also have to be considered interaction with
            // external contracts.

            // 1. Conditions
            if (block.timestamp < auctionEndTime)
                revert AuctionNotYetEnded();
            if (ended)
                revert AuctionEndAlreadyCalled();

            // 2. Effects
            ended = true;
            emit AuctionEnded(highestBidder, highestBid);

            // 3. Interaction
            beneficiary.transfer(highestBid);
        }
    }

Blind Auction
=============

The previous open auction is extended to a blind auction in the following. The
advantage of a blind auction is that there is no time pressure towards the end
of the bidding period. Creating a blind auction on a transparent computing
platform might sound like a contradiction, but cryptography comes to the
rescue.

During the **bidding period**, a bidder does not actually send their bid, but
only a hashed version of it.  Since it is currently considered practically
impossible to find two (sufficiently long) values whose hash values are equal,
the bidder commits to the bid by that.  After the end of the bidding period,
the bidders have to reveal their bids: They send their values unencrypted and
the contract checks that the hash value is the same as the one provided during
the bidding period.

Another challenge is how to make the auction **binding and blind** at the same
time: The only way to prevent the bidder from just not sending the money after
they won the auction is to make them send it together with the bid. Since value
transfers cannot be blinded in Ethereum, anyone can see the value.

The following contract solves this problem by accepting any value that is
larger than the highest bid. Since this can of course only be checked during
the reveal phase, some bids might be **invalid**, and this is on purpose (it
even provides an explicit flag to place invalid bids with high value
transfers): Bidders can confuse competition by placing several high or low
invalid bids.


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

        // Allowed withdrawals of previous bids
        mapping(address => uint) pendingReturns;

        event AuctionEnded(address winner, uint highestBid);

        // Errors that describe failures.

        /// The function has been called too early.
        /// Try again at `time`.
        error TooEarly(uint time);
        /// The function has been called too late.
        /// It cannot be called after `time`.
        error TooLate(uint time);
        /// The function auctionEnd has already been called.
        error AuctionEndAlreadyCalled();

        // Modifiers are a convenient way to validate inputs to
        // functions. `onlyBefore` is applied to `bid` below:
        // The new function body is the modifier's body where
        // `_` is replaced by the old function body.
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

        /// Place a blinded bid with `blindedBid` =
        /// keccak256(abi.encodePacked(value, fake, secret)).
        /// The sent ether is only refunded if the bid is correctly
        /// revealed in the revealing phase. The bid is valid if the
        /// ether sent together with the bid is at least "value" and
        /// "fake" is not true. Setting "fake" to true and sending
        /// not the exact amount are ways to hide the real bid but
        /// still make the required deposit. The same address can
        /// place multiple bids.
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

        /// Reveal your blinded bids. You will get a refund for all
        /// correctly blinded invalid bids and for all bids except for
        /// the totally highest.
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
                    // Bid was not actually revealed.
                    // Do not refund deposit.
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

        /// Withdraw a bid that was overbid.
        function withdraw() external {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // It is important to set this to zero because the recipient
                // can call this function again as part of the receiving call
                // before `transfer` returns (see the remark above about
                // conditions -> effects -> interaction).
                pendingReturns[msg.sender] = 0;

                payable(msg.sender).transfer(amount);
            }
        }

        /// End the auction and send the highest bid
        /// to the beneficiary.
        function auctionEnd()
            external
            onlyAfter(revealEnd)
        {
            if (ended) revert AuctionEndAlreadyCalled();
            emit AuctionEnded(highestBidder, highestBid);
            ended = true;
            beneficiary.transfer(highestBid);
        }

        // This is an "internal" function which means that it
        // can only be called from the contract itself (or from
        // derived contracts).
        function placeBid(address bidder, uint value) internal
                returns (bool success)
        {
            if (value <= highestBid) {
                return false;
            }
            if (highestBidder != address(0)) {
                // Refund the previously highest bidder.
                pendingReturns[highestBidder] += highestBid;
            }
            highestBid = value;
            highestBidder = bidder;
            return true;
        }
    }
