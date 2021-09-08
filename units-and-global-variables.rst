.. include:: glossaries.rst

**************************************
单位和全局可用变量
**************************************

.. index:: wei, finney, szabo, gwei, ether

|ether| 单位
==============

|ether| 的单位面额是在数字后边加上 ``wei``、``gwei``  或 ``ether`` ，如果后面没有单位，则缺省为 ``wei`` 。例如 ``2 ether == 2000 finney`` 的逻辑判断结果为 ``true``。

.. code-block:: solidity
    :force:

    assert(1 wei == 1);
    assert(1 gwei == 1e9);
    assert(1 ether == 1e18);

货币单位后缀的的效果相当于乘以10的幂。

.. 注解::
    从0.7.0开始 ``finney`` 和 ``szabo`` 被移除了。

.. index:: time, seconds, minutes, hours, days, weeks, years

时间单位
==========

秒是缺省时间单位，在时间单位之间，数字后面带有 ``seconds``、 ``minutes``、 ``hours``、 ``days`` 和 ``weeks`` 的可以进行换算，基本换算关系如下：

* ``1 == 1 seconds``
* ``1 minutes == 60 seconds``
* ``1 hours == 60 minutes``
* ``1 days == 24 hours``
* ``1 weeks == 7 days``


如果您使用这些单位执行日历计算，请小心，因为 `闰秒 <https://zh.wikipedia.org/wiki/%E9%97%B0%E7%A7%92>`_ 的存在，并不是每年都等于 365 天，甚至每天都有可能不等于 24 小时。

由于无法预测闰秒，因此必须通过外部预言机更新精确的日历库，进行时间矫正。

.. 注解::
    由于上述原因，后缀 `years` 已在 0.5.0 版本中删除。

这些后缀不能应用于变量。例如，如果想要以天为单位设置参数，则可以通过以下方式：

.. code-block:: solidity

    function f(uint start, uint daysAfter) public {
        if (block.timestamp >= start + daysAfter * 1 days) {
          // ...
        }
    }

.. _special-variables-functions:

特殊变量和方法
===============================

全局命名空间中始终存在一些特殊的变量和函数，主要用于提供有关区块链的信息和一些通用方法。

.. index:: abi, block, coinbase, difficulty, encode, number, block;number, timestamp, block;timestamp, msg, data, gas, sender, value, gas price, origin


区块和交易属性
--------------------------------

- ``blockhash(uint blockNumber) returns (bytes32)``:  当 ``blocknumber`` 是 256 个最近的区块之一时，返回指定区块的哈希值； 否则返回零
- ``block.basefee`` (``uint``): 当前区块的基本费用 (`EIP-3198 <https://eips.ethereum.org/EIPS/eip-3198>`_ 和 `EIP-1559 <https://eips.ethereum.org/EIPS/eip-1559>`_)
- ``block.chainid`` (``uint``): 当前区块链id
- ``block.coinbase`` (``address payable``): 当前区块矿工的地址
- ``block.difficulty`` (``uint``): 当前区块难度
- ``block.gaslimit`` (``uint``): 当前区块gas限制
- ``block.number`` (``uint``): 当前区块号
- ``block.timestamp`` (``uint``): 当前区块UTC时间戳（秒单位）
- ``gasleft() returns (uint256)``: 剩余gas
- ``msg.data`` (``bytes calldata``): 完整的数据
- ``msg.sender`` (``address``): 消息的发送者（当前调用）
- ``msg.sig`` (``bytes4``): calldata 的前四个字节（即方法标识符）
- ``msg.value`` (``uint``): 随消息一起发送的 wei 数量
- ``tx.gasprice`` (``uint``): 交易的gas价格
- ``tx.origin`` (``address``): 交易的发送者（完整的调用链）

.. 注解::
    ``msg`` 的所有成员的值，包括 ``msg.sender`` 和 ``msg.value`` 可以因每个 **外部** 方法调用而改变。这包括对库方法的调用。

