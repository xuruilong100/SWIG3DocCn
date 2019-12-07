[TOC]

# 2 引言

## 2.1 SWIG 是什么？

SWIG is a software development tool that simplifies the task of interfacing different languages to C and C++ programs. In a nutshell, SWIG is a compiler that takes C/C++ declarations and creates the wrappers needed to access those declarations from other languages including Perl, Python, Tcl, Ruby, Guile, and Java. SWIG normally requires no modifications to existing code and can often be used to build a usable interface in only a few minutes. Possible applications of SWIG include:

* Building interpreted interfaces to existing C programs.
* Rapid prototyping and application development.
* Interactive debugging.
* Reengineering or refactoring of legacy software into scripting language components.
* Making a graphical user interface (using Tk for example).
* Testing of C libraries and programs (using scripts).
* Building high performance C modules for scripting languages.
* Making C programming more enjoyable (or tolerable depending on your point of view).
* Impressing your friends.
* Obtaining vast sums of research funding (although obviously not applicable to the author).

SWIG was originally designed to make it extremely easy for scientists and engineers to build extensible scientific software without having to get a degree in software engineering. Because of this, the use of SWIG tends to be somewhat informal and ad-hoc (e.g., SWIG does not require users to provide formal interface specifications as you would find in a dedicated IDL compiler). Although this style of development isn't appropriate for every project, it is particularly well suited to software development in the small; especially the research and development work that is commonly found in scientific and engineering projects. However, nowadays SWIG is known to be used in many large open source and commercial projects.

> SWIG 是一款软件开发工具，它简化了将不同语言连接到 C 和 C++ 程序的任务。简而言之，SWIG 是一个编译器，它接受 C/C++ 声明并创建从其他语言（包括 Perl、Python、Tcl、Ruby、Guile 和 Java）访问这些声明所需的包装器。SWIG 通常不需要修改现有代码，并且可以在几分钟内构建可用的接口。SWIG 可能的应用包括：
>
> * 构建现有 C 程序的解释接口。
> * 快速原型设计和应用程序开发。
> * 交互式调试。
> * 将历史遗留软件重新设计或重构为脚本语言组件。
> * 制作图形用户（例如使用 Tk）。
> * 测试 C 代码库和程序（使用脚本）。
> * 为脚本语言构建高性能 C 模块。
> * 使 C 编程更愉快（或视你的情况而定）。
> * 给你的朋友留下深刻印象
> * 获得大量研究经费（虽然显然不适用于作者本人）。
>
> SWIG 最初旨在使科学家和工程师能够非常轻松地构建可扩展的科学软件，而无需软件工程学位。因此，SWIG 的使用往往是非正式的和临时的（例如，SWIG 不要求用户提供正式的接口规范，就像你在专用的 IDL 编译器中找到的那样）。虽然这种开发方式并不适合每个项目，但它特别适合于小型软件开发，特别是科学和工程项目中常见的研究和开发工作。但是，如今已知 SWIG 可用于许多大型开源和商业项目。

## 2.2 为什么使用 SWIG？

As stated in the previous section, the primary purpose of SWIG is to simplify the task of integrating C/C++ with other programming languages. However, why would anyone want to do that? To answer that question, it is useful to list a few strengths of C/C++ programming:

* Excellent support for writing programming libraries.
* High performance (number crunching, data processing, graphics, etc.).
* Systems programming and systems integration.
* Large user community and software base.

Next, let's list a few problems with C/C++ programming

* Writing a user interface is rather painful (i.e., consider programming with MFC, X11, GTK, or any number of other libraries).
* Testing is time consuming (the compile/debug cycle).
* Not easy to reconfigure or customize without recompilation.
* Modularization can be tricky.
* Security concerns (buffer overflows for instance).

