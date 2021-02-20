[TOC]

# 16 处理模块

## 16.1 模块介绍

Each invocation of SWIG requires a module name to be specified. The module name is used to name the resulting target language extension module. Exactly what this means and what the name is used for depends on the target language, for example the name can define a target language namespace or merely be a useful name for naming files or helper classes. Essentially, a module comprises target language wrappers for a chosen collection of global variables/functions, structs/classes and other C/C++ types.

The module name can be supplied in one of two ways. The first is to specify it with the special `%module` directive. This directive must appear at the beginning of the interface file. The general form of this directive is:

> 每次调用 SWIG 都需要指定一个模块名称。模块名称用于命名生成的目标语言扩展模块。确切的含义和名称的使用取决于目标语言，例如，名称可以定义目标语言的名称空间，或者仅是用于命名文件或辅助类的有用名称。本质上，一个模块包括目标语言包装器，用于选择的全局变量/函数，结构体/类和其他 C/C++ 类型的集合。
>
> 可以通过以下两种方式之一提供模块名称。首先是用特殊的 `%module` 指令指定它。该指令必须出现在接口文件的开头。该指令的一般形式为：

```
%module(option1="value1", option2="value2", ...) modulename
```

where the modulename is mandatory and the options add one or more optional additional features. Typically no options are specified, for example:

> 其中，`modulename` 是必需的，而选项则添加一个或多个可选的附加功能。通常不指定任何选项，例如：

```
%module mymodule
```

The second way to specify the module name is with the `-module` command line option, for example `-module mymodule`. If the module name is supplied on the command line, it overrides the name specified by the `%module` directive.

When first working with SWIG, users commonly start by creating a single module. That is, you might define a single SWIG interface that wraps some set of C/C++ code. You then compile all of the generated wrapper code together and use it. For large applications, however, this approach is problematic---the size of the generated wrapper code can be rather large. Moreover, it is probably easier to manage the target language interface when it is broken up into smaller pieces.

This chapter describes the problem of using SWIG in programs where you want to create a collection of modules. Each module in the collection is created via separate invocations of SWIG.

> 指定模块名称的第二种方法是使用 `-module` 命令行选项，例如 `-module mymodule`。如果模块名称是在命令行中提供的，它将覆盖由 `%module` 指令指定的名称。
>
> 首次使用 SWIG 时，用户通常首先创建一个模块。也就是说，你可以定义单个 SWIG 接口，该接口包装一些 C/C++ 代码集。然后，你将所有生成的包装器代码一起编译并使用。但是，对于大型应用程序，此方法存在问题——生成的包装器代码的大小可能会很大。此外，将目标语言接口分解成较小的部分时，可能更容易管理目标语言接口。
>
> 本章描述了在要创建模块集合的程序中使用 SWIG 的问题。集合中的每个模块都是通过 SWIG 的单独调用创建的。

## 16.2 基本用法

The basic usage case with multiple modules is when modules do not have cross-references (ie. when wrapping multiple independent C APIs). In that case, swig input files should just work out of the box - you simply create multiple wrapper `.cxx` files, link them into your application, and insert/load each in the scripting language runtime as you would do for the single module case.

A bit more complex is the case in which modules need to share information. For example, when one module extends the class of another by deriving from it:

> 多个模块的基本用法是模块不具有交叉引用时（即包装多个独立的 C API 时）。在这种情况下，swig 输入文件应该是开箱即用的——你只需创建多个包装器 `.cxx` 文件，将它们链接到你的应用程序中，然后像在单模块情况下那样在脚本语言运行时中插入/加载每个文件即可。
>
> 模块需要共享信息的情况稍微复杂一些。例如，当一个模块通过派生而扩展了另一个模块的类时：

```c++
// File: base.h
class base {
public:
  int foo();
};
```

```
// File: base_module.i
%module base_module

%{
#include "base.h"
%}
%include "base.h"
```

```
// File: derived_module.i
%module derived_module

%import "base_module.i"

%inline %{
class derived : public base {
public:
  int bar();
};
%}
```

To create the wrapper properly, module `derived_module` needs to know about the `base` class and that its interface is covered in another module. The line `%import "base_module.i"` lets SWIG know exactly that. Often the `.h` file is passed to `%import` instead of the `.i`, which unfortunately doesn't work for all language modules. For example, Python requires the name of module that the base class exists in so that the proxy classes can fully inherit the base class's methods. Typically you will get a warning when the module name is missing, eg:

