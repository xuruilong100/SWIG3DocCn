[TOC]

# 1 前言

## 1.1 引言

SWIG (Simplified Wrapper and Interface Generator) is a software development tool for building scripting language interfaces to C and C++ programs. Originally developed in 1995, SWIG was first used by scientists in the Theoretical Physics Division at Los Alamos National Laboratory for building user interfaces to simulation codes running on the Connection Machine 5 supercomputer. In this environment, scientists needed to work with huge amounts of simulation data, complex hardware, and a constantly changing code base. The use of a scripting language interface provided a simple yet highly flexible foundation for solving these types of problems. SWIG simplifies development by largely automating the task of scripting language integration--allowing developers and users to focus on more important problems.

Although SWIG was originally developed for scientific applications, it has since evolved into a general purpose tool that is used in a wide variety of applications--in fact almost anything where C/C++ programming is involved.

> SWIG（Simplified Wrapper and Interface Generator）是一个用于构建 C 和 C++ 程序脚本语言接口的软件开发工具。SWIG 开发于 1995 年，最初由洛斯·阿拉莫斯国家实验室理论物理部门的科学家使用，用于构建运行在 Connection Machine 5 超级计算机上的模拟代码的用户接口。在这种环境中，科学家们需要处理大量的模拟数据、复杂的硬件和不断变化的代码库。脚本语言接口的使用为解决这些类型的问题提供了简单而高度灵活的基础。SWIG 通过大大自动化脚本语言集成任务简化了开发，这允许开发人员和用户专注于更重要的问题。
>
> 尽管 SWIG 最初是为科学应用程序而开发的，但它已经发展成为一种通用工具，可用于各种应用程序，实际上凡是涉及 C/C++ 编程的都可以。

## 1.2 SWIG 版本

In the late 1990's, the most stable version of SWIG was release 1.1p5. Versions 1.3.x were officially development versions and these were released over a period of 10 years starting from the year 2000. The final version in the 1.3.x series was 1.3.40, but in truth the 1.3.x series had been stable for many years. An official stable version was released along with the decision to make SWIG license changes and this gave rise to version 2.0.0 in 2010.

> 在 1990 年代后期，最稳定的 SWIG 版本是 1.1p5 版本。版本 1.3.x 是正式开发版本，从 2000 年开始这些版本陆续在 10 年内发布。1.3.x 系列的最终版本是 1.3.40，但事实上 1.3.x 系列已经稳定了很多年。随着修改 SWIG 许可证的决定发布了官方稳定版本，这导致了 2010 年的版本 2.0.0。

## 1.3 SWIG 许可证

The LICENSE file shipped with SWIG in the top level directory contains the SWIG license. For further insight into the license including the license of SWIG's output code, please visit the SWIG legal page - http://www.swig.org/legal.html.

The license was clarified in version 2.0.0 so that the code that SWIG generated could be distributed under license terms of the user's choice/requirements and at the same time the SWIG source was placed under the GNU General Public License version 3.

> SWIG 在顶层目录中附带的 LICENSE 文件包含 SWIG 许可证。如需进一步了解许可证，包括 SWIG 输出代码的许可证，请访问 SWIG 的法律页面——<http://www.swig.org/legal.html>。
>
> 许可证在 2.0.0 版本中得到了澄清，因此 SWIG 生成的代码可以根据用户选择和要求的许可条款进行发布，同时 SWIG 源代码遵循 GPL 3 许可证。

## 1.4 SWIG 资源

The official location of SWIG related material is

> SWIG 相关资料的官方网址是

```
http://www.swig.org
```

This site contains the latest version of the software, users guide, and information regarding bugs, installation problems, and implementation tricks.

You can also subscribe to the swig-user mailing list by visiting the page

> 该站点包含最新版本的软件、用户指南以及有关错误、安装问题和实现技巧的信息。
>
> 你还可以通过访问该页面订阅 swig-user 邮件列表

