********************
小额支付通道
********************

在本节中，我们将学习如何构建支付通道的示例实现。
它使用加密签名在同一方之间安全、即时且无交易费用地重复转移 以太币。 例如，我们需要了解如何签名和验证签名，以及设置支付通道。

创建和验证签名
=================================

想象一下 Alice 想要向 Bob 发送一定数量的 以太币，即 Alice 是发送者，而 Bob 是接收者。

Alice 只需要在链外（例如通过电子邮件）向 Bob 发送加密签名的消息，这类似于写支票。

Alice 和 Bob 使用签名来授权交易，这可以通过以太坊上的智能合约实现。
Alice 将构建一个简单的智能合约，让她转移 以太币，但她不会自己调用方法来发起支付，而是让 Bob 这样做，从而支付交易费用。

该合约将按以下方式运作：

    1. Alice 部署 ``ReceiverPays`` 合约，附加足够的以太币来支付将要支付的款项。
    2. Alice 通过使用她的私钥签署消息来授权付款。
    3. Alice 将加密签名的消息发送给 Bob。 消息不需要保密（稍后解释），发送它的机制无关紧要。
    4. Bob 通过将签名的消息提交给智能合约来索取他的付款，它验证消息的真实性，然后交付资金。

创建签名
----------------------

Alice 不需要与以太坊网络交互来签署交易，整个过程是完全离线的。
在本教程中，我们将使用 `web3.js <https://github.com/ethereum/web3.js>`_ 和 `MetaMask <https://metamask.io>`_ 在浏览器中签署消息，使用 `EIP-762 <https://github.com/ethereum/EIPs/pull/712>`_ 中描述的方法，因为它提供了许多其他安全优势。

.. code-block:: javascript

    /// 首先哈希使事情变得更容易
    var hash = web3.utils.sha3("message to sign");
    web3.eth.personal.sign(hash, web3.eth.defaultAccount, function () { console.log("Signed"); });

.. 注解::
  ``web3.eth.personal.sign`` 将消息的长度添加到签名数据之前。因为我们先哈希，所以消息总是正好是 32 字节长，因此这个长度前缀总是相同的。

签署什么
------------

对于履行付款的合同，签署的消息必须包括：

    1. 收件人的地址。
    2. 要转移的金额。
    3. 防止重放攻击。

重放攻击是指重新使用已签名的消息来声明对第二个操作的授权。
为了避免重放攻击，我们引入一个 **nonce** (以太坊链上交易也是使用这个方式来防止重放攻击), 它表示一个账号已经发送交易的次数。智能合约将检查 **nonce** 是否使用过。

当所有者部署 ``ReceiverPays`` 智能合约，进行一些付款，然后销毁合约时，可能会发生另一种类型的重放攻击。当他们决定再次部署 ``ReceiverPays`` 智能合约，新合约不知道之前部署中使用的 **nonce**，因此攻击者可以再次使用旧消息。

Alice 可以通过在消息中包含合约地址来防止这种攻击，并且只有在消息中包含
合约自身地址才会被接受。您可以在本节末尾的完整合约的 ``claimPayment()`` 方法的前两行中找到这个示例。

封装参数
-----------------

现在我们已经确定了要在签名消息中包含哪些信息，我们就可以将消息放在一起、哈希化并签名。为简单起见，我们连接数据。 `ethereumjs-abi <https://github.com/ethereumjs/ethereumjs-abi>`_ 库提供了一个名为 ``soliditySHA3`` 的方法，它模仿了 Solidity 的 ``keccak256`` 方法的行为，该方法用于编码 ``abi.encodePacked`` 。

这里有一个 **JavaScript** 方法，它为 ``ReceiverPays`` 示例创建正确的签名：

.. code-block:: javascript

    // recipient 是收款人地址。
    // amount 是要发送的以太币数量（单位: wei）。
    // nonce 可以是任何唯一的数字以防止重放攻击
    // contractAddress 用于防止跨合约重放攻击
    function signPayment(recipient, amount, nonce, contractAddress, callback) {
        var hash = "0x" + abi.soliditySHA3(
            ["address", "uint256", "uint256", "address"],
            [recipient, amount, nonce, contractAddress]
        ).toString("hex");

        web3.eth.personal.sign(hash, web3.eth.defaultAccount, callback);
    }