> 为了正确地创建包装器，模块 `derived_module` 需要了解 `base` 类，并且其接口包含在另一个模块中。`%import "base_module.i"` 使 SWIG 确切知道这一点。通常，`.h` 文件会传递给 `%import` 而不是 `.i` 传递，不幸的是，该文件不适用于所有语言模块。例如，Python 需要基类所在的模块的名称，以便代理类可以完全继承基类的方法。通常，当缺少模块名称时，你会收到警告，例如：

```
derived_module.i:8: Warning 401: Base class 'base' ignored - unknown module name for base. Either
import
the appropriate module interface file or specify the name of the module in the %import directive.
```

It is sometimes desirable to import the header file rather than the interface file and overcome the above warning. For example in the case of the imported interface being quite large, it may be desirable to simplify matters and just import a small header file of dependent types. This can be done by specifying the optional `module` attribute in the `%import` directive. The `derived_module.i` file shown above could be replaced with the following:

> 有时需要导入头文件而不是接口文件并克服上述警告。例如，在导入的接口文件很大的情况下，可能希望简化事务并仅导入依赖类型的小头文件。这可以通过在 `%import` 指令中指定可选的 `module` 属性来完成。上面显示的 `derived_module.i` 文件可以替换为以下内容：

```
// File: derived_module.i
%module derived_module

%import(module="base_module") "base.h"

%inline %{
class derived : public base {
public:
  int bar();
};
```

Note that "base_module" is the module name and is the same as that specified in `%module` in `base_module.i` as well as the `%import` in `derived_module.i`.

Another issue to beware of is that multiple dependent wrappers should not be linked/loaded in parallel from multiple threads as SWIG provides no locking - for more on that issue, read on.

> 注意，`base_module` 是模块名称，与` base_module.i` 中的 `%module` 以及 ` derived_module.i` 中的 `%import` 所指定的名称相同。
>
> 要注意的另一个问题是，由于 SWIG 不提供锁定，因此不应从多个线程并行链接/加载多个从属包装，有关该问题的更多信息，请继续阅读。

## 16.3 SWIG 运行时代码

Many of SWIG's target languages generate a set of functions commonly known as the "SWIG runtime." These functions are primarily related to the runtime type system which checks pointer types and performs other tasks such as proper casting of pointer values in C++. As a general rule, the statically typed target languages, such as Java, use the language's built in static type checking and have no need for a SWIG runtime. All the dynamically typed / interpreted languages rely on the SWIG runtime.

The runtime functions are private to each SWIG-generated module. That is, the runtime functions are declared with "static" linkage and are visible only to the wrapper functions defined in that module. The only problem with this approach is that when more than one SWIG module is used in the same application, those modules often need to share type information. This is especially true for C++ programs where SWIG must collect and share information about inheritance relationships that cross module boundaries.

To solve the problem of sharing information across modules, a pointer to the type information is stored in a global variable in the target language namespace. During module initialization, type information is loaded into the global data structure of type information from all modules.

There are a few trade offs with this approach. This type information is global across all SWIG modules loaded, and can cause type conflicts between modules that were not designed to work together. To solve this approach, the SWIG runtime code uses a define SWIG_TYPE_TABLE to provide a unique type table. This behavior can be enabled when compiling the generated _wrap.cxx or _wrap.c file by adding -DSWIG_TYPE_TABLE=myprojectname to the command line argument.

Then, only modules compiled with SWIG_TYPE_TABLE set to myprojectname will share type information. So if your project has three modules, all three should be compiled with -DSWIG_TYPE_TABLE=myprojectname, and then these three modules will share type information. But any other project's types will not interfere or clash with the types in your module.

Another issue relating to the global type table is thread safety. If two modules try and load at the same time, the type information can become corrupt. SWIG currently does not provide any locking, and if you use threads, you must make sure that modules are loaded serially. Be careful if you use threads and the automatic module loading that some scripting languages provide. One solution is to load all modules before spawning any threads, or use SWIG_TYPE_TABLE to separate type tables so they do not clash with each other.

Lastly, SWIG uses a #define SWIG_RUNTIME_VERSION, located in Lib/swigrun.swg and near the top of every generated module. This number gets incremented when the data structures change, so that SWIG modules generated with different versions can peacefully coexist. So the type structures are separated by the (SWIG_TYPE_TABLE, SWIG_RUNTIME_VERSION) pair, where by default SWIG_TYPE_TABLE is empty. Only modules compiled with the same pair will share type information.

