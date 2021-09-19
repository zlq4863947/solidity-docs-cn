.. index:: purchase, remote purchase, escrow

********************
安全的远程购买
********************

目前远程购买商品需要多方相互信任。
最简单的配置涉及卖方和买方。买方希望收到卖方的物品，而卖方希望获得金钱（或等价物）作为回报。

有问题的部分是这里的输送物品：无法确定物品是否送达买方。

有多种方法可以解决这个问题，但都有或多或少的不足。
在以下示例中，双方必须将物品价值的两倍作为托管放入合约中。一旦发生意外情况，这笔钱将被锁定在合约中，直到买方确认他们收到了物品。然后，买家将获得物品价值（他们押金的一半）的返还，而卖家则获得三倍的价值（他们的押金加上物品价值）。
这背后的想法是，使双方都有解决问题的动力，否则他们的钱将被永远锁定。

这个合约当然不能解决所有问题，但提供了如何在合约中使用类似状态机结构的概述。


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;
    contract Purchase {
        uint public value;
        address payable public seller;
        address payable public buyer;

        enum State { Created, Locked, Release, Inactive }
        // 状态变量的默认值是第一个成员，`State.created`
        State public state;

        modifier condition(bool condition_) {
            require(condition_);
            _;
        }

        /// 只有买方可以调用此方法。
        error OnlyBuyer();
        /// 只有卖方可以调用此方法。
        error OnlySeller();
        /// 在当前状态下无法调用该方法。
        error InvalidState();
        /// 提供的值必须是偶数。
        error ValueNotEven();

        modifier onlyBuyer() {
            if (msg.sender != buyer)
                revert OnlyBuyer();
            _;
        }

        modifier onlySeller() {
            if (msg.sender != seller)
                revert OnlySeller();
            _;
        }

        modifier inState(State state_) {
            if (state != state_)
                revert InvalidState();
            _;
        }

        event Aborted();
        event PurchaseConfirmed();
        event ItemReceived();
        event SellerRefunded();

        // 确保 `msg.value` 是偶数。
        // 如果是奇数，除法将被截断（Solidity没有浮点数）。
        // 通过乘法检查它不是奇数。
        constructor() payable {
            seller = payable(msg.sender);
            value = msg.value / 2;
            if ((2 * value) != msg.value)
                revert ValueNotEven();
        }

        /// 中止购买并回收以太币。
        /// 只能在合约锁定前由卖家调用。
        function abort()
            external
            onlySeller
            inState(State.Created)
        {
            emit Aborted();
            state = State.Inactive;
            // 我们在这里直接使用 transfer。
            // 它是可重入安全的，因为它是最终方法调用，并且我们已经更改了状态。
            seller.transfer(address(this).balance);
        }

        /// 由买家确认购买。
        /// 交易必须包含`2 * value` 以太币。
        /// 在调用 confirmReceived 之前，以太币将被锁定。
        function confirmPurchase()
            external
            inState(State.Created)
            condition(msg.value == (2 * value))
            payable
        {
            emit PurchaseConfirmed();
            buyer = payable(msg.sender);
            state = State.Locked;
        }

        /// 买家确认您收到了该物品。
        /// 这将释放锁定的以太币。
        function confirmReceived()
            external
            onlyBuyer
            inState(State.Locked)
        {
            emit ItemReceived();
            // 首先更改状态很重要，否则，下面使用 `send` 调用的合约可以在这里再次调用。
            state = State.Release;

            buyer.transfer(value);
        }

        /// 此方法向卖家退款（偿还卖方的锁定资金。）
        function refundSeller()
            external
            onlySeller
            inState(State.Release)
        {
            emit SellerRefunded();
            // 首先更改状态很重要，否则，下面使用 `send` 调用的合约可以在这里再次调用。
            state = State.Inactive;

            seller.transfer(3 * value);
        }
    }
