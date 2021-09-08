.. index:: ! installing

.. _installing-solidity:

################################
安装 Solidity 编译器
################################

版本控制
==========

Solidity 版本遵循`语义版本控制 <https://semver.org>`_ 并且除了发布之外，还提供 **每日开发构建**。

不保证每日构建都能正常工作，尽管尽了最大努力，它们仍可能包含未记录和/或损坏的更改。我们建议使用最新发布的版本。下面的软件包安装程序将使用最新版本。

Remix
=====

*我们推荐 Remix 用于小型合约和快速学习 Solidity。*

`在线访问Remix <https://remix.ethereum.org/>`_，你不需要安装任何东西。
如果您想在不连接 **互联网** 的情况下使用它，请转到 https://github.com/ethereum/remix-live/tree/gh-pages 并下载该页面上的 ``.zip`` 文件。 Remix 也是测试夜间构建的便捷选项，无需安装多个 Solidity 版本。

.. _solcjs:

npm / Node.js
=============

使用 ``npm`` 以方便且可移植的方式安装 ``solcjs``，一个 Solidity 编译器。 `solcjs` 程序的功能比本页后面描述的访问编译器的方法要少。 :ref:`commandline-compiler` 文档假定您使用的是全功能编译器 ``solc``。 ``solcjs`` 的用法记录在它自己的 `repository <https://github.com/ethereum/solc-js>`_ 中。

注意：solc-js 项目是通过使用 Emscripten 从 C++ `solc` 派生的，这意味着两者使用相同的编译器源代码。
`solc-js` 可以直接在 JavaScript 项目中使用（比如 Remix）。
有关说明，请参阅 solc-js 资源库。

.. code-block:: bash

    npm install -g solc

.. 注解::

    在命令行中，使用 ``solcjs`` 而非 ``solc`` 。 ``solcjs`` 的命令行选项同 ``solc`` 和一些工具（如 ``geth`` )是不兼容的，因此不要期望 ``solcjs`` 能像 ``solc`` 一样工作。

Docker
======

Solidity 构建的 Docker 镜像可以使用来自 ``ethereum`` 组织的 ``solc`` 镜像。
对于最新发布的版本，使用 ``stable`` 标签，对于 develop 分支中潜在的不稳定变更，使用 ``nightly`` 标签。

Docker 镜像运行编译器可执行文件，因此您可以将所有编译器参数传递给它。
例如，下面的命令拉取 ``solc`` 镜像的稳定版本（如果您还没有它），在新容器中运行它，并传递 ``--help`` 参数。

.. code-block:: bash

    docker run ethereum/solc:stable --help

您还可以在标记中指定发布构建版本，例如，指定 0.5.4 版本。

.. code-block:: bash

    docker run ethereum/solc:0.5.4 --help

使用Docker镜像在宿主机上编译Solidity文件需要在本地挂载一个输入输出文件夹，并指定要编译的合约。 例如：

.. code-block:: bash

    docker run -v /local/path:/sources ethereum/solc:stable -o /sources/output --abi --bin /sources/Contract.sol

您还可以使用标准的 JSON 接口（推荐使用在带有工具的编译器）。
使用此接口时，只要 JSON 输入是自包含的，就不需要挂载任何目录（即它不引用任何必须 :ref:`由导入回调加载 <initial-vfs-content-standard-json-with-import-callback>`）的外部文件。

.. code-block:: bash

    docker run ethereum/solc:stable --standard-json < input.json > output.json

Linux 软件包
==============

Solidity 的二进制包可在 `solidity/releases <https://github.com/ethereum/solidity/releases>`_ 获得。

我们也有适用于 Ubuntu 的 PPA，您可以使用以下命令获得最新的稳定版本：

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install solc

可以使用以下命令安装nightly版本：

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo add-apt-repository ppa:ethereum/ethereum-dev
    sudo apt-get update
    sudo apt-get install solc

我们还发布了一个 `snap包 <https://snapcraft.io/>`_，它可以安装在所有 `支持的 Linux 发行版 <https://snapcraft.io/docs/core/install>`_ 中。 要安装最新的稳定版 solc：

.. code-block:: bash

    sudo snap install solc

如果您想帮助测试具有最新更改的 Solidity 的最新开发版本，请使用以下命令：