> SWIG 的许多目标语言都会生成一组函数，通常称为 `SWIG runtime`。这些函数主要与运行时类型系统有关，该系统检查指针类型并执行其他任务，例如在 C++ 中正确转换指针值。通常，静态类型的目标语言（例如 Java）使用该语言的内置静态类型检查，并且不需要 SWIG 运行时。所有动态类型化/解释性语言都依赖于 SWIG 运行时。
>
> 运行时函数对于每个 SWIG 生成的模块都是私有的。即，运行时函数是通过静态链接声明的，并且仅对该模块中定义的包装器函数可见。这种方法的唯一问题是，在同一应用程序中使用多个 SWIG 模块时，这些模块通常需要共享类型信息。对于 C++ 程序尤其如此，其中 SWIG 必须收集和共享有关跨越模块边界的继承关系的信息。
>
> 为了解决跨模块共享信息的问题，将指向类型信息的指针存储在目标语言名称空间的全局变量中。在模块初始化期间，类型信息将从所有模块加载到类型信息的全局数据结构中。
>
> 这种方法需要权衡取舍。此类型信息在所有已加载的 SWIG 模块中是全局的，并且可能导致未设计为协同工作的模块之间发生类型冲突。为了解决此问题，SWIG 运行时代码使用定义 `SWIG_TYPE_TABLE` 提供唯一的类型表。通过将 `-DSWIG_TYPE_TABLE = myprojectname` 添加到命令行参数来编译生成的 `_wrap.cxx` 或 `_wrap.c` 文件时，可以启用此行为。
>
> 然后，仅将 `SWIG_TYPE_TABLE` 设置为 `myprojectname` 编译的模块将共享类型信息。因此，如果你的项目有三个模块，则应使用 `-DSWIG_TYPE_TABLE = myprojectname` 编译所有三个模块，然后这三个模块将共享类型信息。但是任何其他项目的类型都不会干扰或冲突你模块中的类型。
>
> 与全局类型表有关的另一个问题是线程安全性。如果两个模块尝试同时加载，则类型信息可能会损坏。SWIG 当前不提供任何锁定，如果使用线程，则必须确保模块已串行加载。如果使用某些脚本语言提供的线程和自动模块加载，请小心。一种解决方案是在产生任何线程之前加载所有模块，或使用 `SWIG_TYPE_TABLE` 分隔类型表，以使它们彼此不冲突。
>
> 最后，SWIG 使用 `#define SWIG_RUNTIME_VERSION`，位于 `Lib/swigrun.swg` 中，在每个生成的模块的顶部附近。当数据结构更改时，此数字将递增，以便可以使用不同版本生成的 SWIG 模块和平共处。因此，类型结构由（`SWIG_TYPE_TABLE`，`SWIG_RUNTIME_VERSION`）对分隔，其中默认情况下 `SWIG_TYPE_TABLE` 为空。只有使用同一对编译的模块才能共享类型信息。

## 16.4 从外部访问运行时代码