```
http://www.swig.org/mail.html
```

The mailing list often discusses some of the more technical aspects of SWIG along with information about beta releases and future work.

Git and Subversion access to the latest version of SWIG is also available. More information about this can be obtained at:

> 邮件列表更多讨论一些 SWIG 的技术内容，以及有关 beta 版本和未来工作的信息。
>
> 也可以通过 Git 和 Subversion 获得最新版本的 SWIG。有关这方面的更多信息，请访问：

```
SWIG Bleeding Edge
```

## 1.5 前提要求

This manual assumes that you know how to write C/C++ programs and that you have at least heard of scripting languages such as Tcl, Python, and Perl. A detailed knowledge of these scripting languages is not required although some familiarity won't hurt. No prior experience with building C extensions to these languages is required---after all, this is what SWIG does automatically. However, you should be reasonably familiar with the use of compilers, linkers, and makefiles since making scripting language extensions is somewhat more complicated than writing a normal C program.

Over time SWIG releases have become significantly more capable in their C++ handling--especially support for advanced features like namespaces, overloaded operators, and templates. Whenever possible, this manual tries to cover the technicalities of this interface. However, this isn't meant to be a tutorial on C++ programming. For many of the gory details, you will almost certainly want to consult a good C++ reference. If you don't program in C++, you may just want to skip those parts of the manual.

> 本手册假定你知道如何编写 C/C++ 程序，并且至少听说过 Tcl、Python 和 Perl 等脚本语言。不要求详细了解这些脚本语言，但熟悉总没坏处。不要求有为这些语言构建 C 扩展的经验——毕竟，这是 SWIG 自动执行的操作。但是，你应该相当熟悉编译器、链接器和 makefile 的使用，因为编写脚本语言扩展比编写普通的 C 程序要复杂一些。
>
> 随着时间的推移，SWIG 在处理 C++ 方面的能力显着提高——尤其是对命名空间、重载运算符和模板等高级功能的支持。只要有可能，本手册将尝试涵盖此接口的技术细节。但是，这并不是一本关于 C++ 编程的教程。对于许多深层的细节，你几乎肯定想要一本很好的 C++ 参考手册。如果你不使用 C++ 编程，你可能想直接跳过本手册的这些部分。

## 1.6 本手册的组织构成

The first few chapters of this manual describe SWIG in general and provide an overview of its capabilities. The remaining chapters are devoted to specific SWIG language modules and are self contained. Thus, if you are using SWIG to build Python interfaces, you can probably skip to that chapter and find almost everything you need to know.

> 本手册的前几章概述了 SWIG，提供了其功能的概览。其余章节专门介绍特定的 SWIG 语言模块，并且它们是自给自足的。因此，如果你使用 SWIG 构建 Python 接口，你可以跳到该章并能找到你需要知道的几乎所有内容。

## 1.7 如何避免阅读手册

If you hate reading manuals, glance at the "Introduction" which contains a few simple examples. These examples contain about 95% of everything you need to know to use SWIG. After that, simply use the language-specific chapters as a reference. The SWIG distribution also comes with a large directory of examples that illustrate different topics.

> 如果你讨厌阅读手册，请浏览一下“引言”，其中包含一些简单的例子。这些示例包含使用 SWIG 所有需要了解内容的 95%。之后，只需使用特定语言的章节作为参考。SWIG 发行版还附带了一个涵盖不同主题的大型示例目录。

## 1.8 向后兼容