To address these limitations, many programmers have arrived at the conclusion that it is much easier to use different programming languages for different tasks. For instance, writing a graphical user interface may be significantly easier in a scripting language like Python or Tcl (consider the reasons why millions of programmers have used languages like Visual Basic if you need more proof). An interactive interpreter might also serve as a useful debugging and testing tool. Other languages like Java might greatly simplify the task of writing distributed computing software. The key point is that different programming languages offer different strengths and weaknesses. Moreover, it is extremely unlikely that any programming is ever going to be perfect. Therefore, by combining languages together, you can utilize the best features of each language and greatly simplify certain aspects of software development.

From the standpoint of C/C++, a lot of people use SWIG because they want to break out of the traditional monolithic C programming model which usually results in programs that resemble this:

* A collection of functions and variables that do something useful.
* A `main()` program that starts everything.
* A horrible collection of hacks that form some kind of user interface (but which no-one really wants to touch).

Instead of going down that route, incorporating C/C++ into a higher level language often results in a more modular design, less code, better flexibility, and increased programmer productivity.

SWIG tries to make the problem of C/C++ integration as painless as possible. This allows you to focus on the underlying C program and using the high-level language interface, but not the tedious and complex chore of making the two languages talk to each other. At the same time, SWIG recognizes that all applications are different. Therefore, it provides a wide variety of customization features that let you change almost every aspect of the language bindings. This is the main reason why SWIG has such a large user manual ;-).

> 如上一节所述，SWIG 的主要目的是简化 C/C++ 与其他编程语言集成的任务。但是，为什么有人想这样做呢？要回答这个问题，列出一些 C/C++ 编程的优点很有用：
>
> * 对编写程序库的出色支持。
> * 高性能（数值运算、数据处理、图形等）。
> * 系统编程和系统集成。
> * 庞大的用户社区和软件基础。
>
> 接着，列出一些 C/C++ 编程的问题
>
> * 编写用户界面相当痛苦（即考虑使用 MFC、X11、GTK 或其他库进行编程）。
> * 测试耗时（编译/调试周期）。
> * 不重新编译就不容易重新配置或自定义。
> * 模块化可能很棘手。
> * 安全问题（例如缓冲区溢出）。
>
> 为了解决这些限制，许多程序员得出的结论是，在不同的任务使用不同的编程语言要容易得多。例如，在 Python 或 Tcl 等脚本语言中编写图形用户界面可能要容易得多（如果需要更多证据，请考虑数百万程序员使用 Visual Basic 等语言的原因）。交互式解释器也可以作为有用的调试和测试工具。像 Java 这样的其他语言可以大大简化编写分布式计算软件的任务。关键是不同的编程语言提供不同的优点和缺点。而且，任何程序都不太可能是完美的。因此，通过将语言组合在一起，你可以利用每种语言的最佳功能，并大大简化软件开发的某些方面。
>
> 从 C/C++ 的角度来看，很多人都使用 SWIG，因为他们想要打破传统的单片机 C 编程模型，那通常会产生类似于下面的程序：
>
> * 一组有用的函数和变量。
> * 一个启动所有内容的 `main()` 程序。
> * 一个可怕的黑客集合，形成了某种用户界面（但没有人真正想要碰一碰）。
>
> 将 C/C++ 结合到更高级的语言中，通常会导致更加模块化的设计、更少的代码、更好的灵活性，以及更高的程序员生产力，而不是沿着老路走下去。
>
> SWIG 试图使 C/C++ 集成问题尽可能轻松。这使你可以专注于底层的 C 程序，并使用高级语言接口，而不是让两种语言相互交流的繁琐复杂的工作。同时，SWIG 认识到所有应用程序都是不同的。因此，它提供了各种自定义功能，使你可以更改语言绑定的几乎每个方面。这是 SWIG 拥有如此厚的用户手册的主要原因 ;-)。

## 2.3 一个 SWIG 示例

The best way to illustrate SWIG is with a simple example. Consider the following C code:

> 阐释 SWIG 的最佳方式是一个简单的例子。考虑以下 C 代码：