在 Solidity 中还原消息签名者
-----------------------------------------

一般来说，ECDSA 签名由两个参数组成，``r`` 和 ``s``。 以太坊中的签名包含第三个名为 ``v`` 的参数，您可以使用它来验证哪个帐户的私钥签名了此消息，以及交易的发送者。 Solidity 提供了一个内置方法 :ref:`ecrecover <mathematical-and-cryptographic-functions>` ，它接受消息以及 ``r``、``s`` 和 ``v`` 参数并返回签名此消息的地址。

提取签名参数
-----------------------------------

web3.js 生成的签名是由 ``r``、``s`` 和 ``v`` 组合而成，所以第一步需要将这些参数分开。您可以在客户端执行此操作，但在智能合约中执行此操作意味着您只需要发送一个签名参数而不是三个。将字节数组拆分为其组成部分是非常麻烦的，因此我们使用 :doc:`inline assembly <assembly>` 来完成 `splitSignature` 方法（完整合约末尾的第三个函数）中的这部分工作）。

计算消息哈希
--------------------------

智能合约需要确切知道签署了哪些参数，因此它必须根据参数重新创建消息并将其用于签名验证。

方法 ``prefixed`` 和 ``recoverSigner`` 在 ``claimPayment`` 方法中执行此操作。

完整合约
-----------------

