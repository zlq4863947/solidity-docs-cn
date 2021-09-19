.. index:: contract;modular, modular contract

*****************
模块化合约
*****************

构建合约的模块化方法可帮助您降低复杂性并提高可读性，这将有助于在开发和代码审查期间发现错误和漏洞。
如果您单独指定和控制每个模块的行为，则您必须考虑的交互仅是模块规范之间的交互，而不应将它放到合约部分中。

在下面的示例中，合约使用 ``Balances`` :ref:`library <libraries>` 的 ``move`` 方法来检查地址之间发送的余额是否与您期望的相符。通过这种方式，``Balances`` 库提供了一个独立的组件，可以正确跟踪账户余额。
很容易验证 ``Balances`` 库永远不会产生负余额或溢出，并且所有余额的总和在合约的整个生命周期中都是不变的。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.5.0 <0.9.0;

    library Balances {
        function move(mapping(address => uint256) storage balances, address from, address to, uint amount) internal {
            require(balances[from] >= amount);
            require(balances[to] + amount >= balances[to]);
            balances[from] -= amount;
            balances[to] += amount;
        }
    }

    contract Token {
        mapping(address => uint256) balances;
        using Balances for *;
        mapping(address => mapping (address => uint256)) allowed;

        event Transfer(address from, address to, uint amount);
        event Approval(address owner, address spender, uint amount);

        function transfer(address to, uint amount) external returns (bool success) {
            balances.move(msg.sender, to, amount);
            emit Transfer(msg.sender, to, amount);
            return true;

        }

        function transferFrom(address from, address to, uint amount) external returns (bool success) {
            require(allowed[from][msg.sender] >= amount);
            allowed[from][msg.sender] -= amount;
            balances.move(from, to, amount);
            emit Transfer(from, to, amount);
            return true;
        }

        function approve(address spender, uint tokens) external returns (bool success) {
            require(allowed[msg.sender][spender] == 0, "");
            allowed[msg.sender][spender] = tokens;
            emit Approval(msg.sender, spender, tokens);
            return true;
        }

        function balanceOf(address tokenOwner) external view returns (uint balance) {
            return balances[tokenOwner];
        }
    }