.. 注解::
    不要依赖 ``block.timestamp`` 或 ``blockhash`` 作为随机数的来源，除非你知道自己在做什么。

    时间戳和区块哈希都可以在某种程度上受到矿工的影响。
    例如，挖矿社区中的不良参与者可以在选定的哈希上运行赌场支付功能，如果他们没有收到任何钱，只需重试不同的哈希。

    当前区块时间戳必须严格大于最后一个区块的时间戳，但唯一的保证是它会在规范链中两个连续区块的时间戳之间。

.. 注解::
    出于可扩展性的原因，并非所有块都可以使用块哈希。
    您只能访问最近的 256 个区块的哈希值，其他值都为零。

.. 注解::
    方法 ``blockhash`` 以前叫做 ``block.blockhash``，它在 0.4.22 版本中被弃用并在 0.5.0 版本中删除。

.. 注解::
    函数 ``gasleft`` 以前叫做 ``msg.gas``，它在 0.4.21 版本中被弃用并在 0.5.0 版本中删除。

.. 注解::
    在 0.7.0 版本中，``now`` （  ``block.timestamp`` 的别名）被移除。

.. index:: abi, encoding, packed

ABI 编码和解码方法
-----------------------------------

- ``abi.decode(bytes memory encodedData, (...)) returns (...)``: 对指定数据进行ABI解码，而类型在括号中作为第二个参数指定。 示例： ``(uint a, uint[2] memory b, bytes memory c) = abi.decode(data, (uint, uint[2], bytes))``
- ``abi.encode(...) returns (bytes memory)``: 对指定参数进行ABI编码
- ``abi.encodePacked(...) returns (bytes memory)``: 对给定参数执行 :ref: `打包编码 <abi_packed_mode>`。 请注意，打包编码可以不明确！
- ``abi.encodeWithSelector(bytes4 selector, ...) returns (bytes memory)``: 对指定第二个开始的参数进行编码，并以指定的方法选择器作为起始的 4 字节数据一起返回
- ``abi.encodeWithSignature(string memory signature, ...) returns (bytes memory)``: 相当于 ``abi.encodeWithSelector(bytes4(keccak256(bytes(signature))), ...)``

.. 注解::
    这些编码方法可用于为外部方法调用以制作数据，而无需实际调用外部方法。 此外，``keccak256(abi.encodePacked(a, b))`` 是一种计算结构化数据哈希的方法（但请注意，可以使用不同的方法参数类型来造成 "哈希碰撞"）。

有关编码的详细信息，请参阅有关 :ref:`ABI <ABI>` 和 :ref:`紧压缩编码 <abi_packed_mode>` 的文档。

.. index:: bytes members

字节成员
----------------

- ``bytes.concat(...) returns (bytes memory)``: :ref:`将可变数量的字节和 bytes1, ..., bytes32 参数连接成一个字节数组 <bytes-concat>`

.. index:: assert, revert, require

错误处理
--------------

请参阅 :ref:`assert 和 require<assert-and-require>` 的部分
有关错误处理以及何时使用哪些方法的更多详细信息。

``assert(bool condition)``
    如果条件不满足，则会导致 Panic 错误，从而恢复状态更改 - 用于内部错误。

``require(bool condition)``
    如果条件不满足，则回滚 - 用于输入或外部组件中的错误。

``require(bool condition, string memory message)``
    如果条件不满足，则回滚 - 用于输入或外部组件中的错误。还提供错误信息。

``revert()``
    中止执行并恢复状态更改

``revert(string memory reason)``
    中止执行并恢复状态更改，提供解释性字符串

.. index:: keccak256, ripemd160, sha256, ecrecover, addmod, mulmod, cryptography,

.. _mathematical-and-cryptographic-functions:

数学和密码方法
----------------------------------------

``addmod(uint x, uint y, uint k) returns (uint)``
    计算 ``(x + y) % k`` ，其中加法以任意精度执行，并且不会在 ``2**256`` 处被截取。从版本 0.5.0 开始加入 断言 ``k != 0`` 。

``mulmod(uint x, uint y, uint k) returns (uint)``
    计算 ``(x * y) % k`` ，其中乘法以任意精度执行，并且不会在 ``2**256`` 处被截取。从版本 0.5.0 开始加入 断言 ``k != 0`` 。