```c
/* File : example.c */

double  My_variable  = 3.0;

/* Compute factorial of n */
int fact(int n) {
  if (n <= 1)
    return 1;
  else
    return n*fact(n-1);
}

/* Compute n mod m */
int my_mod(int n, int m) {
  return(n % m);
}
```

Suppose that you wanted to access these functions and the global variable `My_variable` from Tcl. You start by making a SWIG interface file as shown below (by convention, these files carry a .i suffix) :

> 假设你想要从 Tcl 访问这些函数和全局变量 `My_variable`。首先制作一个 SWIG 接口文件，如下所示（按照惯例，这些文件带有 `.i` 后缀）：

### 2.3.1 SWIG 接口文件

```
/* File : example.i */
%module example
%{
/* Put headers and other declarations here */
extern double My_variable;
extern int    fact(int);
extern int    my_mod(int n, int m);
%}

extern double My_variable;
extern int    fact(int);
extern int    my_mod(int n, int m);
```

The interface file contains ANSI C function prototypes and variable declarations. The `%module` directive defines the name of the module that will be created by SWIG. The `%{ %}` block provides a location for inserting additional code, such as C header files or additional C declarations, into the generated C wrapper code.

> 接口文件包含 ANSI C 函数原型和变量声明。`%module` 指令定义 SWIG 创建的模块的名称。`%{ %}` 块提供了一个位置，用于在生成的 C 包装器代码中插入其他代码，例如 C 头文件或其他 C 声明。

### 2.3.2 `swig` 命令

SWIG is invoked using the `swig` command. We can use this to build a Tcl module (under Linux) as follows :

> 使用 `swig` 命令调用 SWIG。我们可以使用它来构建一个 Tcl 模块（在 Linux 下），如下所示：

```
unix > swig -tcl example.i
unix > gcc -c -fpic example.c example_wrap.c -I/usr/local/include
unix > gcc -shared example.o example_wrap.o -o example.so
unix > tclsh
% load ./example.so
% fact 4
24
% my_mod 23 7
2
% expr $My_variable + 4.5
7.5
%
```

The `swig` command produced a new file called `example_wrap.c`that should be compiled along with the `example.c` file. Most operating systems and scripting languages now support dynamic loading of modules. In our example, our Tcl module has been compiled into a shared library that can be loaded into Tcl. When loaded, Tcl can now access the functions and variables declared in the SWIG interface. A look at the file `example_wrap.c` reveals a hideous mess. However, you almost never need to worry about it.

> `swig` 命令生成了一个名为 `example_wrap.c` 的新文件，该文件应与 `example.c` 文件一起编译。现在，大多数操作系统和脚本语言都支持动态加载模块。在我们的示例中，我们的 Tcl 模块已编译为可以加载到 Tcl 的动态库。加载后，Tcl 现在可以访问 SWIG 接口中声明的函数和变量。看看文件 `example_wrap.c` 就会发现可怕的混乱。但是，你几乎不需要担心它。

### 2.3.3 构建 Perl5 模块

Now, let's turn these functions into a Perl5 module. Without making any changes type the following (shown for Solaris):

> 现在，让我们将这些函数转换为 Perl5 模块。在不进行任何更改的情况下键入以下内容（针对 Solary 显示）：

```
unix > swig -perl5 example.i
unix > gcc -c example.c example_wrap.c \
        -I/usr/local/lib/perl5/sun4-solaris/5.003/CORE
unix > ld -G example.o example_wrap.o -o example.so # This is for Solaris
unix > perl5.003
use example;
print example::fact(4), "\n";
print example::my_mod(23, 7), "\n";
print $example::My_variable + 4.5, "\n";
<ctrl-d>
24
2
7.5
unix >
```

### 2.3.4 构建 Python 模块

Finally, let's build a module for Python (shown for Irix).

> 最后，让我们为 Python 构建一个模块（针对 Irix 显示）。