.. code-block:: bash

    sudo snap install solc --edge

.. 注解::

    ``solc`` 使用严格的限制。 这是 snap 包最安全的模式，它有一些限制，比如只能访问 ``/home`` 和 ``/media`` 目录中的文件。
    有关更多信息，请转到 `揭秘 Snap Confinement <https://snapcraft.io/blog/demystifying-snap-confinement>`_。

Arch Linux 也有软件包，但仅限于最新的开发版本：

.. code-block:: bash

    pacman -S solidity

Gentoo Linux 有一个包含 Solidity包的 `以太坊覆盖 <https://overlays.gentoo.org/#ethereum>`_。
覆盖设置完成后，可以通过以下方式在 x86_64 架构中安装 ``solc``：

.. code-block:: bash

    emerge dev-lang/solidity

macOS软件包
==============

我们通过 Homebrew 将 Solidity 编译器作为从源代码构建的版本分发。目前不支持Pre-built bottles。

.. code-block:: bash

    brew update
    brew upgrade
    brew tap ethereum/ethereum
    brew install solidity

要安装最新的 0.4.x / 0.5.x 版本的 Solidity，您还可以分别使用 ``brew install solidity@4`` 和 ``brew install solidity@5``。

If you need a specific version of Solidity you can install a Homebrew formula directly from Github.
如果您需要特定版本的 Solidity，您可以直接从 Github 安装 Homebrew formula。

参照: `solidity.rb commits on Github <https://github.com/ethereum/homebrew-ethereum/commits/master/solidity.rb>`_.

复制您想要的版本的提交哈希并在您的机器上检出它。

.. code-block:: bash

    git clone https://github.com/ethereum/homebrew-ethereum.git
    cd homebrew-ethereum
    git checkout <your-hash-goes-here>

使用 ``brew`` 安装它：

.. code-block:: bash

    brew unlink solidity
    # eg. Install 0.4.8
    brew install solidity.rb

静态二进制文件
===============

我们在 `solc-bin`_ 维护一个包含所有支持平台的过去和当前编译器版本的静态构建的存储库。 这也是您可以找到夜间构建的位置。

该存储库不仅是最终用户准备开箱即用的二进制文件的一种快速简便的方法，而且还意味着对第三方工具友好：

- 内容镜像到 https://binaries.soliditylang.org，可以通过 HTTPS 轻松下载，无需任何身份验证、速率限制或需要使用 git。
- 内容使用正确的 `Content-Type` 标头和宽松的 CORS 配置提供，以便它可以由浏览器中运行的工具直接加载。
- 二进制文件不需要安装或解包（与必要的 DLL 捆绑在一起的旧版 Windows 版本除外）。
- 我们力求高水平的向后兼容性。 文件一旦添加，就不会删除或移动，而无需在旧位置提供符号链接/重定向。 它们也永远不会就地修改，并且应始终与原始校验和匹配。 唯一的例外是损坏或无法使用的文件，如果保持原样，可能会造成弊大于利。
- 文件通过 HTTP 和 HTTPS 提供。 只要您以安全的方式（通过 git、HTTPS、IPFS 或仅将其缓存在本地）获取文件列表并在下载后验证二进制文件的哈希值，您就不必为二进制文件本身使用 HTTPS。

大多数情况下，相同的二进制文件可在 Github 上的 `Solidity 发布页面`_上找到。不同之处在于我们通常不会在 Github 发布页面上更新旧版本。这意味着如果命名约定发生变化，我们不会重命名它们，并且我们不会为发布时不支持的平台添加构建。这只发生在 ``solc-bin`` 中。

``solc-bin`` 存储库包含几个顶级目录，每个目录代表一个平台。
每个都包含一个 ``list.json`` 文件，列出了可用的二进制文件。 例如在 ``emscripten-wasm32/list.json`` 中，您将找到有关 0.7.4 版本的以下信息：

.. code-block:: json

    {
      "path": "solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js",
      "version": "0.7.4",
      "build": "commit.3f05b770",
      "longVersion": "0.7.4+commit.3f05b770",
      "keccak256": "0x300330ecd127756b824aa13e843cb1f43c473cb22eaf3750d5fb9c99279af8c3",
      "sha256": "0x2b55ed5fec4d9625b6c7b3ab1abd2b7fb7dd2a9c68543bf0323db2c7e2d55af2",
      "urls": [
        "bzzr://16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1",
        "dweb:/ipfs/QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS"
      ]
    }