As described in [The run-time type checker](http://swig.org/Doc3.0/Typemaps.html#Typemaps_runtime_type_checker), the functions `SWIG_TypeQuery`, `SWIG_NewPointerObj`, and others sometimes need to be called. Calling these functions from a typemap is supported, since the typemap code is embedded into the `_wrap.c` file, which has those declarations available. If you need to call the SWIG run-time functions from another C file, there is one header you need to include. To generate the header that needs to be included, SWIG can be run in a different mode via `-external-runtime` to generate the run-time instead of the normal mode of processing an input interface file. For example:

> 如[运行时类型检查器](http://swig.org/Doc3.0/Typemaps.html#Typemaps_runtime_type_checker)中所述，有时需要调用函数 `SWIG_TypeQuery`、`SWIG_NewPointerObj` 和其他函数。支持从类型图调用这些函数，因为类型图代码已嵌入到 `_wrap.c` 文件中，该文件具有可用的声明。如果需要从另一个 C 文件调用 SWIG 运行时函数，则需要包含一个标头。为了生成需要包含的头文件，可以通过 `-external-runtime` 以不同的模式运行 SWIG 以生成运行时，而不是正常的处理输入接口文件的模式。例如：

```
$ swig -python -external-runtime <filename>
```

The filename argument is optional and if it is not passed, then the default filename will be something like `swigpyrun.h`, depending on the language. This header file should be treated like any of the other `_wrap.c` output files, and should be regenerated when the `_wrap` files are. After including this header, your code will be able to call `SWIG_TypeQuery`, `SWIG_NewPointerObj`, `SWIG_ConvertPtr` and others. The exact argument parameters for these functions might differ between language modules; please check the language module chapters for more information.

Inside this header the functions are declared static and are included inline into the file, and thus the file does not need to be linked against any SWIG libraries or code (you might still need to link against the language libraries like libpython-2.3). Data is shared between this file and the _wrap.c files through a global variable in the scripting language. It is also possible to copy this header file along with the generated wrapper files into your own package, so that you can distribute a package that can be compiled without SWIG installed (this works because the header file is self-contained, and does not need to link with anything).

This header will also use the -DSWIG_TYPE_TABLE described above, so when compiling any code which includes the generated header file should define the SWIG_TYPE_TABLE to be the same as the module whose types you are trying to access.

> `filename` 参数是可选的，如果未传递，则默认文件名将类似于 `swigpyrun.h`，具体取决于语言。该头文件应与其他 `_wrap.c` 输出文件一样对待，并且在 `_wrap` 文件被使用时应重新生成。包含此标头后，你的代码将能够调用 `SWIG_TypeQuery`、` SWIG_NewPointerObj`、`SWIG_ConvertPtr` 等。这些功能的确切参数参数可能在语言模块之间有所不同。请查看语言模块章节以获取更多信息。
>
> 在此标头中，函数被声明为静态的，并被内联包含在文件中，因此不需要将文件与任何 SWIG 库或代码链接（你可能仍需要与 `libpython-2.3` 之类的语言库链接）。通过脚本语言中的全局变量在此文件和 `_wrap.c` 文件之间共享数据。也可以将此头文件与生成的包装文件一起复制到你自己的程序包中，以便你可以分发无需安装 SWIG 即可编译的程序包（这是可行的，因为头文件是自包含的，不需要链接任何东西）。
>
> 该头文件还将使用上述的 `-DSWIG_TYPE_TABLE`，因此，在编译包含生成的头文件的任何代码时，应将 `SWIG_TYPE_TABLE` 定义为与你要访问其类型的模块相同。

## 16.5 关于静态库的警告

When working with multiple SWIG modules, you should take care not to use static libraries. For example, if you have a static library `libfoo.a` and you link a collection of SWIG modules with that library, each module will get its own private copy of the library code inserted into it. This is very often **NOT** what you want and it can lead to unexpected or bizarre program behavior. When working with dynamically loadable modules, you should try to work exclusively with shared libraries.

当使用多个 SWIG 模块时，应注意不要使用静态库。例如，如果你有一个静态库 `libfoo.a`，并且将一个 SWIG 模块集合与该库链接，则每个模块将获得自己的插入的库代码的私有副本。这通常是**不需要**，它可能会导致程序异常或异常。当使用可动态加载的模块时，应尝试专门使用共享库。

## 16.6 引用

Due to the complexity of working with shared libraries and multiple modules, it might be a good idea to consult an outside reference. John Levine's "Linkers and Loaders" is highly recommended.

由于使用共享库和多个模块的复杂性，最好参考外部引用。强烈建议使用 John Levine 的《链接器和加载器》。

## 16.7 缩减包装器文件大小

Using multiple modules with the `%import` directive is the most common approach to modularising large projects. In this way a number of different wrapper files can be generated, thereby avoiding the generation of a single large wrapper file. There are a couple of alternative solutions for reducing the size of a wrapper file through the use of command line options and features.

使用带有 `%import` 指令的多个模块是模块化大型项目的最常用方法。这样，可以生成许多不同的包装文件，从而避免了生成单个大包装文件。有两种替代解决方案，可通过使用命令行选项和功能来减少包装文件的大小。

**`-fcompact`**

This command line option will compact the size of the wrapper file without changing the code generated into the wrapper file. It simply removes blank lines and joins lines of code together. This is useful for compilers that have a maximum file size that can be handled.

此命令行选项将压缩包装文件的大小，而无需更改生成到包装文件中的代码。它只是删除空白行并将代码行连接在一起。这对于具有可以处理的最大文件大小的编译器很有用。

**`-fvirtual`**

This command line option will remove the generation of superfluous virtual method wrappers. Consider the following inheritance hierarchy:

此命令行选项将删除多余的虚拟方法包装器的生成。考虑以下继承层次结构：

```c++
struct Base {
  virtual void method();
  ...
};

struct Derived : Base {
  virtual void method();
  ...
};
```

Normally wrappers are generated for both methods, whereas this command line option will suppress the generation of a wrapper for `Derived::method`. Normal polymorphic behaviour remains as `Derived::method` will still be called should you have a `Derived` instance and call the wrapper for `Base::method`.

> 通常为这两种方法都生成包装器，而此命令行选项将禁止为 `Derived::method` 生成包装器。正常的多态行为仍然保持不变，如果你具有 `Derived` 实例并为 `Base::method` 调用包装器，则仍将调用 `Derived::method`。

**`%feature("compactdefaultargs")`**

This feature can reduce the number of wrapper methods when wrapping methods with default arguments. The section on [default arguments](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args) discusses the feature and its limitations.

> 使用默认参数包装方法时，此功能可以减少包装方法的数量。关于[默认参数](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args)部分讨论了此功能及其限制。