```
unix > swig -python example.i
unix > gcc -c -fpic example.c example_wrap.c -I/usr/local/include/python2.0
unix > gcc -shared example.o example_wrap.o -o _example.so
unix > python
Python 2.0 (#6, Feb 21 2001, 13:29:45)
[GCC egcs-2.91.66 19990314/Linux (egcs-1.1.2 release)] on linux2
Type "copyright", "credits" or "license" for more information.
>>> import example
>>> example.fact(4)
24
>>> example.my_mod(23, 7)
2
>>> example.cvar.My_variable + 4.5
7.5
```

### 2.3.5 快捷方式

To the truly lazy programmer, one may wonder why we needed the extra interface file at all. As it turns out, you can often do without it. For example, you could also build a Perl5 module by just running SWIG on the C header file and specifying a module name as follows

> 对于真正懒惰的程序员，人们可能想知道为什么我们需要额外的接口文件。事实证明，你经常可以不需要它。例如，你还可以通过在 C 头文件上运行 SWIG，并指定模块名称来构建 Perl5 模块，如下所示

```
unix > swig -perl5 -module example example.h
unix > gcc -c example.c example_wrap.c \
        -I/usr/local/lib/perl5/sun4-solaris/5.003/CORE
unix > ld -G example.o example_wrap.o -o example.so
unix > perl5.003
use example;
print example::fact(4), "\n";
print example::my_mod(23, 7), "\n";
print $example::My_variable + 4.5, "\n";
<ctrl-d>
24
2
7.5
```

## 2.4 支持的 C/C++ 语言特性

A primary goal of the SWIG project is to make the language binding process extremely easy. Although a few simple examples have been shown, SWIG is quite capable in supporting most of C++. Some of the major features include:

* Full C99 preprocessing.
* All ANSI C and C++ datatypes.
* Functions, variables, and constants.
* Classes.
* Single and multiple inheritance.
* Overloaded functions and methods.
* Overloaded operators.
* C++ templates (including member templates, specialization, and partial specialization).
* Namespaces.
* Variable length arguments.
* C++ smart pointers.