This means that:

- You can find the binary in the same directory under the name
  `solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js <https://github.com/ethereum/solc-bin/blob/gh-pages/emscripten-wasm32/solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js>`_.
  Note that the file might be a symlink, and you will need to resolve it yourself if you are not using
  git to download it or your file system does not support symlinks.
- The binary is also mirrored at https://binaries.soliditylang.org/emscripten-wasm32/solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js.
  In this case git is not necessary and symlinks are resolved transparently, either by serving a copy
  of the file or returning a HTTP redirect.
- The file is also available on IPFS at `QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS`_.
- The file might in future be available on Swarm at `16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1`_.
- You can verify the integrity of the binary by comparing its keccak256 hash to
  ``0x300330ecd127756b824aa13e843cb1f43c473cb22eaf3750d5fb9c99279af8c3``.  The hash can be computed
  on the command line using ``keccak256sum`` utility provided by `sha3sum`_ or `keccak256() function
  from ethereumjs-util`_ in JavaScript.

.. warning::

   Due to the strong backwards compatibility requirement the repository contains some legacy elements
   but you should avoid using them when writing new tools:

   - Use ``emscripten-wasm32/`` (with a fallback to ``emscripten-asmjs/``) instead of ``bin/`` if
     you want the best performance. Until version 0.6.1 we only provided asm.js binaries.
     Starting with 0.6.2 we switched to `WebAssembly builds`_ with much better performance. We have
     rebuilt the older versions for wasm but the original asm.js files remain in ``bin/``.
     The new ones had to be placed in a separate directory to avoid name clashes.
   - Use ``emscripten-asmjs/`` and ``emscripten-wasm32/`` instead of ``bin/`` and ``wasm/`` directories
     if you want to be sure whether you are downloading a wasm or an asm.js binary.
   - Use ``list.json`` instead of ``list.js`` and ``list.txt``. The JSON list format contains all
     the information from the old ones and more.
   - Use https://binaries.soliditylang.org instead of https://solc-bin.ethereum.org. To keep things
     simple we moved almost everything related to the compiler under the new ``soliditylang.org``
     domain and this applies to ``solc-bin`` too. While the new domain is recommended, the old one
     is still fully supported and guaranteed to point at the same location.

.. warning::

    The binaries are also available at https://ethereum.github.io/solc-bin/ but this page
    stopped being updated just after the release of version 0.7.2, will not receive any new releases
    or nightly builds for any platform and does not serve the new directory structure, including
    non-emscripten builds.

    If you are using it, please switch to https://binaries.soliditylang.org, which is a drop-in
    replacement. This allows us to make changes to the underlying hosting in a transparent way and
    minimize disruption. Unlike the ``ethereum.github.io`` domain, which we do not have any control
    over, ``binaries.soliditylang.org`` is guaranteed to work and maintain the same URL structure
    in the long-term.

.. _IPFS: https://ipfs.io
.. _Swarm: https://swarm-gateways.net/bzz:/swarm.eth
.. _solc-bin: https://github.com/ethereum/solc-bin/
.. _Solidity release page on github: https://github.com/ethereum/solidity/releases
.. _sha3sum: https://github.com/maandree/sha3sum
.. _keccak256() function from ethereumjs-util: https://github.com/ethereumjs/ethereumjs-util/blob/master/docs/modules/_hash_.md#const-keccak256
.. _WebAssembly builds: https://emscripten.org/docs/compiling/WebAssembly.html
.. _QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS: https://gateway.ipfs.io/ipfs/QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS
.. _16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1: https://swarm-gateways.net/bzz:/16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1/


.. _building-from-source:

从源代码编译
====================

预先安装环境 - 所有平台
-------------------------------------

以下是所有编译Solidity的依赖关系:

+-----------------------------------+-------------------------------------------------------+
| 软件                          | 备注                                                 |
+===================================+=======================================================+
| `CMake`_ (version 3.13+)           | Cross-platform build file generator.                  |
+-----------------------------------+-------------------------------------------------------+
| `Boost`_  (version 1.65+)         | C++ libraries.                                        |
+-----------------------------------+-------------------------------------------------------+
| `Git`_                            | 获取源代码的命令行工具         |
+-----------------------------------+-------------------------------------------------------+
| `z3`_ (version 4.8+, Optional)    | For use with SMT checker.                             |
+-----------------------------------+-------------------------------------------------------+
| `cvc4`_ (Optional)                | For use with SMT checker.                             |
+-----------------------------------+-------------------------------------------------------+

