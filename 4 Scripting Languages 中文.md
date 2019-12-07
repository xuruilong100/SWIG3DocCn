[TOC]

# 4 脚本语言

This chapter provides a brief overview of scripting language extension programming and the mechanisms by which scripting language interpreters access C and C++ code.

> 本章简要概述了脚本语言扩展编程，以及脚本语言解释器访问 C 和 C++ 代码的机制。

## 4.1 两种语言的概览

When a scripting language is used to control a C program, the resulting system tends to look as follows:

> 当使用脚本语言来控制 C 程序时，生成的系统往往如下所示：

![Scripting language input - C/C++ functions output](http://www.swig.org/Doc3.0/ch2.1.png)

In this programming model, the scripting language interpreter is used for high level control whereas the underlying functionality of the C/C++ program is accessed through special scripting language "commands." If you have ever tried to write your own simple command interpreter, you might view the scripting language approach to be a highly advanced implementation of that. Likewise, If you have ever used a package such as MATLAB or IDL, it is a very similar model--the interpreter executes user commands and scripts. However, most of the underlying functionality is written in a low-level language like C or Fortran.

The two-language model of computing is extremely powerful because it exploits the strengths of each language. C/C++ can be used for maximal performance and complicated systems programming tasks. Scripting languages can be used for rapid prototyping, interactive debugging, scripting, and access to high-level data structures such associative arrays.

> 在此编程模型中，脚本语言解释器用于高级控制，而 C/C++ 程序的基础功能通过特殊脚本语言“命令”访问。如果你曾尝试编写自己的简单命令解释器，则可能会将脚本语言方法视为其高级实现。同样，如果你曾经使用过像 MATLAB 或 IDL 这样的软件包，它就是一个非常相似的模型——解释器执行用户命令和脚本。但是，大多数底层功能都是用 C 或 Fortran 等低级语言编写的。
>
> 双语计算模型非常强大，因为它充分利用了每种语言的优势。C/C++ 可用于最大化性能和复杂系统编程任务。脚本语言可用于快速原型设计、交互式调试、脚本编写，以及对高级数据结构（如关联数组）的访问。

## 4.2 脚本语言如何调用 C？

Scripting languages are built around a parser that knows how to execute commands and scripts. Within this parser, there is a mechanism for executing commands and accessing variables. Normally, this is used to implement the builtin features of the language. However, by extending the interpreter, it is usually possible to add new commands and variables. To do this, most languages define a special API for adding new commands. Furthermore, a special foreign function interface defines how these new commands are supposed to hook into the interpreter.

Typically, when you add a new command to a scripting interpreter you need to do two things; first you need to write a special "wrapper" function that serves as the glue between the interpreter and the underlying C function. Then you need to give the interpreter information about the wrapper by providing details about the name of the function, arguments, and so forth. The next few sections illustrate the process.

> 脚本语言围绕一个知道如何执行命令和脚本的解析器构建。在此解析器中，有一种执行命令和访问变量的机制。通常，这用于实现语言的内置功能。但是，通过扩展解释器，通常可以添加新的命令和变量。为此，大多数语言都定义了一个用于添加新命令的特殊 API。此外，一个特殊的外部函数接口定义了这些新命令应该如何挂钩到解释器中。
>
> 通常，当你向脚本解释器添加新命令时，你需要做两件事。首先，你需要编写一个特殊的“包装器”函数，该函数充当解释器和底层 C 函数之间的粘合剂。然后，你需要通过提供有关函数名称、参数等的详细信息，为解释器提供有关包装器的信息。接下来的几节将说明这一过程。

### 4.2.1 包装器函数

Suppose you have an ordinary C function like this :

> 假定你的初始 C 函数如下：

```c
int fact(int n) {
  if (n <= 1)
    return 1;
  else
    return n*fact(n-1);
}
```

In order to access this function from a scripting language, it is necessary to write a special "wrapper" function that serves as the glue between the scripting language and the underlying C function. A wrapper function must do three things :

* Gather function arguments and make sure they are valid.
* Call the C function.
* Convert the return value into a form recognized by the scripting language.

As an example, the Tcl wrapper function for the `fact()` function above example might look like the following :

> 为了从脚本语言访问此函数，有必要编写一个特殊的“包装器”函数，作为脚本语言和底层 C 函数之间的粘合剂。包装函数必须做三件事：
>
> * 收集函数参数并确保它们有效。
> * 调用 C 函数。
> * 将返回值转换为脚本语言识别的形式。
>
> 举个例子，上面例子中 `fact()` 函数的 Tcl 包装器函数可能如下所示：

```c
int wrap_fact(ClientData clientData, Tcl_Interp *interp, int argc, char *argv[]) {
  int result;
  int arg0;
  if (argc != 2) {
    interp->result = "wrong # args";
    return TCL_ERROR;
  }
  arg0 = atoi(argv[1]);
  result = fact(arg0);
  sprintf(interp->result, "%d", result);
  return TCL_OK;
}
```

Once you have created a wrapper function, the final step is to tell the scripting language about the new function. This is usually done in an initialization function called by the language when the module is loaded. For example, adding the above function to the Tcl interpreter requires code like the following :

> 一旦创建了包装函数，最后一步就是告诉脚本语言有关新函数的信息。这通常在加载模块时由语言调用的初始化函数中完成。例如，将上述函数添加到 Tcl 解释器需要如下代码：

```c
int Wrap_Init(Tcl_Interp *interp) {
  Tcl_CreateCommand(interp, "fact", wrap_fact, (ClientData) NULL,
                    (Tcl_CmdDeleteProc *) NULL);
  return TCL_OK;
}
```

When executed, Tcl will now have a new command called "`fact`" that you can use like any other Tcl command.

Although the process of adding a new function to Tcl has been illustrated, the procedure is almost identical for Perl and Python. Both require special wrappers to be written and both need additional initialization code. Only the specific details are different.

> 执行时，Tcl 将有一个名为 `fact` 的新命令，你可以像使用任何其他 Tcl 命令一样使用它。
>
> 虽然只说明了向 Tcl 添加新函数的过程，但 Perl 和 Python 的过程几乎相同。两者都需要编写特殊的包装器，并且都需要额外的初始化代码。只有具体细节不同。

### 4.2.2 变量链接

Variable linking refers to the problem of mapping a C/C++ global variable to a variable in the scripting language interpreter. For example, suppose you had the following variable:

> 变量链接指的是将 C/C++ 全局变量映射到脚本语言解释器中变量的问题。例如，假设你有以下变量：

```c
double Foo = 3.5;
```

It might be nice to access it from a script as follows (shown for Perl):

> 以如下所示从脚本中访问它看起来挺不错（显示为 Perl）：

```perl
$a = $Foo * 2.3;   # Evaluation
$Foo = $a + 2.0;   # Assignment
```

To provide such access, variables are commonly manipulated using a pair of get/set functions. For example, whenever the value of a variable is read, a "get" function is invoked. Similarly, whenever the value of a variable is changed, a "set" function is called.

In many languages, calls to the get/set functions can be attached to evaluation and assignment operators. Therefore, evaluating a variable such as `$Foo` might implicitly call the get function. Similarly, typing `$Foo = 4` would call the underlying set function to change the value.

> 为了提供这种访问，通常使用一对 get/set 函数来操纵变量。例如，每当读取变量的值时，就会调用“get”函数。类似地，只要改变变量的值，就会调用“set”函数。
>
> 在许多语言中，对 get/set 函数的调用可以附加到求值和赋值运算符。因此，评估诸如 `$Foo` 之类的变量可能会隐式调用 get 函数。类似地，键入 `$Foo = 4` 将调用底层 set 函数来更改值。

### 4.2.3 常量

In many cases, a C program or library may define a large collection of constants. For example:

> 在许多情况下，C 程序或库可以定义大量常量。例如：

```c
#define RED   0xff0000
#define BLUE  0x0000ff
#define GREEN 0x00ff00
```

To make constants available, their values can be stored in scripting language variables such as `$RED`, `$BLUE`, and `$GREEN`. Virtually all scripting languages provide C functions for creating variables so installing constants is usually a trivial exercise.

> 要使常量可用，它们的值可以存储在脚本语言变量中，例如 `$RED`，`$BLUE` 和 `$GREEN`。实际上，所有脚本语言都提供了用于创建变量的 C 函数，因此放置常量通常不是一个问题。

### 4.2.4 结构体与类

Although scripting languages have no trouble accessing simple functions and variables, accessing C/C++ structures and classes present a different problem. This is because the implementation of structures is largely related to the problem of data representation and layout. Furthermore, certain language features are difficult to map to an interpreter. For instance, what does C++ inheritance mean in a Perl interface?

The most straightforward technique for handling structures is to implement a collection of accessor functions that hide the underlying representation of a structure. For example,

> 虽然脚本语言在访问简单函数和变量时没有问题，但访问 C/C++ 结构体和类会带来不同的问题。这是因为结构体的实现主要与数据表示和布局问题有关。此外，某些语言特征难以映射到解释器。例如，C++ 继承在 Perl 接口中对应着什么？
>
> 处理结构体最直接的技术是实现一个访问器函数的集合以隐藏结构的底层表示。例如，

```c
struct Vector {
  Vector();
  ~Vector();
  double x, y, z;
};
```

can be transformed into the following set of functions :

> 可以转换为以下一组函数：

```c
Vector *new_Vector();
void delete_Vector(Vector *v);
double Vector_x_get(Vector *v);
double Vector_y_get(Vector *v);
double Vector_z_get(Vector *v);
void Vector_x_set(Vector *v, double x);
void Vector_y_set(Vector *v, double y);
void Vector_z_set(Vector *v, double z);
```

Now, from an interpreter these function might be used as follows:

> 现在，可以从解释器中使用这些函数，如下所示：

```
% set v [new_Vector]
% Vector_x_set $v 3.5
% Vector_y_get $v
% delete_Vector $v
% ...
```

Since accessor functions provide a mechanism for accessing the internals of an object, the interpreter does not need to know anything about the actual representation of a `Vector`.

> 由于访问器函数提供了访问对象内部的机制，因此解释器不需要知道关于 `Vector` 的实际表示的任何信息。

### 4.2.5 代理类

In certain cases, it is possible to use the low-level accessor functions to create a proxy class, also known as a shadow class. A proxy class is a special kind of object that gets created in a scripting language to access a C/C++ class (or struct) in a way that looks like the original structure (that is, it proxies the real C++ class). For example, if you have the following C++ definition :

> 在某些情况下，可以使用低级访问器函数来创建代理类，也称为影子类。代理类是一种特殊类型的对象，它以脚本语言创建，以一种看起来像原始结构体的方式访问 C/C++ 类（或结构体）（即它代理真正的 C++ 类）。例如，如果你有以下 C++ 定义：

```c++
class Vector {
public:
  Vector();
  ~Vector();
  double x, y, z;
};
```

A proxy classing mechanism would allow you to access the structure in a more natural manner from the interpreter. For example, in Python, you might want to do this:

> 代理分类机制允许你以更自然的方式从解释器访问结构体。例如，在 Python 中，你可能希望这样做：

```python
>>> v = Vector()
>>> v.x = 3
>>> v.y = 4
>>> v.z = -13
>>> ...
>>> del v
```

Similarly, in Perl5 you may want the interface to work like this:

> 同样，在 Perl5 中，你可能希望接口像这样工作：

```perl
$v = new Vector;
$v->{x} = 3;
$v->{y} = 4;
$v->{z} = -13;
```

Finally, in Tcl :

> 最后是在 Tcl 中：

```tcl
Vector v
v configure -x 3 -y 4 -z -13
```

When proxy classes are used, two objects are really at work--one in the scripting language, and an underlying C/C++ object. Operations affect both objects equally and for all practical purposes, it appears as if you are simply manipulating a C/C++ object.

> 当使用代理类时，有两个对象实际在起作用——一个在脚本语言中，另一个在底层的 C/C++ 对象中。操作同等地影响两个对象，以及所有实际目的，看起来好像只是在操作 C/C++ 对象。

## 4.3 构建脚本扩展

The final step in using a scripting language with your C/C++ application is adding your extensions to the scripting language itself. There are two primary approaches for doing this. The preferred technique is to build a dynamically loadable extension in the form of a shared library. Alternatively, you can recompile the scripting language interpreter with your extensions added to it.

> 在 C/C++ 应用程序中使用脚本语言的最后一步是向脚本语言本身添加扩展。这有两种主要方法。首选技术是以动态库的形式构建可动态加载的扩展。或者，你可以重新编译脚本语言解释器并添加扩展。

### 4.3.1 动态库与动态加载

To create a shared library or DLL, you often need to look at the manual pages for your compiler and linker. However, the procedure for a few common platforms is shown below:

> 要创建动态库或 DLL，通常需要查看编译器和链接器的手册。但是，一些常见系统的过程如下所示：

```
# Build a shared library for Solaris
gcc -fpic -c example.c example_wrap.c -I/usr/local/include
ld -G example.o example_wrap.o -o example.so

# Build a shared library for Linux
gcc -fpic -c example.c example_wrap.c -I/usr/local/include
gcc -shared example.o example_wrap.o -o example.so
```

To use your shared library, you simply use the corresponding command in the scripting language (load, import, use, etc...). This will import your module and allow you to start using it. For example:

> 要使用动态库，只需使用脚本语言中的相应命令（`load`、`import`、`use` 等）。这将导入你的模块并允许你开始使用它。例如：

```tcl
% load ./example.so
% fact 4
24
%
```

When working with C++ codes, the process of building shared libraries may be more complicated--primarily due to the fact that C++ modules may need additional code in order to operate correctly. On many machines, you can build a shared C++ module by following the above procedures, but changing the link line to the following :

> 使用 C++ 代码时，构建动态库的过程可能会更复杂——主要是因为 C++ 模块可能需要额外的代码才能正常运行。在许多机器上，你可以按照上述过程构建共享 C++ 模块，但将链接行更改为以下内容：

```
c++ -shared example.o example_wrap.o -o example.so
```

### 4.3.2 链接动态库

When building extensions as shared libraries, it is not uncommon for your extension to rely upon other shared libraries on your machine. In order for the extension to work, it needs to be able to find all of these libraries at run-time. Otherwise, you may get an error such as the following :

> 将扩展构建为动态库时，扩展依赖于计算机上的其他动态库的情况并不罕见。为了使扩展能够工作，它需要能够在运行时找到所有这些库。否则，你可能会收到如下错误：

```
>>> import graph
Traceback (innermost last):
  File "<stdin>", line 1, in ?
  File "/home/sci/data1/beazley/graph/graph.py", line 2, in ?
    import graphc
ImportError:  1101:/home/sci/data1/beazley/bin/python: rld: Fatal Error: cannot
successfully map soname 'libgraph.so' under any of the filenames /usr/lib/libgraph.so:/
lib/libgraph.so:/lib/cmplrs/cc/libgraph.so:/usr/lib/cmplrs/cc/libgraph.so:
>>>
```

What this error means is that the extension module created by SWIG depends upon a shared library called "`libgraph.so`" that the system was unable to locate. To fix this problem, there are a few approaches you can take.

* Link your extension and explicitly tell the linker where the required libraries are located. Often times, this can be done with a special linker flag such as `-R`, `-rpath`, etc. This is not implemented in a standard manner so read the man pages for your linker to find out more about how to set the search path for shared libraries.
* Put shared libraries in the same directory as the executable. This technique is sometimes required for correct operation on non-Unix platforms.
* Set the UNIX environment variable `LD_LIBRARY_PATH` to the directory where shared libraries are located before running Python. Although this is an easy solution, it is not recommended. Consider setting the path using linker options instead.

> 这个错误意味着 SWIG 创建的扩展模块所依赖的名为 `libgraph.so` 的动态库在系统中无法找到。要解决此问题，你可以采取一些方法。
>
> * 链接你的扩展并明确告诉链接器所需库所在的位置。通常，这可以使用特殊的链接器标志来完成，例如 `-R`、`-rpath` 等。这不是以标准方式实现的，因此请阅读链接器的手册以了解更多有关如何设置动态库搜索路径的信息。
> * 将动态库放在与可执行文件相同的目录中。在非 Unix 平台上的正确操作有时需要此技术。
> * 在运行 Python 之前，将 UNIX 环境变量 `LD_LIBRARY_PATH` 设置为动态库所在的目录。虽然这是一个简单的解决方案，但不建议这样做。请考虑使用链接器选项设置路径。

### 4.3.3 静态链接

With static linking, you rebuild the scripting language interpreter with extensions. The process usually involves compiling a short main program that adds your customized commands to the language and starts the interpreter. You then link your program with a library to produce a new scripting language executable.

Although static linking is supported on all platforms, this is not the preferred technique for building scripting language extensions. In fact, there are very few practical reasons for doing this--consider using shared libraries instead.

> 使用静态链接，你可以使用扩展来重建脚本语言解释器。该过程通常涉及编译一个简短的主程序，该程序将自定义命令添加到语言中并启动解释程序。然后，将程序与库链接以生成新的脚本语言可执行文件。
>
> 虽然所有平台都支持静态链接，但这不是构建脚本语言扩展的首选技术。实际上，这样做的实际理由很少——请考虑使用动态库。