Most of C++11 is also supported. Details are in the [C++11](http://www.swig.org/Doc3.0/CPlusPlus11.html#CPlusPlus11) section.

It is important to stress that SWIG is not a simplistic C++ lexing tool like several apparently similar wrapper generation tools. SWIG not only parses C++, it implements the full C++ type system and it is able to understand C++ semantics. SWIG generates its wrappers with full knowledge of this information. As a result, you will find SWIG to be just as capable of dealing with nasty corner cases as it is in wrapping simple C++ code. In fact, SWIG is able to handle C++ code that stresses the very limits of many C++ compilers.

> SWIG 项目的主要目标是使语言绑定过程变得非常容易。虽然只展示了一些简单的例子，但 SWIG 能够很好地支持 C++ 的大多数功能。主要包括：
>
> * 完整的 C99 预处理；
> * 全部 ANSI C&C++ 数据类型；
> * 函数、变量与常量；
> * 类；
> * 单继承与多继承；
> * 重载函数与方法；
> * 重载运算符；
> * C++ 模板（包括成员模板、特化和偏特化）；
> * 命名空间；
> * 变长参数；
> * C++ 智能指针。
>
> 支持绝大部分 C++11 功能。细节请见 [C++11](http://www.swig.org/Doc3.0/CPlusPlus11.html#CPlusPlus11) 章节。
>
> 需要着重强调一点，SWIG 不是一个简单的 C++ 词法分析工具，就像几个明显相似的包装器生成工具一样。SWIG 不仅解析 C++，它实现了完整的 C++ 类型系统，并且能够理解 C++ 语义。SWIG 在充分了解这些信息的情况下生成包装器。因此，你会发现 SWIG 能够像处理简单的 C++ 代码一样处理令人讨厌的特殊情况。事实上，SWIG 能够处理强调许多挑战 C++ 编译器极限的代码。

## 2.5 非直觉的接口构建

When used as intended, SWIG requires minimal (if any) modification to existing C or C++ code. This makes SWIG extremely easy to use with existing packages and promotes software reuse and modularity. By making the C/C++ code independent of the high level interface, you can change the interface and reuse the code in other applications. It is also possible to support different types of interfaces depending on the application.

> 当按预期使用时，SWIG 要求对现有 C 或 C++ 代码进行最少（如果有）修改。这使 SWIG 非常易于使用现有软件包，并有助于软件重用和模块化。通过使 C/C++ 代码独立于高级接口，你可以更改接口，并在其他应用程序中重用代码。还可以根据应用程序支持不同类型的接口。

## 2.6 将 SWIG 整合进构建系统

SWIG is a command line tool and as such can be incorporated into any build system that supports invoking external tools/compilers. SWIG is most commonly invoked from within a Makefile, but is also known to be invoked from popular IDEs such as Microsoft Visual Studio.

If you are using the GNU Autotools ([Autoconf](http://www.gnu.org/software/autoconf/)/ [Automake](http://www.gnu.org/software/automake/)/ [Libtool](http://www.gnu.org/software/libtool/)) to configure SWIG use in your project, the SWIG Autoconf macros can be used. The primary macro is `ax_pkg_swig`, see<http://www.gnu.org/software/autoconf-archive/ax_pkg_swig.html#ax_pkg_swig>. The `ax_python_devel`macro is also helpful for generating Python extensions. See the[Autoconf Archive](http://www.gnu.org/software/autoconf-archive/) for further information on this and other Autoconf macros.

There is growing support for SWIG in some build tools, for example [CMake](http://cmake.org/) is a cross-platform, open-source build manager with built in support for SWIG. CMake can detect the SWIG executable and many of the target language libraries for linking against. CMake knows how to build shared libraries and loadable modules on many different operating systems. This allows easy cross platform SWIG development. It can also generate the custom commands necessary for driving SWIG from IDEs and makefiles. All of this can be done from a single cross platform input file. The following example is a CMake input file for creating a python wrapper for the SWIG interface file, example.i:

>SWIG 是一个命令行工具，因此可以合并到任何支持调用外部工具/编译器的构建系统中。SWIG 最常在 Makefile 中调用，但也可以从流行的 IDE（如 Microsoft Visual Studio）调用。
>
>如果你正在使用 GNU 自动工具（[Autoconf](http://www.gnu.org/software/autoconf/)/ [Automake](http://www.gnu.org/software/automake/)/ [Libtool](http://www.gnu.org/software/libtool/)）来为你的项目配置 SWIG，可以使用 SWIG 的 `Autoconf` 宏。主要的宏是 `ax_pkg_swig`，详见 <http://www.gnu.org/software/autoconf-archive/ax_pkg_swig.html#ax_pkg_swig>。在构建 Python 扩展时 `ax_python_devel` 也很有用。细节信息和其他 `Autoconf` 宏详见 [Autoconf Archive](http://www.gnu.org/software/autoconf-archive/)。
>
>一些构建工具中对 SWIG 的支持越来越多，例如 [CMake](http://cmake.org/) 是一个跨平台的开源构建管理器，内置了对 SWIG 的支持。CMake 可以检测 SWIG 可执行文件和许多目标语言库以进行链接。CMake 知道如何在许多不同的操作系统上构建动态库和可加载模块。这样可以轻松实现跨平台 SWIG 开发。它还可以生成从 IDE 和 makefile 驱动 SWIG 所需的自定义命令。所有这些都可以从单个跨平台输入文件中完成。以下示例是一个 CMake 输入文件，用于为 SWIG 接口文件 `example.i` 创建 python 包装器：

```
# This is a CMake example for Python

FIND_PACKAGE(SWIG REQUIRED)
INCLUDE(${SWIG_USE_FILE})

FIND_PACKAGE(PythonLibs)
INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_PATH})

INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})

SET(CMAKE_SWIG_FLAGS "")

SET_SOURCE_FILES_PROPERTIES(example.i PROPERTIES CPLUSPLUS ON)
SET_SOURCE_FILES_PROPERTIES(example.i PROPERTIES SWIG_FLAGS "-includeall")
SWIG_ADD_MODULE(example python example.i example.cxx)
SWIG_LINK_LIBRARIES(example ${PYTHON_LIBRARIES})
```

The above example will generate native build files such as makefiles, nmake files and Visual Studio projects which will invoke SWIG and compile the generated C++ files into _example.so (UNIX) or _example.pyd (Windows). For other target languages on Windows a dll, instead of a .pyd file, is usually generated.

> 上面的示例将生成本机构建文件，例如 makefile、nmake 文件和 Visual Studio 项目文件，它们将调用 SWIG 并将生成的 C++ 文件编译为 `_example.so`（UNIX）或 `_example.pyd`（Windows）。 对于 Windows 上的其他目标语言，通常会生成  dll 而不是 `.pyd` 文件。

## 2.7 自动化代码生成

SWIG is designed to produce working code that needs no hand-modification (in fact, if you look at the output, you probably won't want to modify it). You should think of your target language interface being defined entirely by the input to SWIG, not the resulting output file. While this approach may limit flexibility for hard-core hackers, it allows others to forget about the low-level implementation details.

> SWIG 旨在生成无需手动修改的工作代码（事实上，如果查看输出，你可能不希望对其进行修改）。你应该认为目标语言接口完全由 SWIG 的输入定义，而不是生成的输出文件。虽然这种方法可能会限制硬核黑客的灵活性，但它允许其他人忘记低级的实现细节。

## 2.8 SWIG 与自由

No, this isn't a special section on the sorry state of world politics. However, it may be useful to know that SWIG was written with a certain "philosophy" about programming---namely that programmers are smart and that tools should just stay out of their way. Because of that, you will find that SWIG is extremely permissive in what it lets you get away with. In fact, you can use SWIG to go well beyond "shooting yourself in the foot" if dangerous programming is your goal. On the other hand, this kind of freedom may be exactly what is needed to work with complicated and unusual C/C++ applications.

Ironically, the freedom that SWIG provides is countered by an extremely conservative approach to code generation. At its core, SWIG tries to distill even the most advanced C++ code down to a small well-defined set of interface building techniques based on ANSI C programming. Because of this, you will find that SWIG interfaces can be easily compiled by virtually every C/C++ compiler and that they can be used on any platform. Again, this is an important part of staying out of the programmer's way----the last thing any developer wants to do is to spend their time debugging the output of a tool that relies on non-portable or unreliable programming features. Dependencies are often a source of incompatibilities and problems and so additional third party libraries are not used in the generated code. SWIG will also generally avoid generating code that introduces a dependency on the C++ Standard Template Library (STL). SWIG will generate code that depends on the C libraries though.

> 这不是关于世界政治糟糕状况的特别章节。然而，知道 SWIG 是由编程的某种“哲学”编写的，这可能是有用的——即程序员是聪明的，工具不应该挡路。因此，你会发现 SWIG 极度地放任你。事实上，如果危险的编程是你的目标，那么你可以使用 SWIG 远离“打到自己的脚”。另一方面，这种自由可能正是处理复杂和不寻常的 C/C++ 应用程序所需要的。
>
> 具有讽刺意味的是，SWIG 提供的自由被极端保守的代码生成方法所抵消。从本质上讲，SWIG 是试图将最高级的 C++ 代码提炼为基于 ANSI C 编程的一组定义明确的接口构建技术。因此，你会发现几乎每个 C/C++ 编译器都可以轻松编译 SWIG 接口，并且可以在任何平台上使用它们。再一次，这对程序员很重要——任何开发人员最不想要做的一件事就是花时间调试依赖于非便携或不可靠编程工具的输出。依赖性通常是不兼容性和问题的根源，因此生成的代码中不使用其他第三方库。SWIG 通常还会避免生成依赖于 C++ 标准模板库（STL）的代码。然而，SWIG 将生成依赖于 C 库的代码。