If you are a previous user of SWIG, don't expect SWIG to provide complete backwards compatibility. Although the developers strive to the utmost to keep backwards compatibility, this isn't always possible as the primary goal over time is to make SWIG better---a process that would simply be impossible if the developers are constantly bogged down with backwards compatibility issues. Potential incompatibilities are clearly marked in the detailed [release notes](http://swig.org/Doc3.0/Preface.html#Preface_release_notes).

If you need to work with different versions of SWIG and backwards compatibility is an issue, you can use the SWIG_VERSION preprocessor symbol which holds the version of SWIG being executed. SWIG_VERSION is a hexadecimal integer such as 0x010311 (corresponding to SWIG-1.3.11). This can be used in an interface file to define different typemaps, take advantage of different features etc:

> 如果你是 SWIG 的老用户，请不要期望 SWIG 提供完全的向后兼容性。尽管开发人员力求最大程度地保持向后兼容性，但随着时间的推移，这并不总是可行的，因为开发人员的主要目标是使 SWIG 更好用，如果开发人员不断陷入向后兼容性问题，这个任务根本不可能实现。潜在的不兼容性在 [release notes](http://swig.org/Doc3.0/Preface.html#Preface_release_notes) 中有详细的明确标记。
>
> 如果你需要使用不同版本的 SWIG，并且向后兼容性是一个问题，你可以使用 `SWIG_VERSION` 预处理符来固定正在运行的 SWIG 版本。`SWIG_VERSION` 是十六进制整数，例如 `0x010311`（对应于 SWIG-1.3.11）。这可以在接口文件中用于定义不同的类型映射，利用不同的功能等：

```
#if SWIG_VERSION >= 0x010311
/* Use some fancy new feature */
#endif
```

Note: The version symbol is not defined in the generated SWIG wrapper file. The SWIG preprocessor has defined SWIG_VERSION since SWIG-1.3.11.

> 注意：版本符号未在生成的 SWIG 包装器文件中定义。自 SWIG-1.3.11 起，SWIG 才定义了预处理符 `SWIG_VERSION`。

## 1.9 发行说明

The CHANGES.current, CHANGES and RELEASENOTES files shipped with SWIG in the top level directory contain, respectively, detailed release notes for the current version, detailed release notes for previous releases and summary release notes from SWIG-1.3.22 onwards.

> SWIG 在顶层目录中附带的 CHANGES.current、CHANGES 和 RELEASENOTES 文件分别包含当前版本的详细发行说明，以前版本的详细发行说明，以及 SWIG-1.3.22 以后的发行说明摘要。

## 1.10 捐赠

SWIG is an unfunded project that would not be possible without the contributions of many people working in their spare time. If you have benefitted from using SWIG, please consider [Donating to SWIG](http://www.swig.org/donate.html) to keep development going. There have been a large varied number of people who have made contributions at all levels over time. Contributors are mentioned either in the COPYRIGHT file or CHANGES files shipped with SWIG or in submitted bugs.

> SWIG 是一个没有资金支持的项目，没有许多人在业余时间工作的贡献，该项目是不可能的。如果你从使用 SWIG 中受益，请考虑[捐赠给 SWIG](http://www.swig.org/donate.html) 以支持开发。历史上，大量的人员在各种层面上做出贡献。COPYRIGHT 文件中提及了贡献者，SWIG 附带的 CHANGES 文件或提交的错误中也有提及。

## 1.11 错误报告

Although every attempt has been made to make SWIG bug-free, we are also trying to make feature improvements that may introduce bugs. To report a bug, either send mail to the SWIG developer list at the [swig-devel mailing list](http://www.swig.org/mail.html) or report a bug at the [SWIG bug tracker](http://www.swig.org/bugs.html). In your report, be as specific as possible, including (if applicable), error messages, tracebacks (if a core dump occurred), corresponding portions of the SWIG interface file used, and any important pieces of the SWIG generated wrapper code. We can only fix bugs if we know about them.

> 尽管已经尽一切努力使 SWIG 无错误，但我们也试图进行功能改进，这可能会引入错误。要报告错误，请将邮件发送到 [swig-devel 邮件列表](http://www.swig.org/mail.html)上的 SWIG 开发人员列表，或报告 [SWIG 错误跟踪器](http：//www.swig.org/bugs.html)中的错误。你的报告应该尽可能具体，引用（如果适用的话）、错误消息、回溯（如果发生内核问题）、使用的 SWIG 接口文件的相应部分，以及 SWIG 生成的包装器代码的任何重要部分。有了解它们，我们才能修复错误。

## 1.12 安装

### 1.12.1 Windows 安装

Please see the dedicated [Windows chapter](http://swig.org/Doc3.0/Windows.html#Windows) for instructions on installing SWIG on Windows and running the examples. The Windows distribution is called swigwin and includes a prebuilt SWIG executable, swig.exe, included in the top level directory. Otherwise it is exactly the same as the main SWIG distribution. There is no need to download anything else.

> 有关在 Windows 上安装 SWIG 并运行示例的说明，请参阅专用的 [Windows 章节](http://swig.org/Doc3.0/Windows.html#Windows)。Windows 发行版称为 swigwin，包含一个在顶层目录中的预构建 SWIG 可执行文件 swig.exe。它与 SWIG 主分布版本完全相同。无需下载任何其他内容。

### 1.12.2 Unix 安装

These installation instructions are for using the distributed tarball, for example, `swig-3.0.8.tar.gz`. If you wish to build and install from source on Github, extra steps are required. Please see the [Bleeding Edge](http://swig.org/svn.html) page on the SWIG website.

You must use [GNU make](http://www.gnu.org/software/make/) to build and install SWIG.

[PCRE](http://www.pcre.org/) needs to be installed on your system to build SWIG, in particular pcre-config must be available. If you have PCRE headers and libraries but not pcre-config itself or, alternatively, wish to override the compiler or linker flags returned by pcre-config, you may set PCRE_LIBS and PCRE_CFLAGS variables to be used instead. And if you don't have PCRE at all, the configure script will provide instructions for obtaining it.

To build and install SWIG, simply type the following:

> 这些安装说明适用于使用发布的压缩文件，例如 `swig-3.0.8.tar.gz`。如果你希望通过 Github 从源代码构建和安装，则需要执行额外的步骤。请参阅 SWIG 网站上的 [Bleeding Edge](http://swig.org/svn.html) 页面。
>
> 你必须使用 [GNU make](http://www.gnu.org/software/make/) 来构建和安装 SWIG。
>
> 你的系统需要安装 [PCRE](http://www.pcre.org/) 来构建 SWIG，特别是 pcre-config 必须可用。如果你有 PCRE 头文件和库，而不是 pcre-config 本身，或者希望覆盖 pcre-config 返回的编译器或链接器标志，则可以设置 PCRE_LIBS 和PCRE_CFLAGS 变量来代替使用。如果你根本没有 PCRE，配置脚本将提供获取它的说明。
>
> 要构建和安装 SWIG，只需键入以下内容：

```
$ ./configure
$ make
$ make install
```

By default SWIG installs itself in /usr/local. If you need to install SWIG in a different location or in your home directory, use the `--prefix` option to `./configure`. For example:

> 默认情况下，SWIG 会自行安装在 `/usr/local` 中。如果需要在不同的位置或主目录中安装 SWIG，请使用 `./configure` 的 `--prefix` 选项 。例如：

```
$ ./configure --prefix=/home/yourname/projects
$ make
$ make install
```

Note: the directory given to `--prefix` must be an absolute pathname. Do **not** use the ~ shell-escape to refer to your home directory. SWIG won't work properly if you do this.

The INSTALL file shipped in the top level directory details more about using configure. Also try

> 注意：给 `--prefix` 的目录必须是绝对路径名。不要使用 `~` 来引用你的主目录。如果你这样做，SWIG 将无法正常工作。
>
> 顶层目录中提供的 INSTALL 文件详细介绍了使用 configure 的更多信息。也可以试试

```
$ ./configure --help.
```

The configure script will attempt to locate various packages on your machine including Tcl, Perl5, Python and all the other target languages that SWIG supports. Don't panic if you get 'not found' messages -- SWIG does not need these packages to compile or run. The configure script is actually looking for these packages so that you can try out the SWIG examples contained in the 'Examples' directory without having to hack Makefiles. Note that the `--without-xxx` options, where xxx is a target language, have minimal effect. All they do is reduce the amount of testing done with 'make check'. The SWIG executable and library files installed cannot currently be configured with a subset of target languages.

SWIG used to include a set of runtime libraries for some languages for working with multiple modules. These are no longer built during the installation stage. However, users can build them just like any wrapper module as described in the [Modules chapter](http://swig.org/Doc3.0/Modules.html#Modules). The CHANGES file shipped with SWIG in the top level directory also lists some examples which build the runtime library.

Note:

* If you checked the code out via Git, you will have to run `./autogen.sh` before `./configure`. In addition, a full build of SWIG requires a number of packages to be installed. Full instructions at [SWIG bleeding edge](http://www.swig.org/svn.html).

> configure 脚本将尝试在你的机器上找到各种软件包，包括 Tcl、Perl5、Python 和 SWIG 支持的所有其他目标语言。如果你收到“not found”消息，请不要惊慌——SWIG 不需要这些包来编译或运行。配置脚本实际上正在寻找这些包，以便你可以尝试“Examples”目录中包含的 SWIG 示例，而无需破解 Makefile。请注意 `--with-xxx` 选项，其中 `xxx` 是目标语言，具有微小的影响。它们所做的就是减少“make check”所做的测试。目前，安装的 SWIG 可执行文件和库文件无法配置目标语言的子集。
>
> SWIG 过去包含一组用于某些语言的运行时库，用于处理多个模块。这些不再在安装阶段构建。但是，用户可以像[模块章节](http://swig.org/Doc3.0/Modules.html#Modules)中所述的任何包装器模块一样构建它们。SWIG 在顶层目录中附带的 CHANGES 文件还列出了构建运行时库的一些示例。
>
> 注意：
>
> * 如果你通过 Git 检查了代码，则必须在 `./configure` 之前运行 `./autogen.sh`。此外，SWIG 的完整版本需要安装许多软件包。完整说明在 [SWIG Bleeding Edge](http://www.swig.org/svn.html)中。

### 1.12.3 Macintosh OS X 安装

SWIG is known to work on various flavors of OS X. Follow the Unix installation instructions above. However, as of this writing, there is still great deal of inconsistency with how shared libaries are handled by various scripting languages on OS X.

Users of OS X should be aware that Darwin handles shared libraries and linking in a radically different way than most Unix systems. In order to test SWIG and run the examples, SWIG configures itself to use flat namespaces and to allow undefined symbols (`-flat_namespace -undefined suppress`). This mostly closely follows the Unix model and makes it more likely that the SWIG examples will work with whatever installation of software you might have. However, this is generally not the recommended technique for building larger extension modules. Instead, you should utilize Darwin's two-level namespaces. Some details about this can be found here [Understanding Two-Level Namespaces](https://developer.apple.com/library/mac/documentation/Porting/Conceptual/PortingUnix/compiling/compiling.html#//apple_ref/doc/uid/TP40002850-BCIHJBBF).

Needless to say, you might have to experiment a bit to get things working at first.

> 众所周知，SWIG 可以在各种 OS X 上工作。遵循上面的 Unix 安装说明。但是，在撰写本文时，OS X 上的各种脚本语言在如何处理动态库的问题上仍然存在很大的不一致。
>
> OS X 的用户应该意识到 Darwin 处理动态库和链接的方式与大多数 Unix 系统完全不同。为了测试 SWIG 并运行示例，SWIG 将自己配置为使用平面名称空间（flat namespaces）并允许未定义的符号（`-flat_namespace -undefined suppress`）。这主要是遵循 Unix 模型，并且使 SWIG 示例更可能适用于你可能拥有的任何安装软件。但是，这通常不是构建更大扩展模块的推荐技术。相反，你应该使用 Darwin 的两级命名空间。有关此内容的一些详细信息，请参阅[理解两级命名空间](https://developer.apple.com/library/mac/documentation/Porting/Conceptual/PortingUnix/compiling/compiling.html#//apple_ref/doc/uid/TP40002850-BCIHJBBF)。
>
> 毋庸置疑，你可能需要先尝试一下才能运转起来。

### 1.12.4 测试

If you want to test SWIG after building it, a check can be performed on Unix operating systems. Type the following:

> 如果要在构建 SWIG 之后测试，可以在 Unix 操作系统上执行检查。输入以下内容：

```
$ make -k check
```

This step can be performed either before or after installation. The check requires at least one of the target languages to be installed. If it fails, it may mean that you have an uninstalled language module or that the file 'Examples/Makefile' has been incorrectly configured. It may also fail due to compiler issues such as a broken C++ compiler. Even if the check fails, there is a pretty good chance SWIG still works correctly --- you will just have to mess around with one of the examples and some makefiles to get it to work. Some tests may also fail due to missing dependency packages, eg PCRE or Boost, but this will require careful analysis of the configure output done during configuration.

The test suite executed by the check is designed to stress-test many parts of the implementation including obscure corner cases. If some of these tests fail or generate warning messages, there is no reason for alarm --- the test may be related to some new SWIG feature or a difficult bug that we're trying to resolve. Chances are that SWIG will work just fine for you. Note that if you have more than one CPU/core, then you can use parallel make to speed up the check as it does take quite some time to run, for example:

> 此步骤可在安装之前或之后执行。该检查至少需要安装一种目标语言。如果失败，则可能意味着你已卸载语言模块或文件 `Examples/Makefile` 配置错误。它也可能由于编译器问题（例如损坏的 C++ 编译器）而失败。即使检查失败，SWIG 仍然可以正常工作——你只需要使用其中一个示例和一些 makefile 来使它工作。由于缺少依赖包（例如 PCRE 或 Boost），某些测试也可能会失败，但这需要仔细分析配置期间完成的配置输出。
>
> 检查执行的测试套件旨在对实现的许多部分进行压力测试，包括特殊情况。如果其中一些测试失败或生成警告消息，没有理由慌张——测试可能与某些新的 SWIG 功能，或我们试图解决的难题有关。SWIG 依然可能很好地运行。请注意，如果你有多个 CPU 核心，那么你可以使用并行构建来加速检查，因为它需要相当长的时间来运行，例如：

```
$ make -j2 -k check
```

Also, SWIG's support for C++ is sufficiently advanced that certain tests may fail on older C++ compilers (for instance if your compiler does not support member templates). These errors are harmless if you don't intend to use these features in your own programs.

Note: The test-suite currently contains over 500 tests. If you have many different target languages installed and a slow machine, it might take more than an hour to run the test-suite.

> 此外，SWIG 对 C++ 的支持非常先进，以至于某些测试可能会在较旧的 C++ 编译器上失败（例如，如果你的编译器不支持成员模板的话）。如果你不打算在自己的程序中使用这些功能，则这些错误是无害的。
>
> 注意：测试套件目前包含 500 多个测试。如果安装了许多不同的目标语言并且计算机速度较慢，则运行测试套件可能需要一个多小时。

### 1.12.5 示例

The Examples directory contains a variety of examples of using SWIG and it has some browsable documentation. Simply point your browser to the file "Example/index.html".

The Examples directory also includes Visual C++ project 6 (.dsp) files for building some of the examples on Windows. Later versions of Visual Studio will convert these old style project files into a current solution file.

> `Examples` 目录包含使用 SWIG 的各种示例，它有一些可浏览的文档。只需将浏览器指向 `Example/index.html` 文件即可。
>
> `Examples` 目录还包括用于在 Windows 上构建示例的一些 Visual C++ 6 项目文件（.dsp）。更高版本的 Visual Studio 会将这些旧样式项目文件转换为当前解决方案文件。
