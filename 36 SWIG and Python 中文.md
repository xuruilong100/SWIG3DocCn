[toc]

# 36 SWIG 与 Python

**Caution: This chapter is under repair!**

This chapter describes SWIG's support of Python. SWIG is compatible with most recent Python versions including Python 3.0 and Python 2.6, as well as older versions dating back to Python 2.0. For the best results, consider using Python 2.3 or newer.

This chapter covers most SWIG features, but certain low-level details are covered in less depth than in earlier chapters. At the very least, make sure you read the "[SWIG Basics](http://www.SWIG.org/Doc3.0/SWIG.html#SWIG)" chapter.

> 本章介绍 SWIG 对 Python 的支持。SWIG 兼容包括 Python 3.0 和 Python 2.6 在内的最新 Python 版本，以及可追溯到 Python 2.0 旧版本。为了获得最佳结果，请考虑使用 Python 2.3 或更高版本。
>
> 本章涵盖了大多数 SWIG 功能，但与以前的章节相比，对某些低级细节的深度介绍较少。请确保你至少已阅读 [SWIG 基础知识](http://www.SWIG.org/Doc3.0/SWIG.html#SWIG)一章。

## 36.1 概览

To build Python extension modules, SWIG uses a layered approach in which parts of the extension module are defined in C and other parts are defined in Python. The C layer contains low-level wrappers whereas Python code is used to define high-level features.

This layered approach recognizes the fact that certain aspects of extension building are better accomplished in each language (instead of trying to do everything in C or C++). Furthermore, by generating code in both languages, you get a lot more flexibility since you can enhance the extension module with support code in either language.

In describing the Python interface, this chapter starts by covering the basics of configuration, compiling, and installing Python modules. Next, the Python interface to common C and C++ programming features is described. Advanced customization features such as typemaps are then described followed by a discussion of low-level implementation details.

> 为了构建 Python 扩展模块，SWIG 使用分层方法，其中扩展模块的某些部分用 C 定义，而其他部分用 Python 定义。C 层包含低级包装器，而 Python 代码用于定义高级功能。
>
> 这种分层方法基于以下事实：扩展构建的某些方面可以在每种语言中更好地完成（而不是尝试使用 C 或 C++ 进行所有操作）。此外，通过生成两种语言的代码，你将获得更大的灵活性，因为你可以使用两种语言的支持代码来增强扩展模块。
>
> 在描述 Python 接口时，本章首先介绍配置、编译和安装 Python 模块的基础知识。接下来，描述了一般 C 和 C++ 编程功能的 Python 接口。然后介绍高级自定义功能（如类型映射），然后讨论底层实现细节。

## 36.2 预备知识

### 36.2.1 运行 SWIG

Suppose that you defined a SWIG module such as the following:

> 假设你定义了以下 SWIG 模块：

```
/* File: example.i */
%module example

%{
#define SWIG_FILE_WITH_INIT
#include "example.h"
%}

int fact(int n);
```

The `#define SWIG_FILE_WITH_INIT` line inserts a macro that specifies that the resulting C file should be built as a python extension, inserting the module `init` code. This `.i` file wraps the following simple C file:

> `#define SWIG_FILE_WITH_INIT` 行插入一个宏，该宏指定应将生成的 C 文件构建为 python 扩展，并插入模块 `init` 代码。该 `.i` 文件包装了以下简单的 C 文件：

```c++
/* File: example.c */

#include "example.h"

int fact(int n) {
    if (n < 0){ /* This should probably return an error, but this is simpler */
        return 0;
    }
    if (n == 0) {
        return 1;
    }
    else {
        /* testing for overflow would be a good idea here */
        return n * fact(n-1);
    }
}
```

With the header file:

> 头文件：

```c++
/* File: example.h */

int fact(int n);
```

To build a Python module, run SWIG using the `-python` option:

> 为构建 Python 模块，运行 SWIG 时要添加 `-python` 选项：

```
$ SWIG -python example.i
```

If building a C++ extension, add the `-c++` option:

> 如果构建 C++ 扩展，要添加 `-c++` 选项：

```
$ SWIG -c++ -python example.i
```

This creates two different files; a C/C++ source file `example_wrap.c` or `example_wrap.cxx` and a Python source file `example.py`. The generated C source file contains the low-level wrappers that need to be compiled and linked with the rest of your C/C++ application to create an extension module. The Python source file contains high-level support code. This is the file that you will import to use the module.

The name of the wrapper file is derived from the name of the input file. For example, if the input file is `example.i`, the name of the wrapper file is `example_wrap.c`. To change this, you can use the `-o` option. The name of the Python file is derived from the module name specified with `%module`. If the module name is `example`, then a file `example.py` is created.

The following sections have further practical examples and details on how you might go about compiling and using the generated files.

> 这将创建两个不同的文件。C/C++ 源文件 `example_wrap.c` 或 `example_wrap.cxx` 和 Python 源文件 `example.py`。生成的 C 源文件包含需要编译的低级包装器，并与你的 C/C++ 应用程序的其余部分链接以创建扩展模块。Python 源文件包含高级支持代码。这是你将导入，并用以使用该模块的文件。
>
> 包装文件的名称是从输入文件的名称派生的。例如，如果输入文件为 `example.i`，则包装文件的名称为 `example_wrap.c`。要更改此设置，可以使用 `-o` 选项。Python 文件的名称是从 `%module` 指定的模块名称派生的。如果模块名称为 `example`，则创建文件 `example.py`。
>
> 以下各节提供了更多实用的示例，以及有关如何编译和使用生成文件的详细信息。

### 36.2.2 使用 `distutils`

The preferred approach to building an extension module for python is to compile it with distutils, which comes with all recent versions of python ([Distutils Docs](https://docs.python.org/library/distutils.html)).

Distutils takes care of making sure that your extension is built with all the correct flags, headers, etc. for the version of Python it is run with. Distutils will compile your extension into a shared object file or DLL (`.so` on Linux, `.pyd` on Windows, etc). In addition, distutils can handle installing your package into site-packages, if that is desired. A configuration file (conventionally called: `setup.py`) describes the extension (and related python modules). The distutils will then generate all the right compiler directives to build it for you.

Here is a sample `setup.py` file for the above example:

> 构建 python 扩展模块的首选方法是使用 `distutils` 对其进行编译，而 `distutils` 随所有 python 的最新版本一起提供（[Distutils Docs](https://docs.python.org/library/distutils.html)）。
>
> `distutils` 负责确保你的扩展程序针对运行它的 Python 版本使用所有正确的标志、头文件等进行构建。`distutils` 会将你的扩展编译为共享对象文件或 DLL（在 Linux 上为 `.so`，在 Windows 上为 `.pyd` 等）。另外，如果需要，`distutils` 可以处理将你的软件包安装到站点软件包中的过程。一个配置文件（通常称为 `setup.py`）描述了扩展（以及相关的 python 模块）。然后，`distutils` 将生成所有正确的编译器指令来为你构建它。
>
> 这是上述示例的样本 `setup.py` 文件：

```python
#!/usr/bin/env python

"""
setup.py file for SWIG example
"""

from distutils.core import setup, Extension


example_module = Extension(
    '_example',
    sources=['example_wrap.c', 'example.c'])

setup(
    name = 'example',
    version = '0.1',
    author      = "SWIG Docs",
    description = """Simple SWIG example from docs""",
    ext_modules = [example_module],
    py_modules  = ["example"])
```

In this example, the line: `example_module = Extension(....)` creates an Extension module object, defining the name as `_example`, and using the source code files: `example_wrap.c`, generated by SWIG , and `example.c`, your original c source. The SWIG (and other python extension modules) tradition is for the compiled extension to have the name of the python portion, prefixed by an underscore. If the name of your python module is `example.py`, then the name of the corresponding object file will be `_example.so`.

The `setup` call then sets up distutils to build your package, defining some meta data, and passing in your extension module object. Once this is saved as `setup.py`, you can build your extension with these commands:

> 在此示例中，行：`example_module = Extension(....)` 创建一个扩展模块对象，将名称定义为 `_example`，并使用由 SWIG 生成的源代码文件 `example_wrap.c` 和 `example.c`，你的原始 C 源代码。SWIG（和其他 python 扩展模块）的传统是使编译后的扩展具有 python 部分的名称，并以下划线作为前缀。如果你的 python 模块的名称为 `example.py`，则对应的目标文件的名称将为 `_example.so`。
>
> 然后，`setup` 调用会设置 `distutils` 来构建你的包，定义一些元数据，并传入你的扩展模块对象。将其保存为 `setup.py` 后，你可以使用以下命令构建扩展程序：

```
$ SWIG -python example.i
$ python setup.py build_ext --inplace
```

And a `.so`, or `.pyd` or ... will be created for you. It will build a version that matches the python that you run the command with. Taking apart the command line:
- `python` -- the version of python you want to build for
- `setup.py` -- the name of your setup script (it can be called anything, but `setup.py` is the tradition)
- `build_ext` -- telling distutils to build extensions
- `--inplace` -- this tells distutils to put the extension lib in the current dir. Otherwise, it will put it inside a build hierarchy, and you'd have to move it to use it.

The distutils have many other features, consult the python distutils docs for details.

This same approach works on all platforms if the appropriate compiler is installed. (it can even build extensions to the standard Windows Python using MingGW)

> 然后将为你创建一个 `.so` 或 `.pyd` 或其他。它将构建与你运行命令的 python 匹配的版本。分析下命令行：
> - `python`—— 你要针对构建的 python 版本
> - `setup.py`——安装脚本的名称（可以称为任何名称，但 `setup.py` 是传统名称）
> - `build_ext`——告诉 `distutils` 建立扩展
> - `--inplace`——告诉 `distutils` 将扩展库 lib 放在当前目录中。否则，它将被放置在构建层次结构中，你必须将其移动以使用它。
>
> `distutils` 还有许多其他功能，有关详细信息，请查阅 python `distutils` 的文档。
>
> 如果安装了适当的编译器，则此方法适用于所有平台。（它甚至可以使用 MingGW 构建对标准 Windows Python 的扩展）

### 36.2.3 手动编译一个动态模块

While the preferred approach to building an extension module is to use the distutils, some people like to integrate building extensions with a larger build system, and thus may wish to compile their modules without the distutils. To do this, you need to compile your program using commands like this (shown for Linux):

> 虽然构建扩展模块的首选方法是使用 `distutils`，但有些人喜欢将构建扩展与更大的构建系统集成在一起，因此，可能希望在没有 `distutils` 的情况下编译其模块。为此，你需要使用以下命令编译程序（针对 Linux，该命令如下所示）：

```
$ SWIG -python example.i
$ gcc -O2 -fPIC -c example.c
$ gcc -O2 -fPIC -c example_wrap.c -I/usr/local/include/python2.5
$ gcc -shared example.o example_wrap.o -o _example.so
```

The exact commands for doing this vary from platform to platform. However, SWIG tries to guess the right options when it is installed. Therefore, you may want to start with one of the examples in the `SWIG/Examples/python` directory. If that doesn't work, you will need to read the man-pages for your compiler and linker to get the right set of options. You might also check the [SWIG Wiki](https://github.com/SWIG/SWIG/wiki) for additional information.

When linking the module, **the name of the output file has to match the name of the module prefixed by an underscore**. If the name of your module is `example`, then the name of the corresponding object file should be `_example.so` or `_examplemodule.so`. The name of the module is specified using the `%module` directive or the `-module` command line option.

**Compatibility Note:** In SWIG-1.3.13 and earlier releases, module names did not include the leading underscore. This is because modules were normally created as C-only extensions without the extra Python support file (instead, creating Python code was supported as an optional feature). This has been changed in SWIG-1.3.14 and is consistent with other Python extension modules. For example, the `socket` module actually consists of two files; `socket.py` and `_socket.so`. Many other built-in Python modules follow a similar convention.

> 具体执行的命令因平台而异。但是，SWIG 会在安装时尝试猜测正确的选项。因此，你可以想从 `SWIG/Examples/python` 目录中的某个示例开始。如果这不起作用，则需要阅读编译器和链接器的手册，以获取正确的选项集。你也可以查看 [SWIG Wiki](https://github.com/SWIG/SWIG/wiki) 以获取更多信息。
>
> 链接模块时，**输出文件的名称必须与带有下划线前缀的模块的名称匹配**。如果模块的名称为 `example`，则相应的目标文件的名称应为 `_example.so` 或 `_examplemodule.so`。使用 `%module` 指令或 `-module` 命令行选项指定模块的名称。
>
> **兼容性说明**：在 SWIG-1.3.13 和更早版本中，模块名称不包括前导下划线。这是因为模块通常仅被创建为 C 扩展，而没有额外的 Python 支持文件（相反，创建 Python 代码作为可选功能）。这已在 SWIG-1.3.14 中更改，并与其他 Python 扩展模块一致。例如，`socket` 模块实际上由两个文件组成：`socket.py` 和 `_socket.so`。许多其他内置的 Python 模块遵循类似的约定。

### 36.2.4 静态链接

An alternative approach to dynamic linking is to rebuild the Python interpreter with your extension module added to it. In the past, this approach was sometimes necessary due to limitations in dynamic loading support on certain machines. However, the situation has improved greatly over the last few years and you should not consider this approach unless there is really no other option.

The usual procedure for adding a new module to Python involves finding the Python source, adding an entry to the `Modules/Setup` file, and rebuilding the interpreter using the Python Makefile. However, newer Python versions have changed the build process. You may need to edit the 'setup.py' file in the Python distribution instead.

In earlier versions of SWIG , the `embed.i` library file could be used to rebuild the interpreter. For example:

> 动态链接的另一种方法是使用扩展模块来重建 Python 解释器。过去，由于某些机器上动态加载支持的限制，有时需要使用此方法。但是，在过去几年中，这种情况已大大改善，除非你真的没有其他选择，否则你不应该考虑采用这种方法。
>
> 向 Python 添加新模块的一般过程包括查找 Python 源代码，向 `Modules/Setup` 文件添加条目，以及使用 Python Makefile 重建解释器。但是，较新的 Python 版本改变了构建过程。你可能需要在 Python 发行版中编辑 `setup.py` 文件。
>
> 在早期版本的 SWIG 中，`embed.i` 库文件可用于重建解释器。例如：

```
%module example

%inline %{
extern int fact(int);
extern int mod(int, int);
extern double My_variable;
%}

%include "embed.i"       // Include code for a static version of Python
```

The `embed.i` library file includes supporting code that contains everything needed to rebuild Python. To rebuild the interpreter, you simply do something like this:

> `embed.i` 库文件包含支持代码，其中包含重建 Python 所需的所有内容。要重建解释器，你只需执行以下操作：

```
$ SWIG -python -lembed.i example.i
$ gcc example.c example_wrap.c \
        -Xlinker -export-dynamic \
        -DHAVE_CONFIG_H -I/usr/include/python2.7 \
        -I/usr/lib/python2.7/config-x86_64-linux-gnu \
        -I/usr/lib/python2.7/config \
        -L/usr/lib/python2.7/config -lpython2.7 -lm -ldl \
        -o mypython
```

You will need to supply the same libraries that were used to build Python the first time. This may include system libraries such as `-lsocket`, `-lnsl`, and `-lpthread`. Assuming this actually works, the new version of Python should be identical to the default version except that your extension module will be a built-in part of the interpreter.

**Comment:** In practice, you should probably try to avoid static linking if possible. Some programmers may be inclined to use static linking in the interest of getting better performance. However, the performance gained by static linking tends to be rather minimal in most situations (and quite frankly not worth the extra hassle in the opinion of this author).

**Compatibility note:** The `embed.i` library file is deprecated and has not been actively maintained for many years. Even though it appears to "work" with Python 2.7, no future support is guaranteed. If using static linking, you might want to rely on a different approach (perhaps using distutils).

> 你将需要提供与第一次构建 Python 相同的库。这可能包括系统库，例如 `-lsocket`、`-lnsl` 和 `-lpthread`。假设这确实有效，则 Python 的新版本应与默认版本相同，不同之处在于扩展模块将是解释器的内置部分。
>
> **评论**：实际上，你应该尽可能避免静态链接。为了获得更好的性能，某些程序员可能倾向于使用静态链接。但是，在大多数情况下，通过静态链接获得的性能往往会非常低（坦率地说，依此作者看来，这不值得额外的麻烦）。
>
> **兼容性说明**：不推荐使用 `embed.i` 库文件，并且多年来没有对其进行积极维护。即使它似乎可以在 Python 2.7 中使用，也不能保证将来会提供支持。如果使用静态链接，则可能需要依靠其他方法（也许使用 `distutils`）。

### 36.2.5 使用你的模块

To use your module, simply use the Python `import` statement. If all goes well, you will be able to run this:

> 要使用你的模块，只需使用 Python 的 `import` 语句即可。如果一切顺利，你将可以运行以下命令：

```python
$ python
>>> import example
>>> example.fact(4)
24
>>>
```

A common error received by first-time users is the following:

> 初次使用的用户收到的常见错误如下：

```python
>>> import example
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
  File "example.py", line 2, in ?
    import _example
ImportError: No module named _example
```

If you get this message, it means that you either forgot to compile the wrapper code into an extension module or you didn't give the extension module the right name. Make sure that you compiled the wrappers into a module called `_example.so`. And don't forget the leading underscore (`_`).

Another possible error is the following:

> 如果收到此消息，则意味着你要么忘记将包装器代码编译到扩展模块中，要么没有给扩展模块指定正确的名称。确保将包装器编译到名为 `_example.so` 的模块中。并且不要忘了下划线（`_`）。
>
> 另一个可能的错误如下：

```python
>>> import example
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
ImportError: dynamic module does not define init function (init_example)
>>>
```

This error is almost always caused when a bad name is given to the shared object file. For example, if you created a file `example.so` instead of `_example.so`you would get this error. Alternatively, this error could arise if the name of the module is inconsistent with the module name supplied with the `%module`directive. Double-check the interface to make sure the module name and the shared object filename match. Another possible cause of this error is forgetting to link the SWIG-generated wrapper code with the rest of your application when creating the extension module.

Another common error is something similar to the following:

> 共享对象文件的名称错误时，几乎总是会导致此错误。例如，如果你创建的文件名为 `example.so` 而不是 `_example.so`，则会出现此错误。或者，如果模块名称与 `%module` 指令提供的模块名称不一致，则可能会发生此错误。仔细检查接口文件以确保模块名称和动态链接库文件名匹配。造成此错误的另一个可能原因是，在创建扩展模块时，忘记将 SWIG 生成的包装器代码与应用程序的其余部分链接。
>
> 另一个常见错误类似于以下内容：

```python
Traceback (most recent call last):
  File "example.py", line 3, in ?
    import example
ImportError: ./_example.so: undefined symbol: fact
```

This error usually indicates that you forgot to include some object files or libraries in the linking of the shared library file. Make sure you compile both the SWIG wrapper file and your original program into a shared library file. Make sure you pass all of the required libraries to the linker.

Sometimes unresolved symbols occur because a wrapper has been created for a function that doesn't actually exist in a library. This usually occurs when a header file includes a declaration for a function that was never actually implemented or it was removed from a library without updating the header file. To fix this, you can either edit the SWIG input file to remove the offending declaration or you can use the `%ignore` directive to ignore the declaration.

Finally, suppose that your extension module is linked with another library like this:

> 此错误通常表明你忘记了动态链接库文件的链接中包含一些目标文件或库。确保将 SWIG 包装文件和原始程序都编译到动态链接库文件中。确保将所有必需的库传递给链接器。
>
> 有时会出现未解析的符号，因为已为库中实际上不存在的函数创建了包装器。当头文件包含一个从未真正实现的函数的声明，或者在不更新头文件的情况下将其从库中删除时，通常会发生这种情况。为了解决这个问题，你可以编辑 SWIG 输入文件以删除有问题的声明，也可以使用 `%ignore` 指令忽略该声明。
>
> 最后，假设你的扩展模块已与另一个库链接，如下所示：

```
$ gcc -shared example.o example_wrap.o -L/home/beazley/projects/lib -lfoo \
      -o _example.so
```

If the `foo` library is compiled as a shared library, you might encounter the following problem when you try to use your module:

> 如果将 `foo` 库编译为动态链接库，则在尝试使用模块时可能会遇到以下问题：

```python
>>> import example
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
ImportError: libfoo.so: cannot open shared object file: No such file or directory
>>>
```

This error is generated because the dynamic linker can't locate the `libfoo.so` library. When shared libraries are loaded, the system normally only checks a few standard locations such as `/usr/lib` and `/usr/local/lib`. To fix this problem, there are several things you can do. First, you can recompile your extension module with extra path information. For example, on Linux you can do this:

> 生成此错误是因为动态链接器无法找到 `liblib.so` 库。加载动态链接库后，系统通常仅检查几个标准位置，例如 `/usr/lib` 和 `/usr/local/lib`。要解决此问题，你可以执行几项操作。首先，你可以使用额外的路径信息重新编译扩展模块。例如，在 Linux 上，你可以执行以下操作：

```
$ gcc -shared example.o example_wrap.o -L/home/beazley/projects/lib -lfoo \
      -Xlinker -rpath /home/beazley/projects/lib  \
      -o _example.so
```

Alternatively, you can set the `LD_LIBRARY_PATH` environment variable to include the directory with your shared libraries. If setting `LD_LIBRARY_PATH`, be aware that setting this variable can introduce a noticeable performance impact on all other applications that you run. To set it only for Python, you might want to do this instead:

> 另外，你可以设置 `LD_LIBRARY_PATH` 环境变量以将目录包含在动态链接库中。如果设置为 `LD_LIBRARY_PATH`，请注意，设置此变量会对运行的所有其他应用程序产生明显的性能影响。要仅针对 Python 进行设置，你可能需要这样做：

```
$ env LD_LIBRARY_PATH=/home/beazley/projects/lib python
```

Finally, you can use a command such as `ldconfig` (Linux) or `crle` (Solaris) to add additional search paths to the default system configuration (this requires root access and you will need to read the man pages).

> 最后，可以使用诸如 `ldconfig`（Linux）或 `crle`（Solaris）之类的命令将其他搜索路径添加到默认系统配置（这需要 root 访问权，并且你需要阅读手册）。

### 36.2.6 编译 C++

Compilation of C++ extensions has traditionally been a tricky problem. Since the Python interpreter is written in C, you need to take steps to make sure C++ is properly initialized and that modules are compiled correctly. This should be a non-issue if you're using distutils, as it takes care of all that for you. The following is included for historical reasons, and in case you need to compile on your own.

On most machines, C++ extension modules should be linked using the C++ compiler. For example:

> 传统上，C++ 扩展的编译是一个棘手的问题。由于 Python 解释器是用 C 编写的，因此你需要采取步骤以确保正确地初始化了 C++，并且正确地编译了模块。如果你使用的是 `distutils`，这应该不是问题，因为它会为你解决所有问题。以下内容出于历史原因，以防万一你需要自己进行编译。
>
> 在大多数计算机上，应使用 C++ 编译器链接 C++ 扩展模块。例如：

```
$ SWIG -c++ -python example.i
$ g++ -O2 -fPIC -c example.cxx
$ g++ -O2 -fPIC -c example_wrap.cxx -I/usr/local/include/python2.5
$ g++ -shared example.o example_wrap.o -o _example.so
```

The `-fPIC` option tells GCC to generate position-independent code (PIC) which is required for most architectures (it's not vital on x86, but still a good idea as it allows code pages from the library to be shared between processes). Other compilers may need a different option specified instead of `-fPIC`.

In addition to this, you may need to include additional library files to make it work. For example, if you are using the Sun C++ compiler on Solaris, you often need to add an extra library `-lCrun` like this:

> `-fPIC` 选项告诉 GCC 生成大多数架构所需的地址无关代码（PIC）（在 x86 上不是至关重要的，但仍然是一个好主意，因为它允许库中的代码页在进程之间共享）。其他编译器可能需要指定其他选项，而不是 `-fPIC`。
>
> 除此之外，你可能需要包括其他库文件才能使其正常工作。例如，如果在 Solaris 上使用 Sun C++ 编译器，则通常需要添加一个额外的库 `-lCrun`，如下所示：

```
$ SWIG -c++ -python example.i
$ CC -c example.cxx
$ CC -c example_wrap.cxx -I/usr/local/include/python2.5
$ CC -G example.o example_wrap.o -L/opt/SUNWspro/lib -o _example.so -lCrun
```

Of course, the extra libraries to use are completely non-portable---you will probably need to do some experimentation.

Sometimes people have suggested that it is necessary to relink the Python interpreter using the C++ compiler to make C++ extension modules work. In the experience of this author, this has never actually appeared to be necessary. Relinking the interpreter with C++ really only includes the special run-time libraries described above---as long as you link your extension modules with these libraries, it should not be necessary to rebuild Python.

If you aren't entirely sure about the linking of a C++ extension, you might look at an existing C++ program. On many Unix machines, the `ldd` command will list library dependencies. This should give you some clues about what you might have to include when you link your extension module. For example:

> 当然，要使用的额外库完全是不可移植的——你可能需要做一些实验。
>
> 有时人们建议必须使用 C++ 编译器重新链接 Python 解释器，以使 C++ 扩展模块正常工作。根据作者的经验，这实际上从来没有必要。使用 C++ 重新链接解释器实际上仅包含上述特殊的运行时库——只要将扩展模块与这些库链接，就不必重建 Python。
>
> 如果你不确定 C++ 扩展的链接，则可以查看现有的 C++ 程序。在许多 Unix 机器上，`ldd` 命令将列出库依赖关系。这应该为你提供一些有关链接扩展模块时可能必须包含的内容的线索。例如：

```
$ ldd SWIG
        libstdc++-libc6.1-1.so.2 => /usr/lib/libstdc++-libc6.1-1.so.2 (0x40019000)
        libm.so.6 => /lib/libm.so.6 (0x4005b000)
        libc.so.6 => /lib/libc.so.6 (0x40077000)
        /lib/ld-linux.so.2 => /lib/ld-linux.so.2 (0x40000000)
```

As a final complication, a major weakness of C++ is that it does not define any sort of standard for binary linking of libraries. This means that C++ code compiled by different compilers will not link together properly as libraries nor is the memory layout of classes and data structures implemented in any kind of portable manner. In a monolithic C++ program, this problem may be unnoticed. However, in Python, it is possible for different extension modules to be compiled with different C++ compilers. As long as these modules are self-contained, this probably won't matter. However, if these modules start sharing data, you will need to take steps to avoid segmentation faults and other erratic program behavior. If working with lots of software components, you might want to investigate using a more formal standard such as COM.

> 最后的麻烦是，C++ 的主要缺点是它没有为库的二进制链接定义任何类型的标准。这意味着由不同的编译器编译的 C++ 代码将无法正确链接为库，也无法以任何可移植的方式实现类和数据结构体的内存布局。在单一整体的 C++ 程序中，此问题可能没有引起注意。但是，在 Python 中，可以使用不同的 C++ 编译器来编译不同的扩展模块。只要这些模块是独立的，这可能就无关紧要。但是，如果这些模块开始共享数据，则将需要采取步骤来避免分段错误和其他不稳定的程序行为。如果使用大量软件组件，则可能需要使用更正式的标准（例如 COM）进行调查。

### 36.2.7 为 64 位平台编译

On platforms that support 64-bit applications (Solaris, Irix, etc.), special care is required when building extension modules. On these machines, 64-bit applications are compiled and linked using a different set of compiler/linker options. In addition, it is not generally possible to mix 32-bit and 64-bit code together in the same application.

To utilize 64-bits, the Python executable will need to be recompiled as a 64-bit application. In addition, all libraries, wrapper code, and every other part of your application will need to be compiled for 64-bits. If you plan to use other third-party extension modules, they will also have to be recompiled as 64-bit extensions.

If you are wrapping commercial software for which you have no source code, you will be forced to use the same linking standard as used by that software. This may prevent the use of 64-bit extensions. It may also introduce problems on platforms that support more than one linking standard (e.g., -o32 and -n32 on Irix).

On the Linux x86_64 platform (Opteron or EM64T), besides of the required compiler option -fPIC discussed above, you will need to be careful about the libraries you link with or the library path you use. In general, a Linux distribution will have two set of libraries, one for native x86_64 programs (under /usr/lib64), and another for 32 bits compatibility (under /usr/lib). Also, the compiler options -m32 and -m64 allow you to choose the desired binary format for your python extension.

> 在支持 64 位应用程序的平台（Solaris、Irix 等）上，构建扩展模块时需要特别注意。在这些计算机上，使用一组不同的编译器或链接器选项来编译和链接 64 位应用程序。此外，通常不可能在同一应用程序中将 32 位和 64 位代码混合在一起。
>
> 要利用 64 位，需要将 Python 可执行文件重新编译为 64 位应用程序。此外，所有库、包装代码以及应用程序的所有其他部分都需要编译为 64 位。如果你打算使用其他第三方扩展模块，则它们也必须重新编译为 64 位扩展。
>
> 如果要包装没有源代码的商业软件，则将被迫使用与该软件相同的链接标准。这可能会阻止使用 64 位扩展名。它还可能在支持多个链接标准的平台（例如 Irix 上的 `-o32` 和 `-n32`）上引入问题。
>
> 在 Linux x86_64 平台（Opteron 或 EM64T）上，除了上面讨论的必需的编译器选项 `-fPIC` 外，你还需要注意与之链接的库或使用的库路径。通常，Linux 发行版将具有两组库，一组用于本地 x86_64 程序（在 `/usr/lib64` 下），另一组用于 32 位兼容性（在 `/usr/lib` 下）。另外，编译器选项 `-m32` 和 `-m64` 允许你为 python 扩展选择所需的二进制格式。

### 36.2.8 在 Windows 下构建 Python 扩展

Building a SWIG extension to Python under Windows is roughly similar to the process used with Unix. Using the distutils, it is essentially identical. If you have the same version of the MS compiler that Python was built with (the python2.4 and python2.5 distributed by python.org are built with Visual Studio 2003), the standard `python setup.py build` should just work.

As of python2.5, the distutils support building extensions with MingGW out of the box. Following the instruction here: [Building Python extensions for Windows with only free tools](http://boodebr.org/main/python/build-windows-extensions) should get you started.

If you need to build it on your own, the following notes are provided:

You will need to create a DLL that can be loaded into the interpreter. This section briefly describes the use of SWIG with Microsoft Visual C++. As a starting point, many of SWIG's examples include project files (.dsp files) for Visual C++ 6. These can be opened by more recent versions of Visual Studio. You might want to take a quick look at these examples in addition to reading this section.

In Developer Studio, SWIG should be invoked as a custom build option. This is usually done as follows:

- Open up a new workspace and use the AppWizard to select a DLL project.
- Add both the SWIG interface file (the .i file), any supporting C files, and the name of the wrapper file that will be created by SWIG (ie. `example_wrap.c`). Note : If using C++, choose a different suffix for the wrapper file such as `example_wrap.cxx`. Don't worry if the wrapper file doesn't exist yet--Developer Studio keeps a reference to it.
- Select the SWIG interface file and go to the settings menu. Under settings, select the "Custom Build" option.
- Enter "SWIG" in the description field.
- Enter `SWIG -python -o $(ProjDir)\$(InputName)_wrap.c $(InputPath)` in the "Build command(s) field"
- Enter `$(ProjDir)\$(InputName)_wrap.c` in the "Output files(s) field".
- Next, select the settings for the entire project and go to "C++:Preprocessor". Add the include directories for your Python installation under "Additional include directories".
- Define the symbol `__WIN32__` under preprocessor options.
- Finally, select the settings for the entire project and go to "Link Options". Add the Python library file to your link libraries. For example "python21.lib". Also, set the name of the output file to match the name of your Python module, ie. _example.pyd - Note that _example.dll also worked with Python-2.4 and earlier.
- Build your project.

If all went well, SWIG will be automatically invoked whenever you build your project. Any changes made to the interface file will result in SWIG being automatically executed to produce a new version of the wrapper file.

To run your new Python extension, simply run Python and use the `import` command as normal. For example :

> 在 Windows 下为 Python 构建 SWIG 扩展程序与 Unix 大致类似。使用 `distutils`，基本上是相同的。如果你使用与构建 Python 相同的 MS 编译器版本（由 python.org 分发的 python2.4 和 python2.5 是由 Visual Studio 2003 构建的），则标准的 `python setup.py build` 应该可以正常工作。
>
> 从 python2.5 开始，`distutils` 支持开箱即用的 MingGW 构建扩展。请按照此处的说明进行操作：[仅使用免费工具构建 Windows 的 Python 扩展](http://boodebr.org/main/python/build-windows-extensions)可以帮助你入门。
>
> 如果需要自己构建，请提供以下注意事项：
>
> 你将需要创建一个可以加载到解释器中的 DLL。本节简要介绍了将 SWIG 与 Microsoft Visual C++ 一起使用。首先，SWIG 的许多示例都包含用于 Visual C++ 6 的项目文件（`.dsp` 文件）。可以使用最新版本的 Visual Studio 打开这些文件。除了阅读本节之外，你可能还想快速看一下这些示例。
>
> 在 Developer Studio 中，应将 SWIG 作为自定义生成选项调用。通常按以下步骤进行：
> - 打开一个新的工作区，并使用 AppWizard 选择一个 DLL 项目。
> - 添加 SWIG 接口文件（`.i` 文件），所有支持的 C 文件以及 SWIG 将创建的包装文件的名称（即 `example_wrap.c`）。注意：如果使用 C++，请为包装文件选择其他后缀，例如 `example_wrap.cxx`。不用担心包装文件是否不存在，​​Developer Studio 会保留对其的引用。
> - 选择 SWIG 接口文件，然后转到设置菜单。在设置下，选择`自定义版本` 选项。
> - 在说明字段中输入 `SWIG`。
> - 在“构建命令”字段中输入 `SWIG -python -o $(ProjDir)\ $(InputName)_wrap.c $(InputPath)`
> - 在“输出文件”字段中输入 `$(ProjDir)\ $(InputName)_wrap.c`。
> - 接下来，选择整个项目的设置，然后转到 `C++：Preprocessor`。在“其他包含目录”下为你的 Python 安装添加包含目录。
> - 在预处理器选项下定义符号 `__WIN32__`。
> - 最后，选择整个项目的设置，然后转到“链接选项”。将 Python 库文件添加到链接库。例如 `python21.lib`。另外，设置输出文件的名称以匹配你的 Python 模块的名称。`_example.pyd`——注意 `_example.dll` 也可用于 Python2.4 和更早版本。
> - 建立你的项目。
>
> 如果一切顺利，则在你构建项目时将自动调用 SWIG 。对接口文件所做的任何更改都将导致 SWIG 自动执行以产生新版本的包装文件。
>
> 要运行新的 Python 扩展，只需运行 Python 并正常使用 `import` 命令即可。例如：

```python
$ python
>>> import example
>>> print example.fact(4)
24
>>>
```

If you get an `ImportError` exception when importing the module, you may have forgotten to include additional library files when you built your module. If you get an access violation or some kind of general protection fault immediately upon import, you have a more serious problem. This is often caused by linking your extension module against the wrong set of Win32 debug or thread libraries. You will have to fiddle around with the build options of project to try and track this down.

A 'Debug' build of the wrappers requires a debug build of the Python interpreter. This normally requires building the Python interpreter from source, which is not a job for the feint-hearted. Alternatively you can use the 'Release' build of the Python interpreter with a 'Debug' build of your wrappers by defining the `SWIG_PYTHON_INTERPRETER_NO_DEBUG` symbol under the preprocessor options. Or you can ensure this macro is defined at the beginning of the wrapper code using the following in your interface file, where `_MSC_VER`ensures it is only used by the Visual Studio compiler:

> 如果在导入模块时收到 `ImportError` 异常，则可能在构建模块时忘记了包含其他库文件。如果在导入时立即遇到访问冲突或某种常规保护错误，那么你会遇到更严重的问题。这通常是由于将扩展模块链接到错误的 Win32 调试或线程库集引起的。你将不得不弄乱项目的构建选项，以尝试对其进行跟踪。
>
> 包装程序的调试版本需要 Python 解释器的调试版本。这通常需要从源代码构建 Python 解释器，这对胆小的人来说不是一件容易的事。或者，你可以通过在预处理器选项下定义 `SWIG_PYTHON_INTERPRETER_NO_DEBUG` 符号来将 Python 解释器的发布版本与包装的调试版本一起使用。或者，你可以使用接口文件中的以下命令确保在包装代码的开头定义了此宏，其中 `_MSC_VER` 确保仅由 Visual Studio 编译器使用：

```
%begin %{
#ifdef_MSC_VER
#define SWIG_PYTHON_INTERPRETER_NO_DEBUG
#endif
%}
```

Some users have reported success in building extension modules using Cygwin and other compilers. However, the problem of building usable DLLs with these compilers tends to be rather problematic. For the latest information, you may want to consult the [SWIG Wiki](https://github.com/SWIG/SWIG/wiki).

> 一些用户报告说成功使用 Cygwin 和其他编译器构建扩展模块。但是，使用这些编译器构建可用的 DLL 的问题往往很成问题。有关最新信息，你可能需要查阅 [SWIG Wiki](https://github.com/SWIG/SWIG/wiki)。

## 36.3 一个 C/C++ 包装基础教程

By default, SWIG tries to build a very natural Python interface to your C/C++ code. Functions are wrapped as functions, classes are wrapped as classes, and so forth. This section briefly covers the essential aspects of this wrapping.

> 默认情况下，SWIG 会尝试为你的 C/C++ 代码构建一个非常自然的 Python 接口。函数包装为函数，类包装为类，依此类推。本节简要介绍了此包装的基本方面。

### 36.3.1 模块

The SWIG `%module` directive specifies the name of the Python module. If you specify `%module example`, then everything is wrapped into a Python `example` module. Underneath the covers, this module consists of a Python source file `example.py` and a low-level extension module `_example.so`. When choosing a module name, make sure you don't use the same name as a built-in Python command or standard module name.

> SWIG 指令 `%module` 指定 Python 模块的名称。如果你指定了 `%module example`，那么所有的东西都会被包装到一个叫 `example` 的 Python 模块中。在幕后，这个模块由一个 Python 源文件 `example.py` 和一个低级扩展模块 `_example.so` 组成。选择模块名称时，请确保不要使用与内置 Python 命令或标准模块相同的名称。

### 36.3.2 函数

Global functions are wrapped as new Python built-in functions. For example,

> 全局函数被包装为一个新的 Python 内置函数。例如， 

```
%module example
int fact(int n);
```

creates a built-in function `example.fact(n)` that works exactly like you think it does:

> 创建一个内置函数 `example.fact(n)`，工作方式和你想的一样：

```python
>>> import example
>>> print example.fact(4)
24
>>>
```

### 36.3.3 全局变量

C/C++ global variables are fully supported by SWIG . However, the underlying mechanism is somewhat different than you might expect due to the way that Python assignment works. When you type the following in Python

> SWIG 完全支持 C/C++ 全局变量。但是，由于 Python 分配的工作方式，其底层机制与你预期的有所不同。当你在 Python 中键入以下内容时

```python
a = 3.4
```

"a" becomes a name for an object containing the value 3.4. If you later type

> `a` 是一个包含值 3.4 的对象的名字。如果你接着键入

```python
b = a
```

then "a" and "b" are both names for the object containing the value 3.4. Thus, there is only one object containing 3.4 and "a" and "b" are both names that refer to it. This is quite different than C where a variable name refers to a memory location in which a value is stored (and assignment copies data into that location). Because of this, there is no direct way to map variable assignment in C to variable assignment in Python.

To provide access to C global variables, SWIG creates a special object called `cvar`' that is added to each SWIG generated module. Global variables are then accessed as attributes of this object. For example, consider this interface

> 那么 `a` 和 `b` 都是包含值 3.4 的对象的名称。因此，只有一个对象包含 3.4，并且 `a` 和 `b` 都是引用它的名称。这与 C 完全不同，C 的变量名是指存储值的存储位置（赋值将数据复制到该位置）。因此，没有直接的方法可以将 C 中的变量分配映射到 Python 中的变量分配。
>
> 为了提供对 C 全局变量的访问，SWIG 创建了一个名为 `cvar` 的特殊对象，该对象被添加到每个 SWIG 生成的模块中。然后访问全局变量作为该对象的属性。例如，考虑以下接口

```
//SWIG interface file with global variables
%module example
...
%inline %{
extern int My_variable;
extern double density;
%}
...
```

Now look at the Python interface:

> 现在看看 Python 接口：

```python
>>> import example
>>> # Print out value of a C global variable
>>> print example.cvar.My_variable
4
>>> # Set the value of a C global variable
>>> example.cvar.density = 0.8442
>>> # Use in a math operation
>>> example.cvar.density = example.cvar.density*1.10
```

If you make an error in variable assignment, you will receive an error message. For example:

> 如果你在变量声明中犯了错误，你会收到一条错误信息。例如：

```python
>>> example.cvar.density = "Hello"
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
TypeError: C variable 'density (double)'
>>>
```

If a variable is declared as `const`, it is wrapped as a read-only variable. Attempts to modify its value will result in an error.

To make ordinary variables read-only, you can use the `%immutable` directive. For example:

> 如果变量声明为 `const`，则将其包装为只读变量。尝试修改其值将导致错误。
>
> 要使普通变量为只读，可以使用 `%immutable` 指令。例如：

```
%{
extern char *path;
%}
%immutable;
extern char *path;
%mutable;
```

The `%immutable` directive stays in effect until it is explicitly disabled or cleared using `%mutable`. See the [Creating read-only variables](http://www.SWIG.org/Doc3.0/SWIG.html#SWIG_readonly_variables) section for further details.

If you just want to make a specific variable immutable, supply a declaration name. For example:

> `%immutable` 指令将一直有效，直到使用 `%mutable` 将其明确禁用或清除为止。有关更多详细信息，请参见[创建只读变量](http://www.SWIG.org/Doc3.0/SWIG.html#SWIG_readonly_variables)部分。
>
> 如果只想使特定变量不可变，请提供声明名称。例如：

```
%{
extern char *path;
%}
%immutable path;
...
extern char *path;      // Read-only (due to %immutable)
```

If you would like to access variables using a name other than `cvar`, it can be changed using the `-globals` option :

> 如果你想使用 `cvar` 以外的名称访问变量，则可以使用 `-globals` 选项进行更改：

```
$ SWIG -python -globals myvar example.i
```

Some care is in order when importing multiple SWIG modules. If you use the `from  import *` style of importing, you will get a name clash on the variable `cvar`' and you will only be able to access global variables from the last module loaded. To prevent this, you might consider renaming `cvar` or making it private to the module by giving it a name that starts with a leading underscore. SWIG does not create `cvar` if there are no global variables in a module.

> 导入多个 SWIG 模块时需要特别注意。如果使用导入方式 `from import *`，则变量 `cvar` 会发生名称冲突，并且只能从最后加载的模块访问全局变量。为了防止这种情况，你可以考虑重命名 `cvar`，或通过给模块起一个以下划线开头的名称来使其对模块私有。如果模块中没有全局变量，SWIG 不会创建 `cvar`。

### 36.3.4 常量与枚举

C/C++ constants are installed as Python objects containing the appropriate value. To create a constant, use `#define`, `enum`, or the `%constant` directive. For example:

> C/C++ 常量作为包含适当值的 Python 对象安装。要创建一个常量，请使用 `#define`，`enum` 或 `%constant` 指令。例如：

```
#define PI 3.14159
#define VERSION "1.0"

enum Beverage { ALE, LAGER, STOUT, PILSNER };

%constant int FOO = 42;
%constant const char *path = "/usr/local";
```

For enums, make sure that the definition of the enumeration actually appears in a header file or in the wrapper file somehow---if you just stick an enum in a SWIG interface without also telling the C compiler about it, the wrapper code won't compile.

Note: declarations declared as `const` are wrapped as read-only variables and will be accessed using the `cvar` object described in the previous section. They are not wrapped as constants. For further discussion about this, see the [SWIG Basics](http://www.SWIG.org/Doc3.0/SWIG.html#SWIG) chapter.

Constants are not guaranteed to remain constant in Python---the name of the constant could be accidentally reassigned to refer to some other object. Unfortunately, there is no easy way for SWIG to generate code that prevents this. You will just have to be careful.

> 对于枚举，请确保枚举的定义实际上以某种方式出现在头文件或包装器文件中，如果你只是将枚举粘贴在 SWIG 接口文件中而不告知 C 编译器，则包装器代码将不会编译。
>
> 注意：声明为 `const` 的变量被包装为只读变量，将使用上一节中描述的 `cvar` 对象进行访问。它们没有包装为常量。有关此问题的更多讨论，请参见 [SWIG 基础知识](http://www.SWIG.org/Doc3.0/SWIG.html#SWIG)一章。
>
> 不能保证在 Python 中常量会保持不变——常量的名称可能会被意外重分配以引用其他对象。不幸的是，SWIG 没有简单的方法来生成防止这种情况的代码。你只需要小心。

### 36.3.5 指针

C/C++ pointers are fully supported by SWIG . Furthermore, SWIG has no problem working with incomplete type information. Here is a rather simple interface:

> SWIG 完全支持 C/C++ 指针。此外，SWIG 在处理不完整的类型信息时没有问题。这里有一个相当简单的接口：

```
%module example

FILE *fopen(const char *filename, const char *mode);
int fputs(const char *, FILE *);
int fclose(FILE *);
```

When wrapped, you will be able to use the functions in a natural way from Python. For example:

> 打包后，你将能够从 Python 自然地使用这些函数。例如：

```python
>>> import example
>>> f = example.fopen("junk", "w")
>>> example.fputs("Hello World\n", f)
>>> example.fclose(f)
```

If this makes you uneasy, rest assured that there is no deep magic involved. Underneath the covers, pointers to C/C++ objects are simply represented as opaque values using an especial python container object:

> 如果这让你感到不安，请放心，其中不涉及任何深奥的魔法。在幕后，使用特殊的 Python 容器对象将 C/C++ 对象的指针简单地表示为不透明值：

```python
>>> print f
< SWIG Object of type 'FILE *' at 0xb7d6f470>
```

This pointer value can be freely passed around to different C functions that expect to receive an object of type `FILE *`. The only thing you can't do is dereference the pointer from Python. Of course, that isn't much of a concern in this example.

In older versions of SWIG (1.3.22 or older), pointers were represented using a plain string object. If you have an old package that still requires that representation, or you just feel nostalgic, you can always retrieve it by casting the pointer object to a string:

> 这个指针值可以自由地传递给不同的 C 函数，而这些函数希望接受 `FILE *` 类型的对象。你唯一不能做的就是从 Python 解引用指针。当然，在此示例中，这并不是什么大问题。
>
> 在较早版本的 SWIG（1.3.22 或更早版本）中，指针使用纯字符串对象表示。如果你有一个旧的程序包仍然需要该表示形式，或者只是怀旧，可以随时通过将指针对象转换为字符串来检索它：

```python
>>> print str(f)
_c0671108_p_FILE
```

Also, if you need to pass the raw pointer value to some external python library, you can do it by casting the pointer object to an integer:

> 另外，如果你需要将原始指针值传递给某个外部 python 库，则可以通过将指针对象转换为整数来实现：

```python
>>> print int(f)
135833352
```

However, the inverse operation is not possible, i.e., you can't build a SWIG pointer object from a raw integer value.

Note also that the '0' or NULL pointer is always represented by `None`, no matter what type SWIG is addressing. In the previous example, you can call:

> 但是，反向操作是不可能的，你不能从原始整数值构建 SWIG 指针对象。
>
> 还要注意，无论 SWIG 寻址哪种类型，`0` 或 NULL 指针始终由 `None` 表示。在上一个示例中，你可以调用：

```python
>>> example.fclose(None)
```

and that will be equivalent to the following, but not really useful, C code:

> 这等价于下面，但并非真的有用，C 代码：

```c++
FILE *f = NULL;
fclose(f);
```

As much as you might be inclined to modify a pointer value directly from Python, don't. The hexadecimal encoding is not necessarily the same as the logical memory address of the underlying object. Instead it is the raw byte encoding of the pointer value. The encoding will vary depending on the native byte-ordering of the platform (i.e., big-endian vs. little-endian). Similarly, don't try to manually cast a pointer to a new type by simply replacing the type-string. This may not work like you expect, it is particularly dangerous when casting C++ objects. If you need to cast a pointer or change its value, consider writing some helper functions instead. For example:

> 尽管你可能倾向于直接从 Python 修改指针值，但不要这样做。十六进制编码不一定与底层对象的逻辑内存地址相同。相反，它是指针值的原始字节编码。编码会根据平台的原始字节顺序而有所不同（即 big-endian 与 little-endian）。同样，不要尝试通过简单地替换类型字符串来手动将指针转换为新类型。这可能无法按预期工作，在转换 C++ 对象时尤其危险。如果需要强制转换指针或更改其值，请考虑编写一些辅助函数。例如：

```
%inline %{
/* C-style cast */
Bar *FooToBar(Foo *f) {
  return (Bar *) f;
}

/* C++-style cast */
Foo *BarToFoo(Bar *b) {
  return dynamic_cast<Foo*>(b);
}

Foo *IncrFoo(Foo *f, int i) {
    return f+i;
}
%}
```

Also, if working with C++, you should always try to use the new C++ style casts. For example, in the above code, the C-style cast may return a bogus result whereas as the C++-style cast will return `None` if the conversion can't be performed.

> 另外，如果使用 C++，则应始终尝试使用新的 C++ 样式强制转换。例如，在上面的代码中，C 样式转换可能会返回假结果，而 C++ 样式转换会在无法执行转换的情况下返回 `None`。

### 36.3.6 结构体

If you wrap a C structure, it is wrapped by a Python class. This provides a very natural interface. For example,

> 如果包装 C 结构体，则由 Python 类包装。这提供了非常自然的接口。例如，

```c++
struct Vector {
  double x, y, z;
};
```

is used as follows:

> 将以如下方式是用：

```python
>>> v = example.Vector()
>>> v.x = 3.5
>>> v.y = 7.2
>>> print v.x, v.y, v.z
7.8 -4.5 0.0
>>>
```

Similar access is provided for unions and the data members of C++ classes.

If you print out the value of `v` in the above example, you will see something like this:

> 为联合体和 C++ 类的数据成员提供了类似的访问。
>
> 如果在上面的示例中打印出 `v` 的值，你将看到类似以下内容：

```python
>>> print v
<C Vector instance at _18e31408_p_Vector>
```

This object is actually a Python instance that has been wrapped around a pointer to the low-level C structure. This instance doesn't actually do anything--it just serves as a proxy. The pointer to the C object can be found in the `.this`attribute. For example:

> 该对象实际上是一个 Python 实例，该实例被包装在指向低级 C 结构体的指针周围。该实例实际上不执行任何操作，只是充当代理。指向 C 对象的指针可以在 `.this` 属性中找到。例如：

```python
>>> print v.this
_18e31408_p_Vector
>>>
```

Further details about the Python proxy class are covered a little later.

`const` members of a structure are read-only. Data members can also be forced to be read-only using the `%immutable` directive. For example:

> 有关 Python 代理类的更多详细信息将在稍后介绍。
>
> 结构体的 `const` 成员是只读的。还可以使用 `%immutable` 指令将数据成员强制为只读。例如：

```
struct Foo {
  ...
  %immutable;
  int x;        /* Read-only members */
  char *name;
  %mutable;
  ...
};
```

When `char *` members of a structure are wrapped, the contents are assumed to be dynamically allocated using `malloc` or `new` (depending on whether or not SWIG is run with the -c++ option). When the structure member is set, the old contents will be released and a new value created. If this is not the behavior you want, you will have to use a typemap (described later).

If a structure contains arrays, access to those arrays is managed through pointers. For example, consider this:

> 当包装结构体的 `char *` 成员时，假定内容是使用 `malloc` 或 `new` 动态分配的（取决于 SWIG 是否使用 `-c++` 选项运行）。设置结构体成员后，将释放旧内容并创建新值。如果这不是你想要的行为，则必须使用一个类型映射（稍后描述）。
>
> 如果结构体包含数组，则通过指针管理对这些数组的访问。例如，考虑以下：

```c++
struct Bar {
    int  x[16];
};
```

If accessed in Python, you will see behavior like this:

> 如果使用 Python 访问，你将看到以下行为：

```python
>>> b = example.Bar()
>>> print b.x
_801861a4_p_int
>>>
```

This pointer can be passed around to functions that expect to receive an `int *`(just like C). You can also set the value of an array member using another pointer. For example:

> 该指针可以传递给期望接受 `int *` 的函数（就像 C 一样）。你还可以使用另一个指针设置数组成员的值。例如：

```python
>>> c = example.Bar()
>>> c.x = b.x             # Copy contents of b.x to c.x
```

For array assignment, SWIG copies the entire contents of the array starting with the data pointed to by `b.x`. In this example, 16 integers would be copied. Like C, SWIG makes no assumptions about bounds checking---if you pass a bad pointer, you may get a segmentation fault or access violation.

When a member of a structure is itself a structure, it is handled as a pointer. For example, suppose you have two structures like this:

> 对于数组分配，SWIG 从 `b.x` 指向的数据开始复制数组的全部内容。在此示例中，将复制 16 个整数。与 C 一样，SWIG 也不做边界检查的假设，如果传递错误的指针，则可能会遇到分段错误或访问冲突。
>
> 当结构体的成员本身是结构体时，它将作为指针处理。例如，假设你有两个这样的结构体：

```c++
struct Foo {
  int a;
};

struct Bar {
  Foo f;
};
```

Now, suppose that you access the `f` attribute of `Bar` like this:

> 现在，假设你访问 `Bar` 的 `f` 属性：

```
>>> b = Bar()
>>> x = b.f
```

In this case, `x` is a pointer that points to the `Foo` that is inside `b`. This is the same value as generated by this C code:

> 在这种情况下，`x` 是指向 `b` 内部的 `Foo` 的指针。此值与此 C 代码生成的值相同：

```c++
Bar b;
Foo *x = &b->f;       /* Points inside b */
```

Because the pointer points inside the structure, you can modify the contents and everything works just like you would expect. For example:

> 因为指针指向结构体内部，所以你可以修改内容，一切都可以按你期望的方式进行。例如：

```
>>> b = Bar()
>>> b.f.a = 3               # Modify attribute of structure member
>>> x = b.f
>>> x.a = 3                 # Modifies the same structure
```

### 36.3.7 C++ 类

C++ classes are wrapped by Python classes as well. For example, if you have this class,

> C++ 类也由 Python 类包装。例如，如果你有这样的类，

```c++
class List {
public:
  List();
  ~List();
  int  search(char *item);
  void insert(char *item);
  void remove(char *item);
  char *get(int n);
  int  length;
};
```

you can use it in Python like this:

> 在 Python 中你可以这样是用：

```
>>> l = example.List()
>>> l.insert("Ale")
>>> l.insert("Stout")
>>> l.insert("Lager")
>>> l.get(1)
'Stout'
>>> print l.length
3
>>>
```

Class data members are accessed in the same manner as C structures.

Static class members present a special problem for Python. Prior to Python-2.2, Python classes had no support for static methods and no version of Python supports static member variables in a manner that SWIG can utilize. Therefore, SWIG generates wrappers that try to work around some of these issues. To illustrate, suppose you have a class like this:

> 类数据成员的访问方式与 C 结构体相同。
>
> 静态类成员为 Python 带来了一个特殊问题。在 Python-2.2 之前，Python 类不支持静态方法，并且没有任何版本的 Python 支持 SWIG 可以利用的静态成员变量。因此，SWIG 会生成包装程序，以尝试解决其中一些问题。为了说明这一点，假设你有一个像这样的类：

```c++
class Spam {
public:
  static void foo();
  static int bar;
};
```

In Python, the static member can be access in three different ways:

> 在 Python 中，静态方法可以以三种不同的方式访问：

```
>>> example.Spam_foo()    # Spam::foo()
>>> s = example.Spam()
>>> s.foo()               # Spam::foo() via an instance
>>> example.Spam.foo()    # Spam::foo(). Python-2.2 only
```

The first two methods of access are supported in all versions of Python. The last technique is only available in Python-2.2 and later versions.

Static member variables are currently accessed as global variables. This means, they are accessed through `cvar` like this:

> 所有版本的 Python 都支持前两种访问方法。最后一种技术仅在 Python-2.2 和更高版本中可用。
>
> 静态成员变量当前作为全局变量访问。这意味着，可以通过 `cvar` 来访问它们，如下所示：

```
>>> print example.cvar.Spam_bar
7
```

### 36.3.8 C++ 继承

SWIG is fully aware of issues related to C++ inheritance. Therefore, if you have classes like this

> SWIG 完全了解与 C++ 继承有关的问题。因此，如果你有这样的类

```c++
class Foo {
...
};

class Bar : public Foo {
...
};
```

those classes are wrapped into a hierarchy of Python classes that reflect the same inheritance structure. All of the usual Python utility functions work normally:

> 这些类被包装成可反映相同继承结构的 Python 类层次结构。所有常用的 Python 实用程序功能均可正常运行：

```python
>>> b = Bar()
>>> instance(b, Foo)
1
>>> issubclass(Bar, Foo)
1
>>> issubclass(Foo, Bar)
0
```

Furthermore, if you have functions like this

> 更进一步，如果你有这样的函数：

```c++
void spam(Foo *f);
```

then the function `spam()` accepts `Foo *` or a pointer to any class derived from `Foo`.

It is safe to use multiple inheritance with SWIG .

> 然后函数 `spam()` 接受 `Foo *` 或指向从 `Foo` 的任何派生类的指针。
>
> 在 SWIG 中使用多重继承是安全的。

### 36.3.9 指针、引用、值和数组

In C++, there are many different ways a function might receive and manipulate objects. For example:

> 在 C++ 中，函数可以通过许多不同的方式接受和操作对象。例如：

```c++
void spam1(Foo *x);      // Pass by pointer
void spam2(Foo &x);      // Pass by reference
void spam3(const Foo &x);// Pass by const reference
void spam4(Foo x);       // Pass by value
void spam5(Foo x[]);     // Array of objects
```

In Python, there is no detailed distinction like this--specifically, there are only "objects". There are no pointers, references, arrays, and so forth. Because of this, SWIG unifies all of these types together in the wrapper code. For instance, if you actually had the above functions, it is perfectly legal to do this:

> 在 Python 中，没有这样的详细区分。具体来说，只有对象。没有指针，引用，数组等。因此，SWIG 在包装代码中将所有这些类型统一在一起。例如，如果你实际上具有上述函数，则这样做是完全合法的：

```
>>> f = Foo()           # Create a Foo
>>> spam1(f)            # Ok. Pointer
>>> spam2(f)            # Ok. Reference
>>> spam3(f)            # Ok. Const reference
>>> spam4(f)            # Ok. Value.
>>> spam5(f)            # Ok. Array (1 element)
```

Similar behavior occurs for return values. For example, if you had functions like this,

> 类似的行为出现在返回值上。例如，如果你的函数像这样，

```
Foo *spam6();
Foo &spam7();
Foo  spam8();
const Foo &spam9();
```

then all three functions will return a pointer to some `Foo` object. Since the third function (spam8) returns a value, newly allocated memory is used to hold the result and a pointer is returned (Python will release this memory when the return value is garbage collected). The fourth case (spam9) which returns a const reference, in most of the cases will be treated as a returning value, and it will follow the same allocation/deallocation process.

> 那么这三个函数都会返回一个指向 `Foo` 对象的指针。由于第三个函数（`spam8`）返回一个值，因此将使用新分配的内存来保存结果并返回一个指针（当对返回值进行垃圾回收时，Python 将释放该内存）。返回常量引用的第四种情况（`spam9`）在大多数情况下将被视为返回值，并且将遵循相同的分配/取消分配过程。

### 36.3.10 C++ 重载函数

C++ overloaded functions, methods, and constructors are mostly supported by SWIG . For example, if you have two functions like this:

> SWIG 支持巨大部分 C++ 重载函数、方法和构造函数。例如，如果你有两个这样的函数：

```c++
void foo(int);
void foo(char *c);
```

You can use them in Python in a straightforward manner:

> 你可以在 Python 以直接的方式使用它们：

```
>>> foo(3)           # foo(int)
>>> foo("Hello")     # foo(char *c)
```

Similarly, if you have a class like this,

> 类似的，如果你有一个这样的类，

```c++
class Foo {
public:
    Foo();
    Foo(const Foo &);
    ...
};
```

you can write Python code like this:

> 你可以这样写 Python 代码

```
>>> f = Foo()          # Create a Foo
>>> g = Foo(f)         # Copy f
```

Overloading support is not quite as flexible as in C++. Sometimes there are methods that SWIG can't disambiguate. For example:

> 重载支持不像 C++ 那样灵活。有时有些方法 SWIG 无法消除歧义。例如：

```
void spam(int);
void spam(short);
```

or

> 或

```
void foo(Bar *b);
void foo(Bar &b);
```

If declarations such as these appear, you will get a warning message like this:

> 如果出现了这些声明，你活收到一条警告：

```
example.i:12: Warning 509: Overloaded method spam(short) effectively ignored,
example.i:11: Warning 509: as it is shadowed by spam(int).
```

To fix this, you either need to ignore or rename one of the methods. For example:

> 为了修正，你可以省略或重命名其中一个方法。例如：

```
%rename(spam_short) spam(short);
...
void spam(int);
void spam(short);   // Accessed as spam_short
```

or

> 或

```
%ignore spam(short);
...
void spam(int);
void spam(short);   // Ignored
```

SWIG resolves overloaded functions and methods using a disambiguation scheme that ranks and sorts declarations according to a set of type-precedence rules. The order in which declarations appear in the input does not matter except in situations where ambiguity arises--in this case, the first declaration takes precedence.

Please refer to the "SWIG and C++" chapter for more information about overloading.

> SWIG 使用消除歧义的方案来解析重载的函数和方法，该方案根据一组类型优先规则对声明进行排序和排序。声明出现在输入中的顺序无关紧要，除非出现歧义的情况——在这种情况下，第一个声明优先。
>
> 请参阅 **SWIG 和 C++** 一章以获取有关重载的更多信息。

### 36.3.11 C++ 运算符

Certain C++ overloaded operators can be handled automatically by SWIG . For example, consider a class like this:

> SWIG 可以自动处理某些 C++ 重载的运算符。例如，考虑这样的一个类：

```c++
class Complex {
private:
  double rpart, ipart;
public:
  Complex(double r = 0, double i = 0) : rpart(r), ipart(i) { }
  Complex(const Complex &c) : rpart(c.rpart), ipart(c.ipart) { }
  Complex &operator=(const Complex &c);

  Complex operator+=(const Complex &c) const;
  Complex operator+(const Complex &c) const;
  Complex operator-(const Complex &c) const;
  Complex operator*(const Complex &c) const;
  Complex operator-() const;

  double re() const { return rpart; }
  double im() const { return ipart; }
};
```

When wrapped, it works like you expect:

> 包装后将像你预期的一样运行：

```python
>>> c = Complex(3, 4)
>>> d = Complex(7, 8)
>>> e = c + d
>>> e.re()
10.0
>>> e.im()
12.0
>>> c += d
>>> c.re()
10.0
>>> c.im()
12.0
```

One restriction with operator overloading support is that SWIG is not able to fully handle operators that aren't defined as part of the class. For example, if you had code like this

> 支持运算符重载有一个限制，那就是 SWIG 无法完全处理未定义成类的一部分的运算符。例如，如果你有这样的代码

```c++
class Complex {
...
friend Complex operator+(double, const Complex &c);
...
};
```

then SWIG ignores it and issues a warning. You can still wrap the operator, but you may have to encapsulate it in a special function. For example:

> 然后 SWIG 会忽略它并发出警告。你仍然可以包装运算符，但是可能必须将其封装在特殊函数中。例如：

```
%rename(Complex_add_dc) operator+(double, const Complex &);
```

There are ways to make this operator appear as part of the class using the `%extend` directive. Keep reading.

Also, be aware that certain operators don't map cleanly to Python. For instance, overloaded assignment operators don't map to Python semantics and will be ignored.

> 有多种方法可以使用 `%extend` 指令使该运算符出现在类中。继续阅读。
>
> 另外，请注意，某些运算符不能完全映射到 Python。例如，重载的赋值运算符不会映射到 Python 语义，因此将被忽略。

### 36.3.12 C++ 命名空间

SWIG is aware of C++ namespaces, but namespace names do not appear in the module nor do namespaces result in a module that is broken up into submodules or packages. For example, if you have a file like this,

> SWIG 知道 C++ 命名空间，但是命名空间名称不会出现在模块中，命名空间也不会导致模块分解为子模块或程序包。例如，如果你有一个像这样的文件，

```
%module example

namespace foo {
  int fact(int n);
  struct Vector {
    double x, y, z;
  };
};
```

it works in Python as follows:

> 在 Python 中它将如下运行：

```python
>>> import example
>>> example.fact(3)
6
>>> v = example.Vector()
>>> v.x = 3.4
>>> print v.y
0.0
>>>
```

If your program has more than one namespace, name conflicts (if any) can be resolved using `%rename` For example:

> 如果你的程序有多个命名空间，则可以使用 `%rename` 解决名称冲突（如果有的话），例如：

```
%rename(Bar_spam) Bar::spam;

namespace Foo {
    int spam();
}

namespace Bar {
    int spam();
}
```

If you have more than one namespace and your want to keep their symbols separate, consider wrapping them as separate SWIG modules. For example, make the module name the same as the namespace and create extension modules for each namespace separately. If your program utilizes thousands of small deeply nested namespaces each with identical symbol names, well, then you get what you deserve.

> 如果你有多个命名空间，并且想要将它们的符号分开，请考虑将它们包装为单独的 SWIG 模块。例如，使模块名称与命名空间相同，并分别为每个命名空间创建扩展模块。如果你的程序利用了数千个深层嵌套的小型命名空间，每个命名空间都具有相同的符号名称，那么你是自作自受。

### 36.3.13 C++ 模板

C++ templates don't present a huge problem for SWIG . However, in order to create wrappers, you have to tell SWIG to create wrappers for a particular template instantiation. To do this, you use the `%template` directive. For example:

> 对于 SWIG，C++ 模板不会带来很大的问题。但是，为了创建包装器，必须告诉 SWIG 为特定的模板实例创建包装器。为此，你可以使用 `%template` 指令。例如：

```
%module example
%{
#include "pair.h"
%}

template<class T1, class T2>
struct pair {
  typedef T1 first_type;
  typedef T2 second_type;
  T1 first;
  T2 second;
  pair();
  pair(const T1&, const T2&);
 ~pair();
};

%template(pairii) pair<int, int>;
```

In Python:

> 在 Python 中：

```python
>>> import example
>>> p = example.pairii(3, 4)
>>> p.first
3
>>> p.second
4
```

Obviously, there is more to template wrapping than shown in this example. More details can be found in the [SWIG and C++](http://www.SWIG.org/Doc3.0/SWIGPlus.html#SWIGPlus) chapter. Some more complicated examples will appear later.

> 显然，模板包装的实际内容比本示例中所示的更多。可以在 [SWIG 和 C++](http://www.SWIG.org/Doc3.0/SWIGPlus.html#SWIGPlus) 一章中找到更多详细信息。一些更复杂的示例将在以后出现。

### 36.3.14 C++ 智能指针

#### 36.3.14.1 `shared_ptr` 智能指针

The C++11 standard provides `std::shared_ptr` which was derived from the Boost implementation, `boost::shared_ptr`. Both of these are available for Python in the SWIG library and usage is outlined in the [shared_ptr smart pointer](http://www.SWIG.org/Doc3.0/Library.html#Library_std_shared_ptr) library section.

> C++ 11 标准提供了 `std::shared_ptr`，它是从 Boost 实现的 `boost::shared_ptr` 派生的。两者在 SWIG 库中均可用于 Python，[`shared_ptr` 智能指针](http://www.SWIG.org/Doc3.0/Library.html#Library_std_shared_ptr)库部分中概述了它们的用法。

#### 36.3.14.2 泛型智能指针

In certain C++ programs, it is common to use classes that have been wrapped by so-called "smart pointers." Generally, this involves the use of a template class that implements `operator->()` like this:

> 在某些 C++ 程序中，通常使用被所谓的“智能指针”包装的类。通常，这涉及使用实现 `operator->()` 的模板类，如下所示：

```c++
template<class T> class SmartPtr {
  ...
  T *operator->();
  ...
}
```

Then, if you have a class like this,

> 你有这样一个类，

```c++
class Foo {
public:
  int x;
  int bar();
};
```

A smart pointer would be used in C++ as follows:

> C++ 中的智能指针如下这样使用：

```c++
SmartPtr<Foo> p = CreateFoo();   // Created somehow (not shown)
...
p->x = 3;                        // Foo::x
int y = p->bar();                // Foo::bar
```

To wrap this in Python, simply tell SWIG about the `SmartPtr` class and the low-level `Foo` object. Make sure you instantiate `SmartPtr` using `%template` if necessary. For example:

> 要将其包装在 Python 中，只需将其 `SmartPtr` 类和底层 `Foo` 对象告诉 SWIG 。确保必要时使用 `%template` 实例化 `SmartPtr`。例如：

```
%module example
...
%template(SmartPtrFoo) SmartPtr<Foo>;
...
```

Now, in Python, everything should just "work":

> 现在，Python 中一切将如下运行：

```python
>>> p = example.CreateFoo()          # Create a smart-pointer somehow
>>> p.x = 3                          # Foo::x
>>> p.bar()                          # Foo::bar
```

If you ever need to access the underlying pointer returned by `operator->()`itself, simply use the `__deref__()` method. For example:

> 如果你需要自行访问 `operator->()` 返回的底层指针，只需使用 `__deref__()` 方法。例如：

```python
>>> f = p.__deref__()     # Returns underlying Foo *
```

### 36.3.15 C++ 引用计数对象

The [C++ reference counted objects](http://www.SWIG.org/Doc3.0/SWIGPlus.html#SWIGPlus_ref_unref) section contains Python examples of memory management using referencing counting.

> [C++ 引用计数对象](http://www.SWIG.org/Doc3.0/SWIGPlus.html#SWIGPlus_ref_unref)包含使用引用计数进行内存管理的 Python 示例。

## 36.4 Python 类接口的更多细节

In the previous section, a high-level view of Python wrapping was presented. A key component of this wrapping is that structures and classes are wrapped by Python proxy classes. This provides a very natural Python interface and allows SWIG to support a number of advanced features such as operator overloading. However, a number of low-level details were omitted. This section provides a brief overview of how the proxy classes work.

**New in SWIG version 2.0.4:** The use of Python proxy classes has performance implications that may be unacceptable for a high-performance library. The new `-builtin` option instructs SWIG to forego the use of proxy classes, and instead create wrapped types as new built-in Python types. When this option is used, the following section ("Proxy classes") does not apply. Details on the use of the `-builtin` option are in the [Built-in Types](http://www.SWIG.org/Doc3.0/Python.html#Python_builtin_types) section.

> 在上一节中，提供了 Python 包装的高级视图。这种包装的关键部分是结构体和类由 Python 代理类包装。这提供了非常自然的 Python 接口，并允许 SWIG 支持许多高级功能，例如运算符重载。但是，省略了许多底层细节。本节简要概述了代理类的工作方式。
>
> **SWIG 版本 2.0.4 中的新增功能**：使用 Python 代理类可能会对性能产生影响，这对于高性能库而言可能是无法接受的。新的 `-builtin` 选项指示 SWIG 放弃使用代理类，而是将包装的类型创建为新的内置 Python 类型。使用此选项时，以下部分（“代理类”）不适用。有关使用 `-builtin` 选项的详细信息，请参见[内置类型](http://www.SWIG.org/Doc3.0/Python.html#Python_builtin_types)部分。

### 36.4.1 代理类

In the ["SWIG basics"](http://www.SWIG.org/Doc3.0/SWIG.html#SWIG) and [" SWIG and C++"](http://www.SWIG.org/Doc3.0/SWIGPlus.html#SWIGPlus) chapters, details of low-level structure and class wrapping are described. To summarize those chapters, if you have a class like this

> 在[“SWIG 基础知识”](http://www.SWIG.org/Doc3.0/SWIG.html#SWIG)和 [“SWIG 和 C++”](http://www.SWIG.org/Doc3.0/SWIGPlus.html#SWIGPlus)一章，介绍了底层结构体和类包装的详细信息。总结这些章节，如果你有这样的类

```c++
class Foo {
public:
    int x;
    int spam(int);
    ...
```

then SWIG transforms it into a set of low-level procedural wrappers. For example:

> 然后，SWIG 将其转换为一组低级程序包装器。例如：

```c++
Foo *new_Foo() {
    return new Foo();
}
void delete_Foo(Foo *f) {
    delete f;
}
int Foo_x_get(Foo *f) {
    return f->x;
}
void Foo_x_set(Foo *f, int value) {
    f->x = value;
}
int Foo_spam(Foo *f, int arg1) {
    return f->spam(arg1);
}
```

These wrappers can be found in the low-level extension module (e.g., `_example`).

Using these wrappers, SWIG generates a high-level Python proxy class (also known as a shadow class) like this (shown for Python 2.2):

> 这些包装器可以在低级扩展模块（例如 `_example`）中找到。
>
> 使用这些包装器，SWIG 生成了这样的高级 Python 代理类（也称为影子类）（针对 Python 2.2 显示）：

```python
import _example

class Foo(object):
    def__init__(self):
        self.this = _example.new_Foo()
        self.thisown = 1
    def__del__(self):
        if self.thisown:
            _example.delete_Foo(self.this)
    def spam(self, arg1):
        return _example.Foo_spam(self.this, arg1)
    x = property(_example.Foo_x_get, _example.Foo_x_set)
```

This class merely holds a pointer to the underlying C++ object (`.this`) and dispatches methods and member variable access to that object using the low-level accessor functions. From a user's point of view, it makes the class work normally:

> 此类仅持有指向底层 C++ 对象（`.this`）的指针，并使用低级访问器函数将方法和成员变量访问权分配给该对象。从用户的角度来看，它使类正常工作：

```python
>>> f = example.Foo()
>>> f.x = 3
>>> y = f.spam(5)
```

The fact that the class has been wrapped by a real Python class offers certain advantages. For instance, you can attach new Python methods to the class and you can even inherit from it (something not supported by Python built-in types until Python 2.2).

> 该类已经被真正的 Python 类包装了，这一事实提供了某些优势。例如，你可以将新的 Python 方法附加到该类，甚至可以从该类继承（Python 内置类型直到 Python 2.2 才支持该功能）。

### 36.4.2 内置类型

The `-builtin` option provides a significant performance improvement in the wrapped code. To understand the difference between proxy classes and built-in types, let's take a look at what a wrapped object looks like under both circumstances.

When proxy classes are used, each wrapped object in python is an instance of a pure python class. As a reminder, here is what the `__init__` method looks like in a proxy class:

> `-builtin` 选项在包装的代码中显着提高了性能。为了了解代理类和内置类型之间的区别，让我们看一下两种情况下包装对象的外观。
>
> 使用代理类时，Python 中的每个包装对象都是纯 python 类的实例。提醒一下，这是 `__init__` 方法在代理类中的样子：

```python
class Foo(object):
    def__init__(self):
        self.this = _example.new_Foo()
        self.thisown = 1
```

When a `Foo` instance is created, the call to `_example.new_Foo()` creates a new C++ `Foo` instance; wraps that C++ instance inside an instance of a python built-in type called `SWIG PyObject`; and stores the `SWIG PyObject` instance in the 'this' field of the python Foo object. Did you get all that? So, the python `Foo`object is composed of three parts:
- The python `Foo` instance, which contains...
- ... an instance of `struct SWIG PyObject`, which contains...
- ... a C++ `Foo` instance

When `-builtin` is used, the pure python layer is stripped off. Each wrapped class is turned into a new python built-in type which inherits from` SWIG PyObject`, and `SWIG PyObject` instances are returned directly from the wrapped methods. For more information about python built-in extensions, please refer to the python documentation: <http://docs.python.org/extending/newtypes.html>

> 创建 `Foo` 实例后，对 `_example.new_Foo()` 的调用将创建一个新的 C++ `Foo` 实例；将 C++ 实例包装在名为 `SWIG PyObject` 的 Python 内置类型的实例中；并将 `SWIG PyObject` 实例存储在 python `Foo` 对象的 `this` 字段中。你明白了吗？ 因此，Python 的 `Foo` 对象由三部分组成：
> - python `Foo` 实例，其中包含...
> - ...结构体 `SWIG PyObject` 的实例，其中包含...
> - ...一个 C++ `Foo` 实例
>
> 当使用 `-builtin` 时，纯 python 层被剥离。每个包装的类都转换为继承自 `SWIG PyObject` 的新的 Python 内置类型，并且 `SWIG PyObject` 实例直接从包装的方法中返回。有关 Python 内置扩展的更多信息，请参考 python 文档：<http://docs.python.org/extending/newtypes.html>

#### 36.4.2.1 限制

Use of the `-builtin` option implies a couple of limitations:
- python version support:
  - Versions 2.5 and up are fully supported
  - Versions 2.3 and 2.4 are mostly supported; there are problems with director classes and/or sub-classing a wrapped type in python.
  - Versions older than 2.3 are not supported.
- Some legacy syntax is no longer supported; in particular:
  - The functional interface is no longer exposed. For example, you may no longer call `Whizzo.new_CrunchyFrog()`. Instead, you must use `Whizzo.CrunchyFrog()`.
  - Static member variables are no longer accessed through the 'cvar' field (e.g., `Dances.cvar.FishSlap`). They are instead accessed in the idiomatic way (`Dances.FishSlap`).
- Wrapped types may not be raised as python exceptions. Here's why: the python internals expect that all sub-classes of Exception will have this struct layout:

> 使用 `-builtin` 选项意味着两个限制：
> - python 版本支持：
>   - 完全支持 2.5 版及更高版本
>   - 主要支持 2.3 和 2.4版本；Director 类和/或在 Python 中包装类型的子类存在问题。
>   - 不支持 2.3 之前的版本。
> - 不再支持某些旧式语法；尤其是：
>   - 函数接口不再暴露。例如，你可能不再调用 `Whizzo.new_CrunchyFrog()`。相反，你必须使用 `Whizzo.CrunchyFrog()`。
>   - 静态成员变量不再通过 `cvar` 字段访问（例如，`Dances.cvar.FishSlap`）。而是以惯用的方式访问它们（`Dances.FishSlap`）。
> - 包装类型不能作为 python 异常引发。原因如下：python 内部希望 `Exception` 的所有子类都具有以下结构体布局：

```c++
typedef struct {
    PyObject_HEAD
    PyObject *dict;
    PyObject *args;
    PyObject *message;
} PyBaseExceptionObject;
```

But SWIG-generated wrappers expect that all SWIG-wrapped classes will have this struct layout:

> 但是，用 SWIG 生成的包装器希望所有用 SWIG 包裹的类都具有以下结构体布局：

```c++
typedef struct {
    PyObject_HEAD
    void *ptr;
    SWIG_type_info *ty;
    int own;
    PyObject *next;
    PyObject *dict;
} SWIG PyObject;
```

There are workarounds for this. For example, if you wrap this class:

> 有解决方法。例如，如果包装这个类：

```c++
class MyException {
public:
    MyException (const char *msg_);
    ~MyException ();

    const char *what () const;

private:
    char *msg;
};
```

... you can define this python class, which may be raised as an exception:

> ...你可以定义此 python 类，可以引发异常：

```c++
class MyPyException(Exception):
    def__init__(self, msg, *args):
        Exception.__init__(self, *args)
        self.myexc = MyException(msg)
    def what(self):
        return self.myexc.what()
```

- Reverse binary operators (e.g., `__radd__`) are not supported.

To illustrate this point, if you have a wrapped class called `MyString`, and you want to use instances of `MyString` interchangeably with native python strings, you can define an `'operator+ (const char*)'` method :

> - 逆向二元运算符（例如 `__radd__`）不被支持。
>
> 为了说明这一点，如果你有一个名为 `MyString` 的包装类，并且想将 `MyString` 的实例与本地 python 字符串互换使用，则可以定义一个 `operator +(const char *)` 方法：

```c++
class MyString {
public:
    MyString (const char *init);
    MyString operator+ (const char *other) const;
    ...
};
```

SWIG will automatically create an operator overload in python that will allow this:

> SWIG 将自动地在 Python 中创建一个运算符重载，允许这样操作：

```python
from MyModule import MyString

mystr = MyString("No one expects")
episode = mystr + " the Spanish Inquisition"
```

This works because the first operand (`mystr`) defines a way to add a native string to itself. However, the following will **not** work:

> 之所以有效，是因为第一个操作数（`mystr`）定义了一种向其自身添加本地字符串的方法。但是，以下内容将**不**起作用：

```python
from MyModule import MyString

mystr = MyString("Parrot")
episode = "Dead " + mystr
```

The above code fails, because the first operand -- a native python string -- doesn't know how to add an instance of `MyString` to itself.
- If you have multiple SWIG modules that share type information ([more info](http://www.SWIG.org/Doc3.0/Modules.html#Modules_nn2)), the `-builtin` option requires a bit of extra discipline to ensure that base classes are initialized before derived classes. Specifically:
  - There must be an unambiguous dependency graph for the modules.
  - Module dependencies must be explicitly stated with `%import`statements in the SWIG interface file.

As an example, suppose module `A` has this interface in `A.i` :

> 上面的代码失败了，因为第一个操作数，一个本地 python 字符串，不知道如何向自己添加 `MyString` 的实例。
> - 如果你有多个共享类型信息的 SWIG 模块（[更多信息](http://www.SWIG.org/Doc3.0/Modules.html#Modules_nn2)），则 `-builtin` 选项需要一些额外的规则，以确保在派生类之前初始化基类。特别：
>  - 模块必须有明确的依赖关系图。
>  - 必须在 SWIG 接口文件中使用 `%import` 声明明确声明模块依赖性。
>
> 例如，假设模块 `A` 在 `A.i` 中具有此接口：

```
%module "A";

class Base {
...
};
```

If you want to wrap another module containing a class that inherits from `A`, this is how it would look :

> 如果要包装另一个包含从 `A` 继承的类的模块，它将是这样：

```
%module "B";

%import "A.i"

class Derived : public Base {
...
};
```

The `import "A.i"` statement is required, because module `B` depends on module `A`.

As long as you obey these requirements, your python code may import the modules in any order :

> 由于模块 `B` 依赖于模块 `A`，因此必须使用 `import A.i` 声明。
>
> 只要你遵守这些要求，你的 Python 代码就可以按任何顺序导入模块：

```python
import B
import A

assert(issubclass(B.Derived, A.Base))
```

#### 36.4.2.2 运算符重载与 slot——值得一用！

The entire justification for the `-builtin` option is improved performance. To that end, the best way to squeeze maximum performance out of your wrappers is to **use operator overloads.** Named method dispatch is slow in python, even when compared to other scripting languages. However, python built-in types have a large number of "slots", analogous to C++ operator overloads, which allow you to short-circuit named method dispatch for certain common operations.

By default, SWIG will translate most C++ arithmetic operator overloads into python slot entries. For example, suppose you have this class:

> 使用 `-builtin` 选项的全部理由是为了提高性能。为此，从包装程序中获取最大性能的最佳方法是**使用运算符重载**。即使与其他脚本语言相比，在 Python 中命名方法的分发也很慢。但是，python 内置类型具有大量的“slot”，类似于 C++ 运算符重载，这使你可以为某些常用操作短路命名方法分派。
>
> 默认情况下，SWIG 会将大多数 C++ 算术运算符重载转换为 python slot。例如，假设你有此类：

```c++
class Twit {
public:
    Twit operator+ (const Twit& twit) const;

    // Forward to operator+
    Twit add (const Twit& twit) const
    { return *this + twit; }
};
```

SWIG will automatically register `operator+` as a python slot operator for addition. You may write python code like this:

> SWIG 将自动将 `operator+` 注册为 python slot 运算符以进行添加。你可以这样编写 Python 代码：

```python
from MyModule import Twit

nigel = Twit()
emily = Twit()
percival = nigel + emily
percival = nigel.add(emily)
```

The last two lines of the python code are equivalent, but **the line that uses the '+' operator is much faster**.

In-place operators (e.g., `operator+=`) and comparison operators (`operator==, operator<`, etc.) are also converted to python slot operators. For a complete list of C++ operators that are automatically converted to python slot operators, refer to the file `python/pyopers. SWIG ` in the SWIG library.

Read about all of the available python slots here: http://docs.python.org/c-api/typeobj.html

There are two ways to define a python slot function: dispatch to a statically defined function; or dispatch to a method defined on the operand.

To dispatch to a statically defined function, use `%feature("python:<slot>")`, where `<slot>` is the name of a field in a `PyTypeObject, PyNumberMethods, PyMappingMethods, PySequenceMethods` or `PyBufferProcs`. You may override (almost) all of these slots.

Let's consider an example setting the `tp_hash` slot for the `MyClass` type. This is akin to providing a `__hash__` method (for non-builtin types) to make a type hashable. The hashable type can then for example be added to a Python `dict`.

> Python 代码的最后两行是等效的，但是**使用 `+` 运算符的行要快得多**。
>
> 本地运算符（例如 `operator +=`）和比较运算符（`operator ==`、`operator <` 等）也将转换为 python slot 运算符。有关自动转换为 python slot 运算符的 C++ 运算符的完整列表，请参见 SWIG 库中的文件 `python/pyopers.SWIG`。
>
> 在此处阅读有关所有可用 Python slot 的信息：<http://docs.python.org/c-api/typeobj.html>
>
> 定义 python slot 函数有两种方法：分配给静态定义的函数；或调度到在操作数上定义的方法。
>
> 要分派到静态定义的函数，请使用 `%feature("python:<slot>")`，其中 `<slot>` 是 `PyTypeObject`，`PyNumberMethods`，`PyMappingMethods`，`PySequenceMethods` 或 `PyBufferProcs` 中的字段名称。你可以覆盖（几乎）所有这些 slot。
>
> 让我们考虑一个为 `MyClass` 类型设置 `tp_hash` slot 的示例。这类似于提供 `__hash__` 方法（用于非内置类型）以使类型可哈希化。然后可以将可哈希类型添加到 Python 的 `dict` 中。

```
%feature("python:tp_hash") MyClass "myHashFunc";

class MyClass {
public:
  long field1;
  long field2;
  ...
};

%{
#if PY_VERSION_HEX >= 0x03020000
  static Py_hash_t myHashFunc(PyObject *pyobj)
#else
  static long myHashFunc(PyObject *pyobj)
#endif
  {
    MyClass *cobj;
    // Convert pyobj to cobj
    return (cobj->field1 * (cobj->field2 << 7));
  }
%}
```

If you examine the generated code, the supplied hash function will now be the function callback in the tp_hash slot for the builtin type for `MyClass`:

> 如果检查生成的代码，现在提供的哈希函数将成为 `tp_hash` slot 中 `MyClass` 内置类型的函数回调：

```c++
static PyHeapTypeObject SWIG PyBuiltin__MyClass_type = {
    ...
    (hashfunc) myHashFunc,       /* tp_hash */
    ...
```

NOTE: It is the responsibility of the programmer (that's you!) to ensure that a statically defined slot function has the correct signature, the `hashfunc` typedef in this case.

If, instead, you want to dispatch to an instance method, you can use %feature("python:slot"). For example:

> 注意：程序员（就是你！）的责任是确保静态定义的 slot 函数具有正确的签名，在这种情况下为 `hashfunc`。
>
> 相反，如果要分派到实例方法，则可以使用 `%feature(python:slot)`。例如：

```
%feature("python:slot", "tp_hash", functype="hashfunc") MyClass::myHashFunc;

#if PY_VERSION_HEX < 0x03020000
  #define Py_hash_t long
#endif

class MyClass {
  public:
    Py_hash_t myHashFunc() const;
    ...
};
```

NOTE: Some python slots use a method signature which does not match the signature of SWIG-wrapped methods. For those slots, SWIG will automatically generate a "closure" function to re-marshal the arguments before dispatching to the wrapped method. Setting the "functype" attribute of the feature enables SWIG to generate the chosen closure function.

There is further information on `%feature("python:slot")` in the file `python/pyopers. SWIG ` in the SWIG library.

> 注意：某些 python slot 使用的方法签名与 SWIG 封装的方法的签名不匹配。对于这些 slot，SWIG 将自动生成一个“闭包”函数，以在将其分派到包装的方法之前重新编组参数。设置功能的 `functype` 属性可使 SWIG 生成所选的关闭函数。
>
> SWIG 库中的文件 `python/pyopers.SWIG` 中有关于 `%feature(python:slot)` 的更多信息。

### 36.4.3 内存管理

NOTE: Although this section refers to proxy objects, everything here also applies when the `-builtin` option is used.

Associated with proxy object, is an ownership flag `.thisown` The value of this flag determines who is responsible for deleting the underlying C++ object. If set to 1, the Python interpreter will destroy the C++ object when the proxy class is garbage collected. If set to 0 (or if the attribute is missing), then the destruction of the proxy class has no effect on the C++ object.

When an object is created by a constructor or returned by value, Python automatically takes ownership of the result. For example:

> 注意：尽管本节涉及代理对象，但是当使用 `-builtin` 选项时，此处的所有内容也适用。
>
> 所有权标志 `.thisown` 与代理对象相关联，该标志的值确定谁负责删除基础 C++ 对象。如果设置为 1，则在垃圾回收代理类时，Python 解释器将销毁 C++ 对象。如果设置为 0（或缺少属性），则代理类的销毁对 C++ 对象无效。
>
> 当对象由构造函数创建或由值返回时，Python 自动获取结果的所有权。例如：

```c++
class Foo {
public:
    Foo();
    Foo bar();
};
```

In Python:

> 在 Python 中：

```python
>>> f = Foo()
>>> f.thisown
1
>>> g = f.bar()
>>> g.thisown
1
```

On the other hand, when pointers are returned to Python, there is often no way to know where they came from. Therefore, the ownership is set to zero. For example:

> 另一方面，当指针返回到 Python 时，通常没有办法知道它们来自何处。因此，所有权设置为零。例如：

```c++
class Foo {
public:
    ...
    Foo *spam();
    ...
};
```

```python
>>> f = Foo()
>>> s = f.spam()
>>> print s.thisown
0
>>>
```

This behavior is especially important for classes that act as containers. For example, if a method returns a pointer to an object that is contained inside another object, you definitely don't want Python to assume ownership and destroy it!

A good way to indicate that ownership should be set for a returned pointer is to use the [%newobject directive](http://www.SWIG.org/Doc3.0/Library.html#Library_nn11).

Related to containers, ownership issues can arise whenever an object is assigned to a member or global variable. For example, consider this interface:

> 对于充当容器的类，此行为尤其重要。例如，如果方法返回指向另一个对象内包含的对象的指针，则你绝对不希望 Python 承担所有权并销毁它！
>
> 指示应为返回的指针设置所有权的一种好方法是使用 [`%newobject` 指令](http://www.SWIG.org/Doc3.0/Library.html#Library_nn11)。
>
> 与容器相关的是，只要将对象分配给成员或全局变量，就会出现所有权问题。例如，考虑以下接口：

```
%module example

struct Foo {
    int value;
    Foo *next;
};

Foo *head = 0;
```

When wrapped in Python, careful observation will reveal that ownership changes whenever an object is assigned to a global variable. For example:

> 当用 Python 封装时，仔细观察会发现，只要将对象分配给全局变量，所有权就会发生变化。例如：

```python
>>> f = example.Foo()
>>> f.thisown
1
>>> example.cvar.head = f
>>> f.thisown
0
>>>
```

In this case, C is now holding a reference to the object---you probably don't want Python to destroy it. Similarly, this occurs for members. For example:

> 在这种情况下，C 现在持有对该对象的引用，你可能不希望 Python 销毁它。同样，这对于成员也是如此。例如：

```python
>>> f = example.Foo()
>>> g = example.Foo()
>>> f.thisown
1
>>> g.thisown
1
>>> f.next = g
>>> g.thisown
0
>>>
```

For the most part, memory management issues remain hidden. However, there are occasionally situations where you might have to manually change the ownership of an object. For instance, consider code like this:

> 在大多数情况下，内存管理问题仍然隐藏。但是，在某些情况下，你可能必须手动更改对象的所有权。例如，考虑如下代码：

```c++
class Node {
  Object *value;
public:
  void set_value(Object *v) { value = v; }
  ...
};
```

Now, consider the following Python code:

> 现在考虑下面的 Python 代码：

```python
>>> v = Object()           # Create an object
>>> n = Node()             # Create a node
>>> n.set_value(v)         # Set value
>>> v.thisown
1
>>> del v
```

In this case, the object `n` is holding a reference to `v` internally. However, SWIG has no way to know that this has occurred. Therefore, Python still thinks that it has ownership of the object. Should the proxy object be destroyed, then the C++ destructor will be invoked and `n` will be holding a stale-pointer. If you're lucky, you will only get a segmentation fault.

To work around this, it is always possible to flip the ownership flag. For example,

> 在这种情况下，对象 `n` 在内部持有对 `v` 的引用。但是，SWIG 无法知道发生了这种情况。因此，Python 仍然认为它拥有对象的所有权。如果代理对象被销毁，则将调用 C++ 析构函数，并且 `n` 将持有旧指针。如果幸运的话，你只会遇到分段错误。
>
> 要解决此问题，始终可以翻转所有权标志。例如，

```python
>>> v.thisown = 0
```

It is also possible to deal with situations like this using typemaps--an advanced topic discussed later.

> 还可以使用类型映射处理这种情况，这是稍后讨论的高级主题。

### 36.4.4 Python 2.2 与经典类

SWIG makes every attempt to preserve backwards compatibility with older versions of Python to the extent that it is possible. However, in Python-2.2, an entirely new type of class system was introduced. This new-style class system offers many enhancements including static member functions, properties (managed attributes), and class methods. Details about all of these changes can be found on [www.python.org](https://www.python.org/) and is not repeated here.

To address differences between Python versions, SWIG currently emits dual-mode proxy class wrappers. In Python-2.2 and newer releases, these wrappers encapsulate C++ objects in new-style classes that take advantage of new features (static methods and properties). However, if these very same wrappers are imported into an older version of Python, old-style classes are used instead.

This dual-nature of the wrapper code means that you can create extension modules with SWIG and those modules will work with all versions of Python ranging from Python-2.0 to the very latest release. Moreover, the wrappers take advantage of Python-2.2 features when available.

For the most part, the interface presented to users is the same regardless of what version of Python is used. The only incompatibility lies in the handling of static member functions. In Python-2.2, they can be accessed via the class itself. In Python-2.1 and earlier, they have to be accessed as a global function or through an instance (see the earlier section).

> SWIG 尽一切可能保持与旧版本 Python 的向后兼容性。但是，在 Python-2.2 中，引入了一种全新的类系统。这种新型的类系统提供了许多增强功能，包括静态成员函数、属性（托管属性）和类方法。有关所有这些更改的详细信息可以在 [www.python.org](https://www.python.org/) 上找到，在此不再赘述。
>
> 为了解决 Python 版本之间的差异，SWIG 当前发出双模式代理类包装器。在 Python-2.2 和更高版本中，这些包装器将 C++ 对象封装在利用新功能（静态方法和属性）的新型类中。但是，如果将这些完全相同的包装器导入到旧版本的 Python 中，则会使用旧式类。
>
> 包装代码的这种双重特性意味着你可以使用 SWIG 创建扩展模块，并且这些模块将与从 Python-2.0 到最新版本的所有 Python 版本一起使用。而且，包装器在可用时会利用 Python-2.2 功能。
>
> 在大多数情况下，无论使用什么版本的 Python，呈现给用户的接口都是相同的。唯一的不兼容性在于静态成员函数的处理。在 Python-2.2 中，可以通过类本身访问它们。在 Python-2.1 和更早版本中，必须将它们作为全局函数或通过实例进行访问（请参见前面的部分）。

## 36.5 跨语言多态

Proxy classes provide a more natural, object-oriented way to access extension classes. As described above, each proxy instance has an associated C++ instance, and method calls to the proxy are passed to the C++ instance transparently via C wrapper functions.

This arrangement is asymmetric in the sense that no corresponding mechanism exists to pass method calls down the inheritance chain from C++ to Python. In particular, if a C++ class has been extended in Python (by extending the proxy class), these extensions will not be visible from C++ code. Virtual method calls from C++ are thus not able access the lowest implementation in the inheritance chain.

Changes have been made to SWIG 1.3.18 to address this problem and make the relationship between C++ classes and proxy classes more symmetric. To achieve this goal, new classes called directors are introduced at the bottom of the C++ inheritance chain. The job of the directors is to route method calls correctly, either to C++ implementations higher in the inheritance chain or to Python implementations lower in the inheritance chain. The upshot is that C++ classes can be extended in Python and from C++ these extensions look exactly like native C++ classes. Neither C++ code nor Python code needs to know where a particular method is implemented: the combination of proxy classes, director classes, and C wrapper functions takes care of all the cross-language method routing transparently.

> 代理类提供了一种更自然、更面向对象的方法来访问扩展类。如上所述，每个代理实例都有一个关联的 C++ 实例，并且对代理的方法调用通过 C 包装函数透明地传递给 C++ 实例。
>
> 这种安排是不对称的，因为不存在从 C++ 到 Python 的继承链上传递方法调用的相应机制。特别是，如果已经在 Python 中扩展了 C++ 类（通过扩展代理类），则这些扩展将从 C++ 代码中不可见。因此，来自 C++ 的虚方法调用无法访问继承链中最低的实现。
>
> 已对 SWIG 1.3.18 进行了更改，以解决此问题并使 C++ 类和代理类之间的关系更加对称。为了实现此目标，在 C++ 继承链的底部引入了称为 Director 的新类。Director 的工作是将方法调用正确地路由到继承链中较高的 C++ 实现或继承链中较低的 Python 实现。结果是可以在 Python 中扩展 C++ 类，而从 C++ 中扩展这些扩展看起来与本地 C++ 类完全一样。C++ 代码和 Python 代码都不需要知道在哪里实现特定方法：代理类、Director 类和 C 包装函数的组合可透明地处理所有跨语言方法路由。

### 36.5.1 开启 Director

The director feature is disabled by default. To use directors you must make two changes to the interface file. First, add the "directors" option to the %module directive, like this:

> Director 功能默认情况下处于禁用状态。要使用 Director，必须对接口文件进行两项更改。首先，将 `directors` 选项添加到 `%module` 指令，如下所示：

```
%module(directors="1") modulename
```

Without this option no director code will be generated. Second, you must use the %feature("director") directive to tell SWIG which classes and methods should get directors. The %feature directive can be applied globally, to specific classes, and to specific methods, like this:

> 如果没有此选项，将不会生成 Director 代码。其次，你必须使用 `%feature("director")` 指令来告诉 SWIG 哪些类和方法应该获得 Director 。`%feature` 指令可以全局应用于特定的类和特定的方法，如下所示：

```
// generate directors for all classes that have virtual methods
%feature("director");

// generate directors for all virtual methods in class Foo
%feature("director") Foo;
```

You can use the %feature("nodirector") directive to turn off directors for specific classes or methods. So for example,

> 你可以使用 `%feature("nodirector")` 指令关闭特定类或方法的控制器。例如

```
%feature("director") Foo;
%feature("nodirector") Foo::bar;
```

will generate directors for all virtual methods of class Foo except bar().

Directors can also be generated implicitly through inheritance. In the following, class Bar will get a director class that handles the methods one() and two() (but not three()):

> 将为 `Foo` 类的所有虚方法（除 `bar()` 之外）生成 Director 。
>
> Director 也可以通过继承隐式生成。在下面的代码中，`Bar` 类将获得一个处理方法 `one()` 和 `two()`（但不是 `three()`）的 Director 类：

```
%feature("director") Foo;
class Foo {
public:
    Foo(int foo);
    virtual ~Foo();
    virtual void one();
    virtual void two();
};

class Bar: public Foo {
public:
    virtual void three();
};
```

then at the python side you can define

> 在 python 端你可以定义

```python
import mymodule

class MyFoo(mymodule.Foo):
    def __init__(self, foo):
        mymodule.Foo.__init__(self, foo)
#       super().__init__(foo) # Alternative construction for Python3

    def one(self):
        print "one from python"
```

### 36.5.2 Director 类

For each class that has directors enabled, SWIG generates a new class that derives from both the class in question and a special `SWIG::Director` class. These new classes, referred to as director classes, can be loosely thought of as the C++ equivalent of the Python proxy classes. The director classes store a pointer to their underlying Python object and handle various issues related to object ownership. Indeed, this is quite similar to the "this" and "thisown" members of the Python proxy classes.

For simplicity let's ignore the `SWIG::Director` class and refer to the original C++ class as the director's base class. By default, a director class extends all virtual methods in the inheritance chain of its base class (see the preceding section for how to modify this behavior). Thus all virtual method calls, whether they originate in C++ or in Python via proxy classes, eventually end up in at the implementation in the director class. The job of the director methods is to route these method calls to the appropriate place in the inheritance chain. By "appropriate place" we mean the method that would have been called if the C++ base class and its extensions in Python were seamlessly integrated. That seamless integration is exactly what the director classes provide, transparently skipping over all the messy extension API glue that binds the two languages together.

In reality, the "appropriate place" is one of only two possibilities: C++ or Python. Once this decision is made, the rest is fairly easy. If the correct implementation is in C++, then the lowest implementation of the method in the C++ inheritance chain is called explicitly. If the correct implementation is in Python, the Python API is used to call the method of the underlying Python object (after which the usual virtual method resolution in Python automatically finds the right implementation).

Now how does the director decide which language should handle the method call? The basic rule is to handle the method in Python, unless there's a good reason not to. The reason for this is simple: Python has the most "extended" implementation of the method. This assertion is guaranteed, since at a minimum the Python proxy class implements the method. If the method in question has been extended by a class derived from the proxy class, that extended implementation will execute exactly as it should. If not, the proxy class will route the method call into a C wrapper function, expecting that the method will be resolved in C++. The wrapper will call the virtual method of the C++ instance, and since the director extends this the call will end up right back in the director method. Now comes the "good reason not to" part. If the director method were to blindly call the Python method again, it would get stuck in an infinite loop. We avoid this situation by adding special code to the C wrapper function that tells the director method to not do this. The C wrapper function compares the pointer to the Python object that called the wrapper function to the pointer stored by the director. If these are the same, then the C wrapper function tells the director to resolve the method by calling up the C++ inheritance chain, preventing an infinite loop.

One more point needs to be made about the relationship between director classes and proxy classes. When a proxy class instance is created in Python, SWIG creates an instance of the original C++ class and assigns it to `.this`. This is exactly what happens without directors and is true even if directors are enabled for the particular class in question. When a class *derived* from a proxy class is created, however, SWIG then creates an instance of the corresponding C++ director class. The reason for this difference is that user-defined subclasses may override or extend methods of the original class, so the director class is needed to route calls to these methods correctly. For unmodified proxy classes, all methods are ultimately implemented in C++ so there is no need for the extra overhead involved with routing the calls through Python.

> 对于每个启用了 Director 的类，SWIG 都会生成一个新类，该类从相关类和特殊的 `SWIG::Director` 类中派生。可以将这些新类（称为 Director 类）粗略地视为 Python 代理类的 C++ 等效类。Director 类存储指向其基础 Python 对象的指针，并处理与对象所有权有关的各种问题。实际上，这与 Python 代理类的 `this` 和 `thisown` 成员非常相似。
>
> 为了简单起见，让我们忽略 `SWIG::Director` 类，并将原始 C++ 类称为 Director 的基类。默认情况下，director 类在其基类的继承链中扩展了所有虚方法（有关如何修改此行为的信息，请参见上一节）。因此，所有虚方法调用，无论它们是源自 C++ 还是源自 Python，都是通过代理类进行的，最终都以 Director 类中的实现结束。Director 方法的工作是将这些方法调用路由到继承链中的适当位置。“适当的位置”是指如果 C++ 基类及其在 Python 中的扩展被无缝集成的话，该方法将被调用。这种无缝集成正是 Director 类所提供的，透明地跳过了将两种语言绑定在一起的所有混乱扩展 API 胶水。
>
> 实际上，“适当的位置”是仅有的两种可能性之一： C++ 或 Python。一旦做出决定，剩下的就很容易了。如果正确的实现在 C++ 中，则显式调用 C++ 继承链中该方法的最低实现。如果正确的实现是在 Python 中执行的，则使用 Python API 调用基础 Python 对象的方法（此后，Python 中通常的虚方法解析会自动找到正确的实现）。
>
> 现在，Director 如何决定应使用哪种语言处理方法调用？基本规则是在 Python 中处理该方法，除非有充分的理由不这样做。原因很简单：Python 具有该方法最“扩展”的实现。该断言得到保证，因为至少 Python 代理类实现了该方法。如果所讨论的方法已由派生自代理类的类扩展，则该扩展实现将完全按照应有的方式执行。如果不是，则代理类会将方法调用路由到 C 包装函数中，期望该方法将在 C++ 中解析。包装器将调用 C++ 实例的虚方法，并且由于 Director 对其进行了扩展，因此该调用将最终返回到 Director 方法中。现在出现了“拒绝的充分理由”部分。如果 director 方法再次盲目调用 Python 方法，它将陷入无限循环。我们通过在 C 包装函数中添加特殊代码来避免这种情况，该函数告诉 Director 方法不要执行此操作。C 包装函数会将调用包装函数的 Python 对象的指针与控制器存储的指针进行比较。如果这些相同，则C 包装函数将通知主管通过调用 C++ 继承链来解决方法，从而防止无限循环。
>
> 关于 Director 类和代理类之间的关系，还需要指出一点。在 Python 中创建代理类实例时，SWIG 将创建原始 C++ 类的实例并将其分配给 `.this`。这就是没有 Director 的情况，即使针对特定类别启用了 Director，也是如此。但是，当从代理类*派生*一个类时，SWIG 随后将创建相应的 C++ Director 类的实例。造成这种差异的原因是，用户定义的子类可能会覆盖或扩展原始类的方法，因此需要 Director 类将调用正确路由到这些方法。对于未修改的代理类，所有方法最终都用 C++ 实现，因此不需要通过 Python 路由调用而涉及额外的开销。

### 36.5.3 所有权与对象销毁

Memory management issues are slightly more complicated with directors than for proxy classes alone. Python instances hold a pointer to the associated C++ director object, and the director in turn holds a pointer back to the Python object. By default, proxy classes own their C++ director object and take care of deleting it when they are garbage collected.

This relationship can be reversed by calling the special `__disown__()` method of the proxy class. After calling this method, the `.thisown` flag is set to zero, and the director class increments the reference count of the Python object. When the director class is deleted it decrements the reference count. Assuming no outstanding references to the Python object remain, the Python object will be destroyed at the same time. This is a good thing, since directors and proxies refer to each other and so must be created and destroyed together. Destroying one without destroying the other will likely cause your program to segfault.

To help ensure that no references to the Python object remain after calling `__disown__()`, this method returns a weak reference to the Python object. Weak references are only available in Python versions 2.1 and higher, so for older versions you must explicitly delete all references. Here is an example:

> Director 的内存管理问题比仅代理类要复杂得多。Python 实例包含指向关联的 C++  Director 对象的指针，而 Director 又持有指向 Python 对象的指针。默认情况下，代理类拥有其 C++  Director 对象，并在垃圾回收时将其删除。
>
> 可以通过调用代理类的特殊 `__disown__()` 方法来逆转这种关系。调用此方法后，`.thisown` 标志设置为零，并且 Director 类增加 Python 对象的引用计数。删除 Director 类后，它会减少引用计数。假设没有剩余的对 Python 对象的引用，则 Python 对象将同时被销毁。这是一件好事，因为 Director 和代理人相互参照，因此必须一起创建和销毁。销毁一个而不破坏另一个可能会导致你的程序出现段错误。
>
> 为了帮助确保在调用 `__disown__()` 之后没有对 Python 对象的引用，此方法返回对 Python 对象的弱引用。弱引用仅在 Python 2.1 及更高版本中可用，因此对于较旧的版本，你必须明确删除所有引用。这是一个例子：

```c++
class Foo {
public:
    ...
};
class FooContainer {
public:
    void addFoo(Foo *);
    ...
};
```

```python
>>> c = FooContainer()
>>> a = Foo().__disown__()
>>> c.addFoo(a)
>>> b = Foo()
>>> b = b.__disown__()
>>> c.addFoo(b)
>>> c.addFoo(Foo().__disown__())
```

In this example, we are assuming that FooContainer will take care of deleting all the Foo pointers it contains at some point. Note that no hard references to the Foo objects remain in Python.

> 在此示例中，我们假设 `FooContainer` 将在某个时候删除其包含的所有 `Foo` 指针。请注意，Python 中没有对 `Foo` 对象的硬引用。

### 36.5.4 异常回滚

With directors routing method calls to Python, and proxies routing them to C++, the handling of exceptions is an important concern. By default, the directors ignore exceptions that occur during method calls that are resolved in Python. To handle such exceptions correctly, it is necessary to temporarily translate them into C++ exceptions. This can be done with the `%feature("director:except")` directive. The following code should suffice in most cases:

> Director 将方法调用路由到 Python，并将代理路由到 C++，异常处理是一个重要的问题。默认情况下，控制器将忽略在 Python 中解析的方法调用期间发生的异常。为了正确处理此类异常，有必要将它们临时转换为 C++ 异常。这可以通过 `%feature("director:except")` 指令完成。在大多数情况下，以下代码就足够了：

```
%feature("director:except") {
    if ($error != NULL) {
        throw SWIG::DirectorMethodException();
    }
}
```

This code will check the Python error state after each method call from a director into Python, and throw a C++ exception if an error occurred. This exception can be caught in C++ to implement an error handler. Currently no information about the Python error is stored in the SWIG::DirectorMethodException object, but this will likely change in the future.

It may be the case that a method call originates in Python, travels up to C++ through a proxy class, and then back into Python via a director method. If an exception occurs in Python at this point, it would be nice for that exception to find its way back to the original caller. This can be done by combining a normal %exception directive with the `director:except` handler shown above. Here is an example of a suitable exception handler:

> 每次从 Director 调用 Python 的方法后，此代码将检查 Python 错误状态，如果发生错误，则抛出 C++ 异常。可以在 C++ 中捕获此异常以实现错误处理程序。当前在 `SWIG::DirectorMethodException` 对象中没有存储有关 Python 错误的信息，但是将来可能会改变。
>
> 方法调用可能起源于 Python，通过代理类升至 C++，然后通过 Director 方法返回 Python。如果此时在 Python 中发生异常，那么该异常可以找到返回原始调用方的方式会很不错。这可以通过将普通的 `%exception` 指令与上面显示的 `director：except` 处理程序结合使用来完成。这是合适的异常处理程序的示例：

```
%exception {
    try { $action }
    catch ( SWIG::DirectorException &e) { SWIG_fail; }
}
```

The class SWIG::DirectorException used in this example is actually a base class of SWIG::DirectorMethodException, so it will trap this exception. Because the Python error state is still set when SWIG::DirectorMethodException is thrown, Python will register the exception as soon as the C wrapper function returns.

> 本示例中使用的 `SWIG::DirectorException` 类实际上是 `SWIG::DirectorMethodException` 的基类，因此它将捕获此异常。由于抛出 `SWIG::DirectorMethodException` 时仍设置 Python 错误状态，因此 C 包装函数返回后，Python将立即注册异常。

### 36.5.5 开销与代码膨胀

Enabling directors for a class will generate a new director method for every virtual method in the class' inheritance chain. This alone can generate a lot of code bloat for large hierarchies. Method arguments that require complex conversions to and from target language types can result in large director methods. For this reason it is recommended that you selectively enable directors only for specific classes that are likely to be extended in Python and used in C++.

Compared to classes that do not use directors, the call routing in the director methods does add some overhead. In particular, at least one dynamic cast and one extra function call occurs per method call from Python. Relative to the speed of Python execution this is probably completely negligible. For worst case routing, a method call that ultimately resolves in C++ may take one extra detour through Python in order to ensure that the method does not have an extended Python implementation. This could result in a noticeable overhead in some cases.

Although directors make it natural to mix native C++ objects with Python objects (as director objects) via a common base class pointer, one should be aware of the obvious fact that method calls to Python objects will be much slower than calls to C++ objects. This situation can be optimized by selectively enabling director methods (using the %feature directive) for only those methods that are likely to be extended in Python.

> 为类启用控制器将为该类的继承链中的每个虚方法生成一个新的控制器方法。仅此一项就可以为大型层次结构产生大量代码膨胀。需要与目标语言类型进行复杂转换的方法参数可能会导致使用大型 Director 方法。因此，建议仅针对可能在 Python 中扩展并在 C++ 中使用的特定类，选择性地启用 Director。
>
> 与不使用 Director 的类相比， Director 方法中的调用路由确实会增加一些开销。特别是，对于 Python 中的每个方法调用，至少要进行一次动态转换和一个额外的函数调用。相对于 Python 执行速度，这可能是完全可以忽略的。对于最坏情况的路由，最终用 C++ 解析的方法调用可能会通过 Python 进行额外的绕行，以确保该方法没有扩展的 Python 实现。在某些情况下，这可能会导致明显的开销。
>
> 尽管 Director 自然而然地通过一个通用的基类指针将本地 C++ 对象与 Python 对象（作为 Director 对象）混合在一起，但人们应该意识到一个明显的事实，即对 Python 对象的方法调用比对 C++ 对象的调用要慢得多。通过仅针对可能在 Python 中扩展的那些方法有选择地启用 Director 方法（使用 `%feature` 指令），可以优化这种情况。

### 36.5.6 类型映射

Typemaps for input and output of most of the basic types from director classes have been written. These are roughly the reverse of the usual input and output typemaps used by the wrapper code. The typemap operation names are 'directorin', 'directorout', and 'directorargout'. The director code does not currently use any of the other kinds of typemaps. It is not clear at this point which kinds are appropriate and need to be supported.

> 已经编写了 Director 类的大多数基本类型的输入和输出的类型映射。这些大致与包装器代码使用的常规输入和输出类型映射相反。类型映射操作名称为 `directorin`、`directorout` 和 `directorargout`。Director 代码当前不使用任何其他类型的类型映射。目前尚不清楚哪种类型合适并需要支持。

### 36.5.7 其他杂项

Director typemaps for STL classes are in place, and hence you should be able to use std::vector, std::string, etc., as you would any other type.

**Note:** The director typemaps for return types based in const references, such as

> STL 类的 Director 类型映射已经到位，因此你应该能够像使用其他任何类型一样使用 `std::vector`、`std::string` 等。
>
> **注意**：Director 类型映射基于 const 引用的返回类型，例如

```c++
class Foo {
...
    virtual const int& bar();
...
};
```

will work only for simple call scenarios. Usually the resulting code is neither thread or reentrant safe. Hence, the user is advised to avoid returning const references in director methods. For example, the user could modify the method interface to use lvalue return types, wherever possible, for example

> 仅适用于简单的通话场景。通常，结果代码既不是线程安全的也不是可重入的。因此，建议用户避免在 director 方法中返回 const 引用。例如，用户可以在可能的情况下修改方法接口以使用左值返回类型，例如

```c++
class Foo {
...
    virtual int bar();
...
};
```

If that is not possible, the user should avoid enabling the director feature for reentrant, recursive or threaded member methods that return const references.

> 如果不可能，则用户应避免为返回 const 引用的可重入、递归或线程成员方法启用 Director 功能。

## 36.6 一般定制化特性

The last section presented the absolute basics of C/C++ wrapping. If you do nothing but feed SWIG a header file, you will get an interface that mimics the behavior described. However, sometimes this isn't enough to produce a nice module. Certain types of functionality might be missing or the interface to certain functions might be awkward. This section describes some common SWIG features that are used to improve your the interface to an extension module.

> 最后一部分介绍了 C/C++ 包装的绝对基础。如果除了向 SWIG 传递头文件外什么也不做，你将获得一个模仿所描述行为的接口。但是，有时这还不足以产生一个不错的模块。某些类型的功能可能会丢失，或者某些功能的接口可能会很尴尬。本节介绍了一些常用的 SWIG 功能，这些功能用于改善扩展模块的接口。

### 36.6.1 C/C++ 辅助函数

Sometimes when you create a module, it is missing certain bits of functionality. For example, if you had a function like this

> 有时，当你创建模块时，它会缺少某些功能。例如，如果你具有这样的函数

```c++
void set_transform(Image *im, double m[4][4]);
```

it would be accessible from Python, but there may be no easy way to call it. For example, you might get errors like this:

> 可以从 Python 访问它，但是可能没有简单的方法来调用它。例如，你可能会收到如下错误：

```python
>>> a = [
...   [1, 0, 0, 0],
...   [0, 1, 0, 0],
...   [0, 0, 1, 0],
...   [0, 0, 0, 1]]
>>> set_transform(im, a)
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
TypeError: Type error. Expected _p_a_4__double
```

The problem here is that there is no easy way to construct and manipulate a suitable `double [4][4]` value to use. To fix this, you can write some extra C helper functions. Just use the `%inline` directive. For example:

> 这里的问题是，没有简单的方法来构造和操纵要使用的适当的 `double [4][4]` 值。要解决此问题，你可以编写一些额外的 C 辅助函数。只需使用 `%inline` 指令即可。例如：

```
%inline %{
/* Note: double[4][4] is equivalent to a pointer to an array double (*)[4] */
double (*new_mat44())[4] {
  return (double (*)[4]) malloc(16*sizeof(double));
}
void free_mat44(double (*x)[4]) {
  free(x);
}
void mat44_set(double x[4][4], int i, int j, double v) {
  x[i][j] = v;
}
double mat44_get(double x[4][4], int i, int j) {
  return x[i][j];
}
%}
```

From Python, you could then write code like this:

> 在 Python 中你可以这样写：

```python
>>> a = new_mat44()
>>> mat44_set(a, 0, 0, 1.0)
>>> mat44_set(a, 1, 1, 1.0)
>>> mat44_set(a, 2, 2, 1.0)
...
>>> set_transform(im, a)
>>>
```

Admittedly, this is not the most elegant looking approach. However, it works and it wasn't too hard to implement. It is possible to clean this up using Python code, typemaps, and other customization features as covered in later sections.

> 诚然，这不是最优雅的方法。但是，它可以工作，并且实现起来并不难。可以使用 Python 代码，类型映射和其他自定义功能（如稍后部分所述）进行清理。

### 36.6.2 添加而外的 Python 代码

If writing support code in C isn't enough, it is also possible to write code in Python. This code gets inserted in to the `.py` file created by SWIG . One use of Python code might be to supply a high-level interface to certain functions. For example:

> 如果用 C 语言编写支持代码还不够，那么也可以用 Python 编写代码。这段代码被插入到 SWIG 创建的 `.py` 文件中。Python 代码的一种用途可能是为某些函数提供高级接口。例如：

```
void set_transform(Image *im, double x[4][4]);

...
/* Rewrite the high level interface to set_transform */
%pythoncode %{
def set_transform(im, x):
    a = new_mat44()
    for i in range(4):
        for j in range(4):
            mat44_set(a, i, j, x[i][j])
    _example.set_transform(im, a)
    free_mat44(a)
%}
```

In this example, `set_transform()` provides a high-level Python interface built on top of low-level helper functions. For example, this code now seems to work:

> 在这个例子中，`set_transform()` 提供了一个基于低级辅助函数的高级 Python 接口。例如，此代码现在似乎可以工作：

```python
>>> a = [
...   [1, 0, 0, 0],
...   [0, 1, 0, 0],
...   [0, 0, 1, 0],
...   [0, 0, 0, 1]]
>>> set_transform(im, a)
>>>
```

Admittedly, this whole scheme for wrapping the two-dimension array argument is rather ad-hoc. Besides, shouldn't a Python list or a Numeric Python array just work normally? We'll get to those examples soon enough. For now, think of this example as an illustration of what can be done without having to rely on any of the more advanced customization features.

There is also `%pythonbegin` which is another directive very similar to `%pythoncode`, but generates the given Python code at the beginning of the `.py`file. This directive works in the same way as `%pythoncode`, except the code is copied just after the SWIG banner (comment) at the top of the file, before any real code. This provides an opportunity to add your own description in a comment near the top of the file as well as Python imports that have to appear at the top of the file, such as `from __future__ import` statements.

The following shows example usage for Python 2.6 to use `print` as it can in Python 3, that is, as a function instead of a statement:

> 诚然，用于包装二维数组参数的整个方案都是临时的。此外，Python 列表或数字 Python 数组不应该正常工作吗？我们将尽快了解这些示例。现在，请以该示例为例，说明无需依赖任何更高级的自定义功能即可进行的操作。
>
> 还有 `%pythonbegin`，它是另一个类似于 `%pythoncode` 的指令，但是在 `.py` 文件的开头生成给定的 Python 代码。该指令的工作方式与 `%pythoncode` 相同，不同之处在于，该代码仅在文件顶部的 SWIG 标语（注释）之后，任何实际代码之前被复制。这提供了一个机会，可以在文件顶部附近的注释中添加你自己的描述，以及必须出现在文件顶部的 Python 导入，例如 `from __future__ import` 语句。

下面显示了 Python 2.6 在 Python 3 中尽可能使用 `print` 的示例用法，即作为函数而不是语句：

```
%pythonbegin %{
# This module provides wrappers to the Whizz Bang library
%}

%pythonbegin %{
from __future__ import print_function
print("Loading", "Whizz", "Bang", sep=' ... ')
%}
```

which can be seen when viewing the first few lines of the generated `.py` file:

> 查看生成 `.py` 文件的头几行将看到：

```python
# This file was automatically generated by SWIG (http://www.SWIG.org).
# Version 2.0.11
#
# Do not make changes to this file unless you know what you are doing--modify
# the SWIG interface file instead.

# This module provides wrappers to the Whizz Bang library

from __future__ import print_function
print("Loading", "Whizz", "Bang", sep=' ... ')
```

When using `%pythoncode` and `%pythonbegin` you generally want to make sure that the block is delimited by `%{` and `%}`. If you delimit it with `{` and `}` then any lines with a leading `#` will be handled by SWIG as preprocessor directives, when you probably meant them as Python comments. Prior to SWIG 3.0.3, invalid preprocessor directives were silently ignored, so generally using the wrong delimiters resulted in such comments not appearing in the generated output (though a comment starting with a valid preprocessor directive could cause problems, for example: `# error handling`). SWIG 3.0.3 and later report an error for invalid preprocessor directives, so you may have to update existing interface files to delimit blocks of Python code correctly.

As an alternative to providing a block containing Python code, you can include python code from a file. The code is inserted exactly as in the file, so this avoids any issues with the SWIG preprocessor. It's a good approach if you have a non-trivial chunk of Python code to insert. To use this feature you specify a filename in double quotes, for example:

> 当使用 `%pythoncode` 和 `%pythonbegin` 时，通常需要确保该块由 `%{` 和 `%}` 分隔。如果用 `{` 和 `}`分隔行，那么带有 `#` 的任何行都将被 SWIG 作为预处理器指令处理，这时你可能会将它们视为 Python 注释。在 SWIG 3.0.3 之前，无效的预处理器指令会被静默忽略，因此通常使用错误的定界符会导致此类注释未出现在生成的输出中（尽管以有效预处理器指令开头的注释可能会引起问题，例如：`# error handling`）。SWIG 3.0.3 和更高版本报告无效的预处理程序指令错误，因此你可能必须更新现有的接口文件以正确地分隔 Python 代码块。
>
> 作为提供包含 Python 代码的块的替代方法，你可以包括文件中的 Python 代码。代码将完全按照文件中的顺序插入，因此可以避免 SWIG 预处理程序出现任何问题。如果你要插入大量的 Python 代码，这是一个很好的方法。要使用此功能，请在双引号中指定文件名，例如：

```
%pythoncode "somecode.py"
```

Sometimes you may want to replace or modify the wrapper function that SWIG creates in the proxy `.py` file. The Python module in SWIG provides some features that enable you to do this. First, to entirely replace a proxy function you can use `%feature("shadow")`. For example:

> 有时你可能想替换或修改 SWIG 在代理 `.py` 文件中创建的包装函数。SWIG 中的 Python 模块提供了一些使你可以执行此操作的功能。首先，要完全替换代理功能，可以使用 `%feature("shadow")`。例如：

```
%module example

// Rewrite bar() python code

%feature("shadow") Foo::bar(int) %{
def bar(*args):
    #do something before
    $action
    #do something after
%}

class Foo {
public:
    int bar(int x);
};
```

where `$action` will be replaced by the call to the C/C++ proper method.

Often the proxy function created by SWIG is fine, but you simply want to add code to it without touching the rest of the generated function body. For these cases SWIG provides the `pythonprepend` and `pythonappend` features which do exactly as their names suggest. The `pythonprepend` feature will insert its value at the beginning of the proxy function, and `pythonappend` will insert code at the end of the proxy, just before the return statement.

> 其中的 `$action` 将被对 C/C++ 适当方法的调用所代替。
>
> 通常，SWIG 创建的代理功能很好，但是你只想向其中添加代码，而无需接触其余的生成的函数主体。对于这些情况，SWIG 提供了 `pythonprepend` 和 `pythonappend` 功能，它们的功能与名称一样。`pythonprepend` 功能将其值插入代理函数的开头，而 `pythonappend` 将代码插入代理的末尾，即 `return` 语句之前。

```
%module example

// Add python code to bar()

%feature("pythonprepend") Foo::bar(int) %{
    #do something before C++ call
%}

%feature("pythonappend") Foo::bar(int) %{
    #do something after C++ call
%}


class Foo {
public:
    int bar(int x);
};
```

Notes: Usually the `pythonappend` and `pythonprepend` features are safer to use than the `shadow` feature. Also, from SWIG version 1.3.28 you can use the directive forms `%pythonappend` and `%pythonprepend` as follows:

> 注意：通常，`pythonappend` 和 `pythonprepend` 功能比 `shadow` 功能更安全使用。另外，从 SWIG 版本 1.3.28 开始，你可以使用指令形式 `%pythonappend` 和 `%pythonprepend`，如下所示：

```
%module example

// Add python code to bar()

%pythonprepend Foo::bar(int) %{
    #do something before C++ call
%}

%pythonappend Foo::bar(int) %{
    #do something after C++ call
%}


class Foo {
public:
    int bar(int x);
};
```

Note that when the underlying C++ method is overloaded, there is only one proxy Python method for multiple C++ methods. In this case, only one of parsed methods is examined for the feature. You are better off specifying the feature without the argument list to ensure it will get used, as it will then get attached to all the overloaded C++ methods. For example:

> 请注意，当底层 C++ 方法重载时，多个 C++ 方法只有一个代理 Python 方法。在这种情况下，仅检查一种解析方法的功能。最好不要在没有确保可用参数列表的情况下指定该功能，因为它随后将附加到所有重载的 C++ 方法中。例如：

```
%module example

// Add python code to bar()

%pythonprepend Foo::bar %{
    #do something before C++ call
%}

%pythonappend Foo::bar %{
    #do something after C++ call
%}


class Foo {
public:
    int bar(int x);
    int bar();
};
```

The same applies for overloaded constructors.

> 对重载构造函数亦然。

### 36.6.3 用 `%extend` 扩展类

One of the more interesting features of SWIG is that it can extend structures and classes with new methods--at least in the Python interface. Here is a simple example:

> SWIG 的更有趣的功能之一是它可以使用新方法扩展结构体和类——至少在 Python 接口中。这是一个简单的示例：

```
%module example
%{
#include "someheader.h"
%}

struct Vector {
  double x, y, z;
};

%extend Vector {
  char *__str__() {
    static char tmp[1024];
    sprintf(tmp, "Vector(%g, %g, %g)", $self->x, $self->y, $self->z);
    return tmp;
  }
  Vector(double x, double y, double z) {
    Vector *v = (Vector *) malloc(sizeof(Vector));
    v->x = x;
    v->y = y;
    v->z = z;
    return v;
  }
};
```

Now, in Python

> 现在，在 Python 中

```python
>>> v = example.Vector(2, 3, 4)
>>> print v
Vector(2, 3, 4)
>>>
```

`%extend` can be used for many more tasks than this. For example, if you wanted to overload a Python operator, you might do this:

> `%extend` 可以用于更多任务。例如，如果要重载 Python 运算符，则可以这样做：

```
%extend Vector {
  Vector __add__(Vector *other) {
    Vector v;
    v.x = $self->x + other->x;
    v.y = $self->y + other->y;
    v.z = $self->z + other->z;
    return v;
  }
};
```

Use it like this:

> 这样使用：

```python
>>> import example
>>> v = example.Vector(2, 3, 4)
>>> w = example.Vector(10, 11, 12)
>>> print v+w
Vector(12, 14, 16)
>>>
```

`%extend` works with both C and C++ code. It does not modify the underlying object in any way--the extensions only show up in the Python interface.

> `%extend` 可以与 C 和 C++ 代码一起使用。它不会以任何方式修改底层对象——扩展仅显示在 Python 接口中。

### 36.6.4 用 `%exception` 处理异常

If a C or C++ function throws an error, you may want to convert that error into a Python exception. To do this, you can use the `%exception` directive.`%exception` simply lets you rewrite part of the generated wrapper code to include an error check.

In C, a function often indicates an error by returning a status code (a negative number or a NULL pointer perhaps). Here is a simple example of how you might handle that:

> 如果 C 或 C++ 函数引发错误，则可能需要将该错误转换为 Python 异常。为此，你可以使用 `%exception` 指令。`%exception` 仅允许你重写部分生成的包装器代码以包含错误检查。
>
> 在 C 语言中，函数通常通过返回状态码（可能为负数或 NULL 指针）来指示错误。这是你可能如何处理的一个简单示例：

```
%exception malloc {
  $action
  if (!result) {
    PyErr_SetString(PyExc_MemoryError, "Not enough memory");
    SWIG_fail;
  }
}
void *malloc(size_t nbytes);
```

In Python,

> 在 Python 中，

```python
>>> a = example.malloc(2000000000)
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
MemoryError: Not enough memory
>>>
```

If a library provides some kind of general error handling framework, you can also use that. For example:

> 如果库提供某种常规的错误处理框架，你也可以使用它。例如：

```
%exception {
  $action
  if (err_occurred()) {
    PyErr_SetString(PyExc_RuntimeError, err_message());
    SWIG_fail;
  }
}
```

No declaration name is given to `%exception`, it is applied to all wrapper functions.

C++ exceptions are also easy to handle. For example, you can write code like this:

> 没有为 `%exception` 指定任何声明名称，它适用于所有包装函数。
>
> C++ 异常也易于处理。例如，你可以编写如下代码：

```
%exception getitem {
  try {
    $action
  } catch (std::out_of_range &e) {
    PyErr_SetString(PyExc_IndexError, const_cast<char*>(e.what()));
    SWIG_fail;
  }
}

class Base {
public:
  Foo *getitem(int index);      // Exception handled added
  ...
};
```

When raising a Python exception from C, use the `PyErr_SetString()` function as shown above followed by `SWIG_fail`. The following exception types can be used as the first argument.

> 从 C 引发 Python 异常时，请使用如上所示的 `PyErr_SetString()` 函数，然后是 `SWIG_fail`。以下异常类型可以用作第一个参数。

```
PyExc_ArithmeticError
PyExc_AssertionError
PyExc_AttributeError
PyExc_EnvironmentError
PyExc_EOFError
PyExc_Exception
PyExc_FloatingPointError
PyExc_ImportError
PyExc_IndexError
PyExc_IOError
PyExc_KeyError
PyExc_KeyboardInterrupt
PyExc_LookupError
PyExc_MemoryError
PyExc_NameError
PyExc_NotImplementedError
PyExc_OSError
PyExc_OverflowError
PyExc_RuntimeError
PyExc_StandardError
PyExc_SyntaxError
PyExc_SystemError
PyExc_TypeError
PyExc_UnicodeError
PyExc_ValueError
PyExc_ZeroDivisionError
```

`SWIG_fail` is a C macro which when called within the context of SWIG wrapper function, will jump to the error handler code. This will call any cleanup code (freeing any temp variables) and then return from the wrapper function so that the Python interpreter can raise the Python exception. This macro should always be called after setting a Python error in code snippets, such as typemaps and `%exception`, that are ultimately generated into the wrapper function.

The language-independent `exception.i` library file can also be used to raise exceptions. See the [SWIG Library](http://www.SWIG.org/Doc3.0/Library.html#Library) chapter.

> `SWIG_fail` 是一个 C 宏，当在 SWIG 包装函数的上下文中调用时，将跳转到错误处理程序代码。这将调用任何清除代码（释放所有临时变量），然后从包装器函数返回，以便 Python 解释器可以引发 Python 异常。在代码片段中设置 Python 错误（例如类型映射和 `%exception`）后，应始终调用该宏，这些错误最终会生成到包装函数中。
>
> 独立于语言的 `exception.i` 库文件也可以用于引发异常。请参阅 [SWIG 库](http://www.SWIG.org/Doc3.0/Library.html#Library)一章。

## 36.7 小贴士和常用技术

Although SWIG is largely automatic, there are certain types of wrapping problems that require additional user input. Examples include dealing with output parameters, strings, binary data, and arrays. This chapter discusses the common techniques for solving these problems.

> 尽管 SWIG 在很大程度上是自动的，但是某些类型的包装问题需要额外的用户输入。示例包括处理输出参数、字符串、二进制数据和数组。本章讨论解决这些问题的常用技术。

### 36.7.1 输入输出参数

A common problem in some C programs is handling parameters passed as simple pointers. For example:

> 在某些 C 程序中，一个常见的问题是处理作为简单指针传递的参数。例如：

```c++
void add(int x, int y, int *result) {
  *result = x + y;
}
```

or perhaps

> 或者

```c++
int sub(int *x, int *y) {
  return *x-*y;
}
```

The easiest way to handle these situations is to use the `typemaps.i` file. For example:

> 处理这些情况的最简单方法是使用 `typemaps.i` 文件。例如：

```
%module example
%include "typemaps.i"

void add(int, int, int *OUTPUT);
int  sub(int *INPUT, int *INPUT);
```

In Python, this allows you to pass simple values. For example:

> 在 Python 中，这允许你传递简单的值。例如：

```python
>>> a = add(3, 4)
>>> print a
7
>>> b = sub(7, 4)
>>> print b
3
>>>
```

Notice how the `INPUT` parameters allow integer values to be passed instead of pointers and how the `OUTPUT` parameter creates a return result.

If you don't want to use the names `INPUT` or `OUTPUT`, use the `%apply`directive. For example:

> 注意 `INPUT` 参数如何允许传递整数值而不是指针，以及 `OUTPUT` 参数如何创建返回结果。
>
> 如果你不想使用名称 `INPUT` 或 `OUTPUT`，请使用 `%apply` 指令。例如：

```
%module example
%include "typemaps.i"

%apply int *OUTPUT { int *result };
%apply int *INPUT  { int *x, int *y};

void add(int x, int y, int *result);
int  sub(int *x, int *y);
```

If a function mutates one of its parameters like this,

> 如果一个函数像这样改变它的一个参数，

```c++
void negate(int *x) {
  *x = -(*x);
}
```

you can use `INOUT` like this:

> 你可以这样是用 `INOUT`：

```
%include "typemaps.i"
...
void negate(int *INOUT);
```

In Python, a mutated parameter shows up as a return value. For example:

> 在 Python 中，改变的参数显示为返回值。例如：

```python
>>> a = negate(3)
>>> print a
-3
>>>
```

Note: Since most primitive Python objects are immutable, it is not possible to perform in-place modification of a Python object passed as a parameter.

The most common use of these special typemap rules is to handle functions that return more than one value. For example, sometimes a function returns a result as well as a special error code:

> 注意：由于大多数原始 Python 对象都是不可变的，因此无法对作为参数传递的 Python 对象执行就地修改。
>
> 这些特殊类型映射规则最常见的用途是处理返回多个值的函数。例如，有时函数返回结果以及特殊的错误代码：

```c++
/* send message, return number of bytes sent, along with success code */
int send_message(char *text, int len, int *success);
```

To wrap such a function, simply use the `OUTPUT` rule above. For example:

> 要包装这样的函数，只需使用上面的 `OUTPUT` 规则。例如：

```
%module example
%include "typemaps.i"
%apply int *OUTPUT { int *success };
...
int send_message(char *text, int *success);
```

When used in Python, the function will return multiple values.

> 在 Python 中使用时，该函数将返回多个值。

```python
bytes, success = send_message("Hello World")
if not success:
    print "Whoa!"
else:
    print "Sent", bytes
```

Another common use of multiple return values are in query functions. For example:

> 多个返回值的另一个常见用法是查询函数。例如：

```c++
void get_dimensions(Matrix *m, int *rows, int *columns);
```

To wrap this, you might use the following:

> 要包装这样的函数，你可以这样做：

```
%module example
%include "typemaps.i"
%apply int *OUTPUT { int *rows, int *columns };
...
void get_dimensions(Matrix *m, int *rows, *columns);
```

Now, in Python:

> 现在，在 Python 中：

```python
>>> r, c = get_dimensions(m)
```

Be aware that the primary purpose of the `typemaps.i` file is to support primitive datatypes. Writing a function like this

> 请注意，`typemaps.i` 文件的主要目的是支持原始数据类型。编写这样的函数

```c++
void foo(Bar *OUTPUT);
```

may not have the intended effect since `typemaps.i` does not define an OUTPUT rule for `Bar`.

> 可能没有预期的效果，因为 `typemaps.i` 没有为 `Bar` 定义 `OUTPUT` 规则。

### 36.7.2 简单指针

If you must work with simple pointers such as `int *` or `double *` and you don't want to use `typemaps.i`, consider using the `cpointer.i` library file. For example:

> 如果你必须使用诸如 `int *` 或 `double *` 之类的简单指针，并且不想使用 `typemaps.i`，请考虑使用 `cpointer.i` 库文件。例如：

```
%module example
%include "cpointer.i"

%inline %{
extern void add(int x, int y, int *result);
%}

%pointer_functions(int, intp);
```

The `%pointer_functions(type, name)` macro generates five helper functions that can be used to create, destroy, copy, assign, and dereference a pointer. In this case, the functions are as follows:

> `%pointer_functions(type，name)` 宏生成五个辅助函数，可用于创建、销毁、复制、分配和取消引用指针。在这种情况下，函数如下：

```c++
int  *new_intp();
int  *copy_intp(int *x);
void  delete_intp(int *x);
void  intp_assign(int *x, int value);
int   intp_value(int *x);
```

In Python, you would use the functions like this:

> 在 Python 中你可以这样使用这些函数：

```python
>>> result = new_intp()
>>> print result
_108fea8_p_int
>>> add(3, 4, result)
>>> print intp_value(result)
7
>>>
```

If you replace `%pointer_functions()` by `%pointer_class(type, name)`, the interface is more class-like.

> 如果你用 `%pointer_class(type, name)` 替换 `%pointer_functions()`，接口会更像类。

```python
>>> result = intp()
>>> add(3, 4, result)
>>> print result.value()
7
```

See the [SWIG Library](http://www.SWIG.org/Doc3.0/Library.html#Library) chapter for further details.

> 更多细节参见 [SWIG 库](http://www.SWIG.org/Doc3.0/Library.html#Library) 章节。

### 36.7.3 无界 C 数组

Sometimes a C function expects an array to be passed as a pointer. For example,

> 有时，C 函数希望将数组作为指针传递。例如，

```c++
int sumitems(int *first, int nitems) {
    int i, sum = 0;
    for (i = 0; i < nitems; i++) {
        sum += first[i];
    }
    return sum;
}
```

To wrap this into Python, you need to pass an array pointer as the first argument. A simple way to do this is to use the `carrays.i` library file. For example:

> 要将其包装到 Python 中，你需要传递一个数组指针作为第一个参数。一种简单的方法是使用 `carrays.i` 库文件。例如：

```
%include "carrays.i"
%array_class(int, intArray);
```

The `%array_class(type, name)` macro creates wrappers for an unbounded array object that can be passed around as a simple pointer like `int *` or `double *`. For instance, you will be able to do this in Python:

> `%array_class(type, name)` 宏为无边界数组对象创建包装器，该包装器可以作为简单指针如 `int *` 或 `double *` 传递。例如，你将能够在 Python 中执行此操作：

```python
>>> a = intArray(10000000)         # Array of 10-million integers
>>> for i in xrange(10000):        # Set some values
...     a[i] = i
>>> sumitems(a, 10000)
49995000
>>>
```

The array "object" created by `%array_class()` does not encapsulate pointers inside a special array object. In fact, there is no bounds checking or safety of any kind (just like in C). Because of this, the arrays created by this library are extremely low-level indeed. You can't iterate over them nor can you even query their length. In fact, any valid memory address can be accessed if you want (negative indices, indices beyond the end of the array, etc.). Needless to say, this approach is not going to suit all applications. On the other hand, this low-level approach is extremely efficient and well suited for applications in which you need to create buffers, package binary data, etc.

> 由 `%array_class()` 创建的数组“对象”没有将指针封装在特殊的数组对象中。实际上，没有边界检查或任何类型的安全性（就像C中一样）。因此，此库创建的数组确实是非常低级的。你无法遍历它们，甚至无法查询它们的长度。实际上，如果需要，可以访问任何有效的内存地址（负索引，超出数组末尾的索引等）。不用说，这种方法并不适合所有应用程序。另一方面，这种低级方法非常高效，非常适合需要创建缓冲区，打包二进制数据等的应用程序。

### 36.7.4 处理字符串

If a C function has an argument of `char *`, then a Python string can be passed as input. For example:

> 如果 C 函数的参数为 `char *`，则可以将 Python 字符串作为输入传递。例如：

```
// C
void foo(char *s);
# Python
>>> foo("Hello")
```

When a Python string is passed as a parameter, the C function receives a pointer to the raw data contained in the string. Since Python strings are immutable, it is illegal for your program to change the value. In fact, doing so will probably crash the Python interpreter.

If your program modifies the input parameter or uses it to return data, consider using the `cstring.i` library file described in the [SWIG Library](http://www.SWIG.org/Doc3.0/Library.html#Library) chapter.

When functions return a `char *`, it is assumed to be a NULL-terminated string. Data is copied into a new Python string and returned.

If your program needs to work with binary data, you can use a typemap to expand a Python string into a pointer/length argument pair. As luck would have it, just such a typemap is already defined. Just do this:

> 当将 Python 字符串作为参数传递时，C 函数将收到一个指向该字符串中包含的原始数据的指针。由于 Python 字符串是不可变的，因此程序更改该值是非法的。实际上，这样做可能会使 Python 解释器崩溃。
>
> 如果你的程序修改了输入参数或使用它来返回数据，请考虑使用 [SWIG 库](http://www.SWIG.org/Doc3.0/Library.html#Library)一章。
>
> 当函数返回一个 `char *` 时，它被认为是一个以 NULL 结尾的字符串。数据被复制到新的 Python 字符串中并返回。
>
> 如果你的程序需要使用二进制数据，则可以使用类型映射将 Python 字符串扩展为指针/长度参数对。幸运的是，已经定义了这种类型映射。只要这样做：

```
%apply (char *STRING, int LENGTH) { (char *data, int size) };
...
int parity(char *data, int size, int initial);
```

Now in Python:

> 现在 Python 中：

```python
>>> parity("e\x09ffss\x00\x00\x01\nx", 0)
```

If you need to return binary data, you might use the `cstring.i` library file. The `cdata.i` library can also be used to extra binary data from arbitrary pointers.

> 如果需要返回二进制数据，可以使用 `cstring.i` 库文件。`cdata.i` 库也可以用于从任意指针中获取额外的二进制数据。

### 36.7.5 默认参数

C++ default argument code generation is documented in the main [Default arguments](http://www.SWIG.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args)section. There is also an optional Python specific feature that can be used called the `python:cdefaultargs` [feature flag](http://www.SWIG.org/Doc3.0/Customization.html#Customization_feature_flags). By default, SWIG attempts to convert C++ default argument values into Python values and generates code into the Python layer containing these values. For example:

> C++ 默认参数代码的生成主要记录在[默认参数](http://www.SWIG.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args)部分中。还有一个可选的 Python 特定功能，称为 `python:cdefaultargs` [功能标志](http://www.SWIG.org/Doc3.0/Customization.html#Customization_feature_flags)。默认情况下，SWIG 尝试将 C++ 默认参数值转换为 Python 值，并在包含这些值的 Python 层中生成代码。例如：

```c++
struct CDA {
  int fff(int a = 1, bool b = false);
};
```

From Python this can be called as follows:

> Python 中可以这样调用：

```python
>>> CDA().fff()        # C++ layer receives a=1 and b=false
>>> CDA().fff(2)       # C++ layer receives a=2 and b=false
>>> CDA().fff(3, True) # C++ layer receives a=3 and b=true
```

The default code generation in the Python layer is:

> Python 层中的默认参数生成代码如下：

```python
class CDA(object):
    ...
    def fff(self, a=1, b=False):
        return _default_args.CDA_fff(self, a, b)
```

Adding the feature:

> 添加功能：

```
%feature("python:cdefaultargs") CDA::fff;
struct CDA {
  int fff(int a = 1, bool b = false);
```

results in identical behaviour when called from Python, however, it results in different code generation:

> 从 Python 调用时会导致相同的行为，但是会导致生成不同的代码：

```python
class CDA(object):
    ...
    def fff(self, *args):
        return _default_args.CDA_fff(self, *args)
```

The default arguments are obtained in the C++ wrapper layer instead of the Python layer. Some code generation modes are quite different, eg `-builtin` and `-fastproxy`, and are unaffected by `python:cdefaultargs` as the default values are always obtained from the C++ layer.

Note that not all default arguments can be converted into a Python equivalent. When SWIG does not convert them, it will generate code to obtain them from the C++ layer as if `python:cdefaultargs` was specified. This will happen if just one argument cannot be converted into a Python equivalent. This occurs typically when the argument is not fully numeric, such as `int(1)`:

> 默认参数在 C++ 包装器层而不是 Python 层中获得。某些代码生成模式大不相同，例如 `-builtin` 和 `-fastproxy`，并且不受 `python:cdefaultargs` 的影响，因为默认值始终是从 C++ 层获得的。
>
> 请注意，并非所有默认参数都可以转换为等效的 Python 实现。如果 SWIG 不转换它们，它将生成代码以从 C++ 层获取它们，就像指定了 `python:cdefaultargs` 一样。如果仅一个参数不能转换为等效的 Python 就会发生这种情况。通常在参数不是全数字的情况下发生，例如 `int(1)`：

```c++
struct CDA {
  int fff(int a = int(1), bool b = false);
};
```

**Compatibility Note:** SWIG -3.0.6 introduced the `python:cdefaultargs` feature. Versions of SWIG prior to this varied in their ability to convert C++ default values into equivalent Python default argument values.

> **兼容性说明**： SWIG -3.0.6 引入了 `python:cdefaultargs` 功能。在此之前，SWIG 的版本将 C++ 默认值转换为等效的 Python 默认参数值的能力各不相同。

## 36.8 类型映射

This section describes how you can modify SWIG's default wrapping behavior for various C/C++ datatypes using the `%typemap` directive. This is an advanced topic that assumes familiarity with the Python C API as well as the material in the "[Typemaps](http://www.SWIG.org/Doc3.0/Typemaps.html#Typemaps)" chapter.

Before proceeding, it should be stressed that typemaps are not a required part of using SWIG ---the default wrapping behavior is enough in most cases. Typemaps are only used if you want to change some aspect of the primitive C-Python interface or if you want to elevate your guru status.

> 本节描述了如何使用 `%typemap` 指令修改 SWIG 对各种 C/C++ 数据类型的默认包装行为。这是一个高级主题，假定你熟悉 Python C API 以及[类型映射](http://www.SWIG.org/Doc3.0/Typemaps.html#Typemaps)一章中的内容。
>
> 在继续之前，应该强调的是类型映射不是使用 SWIG 的必需部分——在大多数情况下，默认的包装行为就足够了。只有当你要更改原始 C-Python 接口的某些方面或要成为专家时，才使用类型映射。

### 36.8.1 什么是类型映射？

A typemap is nothing more than a code generation rule that is attached to a specific C datatype. For example, to convert integers from Python to C, you might define a typemap like this:

> 类型映射不过是附加到特定 C 数据类型的代码生成规则。例如，要将整数从 Python 转换为 C，你可以定义如下类型映射：

```
%module example

%typemap(in) int {
  $1 = (int) PyLong_AsLong($input);
  printf("Received an integer : %d\n", $1);
}
%inline %{
extern int fact(int n);
%}
```

Typemaps are always associated with some specific aspect of code generation. In this case, the "in" method refers to the conversion of input arguments to C/C++. The datatype `int` is the datatype to which the typemap will be applied. The supplied C code is used to convert values. In this code a number of special variable prefaced by a `$` are used. The `$1` variable is placeholder for a local variable of type `int`. The `$input` variable is the input object of type `PyObject *`.

When this example is compiled into a Python module, it operates as follows:

> 类型映射始终与代码生成的某些特定方面相关联。在这种情况下，`in` 方法是指将输入参数转换为 C/C++ 。数据类型 `int` 是将要应用类型映射的数据类型。提供的 C 代码用于转换值。在此代码中，使用了多个以 `$` 开头的特殊变量。`$1` 变量是类型为 `int` 的局部变量的占位符。`$input` 变量是类型为 `PyObject *` 的输入对象。
>
> 将此示例编译为 Python 模块时，其操作如下：

```python
>>> from example import *
>>> fact(6)
Received an integer : 6
720
```

In this example, the typemap is applied to all occurrences of the `int` datatype. You can refine this by supplying an optional parameter name. For example:

> 在此示例中，类型映射应用于所有出现的 `int` 数据类型。你可以通过提供可选的参数名称来优化此功能。例如：

```
%module example

%typemap(in) int nonnegative {
  $1 = (int) PyLong_AsLong($input);
  if ($1 < 0) {
    PyErr_SetString(PyExc_ValueError, "Expected a nonnegative value.");
    SWIG_fail;
  }
}
%inline %{
extern int fact(int nonnegative);
%}
```

In this case, the typemap code is only attached to arguments that exactly match `int nonnegative`.

The application of a typemap to specific datatypes and argument names involves more than simple text-matching--typemaps are fully integrated into the SWIG C++ type-system. When you define a typemap for `int`, that typemap applies to `int` and qualified variations such as `const int`. In addition, the typemap system follows `typedef` declarations. For example:

> 在这种情况下，类型映射代码仅附加到与 `int nonnegative` 完全匹配的参数上。
>
> 将类型映射应用于特定的数据类型和参数名称所涉及的不仅仅是简单的文本匹配，类型映射已完全集成到 SWIG C++ 类型系统中。当你为 `int` 定义类型映射时，该类型映射适用于 `int` 和限定变量，例如 `const int`。另外，类型映射系统遵循 `typedef` 声明。例如：

```
%typemap(in) int n {
  $1 = (int) PyLong_AsLong($input);
  printf("n = %d\n", $1);
}
%inline %{
typedef int Integer;
extern int fact(Integer n);    // Above typemap is applied
%}
```

Typemaps can also be defined for groups of consecutive arguments. For example:

> 还可以为连续参数组定义类型映射。例如：

```
%typemap(in) (char *str, int len) {
  $1 = PyString_AsString($input);
  $2 = PyString_Size($input);
};

int count(char c, char *str, int len);
```

When a multi-argument typemap is defined, the arguments are always handled as a single Python object. This allows the function to be used like this (notice how the length parameter is omitted):

> 定义多参数类型映射时，参数始终作为单个 Python 对象处理。这允许像下面这样使用该函数（注意如何省略长度参数）：

```python
>>> example.count('e', 'Hello World')
1
>>>
```

### 36.8.2 Python 类型映射

The previous section illustrated an "in" typemap for converting Python objects to C. A variety of different typemap methods are defined by the Python module. For example, to convert a C integer back into a Python object, you might define an "out" typemap like this:

> 上一节说明了用于将 Python 对象转换为 C 的 `in` 类型映射。Python 模块定义了各种不同的类型映射方法。例如，要将 C 整数转换回 Python 对象，可以定义一个 `out` 类型映射，如下所示：

```
%typemap(out) int {
    $result = PyInt_FromLong((long) $1);
}
```

A detailed list of available methods can be found in the "[Typemaps](http://www.SWIG.org/Doc3.0/Typemaps.html#Typemaps)" chapter.

However, the best source of typemap information (and examples) is probably the Python module itself. In fact, all of SWIG's default type handling is defined by typemaps. You can view these typemaps by looking at the files in the SWIG library. Just take into account that in the latest versions of SWIG (1.3.22+), the library files are not very pristine clear for the casual reader, as they used to be. The extensive use of macros and other ugly techniques in the latest version produce a very powerful and consistent python typemap library, but at the cost of simplicity and pedagogic value.

To learn how to write a simple or your first typemap, you better take a look at the SWIG library version 1.3.20 or so.

> 可用方法的详细列表可以在[类型映射](http://www.SWIG.org/Doc3.0/Typemaps.html#Typemaps)一章中找到。
>
> 但是，类型映射信息（和示例）的最佳来源可能是 Python 模块本身。实际上，SWIG 的所有默认类型处理都是由类型映射定义的。你可以通过查看 SWIG 库中的文件来查看这些类型映射。只需考虑到在最新版本的 SWIG（1.3.22+）中，库文件对于临时阅读者来说就不是很原始了。最新版本中大量使用宏和其他难看的技术产生了一个非常强大且一致的 Python typemap 库，但是却以简单性和教学价值为代价。
>
> 要学习如何编写简单的，或你的第一个类型映射，最好查看版本 1.3.20 左右的 SWIG 库。

### 36.8.3 类型映射变量

Within typemap code, a number of special variables prefaced with a `$` may appear. A full list of variables can be found in the "[Typemaps](http://www.SWIG.org/Doc3.0/Typemaps.html#Typemaps)" chapter. This is a list of the most common variables:

> 在类型映射代码中，可能会出现许多以 `$` 开头的特殊变量。变量的完整列表可以在[类型映射](http://www.SWIG.org/Doc3.0/Typemaps.html#Typemaps)一章中找到。这是最常见的变量的列表：

```
$1
```

A C local variable corresponding to the actual type specified in the`%typemap` directive. For input values, this is a C local variable that's supposed to hold an argument value. For output values, this is the raw result that's supposed to be returned to Python.

> 一个 C 局部变量，对应于 `%typemap` 指令中指定的实际类型。对于输入值，这是 C 局部变量，应该保存参数值。对于输出值，这是应该返回给 Python 的原始结果。

```
$input
```

A `PyObject *` holding a raw Python object with an argument or variable value.

> `PyObject *` 持有参数或变量值的原始 Python 对象。

```
$result
```

A `PyObject *` that holds the result to be returned to Python.

> `PyObject *` 持有 Python 返回的结果。

```
$1_name
```

The parameter name that was matched.

> 参数名被匹配。

```
$1_type
```

The actual C datatype matched by the typemap.

> C 数据类型有类型映射匹配。

```
$1_ltype
```

An assignable version of the datatype matched by the typemap (a type that can appear on the left-hand-side of a C assignment operation). This type is stripped of qualifiers and may be an altered version of `$1_type`. All arguments and local variables in wrapper functions are declared using this type so that their values can be properly assigned.

> 与类型映射匹配的数据类型的可分配版本（一种类型，可以显示在 C 分配操作的左侧）。这个类型没有限定符，可以是 `$1_type` 的修改版本。包装函数中的所有参数和局部变量都使用此类型声明，以便可以正确分配它们的值。

```
$symname
```

The Python name of the wrapper function being created.

> 包装函数在 Python 中的名字被创建。

### 36.8.4 有用的 Python 函数

When you write a typemap, you usually have to work directly with Python objects. The following functions may prove to be useful.

> 编写类型映射时，通常必须直接使用 Python 对象。以下函数可能被证明是有用的。

**Python Integer Functions**

```c++
PyObject *PyInt_FromLong(long l);
long      PyInt_AsLong(PyObject *);
int       PyInt_Check(PyObject *);
```

**Python Floating Point Functions**

```c++
PyObject *PyFloat_FromDouble(double);
double    PyFloat_AsDouble(PyObject *);
int       PyFloat_Check(PyObject *);
```

**Python String Functions**

```c++
PyObject *PyString_FromString(char *);
PyObject *PyString_FromStringAndSize(char *, lint len);
int       PyString_Size(PyObject *);
char     *PyString_AsString(PyObject *);
int       PyString_Check(PyObject *);
```

**Python List Functions**

```c++
PyObject *PyList_New(int size);
int       PyList_Size(PyObject *list);
PyObject *PyList_GetItem(PyObject *list, int i);
int       PyList_SetItem(PyObject *list, int i, PyObject *item);
int       PyList_Insert(PyObject *list, int i, PyObject *item);
int       PyList_Append(PyObject *list, PyObject *item);
PyObject *PyList_GetSlice(PyObject *list, int i, int j);
int       PyList_SetSlice(PyObject *list, int i, int , PyObject *list2);
int       PyList_Sort(PyObject *list);
int       PyList_Reverse(PyObject *list);
PyObject *PyList_AsTuple(PyObject *list);
int       PyList_Check(PyObject *);
```

**Python Tuple Functions**

```c++
PyObject *PyTuple_New(int size);
int       PyTuple_Size(PyObject *);
PyObject *PyTuple_GetItem(PyObject *, int i);
int       PyTuple_SetItem(PyObject *, int i, PyObject *item);
PyObject *PyTuple_GetSlice(PyObject *t, int i, int j);
int       PyTuple_Check(PyObject *);
```

**Python Dictionary Functions**

```c++
PyObject *PyDict_New();
int       PyDict_Check(PyObject *);
int       PyDict_SetItem(PyObject *p, PyObject *key, PyObject *val);
int       PyDict_SetItemString(PyObject *p, const char *key, PyObject *val);
int       PyDict_DelItem(PyObject *p, PyObject *key);
int       PyDict_DelItemString(PyObject *p, char *key);
PyObject* PyDict_Keys(PyObject *p);
PyObject* PyDict_Values(PyObject *p);
PyObject* PyDict_GetItem(PyObject *p, PyObject *key);
PyObject* PyDict_GetItemString(PyObject *p, const char *key);
int       PyDict_Next(PyObject *p, Py_ssize_t *ppos, PyObject **pkey, PyObject **pvalue);
Py_ssize_t PyDict_Size(PyObject *p);
int       PyDict_Update(PyObject *a, PyObject *b);
int       PyDict_Merge(PyObject *a, PyObject *b, int override);
PyObject* PyDict_Items(PyObject *p);
```

**Python File Conversion Functions**

```c++
PyObject *PyFile_FromFile(FILE *f);
FILE     *PyFile_AsFile(PyObject *);
int       PyFile_Check(PyObject *);
```

**Abstract Object Interface**

```
write me
```

## 36.9 类型映射示例

This section includes a few examples of typemaps. For more examples, you might look at the files `python.swg` and `typemaps.i` in the SWIG library.

> 本节包含一些类型映射的示例。有关更多示例，你可以查看 SWIG 库中的文件 `python.swg` 和 `typemaps.i`。

### 36.9.1 将 Python list 转换为 `char **`

A common problem in many C programs is the processing of command line arguments, which are usually passed in an array of NULL terminated strings. The following SWIG interface file allows a Python list object to be used as a `char **`object.

> 许多 C 程序中的一个常见问题是命令行参数的处理，这些参数通常以 NULL 终止的字符串数组形式传递。以下 SWIG 接口文件允许将 Python 列表对象用作 `char **` 对象。

```
%module argv

// This tells SWIG to treat char ** as a special case
%typemap(in) char ** {
  /* Check if is a list */
  if (PyList_Check($input)) {
    int size = PyList_Size($input);
    int i = 0;
    $1 = (char **) malloc((size+1)*sizeof(char *));
    for (i = 0; i < size; i++) {
      PyObject *o = PyList_GetItem($input, i);
      if (PyString_Check(o)) {
        $1[i] = PyString_AsString(PyList_GetItem($input, i));
      } else {
        free($1);
        PyErr_SetString(PyExc_TypeError, "list must contain strings");
        SWIG_fail;
      }
    }
    $1[i] = 0;
  } else {
    PyErr_SetString(PyExc_TypeError, "not a list");
    SWIG_fail;
  }
}

// This cleans up the char ** array we malloc'd before the function call
%typemap(freearg) char ** {
  free((char *) $1);
}

// Now a test function
%inline %{
int print_args(char **argv) {
  int i = 0;
  while (argv[i]) {
    printf("argv[%d] = %s\n", i, argv[i]);
    i++;
  }
  return i;
}
%}
```

When this module is compiled, the wrapped C function now operates as follows :

> 当模块被编译后，包装后的 C 函数这样操作：

```python
>>> from argv import *
>>> print_args(["Dave", "Mike", "Mary", "Jane", "John"])
argv[0] = Dave
argv[1] = Mike
argv[2] = Mary
argv[3] = Jane
argv[4] = John
5
```

In the example, two different typemaps are used. The "in" typemap is used to receive an input argument and convert it to a C array. Since dynamic memory allocation is used to allocate memory for the array, the "freearg" typemap is used to later release this memory after the execution of the C function.

> 在示例中，使用了两个不同的类型映射。`in` 类型映射用于接受输入参数并将其转换为 C 数组。由于动态内存分配用于为数组分配内存，因此在执行 C 函数之后，将使用 `freearg` 类型映射稍后释放该内存。

### 36.9.2 扩展一个 Python 对象到多参数

Suppose that you had a collection of C functions with arguments such as the following:

> 假设你有一个带有以下参数的 C 函数集合：

```c++
int foo(int argc, char **argv);
```

In the previous example, a typemap was written to pass a Python list as the `char **argv`. This allows the function to be used from Python as follows:

> 在前面的示例中，编写了一个类型映射以将 Python 列表作为 `char **argv` 传递。这样可以从 Python 中按以下方式使用该函数：

```python
>>> foo(4, ["foo", "bar", "spam", "1"])
```

Although this works, it's a little awkward to specify the argument count. To fix this, a multi-argument typemap can be defined. This is not very difficult--you only have to make slight modifications to the previous example:

> 尽管这可行，但指定参数计数有点尴尬。为了解决这个问题，可以定义一个多参数的类型映射。这不是很困难，你只需对上一个示例进行一些修改：

```
%typemap(in) (int argc, char **argv) {
  /* Check if is a list */
  if (PyList_Check($input)) {
    int i;
    $1 = PyList_Size($input);
    $2 = (char **) malloc(($1+1)*sizeof(char *));
    for (i = 0; i < $1; i++) {
      PyObject *o = PyList_GetItem($input, i);
      if (PyString_Check(o)) {
        $2[i] = PyString_AsString(PyList_GetItem($input, i));
      } else {
        free($2);
        PyErr_SetString(PyExc_TypeError, "list must contain strings");
        SWIG_fail;
      }
    }
    $2[i] = 0;
  } else {
    PyErr_SetString(PyExc_TypeError, "not a list");
    SWIG_fail;
  }
}

%typemap(freearg) (int argc, char **argv) {
  free((char *) $2);
}
```

When writing a multiple-argument typemap, each of the types is referenced by a variable such as `$1` or `$2`. The typemap code simply fills in the appropriate values from the supplied Python object.

With the above typemap in place, you will find it no longer necessary to supply the argument count. This is automatically set by the typemap code. For example:

> 编写多参数类型映射时，每个类型均由变量 `$1` 或 `$2` 引用。类型映射代码只是从提供的 Python 对象中填充适当的值。
>
> 使用上面的类型映射，你将发现不再需要提供参数计数。这是由类型映射代码自动设置的。例如：

```python
>>> foo(["foo", "bar", "spam", "1"])
```

If your function is overloaded in C++, for example:

> 如果你的函数在 C++ 中冲在，例如：

```c++
int foo(int argc, char **argv);
int foo();
```

don't forget to also provide a suitable [typecheck typemap for overloading](http://www.SWIG.org/Doc3.0/Typemaps.html#Typemaps_overloading) such as:

> 不要忘了也为[重载类型映射](http://www.SWIG.org/Doc3.0/Typemaps.html#Typemaps_overloading)提供一个合适的类型检查:

```
%typecheck( SWIG_TYPECHECK_STRING_ARRAY) (int argc, char **argv) {
  $1 = PyList_Check($input) ? 1 : 0;
}
```

If you don't you'll get an error message along the lines of:

> 如果不提供，你将会得到一条错误信息：

```
Traceback (most recent call last):
  File "runme.py", line 3, in >module<
    example.foo(["foo", "bar", "spam", "1"])
NotImplementedError: Wrong number or type of arguments for overloaded function 'foo'.
  Possible C/C++ prototypes are:
    foo(int, char **)
    foo()
```

### 36.9.3 使用类型映射返回参数

A common problem in some C programs is that values may be returned in arguments rather than in the return value of a function. For example:

> 在某些 C 程序中，一个常见的问题是值可能在参数中返回，而不是在函数的返回值中返回。例如：

```c++
/* Returns a status value and two values in out1 and out2 */
int spam(double a, double b, double *out1, double *out2) {
  ... Do a bunch of stuff ...
  *out1 = result1;
  *out2 = result2;
  return status;
}
```

A typemap can be used to handle this case as follows :

> 一个类型映射可以用来处理这种情况，如下所示：

```
%module outarg

// This tells SWIG to treat an double * argument with name 'OutValue' as
// an output value.  We'll append the value to the current result which
// is guaranteed to be a List object by SWIG .

%typemap(argout) double *OutValue {
  PyObject *o, *o2, *o3;
  o = PyFloat_FromDouble(*$1);
  if ((!$result) || ($result == Py_None)) {
    $result = o;
  } else {
    if (!PyTuple_Check($result)) {
      PyObject *o2 = $result;
      $result = PyTuple_New(1);
      PyTuple_SetItem($result, 0, o2);
    }
    o3 = PyTuple_New(1);
    PyTuple_SetItem(o3, 0, o);
    o2 = $result;
    $result = PySequence_Concat(o2, o3);
    Py_DECREF(o2);
    Py_DECREF(o3);
  }
}

int spam(double a, double b, double *OutValue, double *OutValue);
```

The typemap works as follows. First, a check is made to see if any previous result exists. If so, it is turned into a tuple and the new output value is concatenated to it. Otherwise, the result is returned normally. For the sample function `spam()`, there are three output values--meaning that the function will return a 3-tuple of the results.

As written, the function must accept 4 arguments as input values, last two being pointers to doubles. If these arguments are only used to hold output values (and have no meaningful input value), an additional typemap can be written. For example:

> 类型映射的工作方式如下。首先，进行检查以查看是否存在任何先前的结果。如果是这样，它将变成一个元组并将新的输出值连接到它。否则，结果将正常返回。对于样本函数 `spam()`，有三个输出值——这意味着该函数将返回结果的三元组。
>
> 按照编写的方法，该函数必须接受 4 个参数作为输入值，最后两个是指向 `double` 的指针。如果这些参数仅用于保存输出值（并且没有有意义的输入值），则可以编写其他类型映射。例如：

```
%typemap(in, numinputs=0) double *OutValue(double temp) {
  $1 = &temp;
}
```

By specifying numinputs=0, the input value is ignored. However, since the argument still has to be set to some meaningful value before calling C, it is set to point to a local variable `temp`. When the function stores its output value, it will simply be placed in this local variable. As a result, the function can now be used as follows:

> 通过指定 `numinputs = 0`，输入值将被忽略。但是，由于在调用 C 之前仍必须将参数设置为一些有意义的值，因此将其设置为指向局部变量 `temp`。当函数存储其输出值时，它将简单地放置在此局部变量中。结果，该函数现在可以按以下方式使用：

```python
>>> a = spam(4, 5)
>>> print a
(0, 2.45, 5.0)
>>> x, y, z = spam(4, 5)
>>>
```

### 36.9.4 将 Python 元组映射到小数组

In some applications, it is sometimes desirable to pass small arrays of numbers as arguments. For example :

> 在某些应用程序中，有时希望传递数字的小数组作为参数。例如：

```c++
extern void set_direction(double a[4]);       // Set direction vector
```

This too, can be handled used typemaps as follows :

> 类型映射也可以用来处理，如下所示：

```
// Grab a 4 element array as a Python 4-tuple
%typemap(in) double[4](double temp[4]) {   // temp[4] becomes a local variable
  int i;
  if (PyTuple_Check($input)) {
    if (!PyArg_ParseTuple($input, "dddd", temp, temp+1, temp+2, temp+3)) {
      PyErr_SetString(PyExc_TypeError, "tuple must have 4 elements");
      SWIG_fail;
    }
    $1 = &temp[0];
  } else {
    PyErr_SetString(PyExc_TypeError, "expected a tuple.");
    SWIG_fail;
  }
}
```

This allows our `set_direction` function to be called from Python as follows :

> 这让 `set_direction` 函数在 Python 中可以按以下方式使用：

```python
>>> set_direction((0.5, 0.0, 1.0, -0.25))
```

Since our mapping copies the contents of a Python tuple into a C array, such an approach would not be recommended for huge arrays, but for small structures, this approach works fine.

> 由于我们的映射将 Python 元组的内容复制到 C 数组中，因此不建议将这种方法用于大型数组，但对于小型结构体，则此方法效果很好。

### 36.9.5 将序列映射到 C 数组

Suppose that you wanted to generalize the previous example to handle C arrays of different sizes. To do this, you might write a typemap as follows:

> 假设你想推广前面的示例以处理不同大小的 C 数组。为此，你可以按如下方式编写类型映射：

```
// Map a Python sequence into any sized C double array
%typemap(in) double[ANY](double temp[$1_dim0]) {
  int i;
  if (!PySequence_Check($input)) {
    PyErr_SetString(PyExc_TypeError, "Expecting a sequence");
    SWIG_fail;
  }
  if (PyObject_Length($input) != $1_dim0) {
    PyErr_SetString(PyExc_ValueError, "Expecting a sequence with $1_dim0 elements");
    SWIG_fail;
  }
  for (i =0; i < $1_dim0; i++) {
    PyObject *o = PySequence_GetItem($input, i);
    if (!PyFloat_Check(o)) {
      Py_XDECREF(o);
      PyErr_SetString(PyExc_ValueError, "Expecting a sequence of floats");
      SWIG_fail;
    }
    temp[i] = PyFloat_AsDouble(o);
    Py_DECREF(o);
  }
  $1 = &temp[0];
}
```

In this case, the variable `$1_dim0` is expanded to match the array dimensions actually used in the C code. This allows the typemap to be applied to types such as:

> 在这种情况下，变量 `$1_dim0` 被扩展以匹配 C 代码中实际使用的数组尺寸。这允许将类型映射应用于以下类型：

```c++
void foo(double x[10]);
void bar(double a[4], double b[8]);
```

Since the above typemap code gets inserted into every wrapper function where used, it might make sense to use a helper function instead. This will greatly reduce the amount of wrapper code. For example:

> 由于上面的类型映射代码被插入到每个包装函数中使用的位置，因此使用辅助函数可能更有意义。这将大大减少包装器代码的数量。例如：

```
%{
static int convert_darray(PyObject *input, double *ptr, int size) {
  int i;
  if (!PySequence_Check(input)) {
    PyErr_SetString(PyExc_TypeError, "Expecting a sequence");
    return 0;
  }
  if (PyObject_Length(input) != size) {
    PyErr_SetString(PyExc_ValueError, "Sequence size mismatch");
    return 0;
  }
  for (i =0; i < size; i++) {
    PyObject *o = PySequence_GetItem(input, i);
    if (!PyFloat_Check(o)) {
      Py_XDECREF(o);
      PyErr_SetString(PyExc_ValueError, "Expecting a sequence of floats");
      return 0;
    }
    ptr[i] = PyFloat_AsDouble(o);
    Py_DECREF(o);
  }
  return 1;
}
%}

%typemap(in) double [ANY](double temp[$1_dim0]) {
  if (!convert_darray($input, temp, $1_dim0)) {
    SWIG_fail;
  }
  $1 = &temp[0];
}
```

### 36.9.6 处理指针

Occasionally, it might be necessary to convert pointer values that have been stored using the SWIG typed-pointer representation. Since there are several ways in which pointers can be represented, the following two functions are used to safely perform this conversion:

> 有时，可能有必要转换使用 SWIG 类型指针表示形式存储的指针值。由于可以通过多种方式表示指针，因此可以使用以下两个函数安全地执行此转换：

```c++
int SWIG_ConvertPtr(PyObject *obj, void **ptr, SWIG_type_info *ty, int flags)
```

Converts a Python object `obj` to a C pointer. The result of the conversion is placed into the pointer located at `ptr`. `ty` is a SWIG type descriptor structure. `flags` is used to handle error checking and other aspects of conversion. It is the bitwise-or of several flag values including `SWIG_POINTER_EXCEPTION` and `SWIG_POINTER_DISOWN`. The first flag makes the function raise an exception on type error. The second flag additionally steals ownership of an object. Returns 0 on success and -1 on error.

> 将 Python 对象 `obj` 转换为 C 指针。转换结果放置在位于 `ptr` 的指针中。`ty` 是 SWIG 类型的描述符结构体。`flags` 用于处理错误检查和转换的其他方面。它是几个标志值的按位或，包括 `SWIG_POINTER_EXCEPTION` 和 `SWIG_POINTER_DISOWN`。第一个标志使函数在类型错误时引发异常。第二个标志还窃取了对象的所有权。成功返回 0，错误返回 -1。

```c++
PyObject * SWIG_NewPointerObj(void *ptr, SWIG_type_info *ty, int own)
```

Creates a new Python pointer object. `ptr` is the pointer to convert, `ty`is the SWIG type descriptor structure that describes the type, and `own`is a flag that indicates whether or not Python should take ownership of the pointer.

Both of these functions require the use of a special SWIG type-descriptor structure. This structure contains information about the mangled name of the datatype, type-equivalence information, as well as information about converting pointer values under C++ inheritance. For a type of `Foo *`, the type descriptor structure is usually accessed as follows:

> 创建一个新的 Python 指针对象。`ptr` 是要转换的指针，`ty` 是描述该类型的 SWIG 类型描述符结构体，而 `own` 是指示 Python 是否应拥有该指针所有权的标志。
>
> 这两个功能都需要使用特殊的 SWIG 类型描述符结构体。此结构体包含有关数据类型的错误名称，类型等效信息以及有关在 C++ 继承下转换指针值的信息。对于 `Foo *` 类型，通常按以下方式访问类型描述符结构体：

```c++
Foo *f;
if (! SWIG_IsOK( SWIG_ConvertPtr($input, (void **) &f, SWIG TYPE_p_Foo, 0))) {
  SWIG_exception_fail( SWIG_TypeError, "in method '$symname', expecting type Foo");
}

PyObject *obj;
obj = SWIG_NewPointerObj(f, SWIG TYPE_p_Foo, 0);
```

In a typemap, the type descriptor should always be accessed using the special typemap variable `$1_descriptor`. For example:

> 在类型映射中，应该始终使用特殊的类型映射变量 `$1_descriptor` 访问类型描述符。例如：

```
%typemap(in) Foo * {
  if (! SWIG_IsOK( SWIG_ConvertPtr($input, (void **) &$1, $1_descriptor, 0))) {
    SWIG_exception_fail( SWIG_TypeError, "in method '$symname', expecting type Foo");
  }
}
```

If necessary, the descriptor for any type can be obtained using the `$descriptor()` macro in a typemap. For example:

> 如有必要，可以使用类型映射中的 `$descriptor()` 宏获取任何类型的描述符。例如：

```
%typemap(in) Foo * {
  if (! SWIG_IsOK( SWIG_ConvertPtr($input, (void **) &$1, $descriptor(Foo *), 0))) {
    SWIG_exception_fail( SWIG_TypeError, "in method '$symname', expecting type Foo");
  }
}
```

Although the pointer handling functions are primarily intended for manipulating low-level pointers, both functions are fully aware of Python proxy classes. Specifically, `SWIG_ConvertPtr()` will retrieve a pointer from any object that has a `this` attribute. In addition, `SWIG_NewPointerObj()` can automatically generate a proxy class object (if applicable).

> 尽管指针处理功能主要用于操纵低级指针，但两个功能都完全了解 Python 代理类。具体来说，`SWIG_ConvertPtr()` 将从具有 `this` 属性的任何对象中检索指针。另外，`SWIG_NewPointerObj()` 可以自动生成代理类对象（如果适用）。

## 36.10 Docstring 功能

Using docstrings in Python code is becoming more and more important and more tools are coming on the scene that take advantage of them, everything from full-blown documentation generators to class browsers and popup call-tips in Python-aware IDEs. Given the way that SWIG generates the proxy code by default, your users will normally get something like `function_name(*args)` in the popup calltip of their IDE which is next to useless when the real function prototype might be something like this:

> 在 Python 代码中使用 docstring 变得越来越重要，并且越来越多的工具可以利用它们来利用它们，从成熟的文档生成器到类浏览器以及 Python 感知的 IDE 中的弹出式调用提示，无所不包。根据 SWIG 默认生成代理代码的方式，你的用户通常会在其 IDE 的弹出式调用提示中获得类似 `function_name(* args)` 的内容，当实际函数原型可能像这样时，该内容将无效：

```c++
bool function_name(int x, int y, Foo* foo=NULL, Bar* bar=NULL);
```

The features described in this section make it easy for you to add docstrings to your modules, functions and methods that can then be used by the various tools out there to make the programming experience of your users much simpler.

> 本节中描述的功能使你可以轻松地将文档字符串添加到模块，函数和方法中，然后可以被各种工具使用，以使用户的编程体验变得更加简单。

### 36.10.1 模块 `docstring`

Python allows a docstring at the beginning of the `.py` file before any other statements, and it is typically used to give a general description of the entire module. SWIG supports this by setting an option of the `%module` directive. For example:

> Python 允许在 `.py` 文件开头的文档字符串在任何其他语句之前，并且通常用于提供整个模块的一般描述。SWIG 通过设置 `%module` 指令的选项来支持这一点。例如：

```
%module(docstring="This is the example module's docstring") example
```

When you have more than just a line or so then you can retain the easy readability of the `%module` directive by using a macro. For example:

> 如果你不只是一行，那么可以使用宏来保留 `%module` 指令的易读性。例如：

```
%define DOCSTRING
"The `XmlResource` class allows program resources defining menus,
layout of controls on a panel, etc. to be loaded from an XML file."
%enddef

%module(docstring=DOCSTRING) xrc
```

### 36.10.2 `%feature("autodoc")`

As alluded to above SWIG will generate all the function and method proxy wrappers with just "*args" (or "*args, **kwargs" if the -keyword option is used) for a parameter list and will then sort out the individual parameters in the C wrapper code. This is nice and simple for the wrapper code, but makes it difficult to be programmer and tool friendly as anyone looking at the `.py` file will not be able to find out anything about the parameters that the functions accept.

But since SWIG does know everything about the function it is possible to generate a docstring containing the parameter types, names and default values. Since many of the docstring tools are adopting a standard of recognizing if the first thing in the docstring is a function prototype then using that instead of what they found from introspection, then life is good once more.

SWIG's Python module provides support for the "autodoc" feature, which when attached to a node in the parse tree will cause a docstring to be generated that includes the name of the function, parameter names, default values if any, and return type if any. There are also four levels for autodoc controlled by the value given to the feature, `%feature("autodoc", "*level*")`. The four values for *level* are covered in the following sub-sections.

> 如上所述，SWIG 将仅使用参数列表的 `* args`（或 `* args`、`** kwargs`，如果使用 `-keyword` 选项）来生成所有函数和方法代理包装，然后对各个参数进行排序在 C 包装器代码中。对于包装代码来说，这是很好而且很简单的方法，但是却使程序员和工具友好起来变得很困难，因为任何查看 `.py` 文件的人都无法找到有关函数接受的参数的任何信息。
>
> 但是，由于 SWIG 确实了解有关该函数的所有信息，因此可以生成包含参数类型，名称和默认值的文档字符串。由于许多文档字符串工具都采用一种标准，即识别文档字符串中的第一件事是否是函数原型，然后使用它而不是内省的结果，因此生活再好不过了。
>
> SWIG 的 Python 模块提供对“自动文档”功能的支持，该功能附加到解析树中的节点时将导致生成文档字符串，其中包括函数名称，参数名称，默认值（如果有）以及返回类型（如果有） 。`autodoc` 也有四个级别，这些级别由功能值 `%feature("autodoc", "*level*")` 控制。以下各小节将介绍 `level` 的四个值。

#### 36.10.2.1 `%feature("autodoc", "0")`

When level "0" is used then the types of the parameters will *not* be included in the autodoc string. For example, given this function prototype:

> 如果使用级别 `0`，则*自动*字符串中将不包括参数类型。例如，给定此函数原型：

```
%feature("autodoc", "0");
bool function_name(int x, int y, Foo* foo=NULL, Bar* bar=NULL);
```

Then Python code like this will be generated:

> 将产生这样的 Python 代码：

```python
def function_name(*args, **kwargs):
    """function_name(x, y, foo=None, bar=None) -> bool"""
    ...
```

#### 36.10.2.2 `%feature("autodoc", "1")`

When level "1" is used then the parameter types *will* be used in the autodoc string. In addition, an attempt is made to simplify the type name such that it makes more sense to the Python user. Pointer, reference and const info is removed if the associated type is has an associated Python type (`%rename`'s are thus shown correctly). This works most of the time, otherwise a C/C++ type will be used. See the next section for the "docstring" feature for tweaking the docstrings to your liking. Given the example above, then turning on the parameter types with level "1" will result in Python code like this:

> 如果使用级别 `1`，则*将*在自动文档字符串中使用类型的参数。另外，尝试简化类型名称，以使它对 Python 用户更有意义。如果相关的类型具有相关的 Python 类型，则将删除指针、引用和 const 信息（因此正确显示了 `%rename`）。这在大多数情况下都有效，否则将使用 C/C++ 类型。请参阅下一节的“文档字符串”功能，以根据自己的喜好调整文档字符串。给定上面的示例，然后打开级别为 `1` 的参数类型将得到如下 Python 代码：

```python
def function_name(*args, **kwargs):
    """function_name(int x, int y, Foo foo=None, Bar bar=None) -> bool"""
    ...
```

#### 36.10.2.3 `%feature("autodoc", "2")`

Level "2" results in the function prototype as per level "0". In addition, a line of documentation is generated for each parameter using [numpydoc](https://github.com/numpy/numpy/blob/master/doc/HOWTO_DOCUMENT.rst.txt) style. Using the previous example, the generated code will be:

> 级别 `2` 产生了根据级别 `0` 的功能原型。另外，还会使用 [numpydoc](https://github.com/numpy/numpy/blob/master/doc/HOWTO_DOCUMENT.rst.txt) 样式为每个参数生成一行文档。使用前面的示例，生成的代码将是：

```python
def function_name(*args, **kwargs):
    """
    function_name(x, y, foo=None, bar=None) -> bool

    Parameters
    ----------
    x: int
    y: int
    foo: Foo *
    bar: Bar *

    """
    ...
```

Note that the documentation for each parameter is sourced from the "doc" typemap which by default shows the C/C++ type rather than the simplified Python type name described earlier for level "1". Typemaps can of course change the output for any particular type, for example the `int x` parameter:

> 请注意，每个参数的文档均来自 `doc` 类型映射，该映射默认情况下显示 C/C++ 类型，而不是先前为级别 `1` 所述的简化的 Python 类型名称。类型映射当然可以更改任何特定类型的输出，例如 `int x` 参数：

```
%feature("autodoc", "2");
%typemap("doc") int x "$1_name (C++ type: $1_type) -- Input $1_name dimension"
bool function_name(int x, int y, Foo* foo=NULL, Bar* bar=NULL);
```

resulting in

```python
def function_name(*args, **kwargs):
    """
    function_name(x, y, foo=None, bar=None) -> bool

    Parameters
    ----------
    x (C++ type: int) -- Input x dimension
    y: int
    foo: Foo *
    bar: Bar *

    """
```

#### 36.10.2.4 `%feature("autodoc", "3")`

Level "3" results in the function prototype as per level "1" but also contains the same additional line of documentation for each parameter as per level "2". Using our earlier example again, the generated code will be:

> 级别 `3` 产生的函数原型为级别 `1`，但还包含与级别 `2` 相同的每个参数的附加文档行。再次使用前面的示例，生成的代码将是：

```python
def function_name(*args, **kwargs):
    """
    function_name(int x, int y, Foo foo=None, Bar bar=None) -> bool

    Parameters
    ----------
    x: int
    y: int
    foo: Foo *
    bar: Bar *

    """
    ...
```

#### 36.10.2.5 `%feature("autodoc", "docstring")`

Finally, there are times when the automatically generated autodoc string will make no sense for a Python programmer, particularly when a typemap is involved. So if you give an explicit value for the autodoc feature then that string will be used in place of the automatically generated string. For example:

> 最后，有时候自动生成的 autodoc 字符串对于 Python 程序员来说毫无意义，特别是当涉及到类型映射时。因此，如果为 autodoc 功能提供一个明确的值，则将使用该字符串代替自动生成的字符串。例如：

```
%feature("autodoc", "GetPosition() -> (x, y)") GetPosition;
void GetPosition(int* OUTPUT, int* OUTPUT);
```

### 36.10.3 `%feature("docstring")`

In addition to the autodoc strings described above, you can also attach any arbitrary descriptive text to a node in the parse tree with the "docstring" feature. When the proxy module is generated then any docstring associated with classes, function or methods are output. If an item already has an autodoc string then it is combined with the docstring and they are output together. If the docstring is all on a single line then it is output like this:

> 除了上述自动文档字符串之外，你还可以使用“文档字符串”功能将任意描述性文本附加到解析树中的节点上。生成代理模块时，将输出与类，函数或方法关联的任何文档字符串。如果项目已经具有自动文档字符串，则将其与文档字符串合并，然后将它们一起输出。如果文档字符串全部在一行上，则输出如下：

```python
"""This is the docstring"""
```

Otherwise, to aid readability it is output like this:

> 或者增加可读性，则输出：

```python
"""
This is a multi-line docstring
with more than one line.
"""
```

## 36.11 Python 包

Python has concepts of modules and packages. Modules are separate units of code and may be grouped together to form a package. Packages may be nested, that is they may contain subpackages. This leads to tree-like hierarchy, with packages as intermediate nodes and modules as leaf nodes.

The hierarchy of Python packages/modules follows the hierarchy of `*.py` files found in a source tree (or, more generally, in the Python path). Normally, the developer creates new module by placing a `*.py` file somewhere under Python path; the module is then named after that `*.py` file. A package is created by placing an `__init__.py` file within a directory; the package is then named after that directory. For example, the following source tree:

> Python 具有模块和包的概念。模块是单独的代码单元，可以组合在一起形成一个包。程序包可能是嵌套的，即它们可能包含子程序包。这导致树状层次结构，其中包作为中间节点，而模块作为叶节点。
>
> Python 包/模块的层次结构遵循在源树（或更常见的是在 Python 路径中）中找到的 `*.py` 文件的层次结构。通常，开发人员通过在 Python 路径下的某个位置放置一个 `*.py` 文件来创建新模块；然后，该模块以该 `*.py` 文件命名。通过在目录中放置一个 `__init__.py` 文件来创建一个包。然后，以该目录命名该软件包。例如，以下源代码树：

```
mod1.py
pkg1/__init__.py
pkg1/mod2.py
pkg1/pkg2/__init__.py
pkg1/pkg2/mod3.py
```

defines the following Python packages and modules:

> 定义了以下 Python 包和模块：

```
pkg1            # package
pkg1.pkg2       # package
mod1            # module
pkg1.mod2       # module
pkg1.pkg2.mod3  # module
```

The purpose of an `__init__.py` file is two-fold. First, the existence of`__init__.py` in a directory informs the Python interpreter that this directory contains a Python package. Second, the code in `__init__.py` is loaded/executed automatically when the package is initialized (when it or its submodule/subpackage gets `import`'ed). By default, SWIG generates proxy Python code – one `*.py` file for each `*.i` interface. The `__init__.py` files, however, are not generated by SWIG . They should be created by other means. Both files (module `*.py` and `__init__.py`) should be installed in appropriate destination directories in order to obtain a desirable package/module hierarchy.

Python3 adds another option for packages with [PEP 0420](https://www.python.org/dev/peps/pep-0420/) (implicit namespace packages). Implicit namespace packages no longer use `__init__.py` files. SWIG generated Python modules support implicit namespace packages. See [36.11.5 Implicit Namespace Packages](http://www.SWIG.org/Doc3.0/Python.html#Python_implicit_namespace_packages) for more information.

If you place a SWIG generated module into a Python package then there are details concerning the way SWIG [searches for the wrapper module](http://www.SWIG.org/Doc3.0/Python.html#Python_package_search) that you may want to familiarize yourself with.

The way Python defines its modules and packages impacts SWIG users. Some users may need to use special features such as the `package` option in the `%module` directive or import related command line options. These are explained in the following sections.

> `__init__.py` 文件的目的是双重的。首先，目录中存在 `__init__.py` 通知 Python 解释器该目录包含 Python 包。其次，初始化包时（当它或其子模块/子包获得导入时），将自动加载/执行 `__init__.py` 中的代码。默认情况下，SWIG 生成代理 Python 代码——每个 `*.i` 接口一个 `* .py` 文件。但是，`__init__.py` 文件不是由 SWIG 生成的。它们应该通过其他方式创建。这两个文件（模块 `*.py` 和 `__init__.py`）都应安装在适当的目标目录中，以便获得所需的包/模块层次结构。
>
> Python3 为 [PEP 0420](https://www.python.org/dev/peps/pep-0420/) 的软件包（隐式命名空间软件包）添加了另一个选项。隐式命名空间包不再使用 `__init__.py` 文件。SWIG 生成的 Python 模块支持隐式命名空间包。有关更多信息，请参见 [36.11.5 隐式命名空间包](http://www.SWIG.org/Doc3.0/Python.html#Python_implicit_namespace_packages)。
>
> 如果将 SWIG 生成的模块放入 Python 包中，那么可能会有关于 SWIG [搜索包装器模块](http://www.SWIG.org/Doc3.0/Python.html#Python_package_search)的方式的详细信息想熟悉一下。
>
> Python 定义其模块和软件包的方式会影响 SWIG 用户。一些用户可能需要使用特殊功能，例如 `%module` 指令中的 `package` 选项或导入相关的命令行选项。这些将在以下各节中说明。

### 36.11.1 设置 Python 包

Using the `package` option in the `%module` directive allows you to specify a Python package that the module will be in when installed.

> 使用 `%module` 指令中的 `package` 选项，可以指定安装模块时将在其中的 Python 软件包。

```
%module(package="wx") xrc
```

This is useful when the `.i` file is `%import`ed by another `.i` file. By default SWIG will assume that the importer is able to find the importee with just the module name, but if they live in separate Python packages then this won't work. However if the importee specifies what its package is with the `%module` option then the Python code generated for the importer will use that package name when importing the other module and in base class declarations, etc..

SWIG assumes that the `package` option provided to `%module` together with the `module` name (that is, `wx.xrc` in the above example) forms a fully qualified (absolute) name of a module (in Python terms). This is important especially for Python 3, where absolute imports are used by default. It's up to you to place the generated module files (`.py`, `.so`) in appropriate subdirectories. For example, if you have an interface file `foo.i` with:

> 当 `.i` 文件由另一个 `.i` 文件进行导入时，这很有用。默认情况下，SWIG 将假定导入程序仅使用模块名称即可找到该导入对象，但是如果它们位于单独的 Python 包中，则此方法将无效。但是，如果导入对象使用 `%module` 选项指定了其软件包，那么为导入器生成的 Python 代码将在导入其他模块时以及在基类声明等中使用该软件包名称。
>
> SWIG 假定提供给 `%module` 的 `package` 选项和模块名（在上面的示例中为 `wx.xrc`）一起构成了模块的完全合格（绝对）名称（使用 Python 术语） ）。这对于 Python 3 特别重要，在 Python 3 中默认使用绝对导入。你可以将生成的模块文件（`.py`，`.so`）放在适当的子目录中。例如，如果你有一个接口文件 `foo.i`，其中包含：

```
%module(package="pkg1.pkg2") foo
```

then the resulting directory layout should be

> 然后结果目录将是

```
pkg1/
pkg1/__init__.py
pkg1/pkg2/__init__.py
pkg1/pkg2/foo.py        # (generated by SWIG)
pkg1/pkg2/_foo.so       # (shared library built from C/C++ code generated by SWIG)
```

### 36.11.2 绝对与相对导入

Suppose, we have the following hierarchy of files:

> 假设我们有如下架构的文件：

```
pkg1/
pkg1/__init__.py
pkg1/mod2.py
pkg1/pkg2/__init__.py
pkg1/pkg2/mod3.py
```

Let the contents of `pkg1/pkg2/mod3.py` be

> `pkg1/pkg2/mod3.py` 的内容是

```python
class M3: pass
```

We edit `pkg1/mod2.py` and want to import module `pkg1/pkg2/mod3.py` in order to derive from class `M3`. We can write appropriate Python code in several ways, for example:

1. Using `import <>` syntax with absolute package name:

> 我们编辑 `pkg1/mod2.py`，并想导入模块 `pkg1/pkg2/mod3.py`，以便从类 `M3` 派生。我们可以通过几种方式编写适当的 Python 代码，例如：
>
> 1. 使用 `import <>` 语法与包的绝对名字：

```python
# pkg1/mod2.py
import pkg1.pkg2.mod3
class M2(pkg1.pkg2.mod3.M3): 
    pass
```

2. Using `import <>` syntax with package name relative to `pkg1` (only in Python 2.7 and earlier):

> 2. 使用 `import <>` 语法与相对于 `pkg1` 的包名字（仅在 Python 2.7 及更早版本中）：

```python
# pkg1/mod2.py
import pkg2.mod3
class M2(pkg2.mod3.M3): 
    pass
```

3. Using `from <> import <>` syntax (relative import syntax, only in Python 2.5 and later):

> 3. 使用 `from <> import <>` 语法（相对导入语法，仅在 Python 2.5 及更早版本中）：

```python
# pkg1/mod2.py
from .pkg2 import mod3
class M2(mod3.M3): pass
```

4. Other variants, for example the following construction in order to have the `pkg2.mod3.M3` symbol available in `mod2` as in point 2 above (but now under Python 3):

> 4. 其他变体，例如以下构造旨在使 `mod2` 中 `pkg2.mod3.M3` 是可用的，像之前的第 2 点（但现在在 Python 3 中）：

```python
# pkg1/mod2.py
from . import pkg2
from .pkg2 import mod3
class M2(pkg2.mod3.M3): pass
```

Now suppose we have `mod2.i` with

> 现在，假设我们有 `mod2.i`

```
// mod2.i
%module (package="pkg1") mod2
%import "mod3.i"
// ...
```

and `mod3.i` with

> 与 `mod3.i`

```
// mod3.i
%module (package="pkg1.pkg2") mod3
// ...
```

By default, SWIG would generate `mod2.py` proxy file with `import` directive as in point 1. This can be changed with the `-relativeimport` command line option. The `-relativeimport` instructs SWIG to organize imports as in point 2 (for Python < 2.7.0) or as in point 4 for Python 2.7.0 and newer. This is a check done at the time the module is imported. In short, if you have `mod2.i` and `mod3.i` as above, then without `-relativeimport` SWIG will write

> 默认情况下，SWIG 会使用第 1 点中的 `import` 指令生成 `mod2.py` 代理文件。这可以通过 `-relativeimport` 命令行选项进行更改。`-relativeimport` 指示 SWIG 按照第 2 点（对于 Python < 2.7.0）或第 4 点（对于 Python 2.7.0 及更高版本）组织导入。这是在导入模块时进行的检查。简而言之，如果你具有上述的 `mod2.i` 和 `mod3.i`，那么在没有 `-relativeimport` 的情况下，SWIG 会将

```python
import pkg1.pkg2.mod3
```

to `mod2.py` proxy file, and with `-relativeimport` it will write

> 写进 `mod2.py` 代理文件，如果有 `-relativeimport` 它会写

```python
from sys import version_info
if version_info >= (2, 7, 0):
    from . import pkg2
    import pkg1.pkg2.mod3
else:
    import pkg2.mod3
del version_info
```

You should avoid using relative imports and use absolute ones whenever possible. There are some cases, however, when relative imports may be necessary. The first example is, when some (legacy) Python code refers entities imported by proxy files generated by SWIG , and it assumes that the proxy file uses relative imports. Second case is, when one puts import directives in `__init__.py` to import symbols from submodules or subpackages and the submodule depends on other submodules (discussed later).

> 你应避免使用相对导入，并尽可能使用绝对导入。但是，在某些情况下，可能需要相对导入。第一个示例是，当某些（旧式）Python 代码引用由 SWIG 生成的代理文件导入的实体时，它假定代理文件使用相对导入。第二种情况是，当将导入指令放在 `__init__.py` 中以从子模块或子包中导入符号，并且该子模块依赖于其他子模块时（稍后讨论）。

### 36.11.3 增强绝对导入语义

As you may know, there is an incompatibility in import semantics (for the`import <>` syntax) between Python 2 and 3. In Python 2.4 and earlier it is not clear whether

> 如你所知，Python 2 和 3 之间的导入语义（对于 `import <>` 语法）不兼容。在 Python 2.4 和更低版本中，尚不清楚是否

```python
import foo
```

refers to a top-level module or to another module inside the current package. In Python 3 it always refers to a top-level module (see [PEP 328](https://www.python.org/dev/peps/pep-0328/)). To instruct Python 2.5 through 2.7 to use new semantics (that is `import foo` is interpreted as absolute import), one has to put the following line

> 指的是顶层模块或当前包内的另一个模块。在 Python 3 中，它始终是指顶级模块（请参阅 [PEP 328](https://www.python.org/dev/peps/pep-0328/)）。为了指示 Python 2.5 到 2.7 使用新的语义（即，`import foo` 被解释为绝对导入），必须放置以下行

```python
from __future__ import absolute_import
```

at the very beginning of his proxy `*.py` file. In SWIG , it may be accomplished with `%pythonbegin` directive as follows:

> 在它的代理服务器 `*.py` 文件的开头。在 SWIG 中，可以通过 `%pythonbegin` 指令完成，如下所示：

```
%pythonbegin %{
from __future__ import absolute_import
%}
```

### 36.11.4 从 `__init__.py` 导入

Imports in `__init__.py` are handy when you want to populate a package's namespace with names imported from other modules. In SWIG based projects this approach may also be used to split large pieces of code into smaller modules, compile them in parallel and then re-assemble everything at runtime by importing submodules' contents in `__init__.py`, for example.

Unfortunately import directives in `__init__.py` may cause problems, especially if they refer to a package's submodules. This is caused by the way Python initializes packages. If you spot problems with imports from `__init__.py` try using `-relativeimport` option. Below we explain in detail one issue, for which the `-relativeimport` workaround may be helpful.

Consider the following example (Python 3):

> 当你想使用从其他模块导入的名称填充包的命名空间时，在 `__init__.py` 中的导入很方便。在基于 SWIG 的项目中，这种方法也可以用于将大段代码拆分为较小的模块，并行编译它们，然后在运行时通过将子模块的内容导入到 `__init__.py` 中来重新组装所有内容。
>
> 不幸的是，`__init__.py` 中的 `import` 指令可能会引起问题，尤其是当它们引用包的子模块时。这是由 Python 初始化程序包的方式引起的。如果你发现从 `__init__.py` 导入的问题，请尝试使用 `-relativeimport` 选项。下面我们详细解释一个问题，`-relativeimport` 解决方法可能会对你有所帮助。
>
> 考虑以下示例（Python 3）：

```
pkg1/__init__.py        # (empty)
pkg1/pkg2/__init__.py   # (imports something from bar.py)
pkg1/pkg2/foo.py
pkg1/pkg2/bar.py        # (imports foo.py)
```

If the file contents are:

> 如果文件内容是：

- `pkg1/pkg2/__init__.py:`

```python
# pkg1/pkg2/__init__.py
from .bar import Bar
```

- `pkg1/pkg2/foo.py:`

```python
# pkg1/pkg2/foo.py
class Foo: pass
```

- `pkg1/pkg2/bar.py:`

```python
# pkg1/pkg2/bar.py
import pkg1.pkg2.foo
class Bar(pkg1.pkg2.foo.Foo): pass
```

Now if one simply used `import pkg1.pkg2`, it will usually fail:

> 现在，如果有人简单使用 `import pkg1.pkg2`，通常会失败：

```python
>>> import pkg1.pkg2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "./pkg1/pkg2/__init__.py", line 2, in <module>
    from .bar import Bar
  File "./pkg1/pkg2/bar.py", line 3, in <module>
    class Bar(pkg1.pkg2.foo.Foo): pass
AttributeError: 'module' object has no attribute 'pkg2'
```

Surprisingly, if we execute the `import pkg1.pkg2` directive for the second time, it succeeds. The reason seems to be following: when Python spots the `from .bar import Bar` directive in `pkg1/pkg2/__init__.py` it starts loading `pkg1/pkg2/bar.py`. This module imports `pkg1.pkg2.foo` in turn and tries to use `pkg1.pkg2.foo.Foo`, but the package `pkg1` is not fully initialized yet (the initialization procedure is actually in progress) and it seems like the effect of the already seen `import pkg1.pkg2.pkg3.foo` is "delayed" or ignored. Exactly the same may happen to a proxy module generated by SWIG .

One workaround for this case is to use a relative import in `pkg1/pkg2/bar.py`. If we change `bar.py` to be:

> 令人惊讶的是，如果我们第二次执行 `import pkg1.pkg2` 指令，则该指令成功。原因似乎如下：当 Python 在 `pkg1/pkg2/__ init__.py` 中发现 `from .bar import Bar` 指令时，它开始加载 `pkg1/pkg2/bar.py`。这个模块依次导入 `pkg1.pkg2.foo` 并尝试使用 `pkg1.pkg2.foo.Foo`，但是包 `pkg1` 尚未完全初始化（初始化过程实际上正在进行中），看起来像已经看到的 `import pkg1.pkg2.pkg3.foo` 的效果被延迟或忽略。SWIG 生成的代理模块可能完全相同。
>
> 这种情况的一种解决方法是在 `pkg1/pkg2/bar.py` 中使用相对导入。如果我们将 `bar.py` 更改为：

```python
from .pkg3 import foo
class Bar(foo.Foo): pass
```

or

> 或

```python
from . import pkg3
from .pkg3 import foo
class Bar(pkg3.foo.Foo): pass
```

then the example works again. With SWIG , you need to enable the `-relativeimport` option in order to have the above workaround in effect (note, that the Python 2 case also needs the `-relativeimport` workaround).

> 然后该示例再次起作用。使用 SWIG，你需要启用 `-relativeimport` 选项才能生效上述解决方法（请注意，Python 2 案例也需要 `-relativeimport` 解决方法）。

### 36.11.5 隐式命名空间包

Python 3.3 introduced [PEP 0420](https://www.python.org/dev/peps/pep-0420/) which implements implicit namespace packages. In a nutshell, implicit namespace packages remove the requirement of an __init__.py file and allow packages to be split across multiple PATH elements. For example:

> Python 3.3 引入了 [PEP 0420](https://www.python.org/dev/peps/pep-0420/)，它实现了隐式命名空间包。简而言之，隐式命名空间包消除了 `__init__.py` 文件的要求，并允许将包拆分为多个 PATH 元素。例如：

```
/fragment1/pkg1/mod1.py
/fragment2/pkg1/mod2.py
/fragment3/pkg1/mod3.py
```

If PYTHONPATH is set to "/fragment1:/fragment2:/fragment3", then mod1, mod2 and mod3 will be part of pkg1. This allows for splitting of packages into separate pieces. This can be useful for SWIG generated wrappers in the following way.

Suppose you create a SWIG wrapper for a module called robin. The SWIG generated code consists of two files robin.py and _robin.so. You wish to make these modules part of a subpackage (brave.sir). With implicit namespace packages you can place these files in the following configurations:

Using PYTHONPATH="/some/path"

> 如果将 PYTHONPATH 设置为 `/fragment1:/fragment2:/fragment3`，则 `mod1`、`mod2` 和 `mod3` 将成为 `pkg1` 的一部分。这允许将包装分成独立的部分。通过以下方式，这对于 SWIG 生成的包装器很有用。
>
> 假设你为名为 `robin` 的模块创建了 SWIG 包装器。SWIG 生成的代码包含两个文件 `robin.py` 和 `_robin.so`。你希望使这些模块成为子包（`brave.sir`）的一部分。使用隐式命名空间包，可以将这些文件置于以下配置中：
> 
> 使用 `PYTHONPATH="/some/path"`

```
/some/path/brave/sir/robin.py
/some/path/brave/sir/_robin.so
```

Using PYTHONPATH="/some/path:/some/other/path"

> 使用 `PYTHONPATH="/some/path:/some/other/path"`
 
```
/some/path/brave/sir/robin.py
/some/other/path/brave/sir/_robin.so
```

Finally suppose that your pure python code is stored in a .zip file or some other way (database, web service connection, etc). Python can load the robin.py module using a custom importer. But the _robin.so module will need to be located on a file system. Implicit namespace packages make this possible. For example, using PYTHONPATH="/some/path/foo.zip:/some/other/path"

Contents of foo.zip

> 最后，假设你的纯 Python 代码存储在 `.zip` 文件中或其他某种方式（数据库，Web服务连接等）中。Python 可以使用自定义导入器加载 `robin.py` 模块。但是 `_robin.so` 模块将需要位于文件系统上。隐式命名空间包使这成为可能。例如，使用 `PYTHONPATH="/some/path/foo.zip:/some/other/path"`
>
> `foo.zip` 的内容

```
brave/
brave/sir/
brave/sir/robin.py
```

File system contents

> 文件系统内容

```
/some/other/path/brave/sir/_robin.so
```

Support for implicit namespace packages was added to python-3.3. The zipimporter requires python-3.5.1 or newer to work with subpackages.

**Compatibility Note:** Support for implicit namespace packages was added in SWIG -3.0.9.

> 对隐式命名空间包的支持已添加到 python-3.3 中。`zipimporter` 需要 python-3.5.1 或更高版本才能使用子包。
>
> **兼容性说明**：在 SWIG -3.0.9 中添加了对隐式命名空间包的支持。

### 36.11.6 搜索包装模块

When SWIG creates wrappers from an interface file, say foo.i, two Python modules are created. There is a pure Python module module (foo.py) and C/C++ code which is built and linked into a dynamically (or statically) loaded low-level module _foo (see the [Preliminaries section](http://www.SWIG.org/Doc3.0/Python.html#Python_nn3) for details). So, the interface file really defines two Python modules. How these two modules are loaded is covered next.

The pure Python module needs to load the C/C++ module in order to link to the wrapped C/C++ methods. To do this it must make some assumptions about what package the C/C++ module may be located in. The approach the pure Python module uses to find the C/C++ module is as follows:
1. The pure Python module, foo.py, tries to load the C/C++ module, _foo, from the same package foo.py is located in. The package name is determined from the `__name__` attribute given to foo.py by the Python loader that imported foo.py. If foo.py is not in a package then _foo is loaded as a global module.
2. If the above import of_foo results in an ImportError being thrown, then foo.py makes a final attempt to load _foo as a global module.

The Python code implementing the loading logic described above is quite complex to handle multiple versions of Python, but it can be replaced with custom code. This is not recommended unless you understand the full intricacies of importing Python modules. The custom code can be specified by setting the `moduleimport`option of the `%module` directive with the appropriate import code. For example:

> 当 SWIG 从接口文件（例如 `foo.i`）创建包装器时，将创建两个 Python 模块。有一个纯 Python 模块模块（`foo.py`）和 C/C++ 代码，它们被构建并链接到动态（或静态）加载的低级模块 `_foo` 中（请参阅[预备知识](http：//www.SWIG.org/Doc3.0/Python.tml#Python_nn3)部分）。因此，接口文件确实定义了两个 Python 模块。接下来介绍如何加载这两个模块。
>
> 纯 Python 模块需要加载 C/C++ 模块才能链接到包装的 C/C++ 方法。为此，必须对 C/C++ 模块可能位于的包进行一些假设。纯 Python 模块用来查找 C/C++ 模块的方法如下：
> 1.纯 Python 模块 `foo.py` 尝试从 `foo.py` 所在的同一程序包中加载 C/C++ 模块 `_foo`。程序包名称由 `foo.py` 提供的 `__name__` 属性确定导入 `foo.py` 的 Python 加载器。如果 `foo.py` 不在软件包中，则 `_foo` 作为全局模块加载。
> 2.如果以上导入的 `_foo` 导致引发 `ImportError`，则 `foo.py` 最终尝试将 `_foo` 加载为全局模块。
>
> 实现上述加载逻辑的 Python 代码非常复杂，无法处理多个版本的 Python，但可以用自定义代码替换。除非你了解导入 Python 模块的全部复杂性，否则不建议这样做。可以通过使用适当的导入代码设置 `%module` 指令的 `moduleimport` 选项来指定自定义代码。例如：

```
%module(moduleimport="import _foo") foo
```

The special variable `$module` will also be expanded into the low-level C/C++ module name, `_foo` in the case above. When you have more than just a line or so then you can retain the easy readability of the `%module` directive by using a macro. For example:

> 在上述情况下，特殊变量 `$module` 也将扩展为低级 C/C++ 模块名称 `_foo`。如果你不只是一行，那么可以使用宏来保留 `%module` 指令的易读性。例如：

```
%define MODULEIMPORT
"
print 'Loading low-level module $module'
import $module
print 'Module has loaded'
"
%enddef

%module(moduleimport=MODULEIMPORT) foo
```

Now let's consider an example using the SWIG default loading logic. Suppose foo.i is compiled into foo.py and _foo.so. Assuming /dir is on PYTHONPATH, then the two modules can be installed and used in the following ways:

> 现在，让我们考虑一个使用 SWIG 默认加载逻辑的示例。假设 `foo.i` 被编译为 `foo.py` 和 `_foo.so`。假设 `/dir` 在 PYTHONPATH 上，则可以通过以下方式安装和使用这两个模块：

#### 36.11.6.1 同一个包中的两个模块

Both modules are in one package:

> 两个模块在同一个包中：

```
/dir/package/foo.py
/dir/package/__init__.py
/dir/package/_foo.so
```

And imported with

> 如下载入

```python
from package import foo
```

#### 36.11.6.2 切分模块

The pure python module is in a package and the C/C++ module is global:

> 纯 Python 模块在一个包中并且 C/C++ 模块是全局的：

```
/dir/package/foo.py
/dir/package/__init__.py
/dir/_foo.so
```

And imported with

> 如下载入

```python
from package import foo
```

#### 36.11.6.3 两个模块皆为全局

Both modules are global:

> 两个模块皆为全局：

```
/dir/foo.py
/dir/_foo.so
```

And imported with

> 如下载入

```python
import foo
```

If_foo is statically linked into an embedded Python interpreter, then it may or may not be in a Python package. This depends in the exact way the module was loaded statically. The above search order will still be used for statically loaded modules. So, one may place the module either globally or in a package as desired.

> 如果 `_foo` 被静态链接到嵌入式 Python 解释器中，则它可能在 Python 包中，也可能不在。这取决于模块静态加载的确切方式。上面的搜索顺序仍将用于静态加载的模块。因此，可以根据需要将模块全局放置或打包放置。

#### 36.11.6.4 静态链接 C 模块

It is strongly recommended to use dynamically linked modules for the C portion of your pair of Python modules. If for some reason you still need to link the C module of the pair of Python modules generated by SWIG into your interpreter, then this section provides some details on how this impacts the pure Python modules ability to locate the other part of the pair. Please also see the [Static Linking](http://www.SWIG.org/Doc3.0/Python.html#Python_nn8) section.

When Python is extended with C code the Python interpreter needs to be informed about details of the new C functions that have been linked into the executable. The code to do this is created by SWIG and is automatically called in the correct way when the module is dynamically loaded. However when the code is not dynamically loaded (because it is statically linked) Then the initialization method for the module created by SWIG is not called automatically and the Python interpreter has no idea that the new SWIG C module exists.

Before Python 3, one could simply call the init method created by SWIG which would have normally been called when the shared object was dynamically loaded. The specific name of this method is not given here because statically linked modules are not encouraged with SWIG ([Static Linking](http://www.SWIG.org/Doc3.0/Python.html#Python_nn8)). However one can find this init function in the C file generated by SWIG .

If you are really keen on static linking there are two ways to initialize the SWIG generated C module with the init method. Which way you use depends on what version of Python your module is being linked with. Python 2 and Python 3 treat this init function differently. And the way they treat it affects how the pure Python module will be able to locate the C module.

The details concerning this are covered completly in the documentation for Python itself. Links to the relavent sections follow:
- [Extending in python2](https://docs.python.org/2/extending/extending.html#methodtable)
- [Extending in python3](https://docs.python.org/3.6/extending/extending.html#the-module-s-method-table-and-initialization-function)

There are two keys things to understand. The first is that in Python 2 the init() function returns void. In Python 3 the init() function returns a PyObject * which points to the new module. Secondly, when you call the init() method manually, you are the Python importer. So, you determine which package the C module will be located in.

So, if you are using Python 3 it is important that you follow what is described in the Python documentation linked above. In particular, you can't simply call the init() function generated by SWIG and cast the PyObject pointer it returns over the side. If you do then Python 3 will have no idea that your C module exists and the pure Python half of your wrapper will not be able to find it. You need to register your module with the Python interpreter as described in the Python docs.

With Python 2 things are somewhat more simple. In this case the init function returns void. Calling it will register your new C module as a **global** module. The pure Python part of the SWIG wrapper will be able to find it because it tries both the pure Python module it is part of and the global module. If you wish not to have the statically linked module be a global module then you will either need to refer to the Python documentation on how to do this (remember you are now the Python importer) or use dynamic linking.

> 强烈建议对 Python 模块对应的 C 部分使用动态链接的模块。如果由于某种原因你仍然需要将 SWIG 生成的一对 Python 模块中的 C 模块链接到你的解释器中，那么本节将提供一些详细信息如何影响纯 Python 模块的能力来定位配对中的另一部分。另请参见[静态链接](http://www.SWIG.org/Doc3.0/Python.html#Python_nn8)部分。
>
> 使用 C 代码扩展 Python 后，需要通知 Python 解释器有关已链接到可执行文件中的 C 函数的详细信息。执行此操作的代码由 SWIG 创建，并在动态加载模块时以正确的方式自动调用该代码。但是，如果代码不是动态加载的（因为它是静态链接的），则不会自动调用 SWIG 创建的模块的初始化方法，Python 解释器也不知道新的 SWIG C 模块存在。
>
> 在 Python 3 之前，可以简单地调用 SWIG 创建的 `init` 方法，该方法通常在动态加载共享对象时被调用。此处未提供此方法的具体名称，因为 SWIG 不鼓励使用静态链接的模块（[静态链接](http://www.SWIG.org/Doc3.0/Python.html#Python_nn8)）。但是，可以在 SWIG 生成的 C 文件中找到此初始化函数。
>
> 如果你真的热衷于静态链接，则有两种方法可以使用 `init` 方法初始化 SWIG 生成的 C 模块。使用哪种方式取决于要与模块链接的 Python 版本。Python 2 和 Python 3 对此 `init` 函数的处理方式有所不同。它们对待它的方式会影响纯 Python 模块将如何定位 C 模块。
>
> 有关此方面的详细信息，在 Python 本身的文档中已全面介绍。指向相关部分的链接如下：
> - [Extending in python2](https://docs.python.org/2/extending/extending.html#methodtable)
> - [Extending in python3](https://docs.python.org/3.6/extending/extending.html#the-module-s-method-table-and-initialization-function)
>
> 有两个关键的事情要理解。首先是在 Python 2 中，`init()` 函数返回 `void`。在 Python 3 中，`init()` 函数返回一个 `PyObject *`，它指向新模块。其次，当你手动调用 `init()` 方法时，你就是 Python 导入器。因此，你可以确定 C 模块将位于哪个软件包中。
>
> 因此，如果你使用的是 Python 3，请务必遵循上面链接的 Python 文档中所述的内容。特别是，你不能简单地调用 SWIG 生成的 `init()` 函数并将其返回的 `PyObject` 指针强制转换为侧面。如果这样做，Python 3 将不知道 C 模块的存在，并且包装的纯 Python 部分将无法找到它。你需要按照 Python 文档中的说明向 Python 解释器注册模块。
>
> 使用 Python 2，事情会更简单。在这种情况下，`init` 函数将返回 `void`。调用它会将你的新 C 模块注册为 **global** 模块。SWIG 包装程序的纯 Python 部分将能够找到它，因为它会尝试同时包含它的纯 Python 模块和全局模块。如果你不希望将静态链接的模块作为全局模块，则需要参考 Python 文档以了解如何执行此操作（请记住你现在是 Python 导入者）或使用动态链接。

## 36.12 Python 3 支持

SWIG is able to support Python 3.0. The wrapper code generated by SWIG can be compiled with both Python 2.x or 3.0. Further more, by passing the `-py3` command line option to SWIG , wrapper code with some Python 3 specific features can be generated (see below subsections for details of these features). The `-py3`option also disables some incompatible features for Python 3, such as `-classic`.

There is a list of known-to-be-broken features in Python 3:
- No more support for FILE* typemaps, because PyFile_AsFile has been dropped in Python 3.
- The `-apply` command line option is removed and generating code using apply() is no longer supported.

The following are Python 3.0 new features that are currently supported by SWIG .

> SWIG 能够支持 Python 3.0。SWIG 生成的包装器代码可以使用 Python 2.x 或 3.0 进行编译。此外，通过将 `-py3` 命令行选项传递给 SWIG，可以生成具有某些 Python 3 特定功能的包装器代码（有关这些功能的详细信息，请参见以下小节）。`-py3` 选项还禁用了 Python 3 的某些不兼容功能，例如 `-classic`。
>
> Python 3 中列出了一系列已知的功能：
> - 不再支持 `FILE *` 类型映射，因为 `PyFile_AsFile` 已在 Python 3 中删除。
> - 删除了 `-apply` 命令行选项，并且不再支持使用 `apply()` 生成代码。
>
> 以下是 SWIG 当前支持的 Python 3.0 新功能。

### 36.12.1 函数标注

The `-py3` option will enable function annotation support. When used SWIG is able to generate proxy method definitions like this:

> `-py3` 选项将启用功能注释支持。使用 SWIG 时，可以生成如下的代理方法定义：

```python
def foo(self, bar : "int"=0) -> "void" : ...
```

Also, even if without passing SWIG the `-py3` option, the parameter list still could be generated:

> 同样，即使没有通过 SWIG 的 `-py3` 选项，仍然可以生成参数列表：

```python
def foo(self, bar=0): ...
```

But for overloaded function or method, the parameter list would fallback to `*args` or `self, *args`, and `**kwargs` may be append depend on whether you enabled the keyword argument. This fallback is due to all overloaded functions share the same function in SWIG generated proxy class.

For detailed usage of function annotation, see [PEP 3107](https://www.python.org/dev/peps/pep-3107/).

> 但是对于重载的函数或方法，参数列表将回退到 `*args` 或 `self, *args` 和 `**kwargs`，取决于你是否启用了关键字参数。此后退是由于所有重载函数在 SWIG 生成的代理类中共享相同的函数。
> 
> 函数标注的详细信息参见 [PEP 3107](https://www.python.org/dev/peps/pep-3107/)。

### 36.12.2 缓冲区接口

Buffer protocols were revised in Python 3. SWIG also gains a series of new typemaps to support buffer interfaces. These typemap macros are defined in `pybuffer.i`, which must be included in order to use them. By using these typemaps, your wrapped function will be able to accept any Python object that exposes a suitable buffer interface.

For example, the `get_path()` function puts the path string into the memory pointed to by its argument:

> 缓冲区协议已在 Python 3 中进行了修订。SWIG 还获得了一系列新的类型映射，以支持缓冲区接口。这些类型映射宏在 `pybuffer.i` 中定义，必须使用它们才能使用它们。通过使用这些类型映射，你的包装函数将能够接受任何公开适当缓冲区接口的 Python 对象。
>
> 例如，`get_path()` 函数将路径字符串放入其参数所指向的内存中：

```c++
void get_path(char *s);
```

Then you can write a typemap like this: (the following example is applied to both Python 3.0 and 2.6, since the `bytearray` type is backported to 2.6).

> 然后，你可以编写这样的类型映射：（以下示例同时适用于 Python 3.0 和 2.6，因为 `bytearray` 类型已反向移植到 2.6）。

```
%include <pybuffer.i>
%pybuffer_mutable_string(char *str);
void get_path(char *s);
```

And then on the Python side the wrapped `get_path` could be used in this way:

> 然后在 Python 端，可以通过这种方式使用包装的 `get_path`：

```python
>>> p = bytearray(10)
>>> get_path(p)
>>> print(p)
bytearray(b'/Foo/Bar/\x00')
```

The macros defined in `pybuffer.i` are similar to those in `cstring.i`:

> `pybuffer.i` 中的宏定义类似于 `cstring.i`：

**`%pybuffer_mutable_binary(parm, size_parm)`**

The macro can be used to generate a typemap which maps a buffer of an object to a pointer provided by `parm` and a size argument provided by `size_parm`. For example:

> 宏可用于生成类型映射，该映射将对象的缓冲区映射到由 `parm` 提供的指针和由 `size_parm` 提供的 `size` 参数。例如：

```
%pybuffer_mutable_binary(char *str, size_t size);
...
int snprintf(char *str, size_t size, const char *format, ...);
```

In Python:

> 在 Python 中：

```python
>>> buf = bytearray(6)
>>> snprintf(buf, "Hello world!")
>>> print(buf)
bytearray(b'Hello\x00')
>>>
```

**`%pybuffer_mutable_string(parm)`**

This typemap macro requires the buffer to be a zero terminated string, and maps the pointer of the buffer to `parm`. For example:

> 这个类型映射要求缓冲区必须是一个零终止的字符串，并将缓冲区的指针映射到 `parm`。例如：

```
%pybuffer_mutable_string(char *str);
...
size_t make_upper(char *str);
```

In Python:

> 在 Python 中：

```python
>>> buf = bytearray(b'foo\x00')
>>> make_upper(buf)
>>> print(buf)
bytearray(b'FOO\x00')
>>>
```

Both `%pybuffer_mutable_binary` and `%pybuffer_mutable_string`require the provided buffer to be mutable, eg. they can accept a `bytearray` type but can't accept an immutable `byte` type.

> `%pybuffer_mutable_binary` 和 `%pybuffer_mutable_string` 都要求提供的缓冲区是可变的，例如。它们可以接受 `bytearray` 类型，但不能接受不可变的 `byte` 类型。

**`%pybuffer_binary(parm, size_parm)`**

This macro maps an object's buffer to a pointer `parm` and a size `size_parm`. It is similar to `%pybuffer_mutable_binary`, except the`%pybuffer_binary` an accept both mutable and immutable buffers. As a result, the wrapped function should not modify the buffer.

> 该宏将对象的缓冲区映射到指针 `parm` 和大小 `size_parm`。它与 `%pybuffer_mutable_binary` 类似，除了 `%pybuffer_binary` 接受可变和不可变的缓冲区。因此，包装函数不应修改缓冲区。

**`%pybuffer_string(parm)`**

This macro maps an object's buffer as a string pointer `parm`. It is similar to `%pybuffer_mutable_string` but the buffer could be both mutable and immutable. And your function should not modify the buffer.

> 该宏将对象的缓冲区映射为字符串指针 `parm`。它与 `%pybuffer_mutable_string` 类似，但是缓冲区可能是既是可变的又是不可变的。而且你的函数不应修改缓冲区。

### 36.12.3 抽象基类

By including `pyabc.i` and using the `-py3` command line option when calling SWIG , the proxy classes of the STL containers will automatically gain an appropriate abstract base class. For example, the following SWIG interface:

> 通过在调用 SWIG 时包含 `pyabc.i` 并使用 `-py3` 命令行选项，STL 容器的代理类将自动获得适当的抽象基类。例如，以下 SWIG 接口：

```
%include <pyabc.i>
%include <std_map.i>
%include <std_list.i>

namespace std {
  %template(Mapii) map<int, int>;
  %template(IntList) list<int>;
}
```

will generate a Python proxy class `Mapii` inheriting from `collections.MutableMap` and a proxy class `IntList` inheriting from `collections.MutableSequence`.

`pyabc.i` also provides a macro `%pythonabc` that could be used to define an abstract base class for your own C++ class:

> 将产生一个继承自 `collections.MutableMap` 的 Python 代理类 `Mapii` 和一个继承自 `collections.MutableSequence` 的代理类 `IntList`。
>
> `pyabc.i` 还提供了宏 `%pythonabc`，可用于为你自己的 C++ 类定义抽象基类：

```
%pythonabc(MySet, collections.MutableSet);
```

For details of abstract base class, please see [PEP 3119](https://www.python.org/dev/peps/pep-3119/).

> 抽象基类的详细信息参见 [PEP 3119](https://www.python.org/dev/peps/pep-3119/)。

### 36.12.4 字节字符串输出转换

By default, any byte string (`char*` or `std::string`) returned from C or C++ code is decoded to text as UTF-8. This decoding uses the `surrogateescape` error handler under Python 3.1 or higher -- this error handler decodes invalid byte sequences to high surrogate characters in the range U+DC80 to U+DCFF. As an example, consider the following SWIG interface, which exposes a byte string that cannot be completely decoded as UTF-8:

> 默认情况下，将从 C 或 C++ 代码返回的任何字节字符串（`char *` 或 `std::string`）解码为 UTF-8 文本。此解码使用 Python 3.1 或更高版本下的  `surrogateescape` 错误处理程序——该错误处理程序将无效字节序列解码为 U+DC80 到 U+DCFF 范围内的高替代字符。例如，请考虑以下 SWIG 接口，该接口公开了无法完全解码为 UTF-8 的字节字符串：

```
%module example

%include <std_string.i>

%inline %{

const char* non_utf8_c_str(void) {
        return "h\xe9llo w\xc3\xb6rld";
}

%}
```

When this method is called from Python 3, the return value is the following text string:

> 从 Python 3 调用此方法时，返回值为以下文本字符串：

```python
>>> s = example.non_utf8_c_str()
>>> s
'h\udce9llo wörld'
```

Since the C string contains bytes that cannot be decoded as UTF-8, those raw bytes are represented as high surrogate characters that can be used to obtain the original byte sequence:

> 由于 C 字符串包含无法解码为 UTF-8 的字节，因此这些原始字节被表示为高替代字符，可用于获取原始字节序列：

```python
>>> b = s.encode('utf-8', errors='surrogateescape')
>>> b
b'h\xe9llo w\xc3\xb6rld'
```

One can then attempt a different encoding, if desired (or simply leave the byte string as a raw sequence of bytes for use in binary protocols):

> 然后，如果需要，可以尝试进行不同的编码（或简单地将字节字符串保留为原始字节序列以供二进制协议使用）：

```python
>>> b.decode('latin-1')
'héllo wÃ¶rld'
```

Note, however, that text strings containing surrogate characters are rejected with the default `strict` codec error handler. For example:

> 但是请注意，包含代理字符的文本字符串将被默认的 `strict` 编解码器错误处理程序拒绝。例如：

```python
>>> with open('test', 'w') as f:
...     print(s, file=f)
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
UnicodeEncodeError: 'utf-8' codec can't encode character '\udce9' in position 1: surrogates not allowed
```

This requires the user to check most strings returned by SWIG bindings, but the alternative is for a non-UTF8 byte string to be completely inaccessible in Python 3 code.

For more details about the `surrogateescape` error handler, please see [PEP 383](https://www.python.org/dev/peps/pep-0383/).

In some cases, users may wish to instead handle all byte strings as bytes objects in Python 3. This can be accomplished by adding `SWIG_PYTHON_STRICT_BYTE_CHAR` to the generated code:

> 这要求用户检查 SWIG 绑定返回的大多数字符串，但是另一种方法是在 Python 3 代码中完全不可访问非 UTF8 字节的字符串。
>
> 有关 `surrogateescape` 错误处理程序的更多详细信息，请参见 [PEP 383](https://www.python.org/dev/peps/pep-0383/)。
>
> 在某些情况下，用户可能希望将所有字节字符串作为 Python 3 中的字节对象来处理。这可以通过在生成的代码中添加 `SWIG_PYTHON_STRICT_BYTE_CHAR` 来实现：

```
%module char_to_bytes
%begin %{
#define SWIG_PYTHON_STRICT_BYTE_CHAR
%}

char *charstring(char *s) {
  return s;
}
```

This will modify the behavior so that only Python 3 bytes objects will be accepted and converted to a C/C++ string, and any string returned from C/C++ will be converted to a bytes object in Python 3:

> 这将修改行为，以便仅 Python 3 字节对象将被接受并转换为 C/C++ 字符串，并且从 C/C++ 返回的任何字符串都将在 Python 3 中转换为字节对象：

```python
>>> from char_to_bytes import *
>>> charstring(b"hi") # Byte string
b'hi'
>>> charstring("hi")  # Unicode string
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
TypeError: in method 'charstring', argument 1 of type 'char *'
```

Note that in Python 2, defining `SWIG_PYTHON_STRICT_BYTE_CHAR` has no effect, since strings in Python 2 are equivalent to Python 3 bytes objects. However, there is a similar capability to force unicode-only handling for wide characters C/C++ strings (`wchar_t *` or `std::wstring` types) in Python 2. By default, in Python 2 both strings and unicode strings are converted to C/C++ wide strings, and returned wide strings are converted to a Python unicode string. To instead only convert unicode strings to wide strings, users can add `SWIG_PYTHON_STRICT_UNICODE_WCHAR` to the generated code:

> 请注意，在 Python 2 中，定义 `SWIG_PYTHON_STRICT_BYTE_CHAR` 无效，因为 Python 2 中的字符串等效于 Python 3 字节对象。但是，在 Python 2 中，有类似的功能可以对宽字符 C/C++ 字符串（`wchar_t *` 或 `std::wstring` 类型）强制仅对 unicode 进行处理。默认情况下，在 Python 2 中，字符串和 unicode 字符串都是转换为 C/C++ 宽字符串，然后将返回的宽字符串转换为 Python unicode 字符串。要只将 unicode 字符串转换为宽字符串，用户可以将 `SWIG_PYTHON_STRICT_UNICODE_WCHAR` 添加到生成的代码中：

```
%module wchar_to_unicode
%begin %{
#define SWIG_PYTHON_STRICT_UNICODE_WCHAR
%}

wchar_t *wcharstring(wchar_t *s) {
  return s;
}
```

This ensures that only unicode strings are accepted by wcharstring in both Python 2 and Python 3:

> 这确保只有 unicode 字符串被 Python 2 和 Python 3 中的宽字符串接受：

```python
>>> from wchar_to_unicode import *
>>> wcharstring(u"hi") # Unicode string
u'hi'
>>> wcharstring(b"hi") # Byte string
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
TypeError: in method 'charstring', argument 1 of type 'wchar_t *'
```

By defining both `SWIG_PYTHON_STRICT_BYTE_CHAR` and `SWIG_PYTHON_STRICT_UNICODE_WCHAR`, Python wrapper code can support overloads taking both `std::string` (as Python bytes) and `std::wstring` (as Python unicode).

> 通过定义 `SWIG_PYTHON_STRICT_BYTE_CHAR` 和 `SWIG_PYTHON_STRICT_UNICODE_WCHAR`，Python 包装代码可以支持接受 `std::string` （Python 字节）和 `std::wstring` （Python unicode）的重载。

### 36.12.5 Python 2 Unicode

A Python 3 string is a Unicode string so by default a Python 3 string that contains Unicode characters passed to C/C++ will be accepted and converted to a C/C++ string (`char *` or `std::string` types). A Python 2 string is not a unicode string by default and should a Unicode string be passed to C/C++ it will fail to convert to a C/C++ string (`char *` or `std::string` types). The Python 2 behavior can be made more like Python 3 by defining `SWIG_PYTHON_2_UNICODE`when compiling the generated C/C++ code. By default when the following is wrapped:

> Python 3 字符串是 Unicode 字符串，因此默认情况下，包含传递给 C/C++ 的 Unicode 字符的 Python 3 字符串将被接受并转换为 C/C++ 字符串（`char *` 或 `std::string` 类型）。默认情况下，Python 2 字符串不是 Unicode 字符串，并且如果将 Unicode 字符串传递给 C/C++，它将无法转换为 C/C++ 字符串（`char *` 或 `std::string` 类型）。通过在编译生成的 C/C++ 代码时定义 `SWIG_PYTHON_2_UNICODE`，可以使 Python 2 的行为更像 Python 3。默认情况下，包装以下内容时：

```
%module unicode_strings
char *charstring(char *s) {
  return s;
}
```

An error will occur when using Unicode strings in Python 2:

> 当在 Python 2 中使用 Unicode 字符串，会出现一个错误：

```python
>>> from unicode_strings import *
>>> charstring("hi")
'hi'
>>> charstring(u"hi")
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
TypeError: in method 'charstring', argument 1 of type 'char *'
```

When the `SWIG_PYTHON_2_UNICODE` macro is added to the generated code:

> 当宏 `SWIG_PYTHON_2_UNICODE` 被加入生成代码：

```
%module unicode_strings
%begin %{
#define SWIG_PYTHON_2_UNICODE
%}

char *charstring(char *s) {
  return s;
}
```

Unicode strings will be successfully accepted and converted from UTF-8, but note that they are returned as a normal Python 2 string:

> Unicode 字符串将被成功接受并从 UTF-8 转换，但是请注意，它们将作为普通的 Python 2 字符串返回：

```python
>>> from unicode_strings import *
>>> charstring("hi")
'hi'
>>> charstring(u"hi")
'hi'
>>>
```

Note that defining both `SWIG_PYTHON_2_UNICODE` and `SWIG_PYTHON_STRICT_BYTE_CHAR` at the same time is not allowed, since the first is allowing unicode conversion and the second is explicitly prohibiting it.

> 请注意，不允许同时定义 `SWIG_PYTHON_2_UNICODE` 和 `SWIG_PYTHON_STRICT_BYTE_CHAR`，因为前者允许 Unicode 转换，而后者则明确禁止 Unicode 转换。