``keccak256(bytes memory) returns (bytes32)``
    计算输入的 Keccak-256 哈希

.. 注解::

    曾经有一个 ``keccak256`` 的别名叫做 ``sha3``，它在 0.5.0 版本中被删除了。

``sha256(bytes memory) returns (bytes32)``
    计算输入的 SHA-256 哈希

``ripemd160(bytes memory) returns (bytes20)``
    计算输入的 RIPEMD-160 哈希

``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``
    从椭圆曲线签名中恢复与公钥关联的地址或在出错时返回零。
    方法参数对应于签名的 ECDSA 值：

    * ``r`` = 签名的前 32 个字节
    * ``s`` = 第二个 32 字节的签名
    * ``v`` = 签名的最后 1 个字节

    ``ecrecover`` 返回一个 ``address``，而不是 ``address payable``。 如果您需要将资金转移到恢复的地址，请参阅 :ref:`address paid<address>` 进行转换。

    有关更多详细信息，请阅读 `示例用法 <https://ethereum.stackexchange.com/questions/1777/workflow-on-signing-a-string-with-private-key-followed-by-signature-verificatio>`_。

.. 警告::

    如果您使用 ``ecrecover``，请注意有效签名可以转换为不同的有效签名，而无需知道相应的私钥。 在 Homestead 硬分叉中，此问题已针对 _transaction_ 签名修复（参见 `EIP-2 <https://eips.ethereum.org/EIPS/eip-2#specification>`_），但 ``ecrecover`` 功能保持不变。

    这通常不是问题，除非您要求签名是唯一的或使用它们来识别项目。OpenZeppelin 有一个 `ECDSA 帮助程序库 <https://docs.openzeppelin.com/contracts/2.x/api/cryptography#ECDSA>`_，您可以将其用作 ``ecrecover`` 的包装器而不会出现此问题。

.. 注解::

    在 *私有区块链* 上运行 ``sha256``、 ``ripemd160`` 或 ``ecrecover`` 时，您可能会遇到 Out-of-Gas 的情况。 这是因为这些方法被实现为 ``预编译合约``，并且只有在它们收到第一条消息后才真正存在（尽管它们的合约代码是硬编码的）。向不存在的合约发送消息的成本更高，因此执行可能会遇到 Out-of-Gas 错误。 此问题的解决方法是先将 Wei（例如 1）发送到每个合约，然后再在实际合约中使用它们。这不是主网上或测试网上的问题。

.. index:: balance, codehash, send, transfer, call, callcode, delegatecall, staticcall

.. _address_related:

地址类型的成员
------------------------

``<address>.balance`` (``uint256``)
    以 Wei 为单位的 :ref:`address` 的余额

``<address>.code`` (``bytes memory``)
    :ref:`address` 的代码（可以为空）

``<address>.codehash`` (``bytes32``)
    :ref:`address` 的codehash

``<address payable>.transfer(uint256 amount)``
    发送指定数量的 Wei 到 :ref:`address`，失败时回滚，发送2300 gas的费用，不可调整

``<address payable>.send(uint256 amount) returns (bool)``
    发送指定数量的 Wei 到 :ref:`address`，失败返回 ``false``，发送2300 gas的费用，不可调整

``<address>.call(bytes memory) returns (bool, bytes memory)``
    使用指定的有效payload 发出低级别的 ``CALL``，返回成功条件并返回数据，发送所有可用的gas，可调整

``<address>.delegatecall(bytes memory) returns (bool, bytes memory)``
    使用指定的有效payload 发出低级别的 ``DELEGATECALL``，返回成功条件并返回数据，发送所有可用的gas，可调整

``<address>.staticcall(bytes memory) returns (bool, bytes memory)``
    使用指定的有效payload 发出低级别的 ``STATICCALL``，返回成功条件并返回数据，发送所有可用的gas，可调整

有关更多信息，请参阅 :ref:`address` 部分。

.. 警告::
    在执行另一个合约方法时，您应该尽可能避免使用 ``.call()`` ，因为它会绕过类型检查、函数存在性检查和参数打包。