.. _cvc4: https://cvc4.cs.stanford.edu/web/
.. _Git: https://git-scm.com/download
.. _Boost: https://www.boost.org
.. _CMake: https://cmake.org/download/
.. _z3: https://github.com/Z3Prover/z3

.. note::
    Solidity versions prior to 0.5.10 can fail to correctly link against Boost versions 1.70+.
    A possible workaround is to temporarily rename ``<Boost install path>/lib/cmake/Boost-1.70.0``
    prior to running the cmake command to configure solidity.

    Starting from 0.5.10 linking against Boost 1.70+ should work without manual intervention.

最低的编译器版本
^^^^^^^^^^^^^^^^^^^^^^^^^

The following C++ compilers and their minimum versions can build the Solidity codebase:

- `GCC <https://gcc.gnu.org>`_, version 8+
- `Clang <https://clang.llvm.org/>`_, version 7+
- `MSVC <https://visualstudio.microsoft.com/vs/>`_, version 2019+



环境依赖条件 - macOS
---------------------

在 macOS 中，需确保有安装最新版的
`Xcode <https://developer.apple.com/xcode/download/>`_，
Xcode 包含 `Clang C++ 编译器 <https://en.wikipedia.org/wiki/Clang>`_， 而
`Xcode IDE <https://en.wikipedia.org/wiki/Xcode>`_ 和其他苹果 OS X 下编译 C++ 应用所必须的开发工具。
如果你是第一次安装 Xcode 或者刚好更新了 Xcode 新版本，则在使用命令行构建前，需同意 Xcode 的使用协议：

.. code:: bash

    sudo xcodebuild -license accept

Solidity 在 OS X 下构建，必须 `安装 Homebrew <https://brew.sh>`_
包管理器来安装依赖。
如果你想从头开始，这里是 `卸载 Homebrew 的方法
<https://docs.brew.sh/FAQ#how-do-i-uninstall-homebrew>`_。


环境依赖条件 - Windows
-----------------------

在Windows下构建Solidity，需下载的依赖软件包：

+-----------------------------------+-------------------------------------------------------+
| 软件                              | 备注                                                  |
+===================================+=======================================================+
| `Visual Studio 2019 Build Tools`_ | C++ 编译器                                            |
+-----------------------------------+-------------------------------------------------------+
| `Visual Studio 2019`_  (Optional) | C++ 编译器和开发环境                                  |
+-----------------------------------+-------------------------------------------------------+

如果你已经有了 IDE，仅需要编译器和相关的库，你可以安装 Visual Studio 2019 Build Tools。

Visual Studio 2019 提供了 IDE 以及必要的编译器和库。所以如果你还没有一个 IDE 并且想要开发 Solidity，那么 Visual Studio 2019 将是一个可以使你获得所有工具的简单选择。

这里是一个在 Visual Studio 2019 Build Tools 或 Visual Studio 2019 中应该安装的组件列表：

* Visual Studio C++ core features
* VC++ 2019 v141 toolset (x86,x64)
* Windows Universal CRT SDK
* Windows 8.1 SDK
* C++/CLI support

.. _Git for Windows: https://git-scm.com/download/win
.. _CMake: https://cmake.org/download/
.. _Visual Studio 2019: https://www.visualstudio.com/vs/
.. _Visual Studio 2019 Build Tools: https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2019


依赖的帮助脚本
---------------------

在 macOS、Windows和其他 Linux 发行版上，有一个脚本可以“一键”安装所需的外部依赖库。本来是需要人工参与的多步操作，现在只需一行命令:

.. code:: bash

    ./scripts/install_deps.sh

Windows 下执行：

.. code:: bat

    scripts\install_deps.ps1

请注意，后一个命令将在``deps``子目录中安装  ``boost`` 和``cmake``，而前一个命令将尝试在全局安装依赖项。



克隆代码库
--------------------

执行以下命令，克隆源代码：

