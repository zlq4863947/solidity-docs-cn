.. include:: README.rst

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

   README.rst
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
