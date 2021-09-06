Solidity中文开发文档 v0.8.8
========

最新翻译日期: 2021-09-06

.. image:: logo.svg
    :width: 120px
    :alt: Solidity logo
    :align: center

Solidity 是一种面向对象的高级语言，用于实现智能合约。智能合约是管理以太坊状态内账户行为的程序。

Solidity 是一种 `大括号编程语言 <https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Curly-bracket_languages>`_。
它受 C++、Python 和 JavaScript 的影响，旨在针对以太坊虚拟机 (EVM)运行智能合约。
您可以找到有关 Solidity 受哪些语言启发的更多详细信息
:doc:`语言影响 <language-influences>` 部分。

Solidity 是静态类型的，支持继承、库和复杂的用户定义类型以及其他功能。

使用 Solidity，您可以创建投票、众筹、盲拍、和多重签名钱包。

部署合约时，应使用最新发布的Solidity 版本。这是为了防止破坏性变化以及定期推出新功能和错误修复。
我们目前使用一个 0.x 版本号 `以表明版本的变化 <https://semver.org/lang/zh-CN/#spec-item-4>`_。

.. 警告::

  Solidity 最近发布了 0.8.x 版本，引入了很多破坏性变化。请务必阅读 :doc:`完整列表 <080-breaking-changes>`。

欢迎提出改进 Solidity 或本文档的想法。

入门
---------------

**1. 理解智能合约基础概念**

如果您不熟悉智能合约的概念，我们建议您从智能合约简介部分开始出现，其中包括:

* :ref:`一个用 Solidity 编写的简单智能合约示例 <simple-smart-contract>`。
* :ref:`区块链基础知识 <blockchain-basics>`。
* :ref:`以太坊虚拟机 <the-ethereum-virtual-machine>`。

**2. 了解 Solidity**

一旦您熟悉了基础知识，我们建议您阅读 :doc:`"Solidity例子" <solidity-by-example>`
和“语言描述”部分，以了解语言的核心概念。

**3. 安装 Solidity 编译器**

安装 Solidity 编译器有多种方法，
只需选择您喜欢的方法，然后按照 :ref:`安装页面 <installing-solidity>` 中列出的步骤进行操作。

.. 提示::
  您可以使用以下命令直接在浏览器中试用代码示例
  `Remix IDE <https://remix.ethereum.org>`_。 Remix 是一个基于 Web 浏览器的 IDE
  允许您编写、部署和管理 Solidity 智能合约，而无需在本地安装 Solidity。

.. 警告::
    人编写软件时，难免会有错误。您应该遵循既定的
    编写智能合约时的软件开发最佳实践。 这
    包括代码审查、测试、审计和正确性证明。智能合约
    用户有时比他们的作者对代码更有信心，并且
    区块链和智能合约有其独特的问题
    注意，所以在处理生产代码之前，请确保您阅读了
    :ref:`安全注意事项 <security_considerations>` 部分。

**4. 了解更多**

如果您想了解有关在以太坊上构建去中心化应用程序的更多信息，
`以太坊开发者资源 <https://ethereum.org/zh/developers/>`_
可以帮助您提供有关以太坊的更多说明文档，以及各种教程，工具和开发框架。

如果您有任何问题，可以尝试在此连接搜索答案
`以太坊 StackExchange <https://ethereum.stackexchange.com/>`_，或
我们的 `Gitter 频道 <https://gitter.im/ethereum/solidity/>`_。

.. _translations:

内容
========

:ref:`关键词索引 <genindex>`

.. toctree::
   :maxdepth: 2
   :caption: 基础

   introduction-to-smart-contracts.rst
   installing-solidity.rst
   solidity-by-example.rst

.. toctree::
   :maxdepth: 2
   :caption: Language Description

   layout-of-source-files.rst
   structure-of-a-contract.rst
   types.rst
   units-and-global-variables.rst
   control-structures.rst
   contracts.rst
   assembly.rst
   cheatsheet.rst
   grammar.rst

.. toctree::
   :maxdepth: 2
   :caption: Compiler

   using-the-compiler.rst
   analysing-compilation-output.rst

.. toctree::
   :maxdepth: 2
   :caption: Internals

   internals/layout_in_storage.rst
   internals/layout_in_memory.rst
   internals/layout_in_calldata.rst
   internals/variable_cleanup.rst
   internals/source_mappings.rst
   internals/optimizer.rst
   metadata.rst
   abi-spec.rst

.. toctree::
   :maxdepth: 2
   :caption: Additional Material

   050-breaking-changes.rst
   060-breaking-changes.rst
   070-breaking-changes.rst
   080-breaking-changes.rst
   natspec-format.rst
   security-considerations.rst
   smtchecker.rst
   resources.rst
   path-resolution.rst
   yul.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   brand-guide.rst
   language-influences.rst