.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    contract ReceiverPays {
        address owner = msg.sender;

        mapping(uint256 => bool) usedNonces;

        constructor() payable {}

        function claimPayment(uint256 amount, uint256 nonce, bytes memory signature) external {
            require(!usedNonces[nonce]);
            usedNonces[nonce] = true;

            // 这将重新创建在客户端上签名的消息
            bytes32 message = prefixed(keccak256(abi.encodePacked(msg.sender, amount, nonce, this)));

            require(recoverSigner(message, signature) == owner);

            payable(msg.sender).transfer(amount);
        }

        /// 销毁合约并收回剩余资金。
        function shutdown() external {
            require(msg.sender == owner);
            selfdestruct(payable(msg.sender));
        }

        /// 签名方法
        function splitSignature(bytes memory sig)
            internal
            pure
            returns (uint8 v, bytes32 r, bytes32 s)
        {
            require(sig.length == 65);

            assembly {
                // 前 32 个字节，在长度前缀之后。
                r := mload(add(sig, 32))
                // 第二个 32 字节。
                s := mload(add(sig, 64))
                // 最后一个字节（接下来的 32 个字节的第一个字节）。
                v := byte(0, mload(add(sig, 96)))
            }

            return (v, r, s);
        }

        function recoverSigner(bytes32 message, bytes memory sig)
            internal
            pure
            returns (address)
        {
            (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

            return ecrecover(message, v, r, s);
        }

        /// 构建一个带前缀的哈希来模仿 eth_sign 的行为。
        function prefixed(bytes32 hash) internal pure returns (bytes32) {
            return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
        }
    }


编写一个简单的支付通道
================================

Alice 现在构建了一个简单但完整的支付通道实现。支付通道使用加密签名来安全、即时且无需交易费用地重复转移以太币。

什么是支付通道？
--------------------------

支付通道允许参与者在不使用交易的情况下重复转移以太币。这意味着您可以避免与交易相关的延迟和费用。 我们将探索两方（Alice 和 Bob）之间的简单单向支付通道。包括三个步骤：

    1. Alice为以太坊的智能合约提供资金。这"打开"了支付通道。
    2. Alice 签署消息，指明给接收者多少以太币。每次付款都会重复此步骤。
    3. Bob"关闭"支付通道，提取他的部分的以太币并将剩余部分发回付款人。

.. 注解::
  只有第 1 步和第 3 步需要以太坊交易，第 2 步意味着付款人通过链下方法（例如电子邮件）将经过加密签名的消息传输给收款人。这意味着只需要两笔交易即可支持任意金额的转移。

Bob 可以保证会收到他的资金，因为智能合约托管 以太币 并遵守有效的签名消息。智能合约还可以强制超时执行，因此即使收款人拒绝关闭通道，Alice 也可以保证最终收回她的资金。支付通道的参与者可以决定将其开放多长时间。对于短期交易，例如为每分钟的网络访问支付网络费用，支付通道可能会在有限的时间内保持开放。另一方面，对于经常性付款，例如向员工支付每小时工资，付款通道可能会保持开放数月或数年。

开通支付通道
---------------------------

为了打开支付通道，Alice 部署了智能合约，添加了要托管的以太币，并指定了预期的收款人和通道存在的最长持续时间。这个处理是本节末尾的合约中的方法 ``SimplePaymentChannel``。

付款
---------------

Alice 通过向 Bob 发送签名消息来付款。
此步骤完全可以在以太坊网络之外进行。
消息由发送者加密签名，然后直接发送给接收者。

每条消息都包含以下信息：

    * 智能合约的地址，用于防止跨合约重放攻击。
    * 到目前为需要发送给接收方的以太币总量。

在一系列转账结束时，支付通道仅关闭一次。
因此，只有一条发送的消息被赎回。这就是为什么每条消息都指定了所欠以太币的累计总量，而不是单个小额支付的金额。收款人自然会选择赎回最新的消息，因为那是总数最高的消息。
不再需要每条消息的 **nonce**，因为智能合约只接受一条消息。智能合约的地址仍然用于防止用于一个支付通道的消息被用在不同的支付通道。

以下是修改后的 JavaScript 代码，用于对上一节中的消息进行加密签名：

.. code-block:: javascript

    function constructPaymentMessage(contractAddress, amount) {
        return abi.soliditySHA3(
            ["address", "uint256"],
            [contractAddress, amount]
        );
    }

    function signMessage(message, callback) {
        web3.eth.personal.sign(
            "0x" + message.toString("hex"),
            web3.eth.defaultAccount,
            callback
        );
    }

    // contractAddress 用于防止跨合约重放攻击。
    // 数量，以 wei 为单位，指定应该发送多少以太币。

    function signPayment(contractAddress, amount, callback) {
        var message = constructPaymentMessage(contractAddress, amount);
        signMessage(message, callback);
    }


关闭支付通道
---------------------------

当 Bob 准备好接收他的资金时，就可以通过调用智能合约上的 ``close`` 方法来关闭支付通道。
关闭通道会向接收者支付他们所欠的以太币并销毁合约，将任何剩余的以太币发送回Alice。要关闭通道，Bob 需要提供一条由 Alice 签名的消息。

智能合约必须验证消息是否包含来自发送者的有效签名。
进行此的验证过程与收款人使用的验证过程相同。
Solidity 方法 ``isValidSignature`` 和 ``recoverSigner`` 的工作方式与上一节中的 JavaScript 对应方法一样，后者的方法借自 ``ReceiverPays`` 合约。

只有支付通道接收方可以调用 ``close`` 方法，他自然会接收最近的支付消息，因为该消息携带的欠款总额最高。如果允许发件人调用此函数，则他们可以提供较低金额的消息，并欺骗收件人以摆脱债务。

该方法验证签名消息与指定参数匹配。
如果一切正常，接收者将收到他们的一部分以太币，而其余部分则通过 ``selfdestruct`` 发回给发送者。
您可以在完整合约中看到 ``close`` 方法。

支付通道有效期
-------------------

Bob 可以随时关闭支付通道，但如果他们不这样做，Alice 需要一种方法来收回她的托管资金。在合约部署时设置了 *到期* 时间。 一旦到了那个时间，Alice 就可以调用 ``claimTimeout`` 来收回她的资金。您可以在完整合约中看到 ``claimTimeout`` 方法。

调用此方法后，Bob 将无法再接收任何 以太币，因此 Bob 在到期之前关闭通道很重要。

完整合约
-----------------

.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    contract SimplePaymentChannel {
        address payable public sender;      // 付款人
        address payable public recipient;   // 收款人
        uint256 public expiration;  // 收款人不关闭通道的超时时间

        constructor (address payable recipientAddress, uint256 duration)
            payable
        {
            sender = payable(msg.sender);
            recipient = recipientAddress;
            expiration = block.timestamp + duration;
        }

        /// 收款人可以随时通过提供来自付款人的签名金额来关闭通道。
        /// 收款人将收到该金额，剩余部分将返还给付款人
        function close(uint256 amount, bytes memory signature) external {
            require(msg.sender == recipient);
            require(isValidSignature(amount, signature));

            recipient.transfer(amount);
            selfdestruct(sender);
        }

        /// 付款人可以随时延长到期时间
        function extend(uint256 newExpiration) external {
            require(msg.sender == sender);
            require(newExpiration > expiration);

            expiration = newExpiration;
        }

        /// 如果超时后接收方未关闭通道的情况下达到超时，则将以太币发回付款方。
        function claimTimeout() external {
            require(block.timestamp >= expiration);
            selfdestruct(sender);
        }

        function isValidSignature(uint256 amount, bytes memory signature)
            internal
            view
            returns (bool)
        {
            bytes32 message = prefixed(keccak256(abi.encodePacked(this, amount)));

            // 检查签名是否来自付款人
            return recoverSigner(message, signature) == sender;
        }

        /// 下面的所有方法都取自 '创建和验证签名' 一章。
        function splitSignature(bytes memory sig)
            internal
            pure
            returns (uint8 v, bytes32 r, bytes32 s)
        {
            require(sig.length == 65);

            assembly {
                // 前 32 个字节，在长度前缀之后。
                r := mload(add(sig, 32))
                // 第二个 32 字节。
                s := mload(add(sig, 64))
                // 最后一个字节（接下来的 32 个字节的第一个字节）。
                v := byte(0, mload(add(sig, 96)))
            }

            return (v, r, s);
        }

        function recoverSigner(bytes32 message, bytes memory sig)
            internal
            pure
            returns (address)
        {
            (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

            return ecrecover(message, v, r, s);
        }

        /// 构建一个带前缀的哈希来模仿 eth_sign 的行为。
        function prefixed(bytes32 hash) internal pure returns (bytes32) {
            return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
        }
    }


.. 注解::
  ``splitSignature`` 方法，不使用任何安全检查。真正的实现应该使用经过更严格测试的库，例如 openzepplin 的 `version <https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol>`_ 这段代码。

验证支付
------------------

与上一节不同，支付通道中的消息不会立即兑换。收款人会跟踪最新的消息，并在需要关闭支付通道时将其赎回。这意味着收款人对每条消息的自我验证至关重要。
否则无法保证收款人最终能够获得报付款。

收款人应使用以下流程验证每条消息：

    1. 验证消息中的合约地址是否与支付通道匹配。
    2. 验证最新总数是否为预期金额。
    3. 验证最新总数不超过托管的以太币数量。
    4. 验证签名是否有效并且来自支付通道付款人。

我们将使用 `ethereumjs-util <https://github.com/ethereumjs/ethereumjs-util>`_ 库来编写此验证。最后一步可以通过多种方式完成，我们使用 JavaScript。 以下代码从上面的签名 **JavaScript 代码** 中借用了 ``constructPaymentMessage`` 方法：

.. code-block:: javascript

    // 这模仿了 eth_sign JSON-RPC 方法的前缀行为。
    function prefixed(hash) {
        return ethereumjs.ABI.soliditySHA3(
            ["string", "bytes32"],
            ["\x19Ethereum Signed Message:\n32", hash]
        );
    }

    function recoverSigner(message, signature) {
        var split = ethereumjs.Util.fromRpcSig(signature);
        var publicKey = ethereumjs.Util.ecrecover(message, split.v, split.r, split.s);
        var signer = ethereumjs.Util.pubToAddress(publicKey).toString("hex");
        return signer;
    }

    function isValidSignature(contractAddress, amount, signature, expectedSigner) {
        var message = prefixed(constructPaymentMessage(contractAddress, amount));
        var signer = recoverSigner(message, signature);
        return signer.toLowerCase() ==
            ethereumjs.Util.stripHexPrefix(expectedSigner).toLowerCase();
    }
