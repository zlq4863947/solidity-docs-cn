.. include:: README.rst

入门
---------------

**1. 理解智能合约基础概念**

如果您不熟悉智能合约的概念，我们建议您从智能合约简介部分开始出现，其中包括:

* :ref:`一个用 Solidity 编写的简单智能合约示例 <simple-smart-contract>`。
* :ref:`区块链基础知识 <blockchain-basics>`。
* :ref:`以太坊虚拟机 <the-ethereum-virtual-machine>`。

**2. 了解 Solidity**

一旦您熟悉了基础知识，我们建议您阅读 :doc:`"Solidity例子" <book/solidity-by-example>`
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

   README.rst
   book/introduction-to-smart-contracts.rst
   book/installing-solidity.rst
   book/solidity-by-example.rst

.. toctree::
   :maxdepth: 2
   :caption: Language Description

   book/layout-of-source-files.rst
   book/structure-of-a-contract.rst
   book/types.rst
   book/units-and-global-variables.rst
   book/control-structures.rst
   book/contracts.rst
   book/assembly.rst
   book/cheatsheet.rst
   book/grammar.rst

.. toctree::
   :maxdepth: 2
   :caption: Compiler

   book/using-the-compiler.rst
   book/analysing-compilation-output.rst

.. toctree::
   :maxdepth: 2
   :caption: Internals

   book/internals/layout_in_storage.rst
   book/internals/layout_in_memory.rst
   book/internals/layout_in_calldata.rst
   book/internals/variable_cleanup.rst
   book/internals/source_mappings.rst
   book/internals/optimizer.rst
   book/metadata.rst
   book/abi-spec.rst

.. toctree::
   :maxdepth: 2
   :caption: Additional Material

   book/050-breaking-changes.rst
   book/060-breaking-changes.rst
   book/070-breaking-changes.rst
   book/080-breaking-changes.rst
   book/natspec-format.rst
   book/security-considerations.rst
   book/smtchecker.rst
   book/resources.rst
   book/path-resolution.rst
   book/yul.rst
   book/style-guide.rst
   book/common-patterns.rst
   book/bugs.rst
   book/contributing.rst
   book/brand-guide.rst
   book/language-influences.rst