.. code:: bash

    git clone --recursive https://github.com/ethereum/solidity.git
    cd solidity

如果你想参与 Solidity 的开发, 你可分叉 Solidity 源码库后，用你个人的分叉库作为第二远程源：

.. code:: bash

    git remote add personal git@github.com:[username]/solidity.git


.. note::
    This method will result in a prerelease build leading to e.g. a flag
    being set in each bytecode produced by such a compiler.
    If you want to re-build a released Solidity compiler, then
    please use the source tarball on the github release page:

    https://github.com/ethereum/solidity/releases/download/v0.X.Y/solidity_0.X.Y.tar.gz

    (not the "Source code" provided by github).


Solidity 有 Git 子模块，需确保完全加载它们：

.. code:: bash

    git submodule update --init --recursive


命令行构建
------------------

**确保你已安装外部依赖（见上面）**

Solidity 使用 CMake 来配置构建。你也许想要安装 `cache`_ 来加速重复构建，CMake自动进行这个工作。
Linux、macOS 和其他 Unix系统上的构建方式都差不多：

.. code:: bash

    mkdir build
    cd build
    cmake .. && make

也有更简单的：

.. code:: bash

    #note: 将安装 solc 和 soltest 到 usr/local/bin 目录
    ./scripts/build.sh

对于 Windows 执行：

.. code:: bash

    mkdir build
    cd build
    cmake -G "Visual Studio 16 2019 Win64" ..

如果你想执行 ``./scripts/install_deps.ps1`` 时使用你安装过的boost版本，可以添加参数 ``-DBoost_DIR="..\deps\boost\lib\cmake\Boost-*"`` 和 ``-DCMAKE_MSVC_RUNTIME_LIBRARY=MultiThreaded`` 去调用  ``cmake``.

这组指令的最后一句，会在 build 目录下创建一个 **solidity.sln** 文件，双击后，默认会使用 Visual Studio 打开。我们建议在VS上创建 **RelWithDebugInfo** 配置文件。

或者用命令创建：

.. code:: bash

    cmake --build . --config RelWithDebInfo

CMake参数
=============

如果你对 CMake 命令选项有兴趣，可执行 ``cmake .. -LH`` 进行查看。

.. _smt_solvers_build:

SMT Solvers
-----------
Solidity can be built against SMT solvers and will do so by default if
they are found in the system. Each solver can be disabled by a `cmake` option.

*Note: In some cases, this can also be a potential workaround for build failures.*


Inside the build folder you can disable them, since they are enabled by default:

.. code-block:: bash

    # disables only Z3 SMT Solver.
    cmake .. -DUSE_Z3=OFF

    # disables only CVC4 SMT Solver.
    cmake .. -DUSE_CVC4=OFF

    # disables both Z3 and CVC4
    cmake .. -DUSE_CVC4=OFF -DUSE_Z3=OFF


版本号字符串详解
============================

Solidity 版本名包含四部分：

- 版本号
- 预发布版本号，通常为 ``develop.YYYY.MM.DD`` 或者 ``nightly.YYYY.MM.DD``
- 以 ``commit.GITHASH`` 格式展示的提交号
- 由若干条平台、编译器详细信息构成的平台标识

如果本地有修改，则 commit 部分有后缀 ``.mod``。

这些部分按照 Semver 的要求来组合， Solidity 预发布版本号等价于 Semver 预发布版本号， Solidity 提交号和平台标识则组成 Semver 的构建元数据。

发行版样例：``0.4.8+commit.60cc1668.Emscripten.clang``.

预发布版样例： ``0.4.9-nightly.2017.1.17+commit.6ecb4aa3.Emscripten.clang``

版本信息详情
=====================================

在版本发布之后，补丁版本号会增加，因为我们假定只有补丁级别的变更会在之后发生。当变更被合并后，版本应该根据semver和变更的剧烈程度进行调整。最后，发行版本总是与当前每日构建版本的版本号一致，但没有 ``prerelease`` 指示符。

例如：

0. 0.4.0 版本发布
1. 从现在开始，每晚构建为 0.4.1 版本
2. 引入非破坏性变更 —— 不改变版本号
3. 引入破坏性变更 —— 版本跳跃到 0.5.0
4. 0.5.0 版本发布

该方式与 :ref:`version pragma <version_pragma>` 一起运行良好。