.. 警告::
    使用 ``send`` 存在一些危险：如果调用堆栈深度为 1024（这总是可以由调用者强制），则传输会失败，并且如果接收者耗尽 gas，传输也会失败。 因此，为了进行安全的以太转账，请始终检查 ``send`` 的返回值，使用 ``transfer`` 会更好：使用收款人提取资金的模式。

.. 警告::
    由于 EVM 认为对不存在的合约的调用总是成功的，Solidity 在执行外部调用时包括使用 ``extcodesize`` 操作码的额外检查。
    这确保了即将被调用的合约要么实际存在（它包含代码）要么引发异常。

    对地址而不是合约实例进行操作的低级调用（即 ``.call()``、 ``.delegatecall()``、 ``.staticcall()``、 ``.send()`` 和 ``.transfer()``) **不包括** 这个检查，这使得它们在gas方面更便宜但也不太安全。

.. 注解::
   在 0.5.0 版本之前，Solidity 允许合约实例访问地址成员，例如 ``this.balance``。
   现在禁止这样做，必须进行显式转换为地址： ``address(this).balance``。

.. 注解::
   如果通过低级委托调用访问状态变量，则两个合约的存储布局必须对齐，以便被调用合约按名称正确访问调用合约的存储变量。
   如果像高级库那样将存储指针作为方法参数传递，则不需如此。

.. 注解::
    在 0.5.0 版本之前，``.call``、 ``.delegatecall`` 和 ``.staticcall`` 只返回成功状态而不返回数据。

.. 注解::
    在 0.5.0 版本之前，有一个名为 ``callcode`` 的成员，其语义与 ``delegatecall`` 相似但略有不同。


.. index:: this, selfdestruct

合约相关
----------------

``this`` (当前合约类型)
    当前合约，显式转换为 :ref:`address`

``selfdestruct(应付收件人地址)``
    销毁当前合约，将其资金发送给指定的 :ref:`address` 并结束执行。
    Note that ``selfdestruct`` has some peculiarities inherited from the EVM:
    请注意， ``selfdestruct`` 有一些继承自 EVM 的特性：

    - 不执行接收合约的receive函数。
    - 合约仅在交易结束时才真正被销毁，并且 ``revert`` 可能会 "撤消" 销毁。

此外，当前合约的所有方法都可以直接调用，包括当前方法。

.. 注解::
    在 0.5.0 版本之前，有一个名为 ``suicide`` 的函数，其语义与 ``selfdestruct`` 相同。

.. index:: type, creationCode, runtimeCode

.. _meta-type:

类型信息
----------------

表达式 ``type(X)`` 可用于检索有关类型 ``X`` 的信息。目前，对该方法的支持有限（ ``X`` 可以是合约或整数类型），但将来可能会扩展。

The following properties are available for a contract type ``C``:
以下属性可用于合约类型 ``C``:

``type(C).name``
    合约名称

``type(C).creationCode``
    包含合约创建字节码的内存字节数组。
    这可用于内联汇编以构建自定义创建例程，尤其是通过使用 ``create2`` 操作码。
    该属性在合约本身或任何派生合约中 **不能访问**。它将导致字节码包含在调用合约的字节码中，因此会引起循环引用。

``type(C).runtimeCode``
    包含合约运行时字节码的内存字节数组。
    这是通常由 ``C`` 的构造函数部署的代码。
    如果 ``C`` 有一个使用内联汇编的构造函数，这可能与实际部署的字节码不同。另请注意，库在部署时修改其运行时字节码以防止常规调用。
    与 ``.creationCode`` 相同的限制也适用于此属性。

除了上述属性外，以下属性可用于接口类型 ``I``:

``type(I).interfaceId``:
    包含 `EIP-165 <https://eips.ethereum.org/EIPS/eip-165>`_ 的 `bytes4` 值
    指定接口 ``I`` 的接口标识符。 这个标识符被定义为 ``XOR``（异或）接口本身定义的所有方法选择器 —— 不包括所有继承的方法。

以下属性可用于整数类型 ``T``:

``type(T).min``
    类型 ``T``可表示的最小值。

``type(T).max``
    类型 ``T``可表示的最大值。
