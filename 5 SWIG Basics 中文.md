[toc]

# 5 SWIG 基础知识

This chapter describes the basic operation of SWIG, the structure of its input files, and how it handles standard ANSI C declarations. C++ support is described in the next chapter. However, C++ programmers should still read this chapter to understand the basics. Specific details about each target language are described in later chapters.

> 本章介绍了 SWIG 的基本操作，输入文件的结构，以及它如何处理标准 ANSI C 声明。对 C++ 的支持将在下一章中介绍。但是，C++ 程序员仍应阅读本章以了解基础知识。有关每种目标语言的具体细节将在后面的章节中介绍。

## 5.1 运行 SWIG

To run SWIG, use the `swig` command with options and a filename like this:

> 要运行 SWIG，请配合选项和文件名使用 `swig` 命令：

```
swig [ options ] filename
```

where `filename` is a SWIG interface file or a C/C++ header file. Below is a subset of `options` that can be used. Additional options are also defined for each target language. A full list can be obtained by typing `swig -help` or `swig *-<lang>* -help` for language `*<lang>*` specific options.

> 其中 `filename` 是一个 SWIG 接口文件或一个 C/C++ 头文件。下面是 `options` 可选范围的子集。也为每一个目标语言定义额外的选项。完整的列表可以通过 `swig -help` 或 `swig -<lang> -help` 获得（`<lang>` 针对特定语言）。

```
-allegrocl            Generate ALLEGROCL wrappers
-chicken              Generate CHICKEN wrappers
-clisp                Generate CLISP wrappers
-cffi                 Generate CFFI wrappers
-csharp               Generate C# wrappers
-d                    Generate D wrappers
-go                   Generate Go wrappers
-guile                Generate Guile wrappers
-java                 Generate Java wrappers
-javascript           Generate Javascript wrappers
-lua                  Generate Lua wrappers
-modula3              Generate Modula 3 wrappers
-mzscheme             Generate Mzscheme wrappers
-ocaml                Generate Ocaml wrappers
-octave               Generate Octave wrappers
-perl                 Generate Perl wrappers
-php5                 Generate PHP5 wrappers
-php7                 Generate PHP7 wrappers
-pike                 Generate Pike wrappers
-python               Generate Python wrappers
-r                    Generate R (aka GNU S) wrappers
-ruby                 Generate Ruby wrappers
-scilab               Generate Scilab wrappers
-sexp                 Generate Lisp S-Expressions wrappers
-tcl                  Generate Tcl wrappers
-uffi                 Generate Common Lisp / UFFI wrappers
-xml                  Generate XML wrappers
-c++                  Enable C++ processing
-cppext ext           Change file extension of C++ generated files to ext
                      (default is cxx, except for PHP5 which uses cpp)
-Dsymbol              Define a preprocessor symbol
-Fmicrosoft           Display error/warning messages in Microsoft format
-Fstandard            Display error/warning messages in commonly used format
-help                 Display all options
-Idir                 Add a directory to the file include path
-lifile               Include SWIG library file <ifile>
-module name          Set the name of the SWIG module
-o outfile            Set name of C/C++ output file to <outfile>
-oh headfile          Set name of C++ output header file for directors to <headfile>
-outcurrentdir        Set default output dir to current dir instead of input file's path
-outdir dir           Set language specific files output directory
-pcreversion          Display PCRE version information
-swiglib              Report location of SWIG library and exit
-version              Display SWIG version number
```

### 5.1.1 输入格式

As input, SWIG expects a file containing ANSI C/C++ declarations and special SWIG directives. More often than not, this is a special SWIG interface file which is usually denoted with a special `.i` or `.swg` suffix. In certain cases, SWIG can be used directly on raw header files or source files. However, this is not the most typical case and there are several reasons why you might not want to do this (described later).

The most common format of a SWIG interface is as follows:

> SWIG 需要一个包含 ANSI C/C++ 声明和特殊 SWIG 指令的文件作为输入。通常，这是一个特殊的 SWIG 接口文件，用特殊的 `.i` 或 `.swg` 后缀表示。在某些情况下，SWIG 可以直接用于原始头文件或源文件。但是，这不是最典型的情况，并且有几个原因可能导致你不想这样做（稍后介绍）。
>
> SWIG 接口最常见的格式如下：

```
%module mymodule
%{
#include "myheader.h"
%}
// Now list ANSI C/C++ declarations
int foo;
int bar(int x);
...
```

The module name is supplied using the special `%module` directive. Modules are described further in the [Modules Introduction](http://www.swig.org/Doc3.0/Modules.html#Modules_introduction) section.

Everything in the `%{ ... %}` block is simply copied verbatim to the resulting wrapper file created by SWIG. This section is almost always used to include header files and other declarations that are required to make the generated wrapper code compile. It is important to emphasize that just because you include a declaration in a SWIG input file, that declaration does *not* automatically appear in the generated wrapper code---therefore you need to make sure you include the proper header files in the `%{ ... %}` section. It should be noted that the text enclosed in `%{ ... %}` is not parsed or interpreted by SWIG. The `%{...%}` syntax and semantics in SWIG is analogous to that of the declarations section used in input files to parser generation tools such as yacc or bison.

> 模块名由特定的 `%module` 指令提供。对模块的进一步讨论在[模块-引言](http://www.swig.org/Doc3.0/Modules.html#Modules_introduction)章节。
>
> `%{...%}` 块中的所有内容都只会逐字复制到 SWIG 最终创建的包装器文件中。此部分几乎总是用于包含头文件和其他声明用于生成的包装器代码的编译。重点强调，因为你仅仅在 SWIG 输入文件中包含一个声明，该声明*不会*自动出现在生成的包装器代码中——因此你需要确保在 `%{...%}` 部分中包含正确的头文件。应该注意的是，SWIG 不解析或解释包含在 `%{...%}` 中的文本。SWIG 中的 `%{...%}` 语法和语义类似于解析器生成工具（如 yacc 或 bison）输入文件中的声明部分。

### 5.1.2 SWIG 输出

The output of SWIG is a C/C++ file that contains all of the wrapper code needed to build an extension module. SWIG may generate some additional files depending on the target language. By default, an input file with the name `file.i` is transformed into a file `file_wrap.c` or `file_wrap.cxx` (depending on whether or not the `-c++` option has been used). The name of the output C/C++ file can be changed using the `-o` option. In certain cases, file suffixes are used by the compiler to determine the source language (C, C++, etc.). Therefore, you have to use the `-o` option to change the suffix of the SWIG-generated wrapper file if you want something different than the default. For example:

> SWIG 的输出是一个 C/C++ 文件，其中包含构建扩展模块所需的所有包装器代码。SWIG 可能会根据目标语言生成一些其他文件。默认情况下，名为 `file.i` 的输入文件将转换为文件 `file_wrap.c` 或 `file_wrap.cxx`（取决于是否使用了 `-c++` 选项）。可以使用 `-o` 选项更改 C/C++ 输出文件的名称。在某些情况下，编译器使用文件后缀来确定源语言（C、C++ 等）。因此，如果需要不同于默认值的东西，则必须使用 `-o` 选项来更改 SWIG 生成的包装器文件的后缀。例如：

```
$ swig -c++ -python -o example_wrap.cpp example.i
```

The C/C++ output file created by SWIG often contains everything that is needed to construct an extension module for the target scripting language. SWIG is not a stub compiler nor is it usually necessary to edit the output file (and if you look at the output, you probably won't want to). To build the final extension module, the SWIG output file is compiled and linked with the rest of your C/C++ program to create a shared library.

For many target languages SWIG will also generate proxy class files in the target language. The default output directory for these language specific files is the same directory as the generated C/C++ file. This can be modified using the `-outdir` option. For example:

> SWIG 创建的 C/C++ 输出文件通常包含为目标脚本语言构建扩展模块所需的所有内容。SWIG 不是存根编译器（stub compiler），通常也不需要编辑输出文件（如果你看一看输出文件，你可能不会想要去编辑）。要构建最终的扩展模块，SWIG 输出文件将被编译并与其余的 C/C++ 程序链接以创建动态库。
>
> 对于许多目标语言，SWIG 还将生成目标语言的代理类文件。这些特定于语言的文件的默认输出目录与生成的 C/C++ 文件是同一目录。这可以使用 `-outdir` 选项进行修改。例如：

```
$ swig -c++ -python -outdir pyfiles -o cppfiles/example_wrap.cpp example.i
```

If the directories `cppfiles` and `pyfiles` exist, the following will be generated:

> 如果目录 `cppfiles` 和 `pyfiles` 存在，将生成以下内容：

```
cppfiles/example_wrap.cpp
pyfiles/example.py
```

If the `-outcurrentdir` option is used (without `-o`) then SWIG behaves like a typical C/C++ compiler and the default output directory is then the current directory. Without this option the default output directory is the path to the input file. If `-o` and `-outcurrentdir` are used together, `-outcurrentdir` is effectively ignored as the output directory for the language files is the same directory as the generated C/C++ file if not overridden with `-outdir`.

> 如果使用 `-outcurrentdir` 选项（没有 `-o`），则 SWIG 的行为类似于典型的 C/C++ 编译器，默认输出目录则是当前目录。如果没有此选项，则默认输出目录是输入文件的路径。如果同时使用 `-o` 和 `-outcurrentdir`，`-outcurrentdir` 实际上会被忽略，因为语言文件的输出目录与生成的 C/C++ 文件是相同的目录，如果没有用 `-outdir` 覆盖的话。

### 5.1.3 注释

C and C++ style comments may appear anywhere in interface files. In previous versions of SWIG, comments were used to generate documentation files. However, this feature is currently under repair and will reappear in a later SWIG release.

> C 和 C++ 样式的注释可以出现在接口文件中的任何位置。在早期版本的 SWIG 中，注释用于生成文档文件。但是，此功能目前正在修复中，并将在稍后的 SWIG 版本中重新出现。

### 5.1.4 C 预处理器

Like C, SWIG preprocesses all input files through an enhanced version of the C preprocessor. All standard preprocessor features are supported including file inclusion, conditional compilation and macros. However, `#include` statements are ignored unless the `-includeall` command line option has been supplied. The reason for disabling includes is that SWIG is sometimes used to process raw C header files. In this case, you usually only want the extension module to include functions in the supplied header file rather than everything that might be included by that header file (i.e., system headers, C library functions, etc.).

It should also be noted that the SWIG preprocessor skips all text enclosed inside a `%{...%}` block. In addition, the preprocessor includes a number of macro handling enhancements that make it more powerful than the normal C preprocessor. These extensions are described in the "[Preprocessor](http://www.swig.org/Doc3.0/Preprocessor.html#Preprocessor)" chapter.

> 与 C 一样，SWIG 通过 C 预处理器的增强版预处理所有输入文件。支持所有标准预处理器功能，包括文件包含、条件编译和宏。但是，除非提供了 `-includeall` 命令行选项，否则将忽略 `#include` 语句。禁用包括的原因是 SWIG 有时用于处理原始 C 头文件。在这种情况下，你通常只希望扩展模块包含所提供的头文件中的函数，而不是该头文件可能包含的所有内容（即系统头文件、C 库函数等）。
>
> 还应注意，SWIG 预处理器会跳过括在 `%{...%}` 块内的所有文本。此外，预处理器还包括许多宏处理增强功能，使其比普通的 C 预处理器更强大。这些扩展在[预处理器](http://www.swig.org/Doc3.0/Preprocessor.html#Preprocessor)一章中描述。

### 5.1.5 SWIG 指令

Most of SWIG's operation is controlled by special directives that are always preceded by a "`%`" to distinguish them from normal C declarations. These directives are used to give SWIG hints or to alter SWIG's parsing behavior in some manner.

Since SWIG directives are not legal C syntax, it is generally not possible to include them in header files. However, SWIG directives can be included in C header files using conditional compilation like this:

> SWIG 的大多数操作都是由特殊指令控制的，这些指令总是以 `%` 开头，以区别于普通的 C 声明。这些指令用于给 SWIG 提供提示或以某种方式更改 SWIG 的解析行为。
>
> 由于 SWIG 指令不是合法的 C 语法，因此通常不可能将它们包含在头文件中。但是，SWIG 指令可以使用条件编译包含在 C 头文件中，如下所示：

```c
/* header.h  --- Some header file */

/* SWIG directives -- only seen if SWIG is running */
#ifdef SWIG
%module foo
#endif
```

`SWIG` is a special preprocessing symbol defined by SWIG when it is parsing an input file.

> `SWIG` 是 SWIG 在解析输入文件时定义的特殊预处理符号。

### 5.1.6 解析限制

Although SWIG can parse most C/C++ declarations, it does not provide a complete C/C++ parser implementation. Most of these limitations pertain to very complicated type declarations and certain advanced C++ features. Specifically, the following features are not currently supported:
* Non-conventional type declarations. For example, SWIG does not support declarations such as the following (even though this is legal C):

> 虽然 SWIG 可以解析大多数 C/C++ 声明，但它不提供完整的 C/C++ 解析器实现。大多数这些限制都与非常复杂的类型声明，以及某些高级 C++ 功能有关。具体而言，目前不支持以下功能：
>
> * 非传统类型声明。例如，SWIG 不支持以下声明（即使这是合法的 C）：

```c
/* Non-conventional placement of storage specifier (extern) */
const int extern Number;

/* Extra declarator grouping */
Matrix (foo);    // A global variable

/* Extra declarator grouping in parameters */
void bar(Spam (Grok)(Doh));
```

In practice, few (if any) C programmers actually write code like this since this style is never featured in programming books. However, if you're feeling particularly obfuscated, you can certainly break SWIG (although why would you want to?).
* Running SWIG on C++ source files (the code in a .c, .cpp or .cxx file) is not recommended. The usual approach is to feed SWIG header files for parsing C++ definitions and declarations. The main reason is if SWIG parses a scoped definition or declaration (as is normal for C++ source files), it is ignored, unless a declaration for the symbol was parsed earlier. For example

> 实际上，很少（如果有的话）有 C 程序员实际编写这样的代码，因为这种风格从未出现在编程书籍中。但是，如果你感到特别困惑，你就确定是在违反 SWIG（为什么非要这么做呢？）。
>
> * 不建议在 C++ 源文件（`.c`、`.cpp` 或 `.cxx` 文件中的代码）上运行 SWIG。通常的方法是给 SWIG 提供头文件以解析 C++ 定义和声明。主要原因是如果 SWIG 解析范围定义或声明（对于 C++ 源文件来说是正常的），源文件将被忽略，除非先前解析了符号的声明。例如

```c++
/* bar not wrapped unless foo has been defined and
the declaration of bar within foo has already been parsed */
int foo::bar(int) {
... whatever ...
}
```

* Certain advanced features of C++ such as nested classes are not yet fully supported. Please see the C++ [Nested classes](http://www.swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_nested_classes) section for more information.

In the event of a parsing error, conditional compilation can be used to skip offending code. For example:

> * C++ 的某些高级功能（如嵌套类）尚未完全支持。有关更多信息，请参阅 C++ [嵌套类](http://www.swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_nested_classes)部分。
>
> 如果出现解析错误，可以使用条件编译来跳过非法代码。例如：

```c
#ifndef SWIG
... some bad declarations ...
#endif
```

Alternatively, you can just delete the offending code from the interface file.

One of the reasons why SWIG does not provide a full C++ parser implementation is that it has been designed to work with incomplete specifications and to be very permissive in its handling of C/C++ datatypes (e.g., SWIG can generate interfaces even when there are missing class declarations or opaque datatypes). Unfortunately, this approach makes it extremely difficult to implement certain parts of a C/C++ parser as most compilers use type information to assist in the parsing of more complex declarations (for the truly curious, the primary complication in the implementation is that the SWIG parser does not utilize a separate *typedef-name* terminal symbol as described on p. 234 of K&R).

> 或者，你可以从接口文件中删除有问题的代码。
>
> SWIG 没有提供完整的 C++ 解析器实现的原因之一是它被设计为使用不完整的规范，并且在处理 C/C++ 数据类型时非常宽松（例如，即使有丢失的类声明或透明数据类型，SWIG 也可以生成接口）。不幸的是，这种方法使得实现 C/C++ 解析器的某些部分变得极其困难，因为大多数编译器使用类型信息来帮助解析更复杂的声明（深层原因是，实现中的主要复杂性是 SWIG 解析器不使用单独的 *typedef-name* 终止符，如 K&R 第 234 页所述）。

## 5.2 包装简单的 C 声明

SWIG wraps simple C declarations by creating an interface that closely matches the way in which the declarations would be used in a C program. For example, consider the following interface file:

> SWIG 通过创建一个接口文件来包装简单的 C 声明，与 C 程序中声明的使用方式非常地匹配。例如，请考虑以下接口文件：

```
%module example

%inline %{
extern double sin(double x);
extern int strcmp(const char *, const char *);
extern int Foo;
%}
#define STATUS 50
#define VERSION "1.1"
```

In this file, there are two functions `sin()` and `strcmp()`, a global variable `Foo`, and two constants `STATUS` and `VERSION`. When SWIG creates an extension module, these declarations are accessible as scripting language functions, variables, and constants respectively. For example, in Tcl:

> 在这个文件中，有两个函数 `sin()` 和 `strcmp()`，一个全局变量 `Foo`，以及两个常量 `STATUS` 和 `VERSION`。当 SWIG 创建扩展模块时，这些声明可以分别作为脚本语言的函数、变量和常量访问。例如，在 Tcl 中：

```tcl
% sin 3
5.2335956
% strcmp Dave Mike
-1
% puts $Foo
42
% puts $STATUS
50
% puts $VERSION
1.1
```

Or in Python:

> 或 Python 中：

```python
>>> example.sin(3)
5.2335956
>>> example.strcmp('Dave', 'Mike')
-1
>>> print example.cvar.Foo
42
>>> print example.STATUS
50
>>> print example.VERSION
1.1
```

Whenever possible, SWIG creates an interface that closely matches the underlying C/C++ code. However, due to subtle differences between languages, run-time environments, and semantics, it is not always possible to do so. The next few sections describe various aspects of this mapping.

> 只要有可能，SWIG 就会创建一个与底层 C/C++ 代码紧密匹配的接口。但是，由于语言、运行时环境和语义之间的细微差别，并不总是可以这样做。接下来的几节将介绍这种映射的各个方面。

### 5.2.1 处理基本类型

In order to build an interface, SWIG has to convert C/C++ datatypes to equivalent types in the target language. Generally, scripting languages provide a more limited set of primitive types than C. Therefore, this conversion process involves a certain amount of type coercion.

Most scripting languages provide a single integer type that is implemented using the `int` or `long` datatype in C. The following list shows all of the C datatypes that SWIG will convert to and from integers in the target language:

> 为了构建接口，SWIG 必须将 C/C++ 数据类型转换为目标语言中的等效类型。通常，脚本语言提供比 C 更有限的一组原始类型。因此，此转换过程涉及一定量的类型强制。
>
> 大多数脚本语言提供单个整数类型，使用 C 中的 `int` 或 `long` 数据类型实现。SWIG 将把下列表中显示的所有 C 数据类型转换为目标语言的整数：

```
int
short
long
unsigned
signed
unsigned short
unsigned long
unsigned char
signed char
bool
```

When an integral value is converted from C, a cast is used to convert it to the representation in the target language. Thus, a 16 bit short in C may be promoted to a 32 bit integer. When integers are converted in the other direction, the value is cast back into the original C type. If the value is too large to fit, it is silently truncated.

`unsigned char` and `signed char` are special cases that are handled as small 8-bit integers. Normally, the `char` datatype is mapped as a one-character ASCII string.

The `bool` datatype is cast to and from an integer value of 0 and 1 unless the target language provides a special boolean type.

Some care is required when working with large integer values. Most scripting languages use 32-bit integers so mapping a 64-bit long integer may lead to truncation errors. Similar problems may arise with 32 bit unsigned integers (which may appear as large negative numbers). As a rule of thumb, the `int` datatype and all variations of `char` and `short` datatypes are safe to use. For `unsigned int` and `long` datatypes, you will need to carefully check the correct operation of your program after it has been wrapped with SWIG.

Although the SWIG parser supports the `long long` datatype, not all language modules support it. This is because `long long` usually exceeds the integer precision available in the target language. In certain modules such as Tcl and Perl5, `long long` integers are encoded as strings. This allows the full range of these numbers to be represented. However, it does not allow `long long` values to be used in arithmetic expressions. It should also be noted that although `long long` is part of the ISO C99 standard, it is not universally supported by all C compilers. Make sure you are using a compiler that supports `long long` before trying to use this type with SWIG.

SWIG recognizes the following floating point types :

> 当从 C 转换整数值时，使用强制转换将其转换为目标语言中的表示。因此，C 中的 16 位整数可以被提升为 32 位整数。当整数在另一个方向上转换时，该值将被转换回原始 C 类型。如果该值太大而无法匹配，则会被默默截断。
>
> `unsigned char` 和 `signed char` 是特殊情况，以 8 位整数处理。通常，`char` 数据类型被映射为单字符 ASCII 字符串。
>
> 除非目标语言提供特殊的布尔类型，否则 `bool` 数据类型将转换为 `0` 和 `1`。
>
> 使用大整数值时需要注意一些事项。大多数脚本语言使用 32 位整数，因此映射 64 位整数可能会导致截断错误。32 位无符号整数可能会出现类似的问题（可能显示为大的负数）。根据经验，`int` 以及 `char` 和 `short` 的所有变体都可以安全使用。对于 `unsigned int` 和 `long`，在使用 SWIG 包装后，你需要仔细检查程序的正确性。
>
> 虽然 SWIG 解析器支持 `long long` 数据类型，但并非所有语言模块都支持它。这是因为 `long long` 通常超过目标语言中可用的整数精度。在某些语言中，如 Tcl 和 Perl5，`long long` 整数被编码为字符串。这允许表示这些数字的全部范围。但是，它不允许在算术表达式中使用 `long long` 值。还应该注意的是，虽然 `long long` 是 ISO C99 标准的一部分，但并非所有 C 编译器普遍支持。在尝试将此类型与 SWIG 一起使用之前，请确保使用支持 `long long` 的编译器。
>
> SWIG 识别以下浮点类型：

```
float
double
```

Floating point numbers are mapped to and from the natural representation of floats in the target language. This is almost always a C `double`. The rarely used datatype of `long double` is not supported by SWIG.

The `char` datatype is mapped into a NULL terminated ASCII string with a single character. When used in a scripting language it shows up as a tiny string containing the character value. When converting the value back into C, SWIG takes a character string from the scripting language and strips off the first character as the char value. Thus if the value "foo" is assigned to a `char` datatype, it gets the value `f'.

The `char *` datatype is handled as a NULL-terminated ASCII string. SWIG maps this into a 8-bit character string in the target scripting language. SWIG converts character strings in the target language to NULL terminated strings before passing them into  C/C++ . The default handling of these strings does not allow them to have embedded NULL bytes. Therefore, the `char *` datatype is not generally suitable for passing binary data. However, it is possible to change this behavior by defining a SWIG typemap. See the chapter on [Typemaps](http://www.swig.org/Doc3.0/Typemaps.html#Typemaps) for details about this.

At this time, SWIG provides limited support for Unicode and wide-character strings (the C `wchar_t` type). Some languages provide typemaps for wchar_t, but bear in mind these might not be portable across different operating systems. This is a delicate topic that is poorly understood by many programmers and not implemented in a consistent manner across languages. For those scripting languages that provide Unicode support, Unicode strings are often available in an 8-bit representation such as UTF-8 that can be mapped to the `char *`type (in which case the SWIG interface will probably work). If the program you are wrapping uses Unicode, there is no guarantee that Unicode characters in the target language will use the same internal representation (e.g., UCS-2 vs. UCS-4). You may need to write some special conversion functions.

> 浮点数映射到目标语言中浮点数的自然表示形式。这几乎总是一个 C `double`。SWIG 不支持很少使用的 `long double` 数据类型。
>
> `char` 数据类型映射到带有 NULL 终止符的单字符 ASCII 字符串。在脚本语言中使用时，它显示为包含字符值的小字符串。将值转换回 C 时，SWIG 从脚本语言中获取一个字符串，并将第一个字符作为字符值剥离。因此，如果将值 `foo` 分配给 `char` 数据类型，则它将获得值 `f`。
>
> `char *` 数据类型作为以 NULL 结尾的 ASCII 字符串处理。SWIG 将此映射为目标脚本语言中的 8 位字符串。SWIG 将目标语言中的字符串转换为 NULL 结尾的字符串，然后再将它们传递给 C/C++ 。这些字符串的默认处理不允许它们具有嵌入的 NULL 字节。因此，`char *` 数据类型通常不适合传递二进制数据。但是，可以通过定义 SWIG 类型映射来更改此行为。有关详细信息，请参阅[类型映射](http://www.swig.org/Doc3.0/Typemaps.html#Typemaps)一章。
>
> 目前，SWIG 对 Unicode 和宽字符串（C `wchar_t` 类型）提供有限的支持。有些语言为 `wchar_t` 提供了类型映射，但请记住，这些语言可能无法在不同的操作系统中移植。这是一个微妙的主题，很多程序员都很难理解，而且没有以一致的方式跨语言实现。对于那些提供 Unicode 支持的脚本语言，Unicode 字符串通常以 8 位表示形式提供，例如 UTF-8，可以映射到 `char *` 类型（在这种情况下，SWIG 接口可能会起作用）。如果要包装的程序使用 Unicode，则无法保证目标语言中的 Unicode 字符将使用相同的内部表示（例如，UCS-2 与 UCS-4）。你可能需要编写一些特殊的转换函数。

### 5.2.2 全局变量

Whenever possible, SWIG maps C/C++ global variables into scripting language variables. For example,

> 只要有可能，SWIG 就会将 C/C++ 全局变量映射到脚本语言变量中。例如，

```
%module example
double foo;
```

results in a scripting language variable like this:

> 最终的脚本语言变量如下：

```
# Tcl
set foo [3.5]                   ;# Set foo to 3.5
puts $foo                       ;# Print the value of foo

# Python
cvar.foo = 3.5                  # Set foo to 3.5
print cvar.foo                  # Print value of foo

# Perl
$foo = 3.5;                     # Set foo to 3.5
print $foo, "\n";               # Print value of foo

# Ruby
Module.foo = 3.5               # Set foo to 3.5
print Module.foo, "\n"         # Print value of foo
```

Whenever the scripting language variable is used, the underlying C global variable is accessed. Although SWIG makes every attempt to make global variables work like scripting language variables, it is not always possible to do so. For instance, in Python, all global variables must be accessed through a special variable object known as `cvar`(shown above). In Ruby, variables are accessed as attributes of the module. Other languages may convert variables to a pair of accessor functions. For example, the Java module generates a pair of functions `double get_foo()` and `set_foo(double val)` that are used to manipulate the value.

Finally, if a global variable has been declared as `const`, it only supports read-only access. Note: this behavior is new to SWIG-1.3. Earlier versions of SWIG incorrectly handled `const` and created constants instead.

> 每当使用脚本语言变量时，都会访问底层 C 全局变量。虽然 SWIG 尽一切努力使全局变量像脚本语言变量一样工作，但并不总是这样做。例如，在 Python 中，必须通过称为 `cvar` 的特殊变量对象（如上所示）访问所有全局变量。在 Ruby 中，变量作为模块的属性进行访问。其他语言可以将变量转换为一对存取函数。例如，Java 模块生成一对函数 `double get_foo()` 和 `set_foo（double val）` 用于操作该值。
>
> 最后，如果全局变量已声明为 `const`，则它仅支持只读访问。注意：此行为是 SWIG-1.3 的新增功能。早期版本的 SWIG 错误地处理了 `const` 并改为创建了常量。

### 5.2.3 常量

Constants can be created using `#define`, enumerations, or a special `%constant` directive. The following interface file shows a few valid constant declarations :

> 可以使用 `#define`、枚举或特殊的 `%constant` 指令创建常量。以下接口文件显示了一些有效的常量声明：

```
#define I_CONST       5               // An integer constant
#define PI            3.14159         // A Floating point constant
#define S_CONST       "hello world"   // A string constant
#define NEWLINE       '\n'            // Character constant

enum boolean {NO=0, YES=1};
enum months {JAN, FEB, MAR, APR, MAY, JUN, JUL, AUG,
             SEP, OCT, NOV, DEC};
%constant double BLAH = 42.37;
#define PI_4 PI/4
#define FLAGS 0x04 | 0x08 | 0x40
```

In `#define` declarations, the type of a constant is inferred by syntax. For example, a number with a decimal point is assumed to be floating point. In addition, SWIG must be able to fully resolve all of the symbols used in a `#define` in order for a constant to actually be created. This restriction is necessary because `#define` is also used to define preprocessor macros that are definitely not meant to be part of the scripting language interface. For example:

> 在 `#define` 声明中，常量的类型由语法推断。例如，假设带小数点的数字是浮点数。此外，SWIG 必须能够完全解析 `#define` 中使用的所有符号，以便实际创建常量。这种限制是必要的，因为 `#define` 也用于定义预处理器宏，这些宏绝对不是脚本语言接口的一部分。例如：

```
#define EXTERN extern

EXTERN void foo();
```

In this case, you probably don't want to create a constant called `EXTERN`(what would the value be?). In general, SWIG will not create constants for macros unless the value can be completely determined by the preprocessor. For instance, in the above example, the declaration

> 在这种情况下，你可能不希望创建一个名为 `EXTERN` 的常量（值是什么？）。通常，SWIG 不会为宏创建常量，除非该值可以由预处理器完全确定。例如，在上面的例子中，声明

```
#define PI_4  PI/4
```

defines a constant because `PI` was already defined as a constant and the value is known. However, for the same conservative reasons even a constant with a simple cast will be ignored, such as

> 定义一个常量，因为 `PI` 已被定义为常量且值已知。但是，出于同样的保守原因，即使是简单强制转换的常量也会被忽略，例如

```
#define F_CONST (double) 5            // A floating point constant with cast
```

The use of constant expressions is allowed, but SWIG does not evaluate them. Rather, it passes them through to the output file and lets the C compiler perform the final evaluation (SWIG does perform a limited form of type-checking however).

For enumerations, it is critical that the original enum definition be included somewhere in the interface file (either in a header file or in the `%{ %}` block). SWIG only translates the enumeration into code needed to add the constants to a scripting language. It needs the original enumeration declaration in order to get the correct enum values as assigned by the C compiler.

The `%constant` directive is used to more precisely create constants corresponding to different C datatypes. Although it is not usually needed for simple values, it is more useful when working with pointers and other more complex datatypes. Typically, `%constant` is only used when you want to add constants to the scripting language interface that are not defined in the original header file.

> 允许使用常量表达式，但 SWIG 不会对它们进行求值。相反，它将它们传递给输出文件，并让 C 编译器执行最终求值（但 SWIG 确实执行了有限形式的类型检查）。
>
> 对于枚举，将原始枚举定义包含在接口文件中的某个位置（在头文件或 `%{ %}` 块中）至关重要。SWIG 仅将枚举转换为向脚本语言添加常量所需的代码。它需要原始的枚举声明才能获得 C 编译器指定的正确枚举值。
>
> `%constant` 指令用于更精确地创建与不同 C 数据类型对应的常量。虽然简单值通常不需要它，但在使用指针和其他更复杂的数据类型时更有用。通常，只有当你想要向脚本语言接口添加原始头文件中未定义的常量时才使用 `%constant`。

### 5.2.4 一点关于 `const` 的文字

A common confusion with C programming is the semantic meaning of the `const` qualifier in declarations--especially when it is mixed with pointers and other type modifiers. In fact, previous versions of SWIG handled `const` incorrectly--a situation that SWIG-1.3.7 and newer releases have fixed.

Starting with SWIG-1.3, all variable declarations, regardless of any use of `const`, are wrapped as global variables. If a declaration happens to be declared as `const`, it is wrapped as a read-only variable. To tell if a variable is `const` or not, you need to look at the right-most occurrence of the `const` qualifier (that appears before the variable name). If the right-most `const` occurs after all other type modifiers (such as pointers), then the variable is `const`. Otherwise, it is not.

Here are some examples of `const` declarations.

> C 编程的一个常见疑惑是声明中 `const` 限定符的语义含义——特别是当它与指针和其他类型修饰符混合时。实际上，早期版本的 SWIG 错误地处理了 `const`，这已在 SWIG-1.3.7 和更新版本修复。
>
> 从 SWIG-1.3 开始，所有变量声明，无论怎样使用 `const`，都被包装为全局变量。如果声明恰好声明为 `const`，则它将被包装为只读变量。要判断一个变量是否为 `const`，你需要查看最右边出现的 `const` 限定符（出现在变量名之前）。如果最右边的 `const` 出现在所有其他类型修饰符（例如指针）之后，那么变量是 `const`。否则，事实并非如此。
>
> 以下是 `const` 声明的一些示例。

```c
const char a;           // A constant character
char const b;           // A constant character (the same)
char *const c;          // A constant pointer to a character
const char *const d;    // A constant pointer to a constant character
```

Here is an example of a declaration that is not `const`:

> 不是 `const` 声明的一些示例：

```c
const char *e;          // A pointer to a constant character.  The pointer
                        // may be modified.
```

In this case, the pointer `e` can change---it's only the value being pointed to that is read-only.

Please note that for const parameters or return types used in a function, SWIG pretty much ignores the fact that these are const, see the section on [const-correctness](http://www.swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_const) for more information.

**Compatibility Note:** One reason for changing SWIG to handle `const` declarations as read-only variables is that there are many situations where the value of a `const` variable might change. For example, a library might export a symbol as `const` in its public API to discourage modification, but still allow the value to change through some other kind of internal mechanism. Furthermore, programmers often overlook the fact that with a constant declaration like `char *const`, the underlying data being pointed to can be modified--it's only the pointer itself that is constant. In an embedded system, a `const`declaration might refer to a read-only memory address such as the location of a memory-mapped I/O device port (where the value changes, but writing to the port is not supported by the hardware). Rather than trying to build a bunch of special cases into the `const`qualifier, the new interpretation of `const` as "read-only" is simple and exactly matches the actual semantics of `const` in  C/C++ . If you really want to create a constant as in older versions of SWIG, use the `%constant` directive instead. For example:

> 在这种情况下，指针 `e` 可以改变——只是它指向的值是只读的。
>
> 请注意，对于函数中使用的 `const` 参数或返回类型，SWIG 几乎忽略了这些是 `const` 的事实，请参阅 [`const` 正确性](http://www.swig.org/Doc3.0/SWIGPlus.html＃SWIGPlus_const)部分了解更多信息。
>
> **注意兼容性：**更改 SWIG 以将 `const` 声明作为只读变量处理的一个原因是，在很多情况下，`const` 变量的值可能会发生变化。例如，库可能会在其公共 API 中将符号导出为 `const` 以阻止修改，但仍允许通过其他类型的内部机制更改该值。此外，程序员经常忽略这样一个事实：使用像 `char *const` 这样的常量声明，可以修改指向的底层数据——它只是指针本身是常量。在嵌入式系统中，`const` 声明可能指的是只读存储器地址，例如内存映射 I/O 设备端口的位置（值发生变化，但硬件不支持写入端口）。不是试图在 `const` 修饰符中构建一堆特殊情况，而是将 `const` 作为“只读”的新解释很简单，并且与 C/C++ 中 `const` 的实际语义完全匹配。如果你真的想在旧版本的 SWIG 中创建一个常量，请改用 `%constant` 指令。例如：

```
%constant double PI = 3.14159;
```

or

> 或

```
#ifdef SWIG
#define const %constant
#endif
const double foo = 3.4;
const double bar = 23.4;
const int    spam = 42;
#ifdef SWIG
#undef const
#endif
...
```

### 5.2.5 `char *` 的注意事项

Before going any further, there is one bit of caution involving `char *` that must now be mentioned. When strings are passed from a scripting language to a C `char *`, the pointer usually points to string data stored inside the interpreter. It is almost always a really bad idea to modify this data. Furthermore, some languages may explicitly disallow it. For instance, in Python, strings are supposed to be immutable. If you violate this, you will probably receive a vast amount of wrath when you unleash your module on the world.

The primary source of problems are functions that might modify string data in place. A classic example would be a function like this:

> 在继续之前，必须提到一点 `char *` 的注意事项。当字符串从脚本语言传递到 C `char *` 时，指针通常指向存储在解释器中的字符串数据。修改这些数据几乎总是一个坏主意。此外，某些语言可能明确禁止它。例如，在 Python 中，字符串应该是不可变的。如果你违反了这一点，那么当你发布你的模块时，你可能会收到一大波愤怒的意见。
>
> 问题的主要来源是可能会修改字符串数据的函数。一个典型的例子是这样的函数：

```c
char *strcat(char *s, const char *t)
```

Although SWIG will certainly generate a wrapper for this, its behavior will be undefined. In fact, it will probably cause your application to crash with a segmentation fault or other memory related problem. This is because `s` refers to some internal data in the target language--data that you shouldn't be touching.

The bottom line: don't rely on `char *` for anything other than read-only input values. However, it must be noted that you could change the behavior of SWIG using [typemaps](http://www.swig.org/Doc3.0/Typemaps.html#Typemaps).

> 虽然 SWIG 肯定会为此生成一个包装器，但它的行为将是未定义的。实际上，它可能会导致应用程序崩溃，导致分段错误或其他与内存相关的问题。这是因为 `s` 指向的是目标语言中的一些内部数据——你不应该碰的数据。
>
> 底线：除了只读输入值之外，不要依赖 `char *`。但是，必须注意，你可以使用[类型映射](http://www.swig.org/Doc3.0/Typemaps.html#Typemaps)更改 SWIG 的行为。

## 5.3 指针与复杂对象

Most C programs manipulate arrays, structures, and other types of objects. This section discusses the handling of these datatypes.

> 大多数 C 程序会操纵数组、结构体和其他类型的对象。本节讨论这些数据类型的处理。

### 5.3.1 简单指针

Pointers to primitive C datatypes such as

> SWIG 完全支持指向原始 C 数据类型的指针。

```c
int *
double ***
char **
```

are fully supported by SWIG. Rather than trying to convert the data being pointed to into a scripting representation, SWIG simply encodes the pointer itself into a representation that contains the actual value of the pointer and a type-tag. Thus, the SWIG representation of the above pointers (in Tcl), might look like this:

> SWIG 不试图将指向的数据转换为脚本表示，而是简单地将指针本身编码为包含指针实际值和类型标记的表示。因此，上述指针（在 Tcl 中）的 SWIG 表示可能如下所示：

```tcl
_10081012_p_int
_1008e124_ppp_double
_f8ac_pp_char
```

A NULL pointer is represented by the string "NULL" or the value 0 encoded with type information.

All pointers are treated as opaque objects by SWIG. Thus, a pointer may be returned by a function and passed around to other C functions as needed. For all practical purposes, the scripting language interface works in exactly the same way as you would use the pointer in a C program. The only difference is that there is no mechanism for dereferencing the pointer since this would require the target language to understand the memory layout of the underlying object.

The scripting language representation of a pointer value should never be manipulated directly. Even though the values shown look like hexadecimal addresses, the numbers used may differ from the actual machine address (e.g., on little-endian machines, the digits may appear in reverse order). Furthermore, SWIG does not normally map pointers into high-level objects such as associative arrays or lists (for example, converting an `int *` into an list of integers). There are several reasons why SWIG does not do this:

* There is not enough information in a C declaration to properly map pointers into higher level constructs. For example, an `int *` may indeed be an array of integers, but if it contains ten million elements, converting it into a list object is probably a bad idea.
* The underlying semantics associated with a pointer is not known by SWIG. For instance, an `int *` might not be an array at all--perhaps it is an output value!
* By handling all pointers in a consistent manner, the implementation of SWIG is greatly simplified and less prone to error.

> NULL 指针由字符串 `NULL` 或用类型信息编码的值 `0` 表示。
>
> SWIG 将所有指针视为不透明对象。因此，指针可以由函数返回并根据需要传递给其他 C 函数。出于所有实际目的，脚本语言接口的工作方式与在 C 程序中使用指针的方式完全相同。唯一的区别是没有解引用指针的机制，因为这需要目标语言来理解底层对象的内存布局。
>
> 永远不要直接操作指针值的脚本语言表示。尽管所示的值看起来像十六进制地址，但所使用的数字可能与实际的机器地址不同（例如，在小端机器上，数字可能以相反的顺序出现）。此外，SWIG 通常不会将指针映射到高级对象，例如关联数组或列表（例如，将 `int *` 转换为整数列表）。SWIG 不这样做有几个原因：
>
> * C 声明中没有足够的信息来将指针正确映射到更高级别的结构。例如，`int *` 可能确实是一个整数数组，但如果它包含一千万个元素，将它转换为一个列表对象可能是一个坏主意。
> * SWIG 不知道与指针相关的基础语义。例如，`int *` 可能根本不是一个数组——也许它是一个输出值！
> * 通过以一致的方式处理所有指针，SWIG 的实现大大简化并且不容易出错。

### 5.3.2 运行时指针类型检查

By allowing pointers to be manipulated from a scripting language, extension modules effectively bypass compile-time type checking in the C/C++ compiler. To prevent errors, a type signature is encoded into all pointer values and is used to perform run-time type checking. This type-checking process is an integral part of SWIG and can not be disabled or modified without using typemaps (described in later chapters).

Like C, `void *` matches any kind of pointer. Furthermore, `NULL` pointers can be passed to any function that expects to receive a pointer. Although this has the potential to cause a crash, `NULL` pointers are also sometimes used as sentinel values or to denote a missing/empty value. Therefore, SWIG leaves NULL pointer checking up to the application.

> 通过允许从脚本语言操作指针，扩展模块有效地绕过了 C/C++ 编译器中的编译时类型检查。为了防止错误，类型签名被编码到所有指针值中，并用于执行运行时类型检查。此类型检查过程是 SWIG 的组成部分，不使用类型映射就无法禁用或修改（在后面的章节中有介绍）。
>
> 像 C 一样，`void *` 匹配任何类型的指针。此外，NULL 指针可以传递给任何期望接受指针的函数。虽然这有可能导致崩溃，但 NULL 指针有时也用作标记值或表示缺失/空值。因此，SWIG 将 NULL 指针对应到应用程序。

### 5.3.3 派生类型、结构体和类

For everything else (structs, classes, arrays, etc...) SWIG applies a very simple rule :

**Everything else is a pointer**

In other words, SWIG manipulates everything else by reference. This model makes sense because most C/C++ programs make heavy use of pointers and SWIG can use the type-checked pointer mechanism already present for handling pointers to basic datatypes.

Although this probably sounds complicated, it's really quite simple. Suppose you have an interface file like this :

> 对于其他一切（结构体、类、数组等），SWIG 应用了一个非常简单的规则：
>
> **其他一切皆是指针**
>
> 换句话说，SWIG 通过引用操纵其他所有内容。这个模型很有意义，因为大多数 C/C++ 程序都大量使用指针，SWIG 可以使用已经存在的指针类型检查机制来处理指向基本数据类型的指针。
>
> 虽然这可能听起来很复杂，但它确实非常简单。假设你有一个这样的接口文件：

```
%module fileio
FILE *fopen(char *, char *);
int fclose(FILE *);
unsigned fread(void *ptr, unsigned size, unsigned nobj, FILE *);
unsigned fwrite(void *ptr, unsigned size, unsigned nobj, FILE *);
void *malloc(int nbytes);
void free(void *);
```

In this file, SWIG doesn't know what a `FILE` is, but since it's used as a pointer, so it doesn't really matter what it is. If you wrapped this module into Python, you can use the functions just like you expect :

> 在这个文件中，SWIG 不知道 `FILE` 是什么，但由于它被用作指针，所以它并不重要。如果将此模块包装到 Python 中，则可以像预期的那样使用这些函数：

```python
# Copy a file
def filecopy(source, target):
    f1 = fopen(source, "r")
    f2 = fopen(target, "w")
    buffer = malloc(8192)
    nbytes = fread(buffer, 8192, 1, f1)
    while (nbytes > 0):
        fwrite(buffer, 8192, 1, f2)
        nbytes = fread(buffer, 8192, 1, f1)
    free(buffer)
```

In this case `f1`, `f2`, and `buffer` are all opaque objects containing C pointers. It doesn't matter what value they contain--our program works just fine without this knowledge.

> 在这种情况下，`f1`、`f2` 和 `buffer` 都是包含 C 指针的不透明对象。它们包含什么值并不重要——我们的程序在没有这些知识的情况下工作得很好。

### 5.3.4 未定义数据类型

When SWIG encounters an undeclared datatype, it automatically assumes that it is a structure or class. For example, suppose the following function appeared in a SWIG input file:

> 当 SWIG 遇到未声明的数据类型时，它会自动假定它是结构体或类。例如，假设 SWIG 输入文件中出现以下函数：

```c
void matrix_multiply(Matrix *a, Matrix *b, Matrix *c);
```

SWIG has no idea what a "`Matrix`" is. However, it is obviously a pointer to something so SWIG generates a wrapper using its generic pointer handling code.

Unlike C or C++, SWIG does not actually care whether `Matrix` has been previously defined in the interface file or not. This allows SWIG to generate interfaces from only partial or limited information. In some cases, you may not care what a `Matrix` really is as long as you can pass an opaque reference to one around in the scripting language interface.

An important detail to mention is that SWIG will gladly generate wrappers for an interface when there are unspecified type names. However, **all unspecified types are internally handled as pointers to structures or classes!** For example, consider the following declaration:

> SWIG 不知道 `Matrix` 是什么。但是，它显然是指向某事物的指针，因此 SWIG 使用其通用指针处理代码生成包装器。
>
> 与 C 或 C++ 不同，SWIG 实际上并不关心先前是否在接口文件中定义了 `Matrix`。这允许 SWIG 仅从部分或有限信息生成接口。在某些情况下，只要你可以在脚本语言接口中传递一个不透明的引用，你可能不关心 `Matrix` 是什么。
>
> 需要注意的一个重要细节是，当存在未指定的类型名称时，SWIG 将很乐意为接口生成包装器。但是，**所有未指定的类型都在内部处理为指向结构体或类的指针！**例如，请考虑以下声明：

```c
void foo(size_t num);
```

If `size_t` is undeclared, SWIG generates wrappers that expect to receive a type of `size_t *` (this mapping is described shortly). As a result, the scripting interface might behave strangely. For example:

> 如果 `size_t` 未声明，SWIG 会生成期望接收 `size_t *` 类型的包装器（稍后将描述此映射）。因此，脚本接口可能会表现得很奇怪。例如：

```
foo(40);
TypeError: expected a _p_size_t.
```

The only way to fix this problem is to make sure you properly declare type names using `typedef`.

> 解决此问题的唯一方法是确保使用 `typedef` 正确声明类型名称。

### 5.3.5 `typedef`

Like C, `typedef` can be used to define new type names in SWIG. For example:

> 与 C 一样，`typedef` 可用于在 SWIG 中定义新的类型名称。例如：

```c
typedef unsigned int size_t;
```

`typedef` definitions appearing in a SWIG interface are not propagated to the generated wrapper code. Therefore, they either need to be defined in an included header file or placed in the declarations section like this:

> 出现在 SWIG 接口中的 `typedef` 定义不会传播到生成的包装器代码。因此，它们需要在包含的头文件中定义，或者放在声明部分中，如下所示：

```
%{
/* Include in the generated wrapper file */
typedef unsigned int size_t;
%}
/* Tell SWIG about it */
typedef unsigned int size_t;
```

or

> 或

```
%inline %{
typedef unsigned int size_t;
%}
```

In certain cases, you might be able to include other header files to collect type information. For example:

> 在某些情况下，你可能能够包含其他头文件来收集类型信息。例如：

```
%module example
%import "sys/types.h"
```

In this case, you might run SWIG as follows:

> 这种情况下，你要如下运行 SWIG：

```
$ swig -I/usr/include -includeall example.i
```

It should be noted that your mileage will vary greatly here. System headers are notoriously complicated and may rely upon a variety of non-standard C coding extensions (e.g., such as special directives to GCC). Unless you exactly specify the right include directories and preprocessor symbols, this may not work correctly (you will have to experiment).

SWIG tracks `typedef` declarations and uses this information for run-time type checking. For instance, if you use the above `typedef` and had the following function declaration:

> 应该注意的是，你的里程在这里会有很大变化。系统头文件众所周知地复杂并且可能依赖于各种非标准 C 编码扩展（例如，诸如对 GCC 的特殊指令）。除非你确切地指定了正确的包含目录和预处理程序符号，否则这可能无法正常工作（你必须进行实验）。
>
> SWIG 跟踪 `typedef` 声明并将此信息用于运行时类型检查。例如，如果你使用上面的 `typedef` 并具有以下函数声明：

```c
void foo(unsigned int *ptr);
```

The corresponding wrapper function will accept arguments of type `unsigned int *` or `size_t *`.

> 相应的包装函数将接受类型为 `unsigned int *` 或 `size_t *` 的参数。

## 5.4 其他实操指南

So far, this chapter has presented almost everything you need to know to use SWIG for simple interfaces. However, some C programs use idioms that are somewhat more difficult to map to a scripting language interface. This section describes some of these issues.

> 到目前为止，本章介绍了使用 SWIG 进行简单接口时需要了解的几乎所有内容。但是，一些 C 程序使用的惯用法更难以映射到脚本语言接口。本节介绍其中一些问题。

### 5.4.1 通过值解析结构体

Sometimes a C function takes structure parameters that are passed by value. For example, consider the following function:

> 有时，C 函数采用按值传递的结构体参数。例如，请考虑以下函数：

```c
double dot_product(Vector a, Vector b);
```

To deal with this, SWIG transforms the function to use pointers by creating a wrapper equivalent to the following:

> 为了解决这个问题，SWIG 通过创建一个等效于以下内容的包装器来转换函数以使用指针：

```c
double wrap_dot_product(Vector *a, Vector *b) {
    Vector x = *a;
    Vector y = *b;
    return dot_product(x, y);
}
```

In the target language, the `dot_product()` function now accepts pointers to Vectors instead of Vectors. For the most part, this transformation is transparent so you might not notice.

> 在目标语言中，`dot_product()` 函数现在接受指向 `Vector` 的指针而不是 `Vector`。在大多数情况下，这种转变是透明的，因此你可能不会注意到。

### 5.4.2 返回值

C functions that return structures or classes datatypes by value are more difficult to handle. Consider the following function:

> 按值返回结构体或类数据类型的 C 函数更难处理。考虑以下函数：

```c
Vector cross_product(Vector v1, Vector v2);
```

This function wants to return `Vector`, but SWIG only really supports pointers. As a result, SWIG creates a wrapper like this:

> 这个函数想要返回 `Vector`，但 SWIG 只支持指针。因此，SWIG 创建了一个这样的包装器：

```c
Vector *wrap_cross_product(Vector *v1, Vector *v2) {
        Vector x = *v1;
        Vector y = *v2;
        Vector *result;
        result = (Vector *) malloc(sizeof(Vector));
        *(result) = cross(x, y);
        return result;
}
```

or if SWIG was run with the `-c++` option:

> 如果在 `-c++` 模式下运行 SWIG：

```c++
Vector *wrap_cross(Vector *v1, Vector *v2) {
        Vector x = *v1;
        Vector y = *v2;
        Vector *result = new Vector(cross(x, y)); // Uses default copy constructor
        return result;
}
```

In both cases, SWIG allocates a new object and returns a reference to it. It is up to the user to delete the returned object when it is no longer in use. Clearly, this will leak memory if you are unaware of the implicit memory allocation and don't take steps to free the result. That said, it should be noted that some language modules can now automatically track newly created objects and reclaim memory for you. Consult the documentation for each language module for more details.

It should also be noted that the handling of pass/return by value in C++ has some special cases. For example, the above code fragments don't work correctly if `Vector` doesn't define a default constructor. The section on SWIG and C++ has more information about this case.

> 在这两种情况下，SWIG 都会分配一个新对象并返回对它的引用。用户在不再使用时删除返回的对象。显然，如果你不知道隐式内存分配并且不采取措施来释放结果，这将导致内存泄漏。也就是说，应该注意的是，某些语言模块现在可以自动跟踪新创建的对象并为你回收内存。有关更多详细信息，请参阅每个语言模块的文档。
>
> 还应该注意的是，C++ 中按值传递/返回的处理有一些特殊情况。例如，如果 `Vector` 没有定义默认构造函数，则上面的代码片段无法正常工作。SWIG 和 C++ 的部分有关于此案例的更多信息。

### 5.4.3 链接结构体变量

When global variables or class members involving structures are encountered, SWIG handles them as pointers. For example, a global variable like this

> 当遇到涉及结构体的全局变量或类成员时，SWIG 将它们作为指针处理。例如，像这样的全局变量

```c
Vector unit_i;
```

gets mapped to an underlying pair of set/get functions like this :

> 被映射到一对底层的 set/get 函数，如下所示：

```c
Vector *unit_i_get() {
  return &unit_i;
}

void unit_i_set(Vector *value) {
  unit_i = *value;
}
```

Again some caution is in order. A global variable created in this manner will show up as a pointer in the target scripting language. It would be an extremely bad idea to free or destroy such a pointer. Also, C++ classes must supply a properly defined copy constructor in order for assignment to work correctly.

> 小心再三。以这种方式创建的全局变量将显示为目标脚本语言中的指针。释放或销毁这样的指针是一个非常糟糕的主意。此外，C++ 类必须提供正确定义的复制构造函数，以使赋值正常工作。

### 5.4.4 链接到 `char *`

When a global variable of type `char *` appears, SWIG uses `malloc()`or `new` to allocate memory for the new value. Specifically, if you have a variable like this

> 当出现 `char *` 类型的全局变量时，SWIG 使用 `malloc()` 或 `new` 为新值分配内存。具体来说，如果你有这样的变量

```c
char *foo;
```

SWIG generates the following code:

> SWIG 产生如下代码：

```c
/* C mode */
void foo_set(char *value) {
  if (foo) free(foo);
  foo = (char *) malloc(strlen(value)+1);
  strcpy(foo, value);
}

/* C++ mode.  When -c++ option is used */
void foo_set(char *value) {
  if (foo) delete [] foo;
  foo = new char[strlen(value)+1];
  strcpy(foo, value);
}
```

If this is not the behavior that you want, consider making the variable read-only using the `%immutable` directive. Alternatively, you might write a short assist-function to set the value exactly like you want. For example:

> 如果这不是你想要的行为，请考虑使用 `%immutable` 指令将变量设置为只读。或者，你可以编写一个简短的辅助函数来完全按照你的意愿设置值。例如：

```
%inline %{
  void set_foo(char *value) {
    strncpy(foo, value, 50);
  }
%}
```

Note: If you write an assist function like this, you will have to call it as a function from the target scripting language (it does not work like a variable). For example, in Python you will have to write:

> **注意：**如果你编写这样的辅助函数，则必须将其作为函数从目标脚本语言中调用（它不像变量那样工作）。例如，在 Python 中你必须写：

```python
>>> set_foo("Hello World")
```

A common mistake with `char *` variables is to link to a variable declared like this:

> `char *` 变量的一个常见错误是链接到这样声明的变量：

```c
char *VERSION = "1.0";
```

In this case, the variable will be readable, but any attempt to change the value results in a segmentation or general protection fault. This is due to the fact that SWIG is trying to release the old value using `free`or `delete` when the string literal value currently assigned to the variable wasn't allocated using `malloc()` or `new`. To fix this behavior, you can either mark the variable as read-only, write a typemap (as described in Chapter 6), or write a special set function as shown. Another alternative is to declare the variable as an array:

> 在这种情况下，变量将是可读的，但任何更改值的尝试都会导致分段或一般保护错误。这是因为当使用 `malloc()` 或 `new` 分配当前分配给变量的字符串文字值时，SWIG 正试图使用 `free` 或 `delete` 释放旧值。要解决此问题，你可以将变量标记为只读，编写类型映射（如第 6 章所述），或者编写一个特殊的 set 函数。另一种方法是将变量声明为数组：

```c
char VERSION[64] = "1.0";
```

When variables of type `const char *` are declared, SWIG still generates functions for setting and getting the value. However, the default behavior does *not* release the previous contents (resulting in a possible memory leak). In fact, you may get a warning message such as this when wrapping such a variable:

> 当声明类型为 `const char *` 的变量时，SWIG 仍会生成用于设置和获取值的函数。但是，默认行为*不*释放先前的内容（导致可能的内存泄漏）。实际上，在包装这样的变量时，你可能会收到一条警告消息：

```
example.i:20. Typemap warning. Setting const char * variable may leak memory
```

The reason for this behavior is that `const char *` variables are often used to point to string literals. For example:

> 这种行为的原因是 `const char *` 变量通常用于指向字符串文字。例如：

```c
const char *foo = "Hello World\n";
```

Therefore, it's a really bad idea to call `free()` on such a pointer. On the other hand, it *is* legal to change the pointer to point to some other value. When setting a variable of this type, SWIG allocates a new string (using malloc or new) and changes the pointer to point to the new value. However, repeated modifications of the value will result in a memory leak since the old value is not released.

> 因此，在这样的指针上调用 `free()` 是一个非常糟糕的主意。另一方面，将指针更改为指向某个其他值是合法的。设置此类型的变量时，SWIG 会分配一个新字符串（使用 `malloc` 或 `new`）并将指针更改为指向新值。但是，重复修改该值将导致内存泄漏，因为旧值未释放。

### 5.4.5 数组

Arrays are fully supported by SWIG, but they are always handled as pointers instead of mapping them to a special array object or list in the target language. Thus, the following declarations :

> SWIG 完全支持数组，但它们总是作为指针处理，而不是将它们映射到目标语言中的特殊数组对象或列表。因此，以下声明：

```c
int foobar(int a[40]);
void grok(char *argv[]);
void transpose(double a[20][20]);
```

are processed as if they were really declared like this:

> 的处理好像它们真的被声明为如下这样：

```c
int foobar(int *a);
void grok(char **argv);
void transpose(double (*a)[20]);
```

Like C, SWIG does not perform array bounds checking. It is up to the user to make sure the pointer points to a suitably allocated region of memory.

Multi-dimensional arrays are transformed into a pointer to an array of one less dimension. For example:

> 与 C 一样，SWIG 不执行数组边界检查。用户可以确保指针指向适当分配的内存区域。
>
> 多维数组被转换为指向较少维度的数组的指针。例如：

```c
int [10];         // Maps to int *
int [10][20];     // Maps to int (*)[20]
int [10][20][30]; // Maps to int (*)[20][30]
```

It is important to note that in the C type system, a multidimensional array `a[][]` is **NOT** equivalent to a single pointer `*a` or a double pointer such as `**a`. Instead, a pointer to an array is used (as shown above) where the actual value of the pointer is the starting memory location of the array. The reader is strongly advised to dust off their C book and re-read the section on arrays before using them with SWIG.

Array variables are supported, but are read-only by default. For example:

> 值得注意的是，在 C 类型系统中，多维数组 `a[][]` 是不等价于单个指针 `*a` 或双指针如 `**a`。而是使用指向数组的指针（如上所示），其中指针的实际值是数组的起始内存位置。强烈建议读者在使用 SWIG 之前拿出尘封的 C 语言书，并重新阅读数组部分。
>
> 支持数组变量，但默认情况下它们是只读的。例如：

```c
int a[100][200];
```

In this case, reading the variable 'a' returns a pointer of type `int (*)[200]` that points to the first element of the array `&a[0][0]`. Trying to modify 'a' results in an error. This is because SWIG does not know how to copy data from the target language into the array. To work around this limitation, you may want to write a few simple assist functions like this:

> 在这种情况下，读取变量 `a` 会返回一个类型为 `int (*)[200]` 的指针，指向数组的第一个元素 `&a[0][0]`。试图修改 `a` 会导致错误。这是因为 SWIG 不知道如何将目标语言中的数据复制到数组中。要解决此限制，你可能需要编写一些简单的辅助函数，如下所示：

```
%inline %{
void a_set(int i, int j, int val) {
  a[i][j] = val;
}
int a_get(int i, int j) {
  return a[i][j];
}
%}
```

To dynamically create arrays of various sizes and shapes, it may be useful to write some helper functions in your interface. For example:

> 要动态创建各种大小和形状的数组，在接口中编写一些辅助函数可能很有用。例如：

```
// Some array helpers
%inline %{
  /* Create any sort of [size] array */
  int *int_array(int size) {
    return (int *) malloc(size*sizeof(int));
  }
  /* Create a two-dimension array [size][10] */
  int (*int_array_10(int size))[10] {
    return (int (*)[10]) malloc(size*10*sizeof(int));
  }
%}
```

Arrays of `char` are handled as a special case by SWIG. In this case, strings in the target language can be stored in the array. For example, if you have a declaration like this,

> `char` 数组由 SWIG 作为特例处理。在这种情况下，目标语言中的字符串可以存储在数组中。例如，如果你有这样的声明，

```c
char pathname[256];
```

SWIG generates functions for both getting and setting the value that are equivalent to the following code:

> SWIG 生成用于获取和设置值的函数，这些函数等效于以下代码：

```c
char *pathname_get() {
  return pathname;
}

void pathname_set(char *value) {
  strncpy(pathname, value, 256);
}
```

In the target language, the value can be set like a normal variable.

> 在目标语言中，可以将值设置为普通变量。

### 5.4.6 创建只读变量

A read-only variable can be created by using the `%immutable` directive as shown :

> 可以使用 `%immutable` 指令创建只读变量，如下所示：

```
// File : interface.i

int a;       // Can read/write
%immutable;
int b, c, d;   // Read only variables
%mutable;
double x, y;  // read/write
```

The `%immutable` directive enables read-only mode until it is explicitly disabled using the `%mutable` directive. As an alternative to turning read-only mode off and on like this, individual declarations can also be tagged as immutable. For example:

> `%immutable` 指令启用只读模式，除非使用 `%mutable` 指令显式禁用它。作为这种关闭和打开只读模式的替代方法，单个声明也可以标记为不可变。例如：

```
%immutable x;                   // Make x read-only
...
double x;                       // Read-only (from earlier %immutable directive)
double y;                       // Read-write
...
```

The `%mutable` and `%immutable` directives are actually [%feature directives](http://www.swig.org/Doc3.0/Customization.html#Customization_features) defined like this:

> `%mutable` 和 `%immutable` 实际上是如下定义的 [`%feature` 指令](http://www.swig.org/Doc3.0/Customization.html#Customization_features)：

```
#define %immutable   %feature("immutable")
#define %mutable     %feature("immutable", "")
```

If you wanted to make all wrapped variables read-only, barring one or two, it might be easier to take this approach:

> 如果你想将所有包装变量设为只读，而不是一两个，否则采用这种方法可能更容易：

```
%immutable;                     // Make all variables read-only
%feature("immutable", "0") x;   // except, make x read/write
...
double x;
double y;
double z;
...
```

Read-only variables are also created when declarations are declared as `const`. For example:

> 当声明声明为 `const` 时也会创建只读变量。例如：

```c
const int foo;               /* Read only variable */
char * const version="1.0";  /* Read only variable */
```

**Compatibility note:** Read-only access used to be controlled by a pair of directives `%readonly` and `%readwrite`. Although these directives still work, they generate a warning message. Simply change the directives to `%immutable;` and `%mutable;` to silence the warning. Don't forget the extra semicolon!

> **注意兼容性：**只读访问过去由一对指令 `%readonly` 和 `%readwrite` 控制。尽管这些指令仍然有效，但它们会生成警告消息。只需将指令更改为 `%immutable;` 和 `%mutable;` 即可使警告静音。**不要忘记额外的分号！**

### 5.4.7 重命名与忽略声明

#### 5.4.7.1 特定标识符的简单重命名

Normally, the name of a C declaration is used when that declaration is wrapped into the target language. However, this may generate a conflict with a keyword or already existing function in the scripting language. To resolve a name conflict, you can use the `%rename`directive as shown :

> 通常，当声明包装到目标语言中时，将使用 C 声明的名称。但是，这可能会与脚本语言中的关键字或已存在的函数产生冲突。要解决名称冲突，可以使用 `%rename` 指令，如下所示：

```
// interface.i

%rename(my_print) print;
extern void print(const char *);

%rename(foo) a_really_long_and_annoying_name;
extern int a_really_long_and_annoying_name;
```

SWIG still calls the correct C function, but in this case the function `print()` will really be called "`my_print()`" in the target language.

The placement of the `%rename` directive is arbitrary as long as it appears before the declarations to be renamed. A common technique is to write code for wrapping a header file like this:

> SWIG 仍然调用正确的 C 函数，但在这种情况下，函数 `print()` 将在目标语言中被称为 `my_print()`。
>
> `%rename` 指令的放置是任意的，只要它出现在要重命名的声明之前。一种常见的技术是编写用于包装头文件的代码，如下所示：

```
// interface.i

%rename(my_print) print;
%rename(foo) a_really_long_and_annoying_name;

%include "header.h"
```

`%rename` applies a renaming operation to all future occurrences of a name. The renaming applies to functions, variables, class and structure names, member functions, and member data. For example, if you had two-dozen C++ classes, all with a member function named `print' (which is a keyword in Python), you could rename them all to `output' by specifying :

> `%rename` 对所有将来出现的名称应用重命名操作。重命名适用于函数、变量、类和结构体名称、成员函数和成员数据。例如，如果你有二十几个 C++ 类，都有一个名为 `print` 的成员函数（它是 Python 中的一个关键字），你可以通过指定它们将它们全部重命名为 `output`：

```
%rename(output) print; // Rename all 'print' functions to 'output'
```

SWIG does not normally perform any checks to see if the functions it wraps are already defined in the target scripting language. However, if you are careful about namespaces and your use of modules, you can usually avoid these problems.

Closely related to `%rename` is the `%ignore` directive. `%ignore` instructs SWIG to ignore declarations that match a given identifier. For example:

> SWIG 通常不执行任何检查以查看它包装的函数是否已在目标脚本语言中定义。但是，如果你对命名空间和模块的使用非常小心，通常可以避免这些问题。
>
> 与 `%rename` 密切相关的是 `%ignore` 指令。`%ignore` 指示 SWIG 忽略与给定标识符匹配的声明。例如：

```
%ignore print;         // Ignore all declarations named print
%ignore MYMACRO;       // Ignore a macro
...
#define MYMACRO 123
void print(const char *);
...
```

Any function, variable etc which matches `%ignore` will not be wrapped and therefore will not be available from the target language. A common usage of `%ignore` is to selectively remove certain declarations from a header file without having to add conditional compilation to the header. However, it should be stressed that this only works for simple declarations. If you need to remove a whole section of problematic code, the SWIG preprocessor should be used instead.

**Compatibility note:** Older versions of SWIG provided a special `%name`directive for renaming declarations. For example:

> 任何与 `%ignore` 匹配的函数、变量等都不会被包装，因此无法在目标语言中使用。`%ignore` 的一个常见用法是有选择地从头文件中删除某些声明，而不必向头部添加条件编译。但是，应该强调的是，这仅适用于简单的声明。如果你需要删除整个有问题的代码段，则应使用 SWIG 预处理器。
>
> **注意兼容性：**旧版本的 SWIG 为重命名声明提供了一个特殊的 `%name` 指令。例如：

```
%name(output) extern void print(const char *);
```

This directive is still supported, but it is deprecated and should probably be avoided. The `%rename` directive is more powerful and better supports wrapping of raw header file information.

> 仍然支持该指令，但它已被弃用，应该避免使用。`%rename` 指令功能更强大、更好地支持包装原始头文件信息。

#### 5.4.7.2 高级重命名支持

While writing `%rename` for specific declarations is simple enough, sometimes the same renaming rule needs to be applied to many, maybe all, identifiers in the SWIG input. For example, it may be necessary to apply some transformation to all the names in the target language to better follow its naming conventions, like adding a specific prefix to all wrapped functions. Doing it individually for each function is impractical so SWIG supports applying a renaming rule to all declarations if the name of the identifier to be renamed is not specified:

> 虽然为特定声明编写 `%rename` 很简单，但有时需要将相同的重命名规则应用于 SWIG 输入中的许多（可能是全部）标识符。例如，可能需要对目标语言中的所有名称应用某些转换，以便更好地遵循其命名约定，例如向所有包装器函数添加特定前缀。为每个函数单独执行操作是不切实际的，因此如果未指定要重命名的标识符的名称，SWIG 支持将重命名规则应用于所有声明：

```
%rename("myprefix_%s") ""; // print -> myprefix_print
```

This also shows that the argument of `%rename` doesn't have to be a literal string but can be a `printf()`-like format string. In the simplest form, `"%s"` is replaced with the name of the original declaration, as shown above. However this is not always enough and SWIG provides extensions to the usual format string syntax to allow applying a (SWIG-defined) function to the argument. For example, to wrap all C functions `do_something_long()` as more Java-like `doSomethingLong()` you can use the `"lowercamelcase"` extended format specifier like this:

> 这也表明 `%rename` 的参数不必是文字字符串，但可以是 `printf()` 类型格式的字符串。在最简单的形式中，`%s` 被替换为原始声明的名称，如上所示。然而，这并不总是足够的，并且 SWIG 提供了对通常的格式字符串语法的扩展，以允许将（SWIG 定义的）函数应用于参数。例如，要将所有 C 函数 `do_something_long()` 包装为更像 Java 的 `doSomethingLong()`，你可以使用这样的 `lowercamelcase` 扩展格式说明符：

```
%rename("%(lowercamelcase)s") ""; // foo_bar -> fooBar; FooBar -> fooBar
```

Some functions can be parametrized, for example the `"strip"` one strips the provided prefix from its argument. The prefix is specified as part of the format string, following a colon after the function name:

> 某些函数可以进行参数化，例如 `strip` 从其参数中剥离提供的前缀。在函数名称后跟冒号后，前缀被指定为格式字符串的一部分：

```
%rename("%(strip:[wx])s") ""; // wxHello -> Hello; FooBar -> FooBar
```

Below is the table summarizing all currently defined functions with an example of applying each one. Note that some of them have two names, a shorter one and a more descriptive one, but the two functions are otherwise equivalent:

> 下面是总结所有当前定义的函数的表，其中包含应用每个函数的示例。请注意，其中一些名称有两个名称，一个较短的名称和一个更具描述性的名称，但这两个函数在其他方面是等效的：

| Function                      | Returns                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Example (in/out) |            |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- | ---------- |
| `uppercase` or `upper`        | Upper case version of the string.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | `Print`          | `PRINT`    |
| `lowercase` or `lower`        | Lower case version of the string.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | `Print`          | `print`    |
| `title`                       | String with first letter capitalized and the rest in lower case.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | `print`          | `Print`    |
| `firstuppercase`              | String with the first letter capitalized and the rest unchanged.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | `printIt`        | `PrintIt`  |
| `firstlowercase`              | String with the first letter in lower case and the rest unchanged.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | `PrintIt`        | `printIt`  |
| `camelcase` or `ctitle`       | String with capitalized first letter and any letter following an underscore (which are removed in the process) and rest in lower case.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | `print_it`       | `PrintIt`  |
| `lowercamelcase` or `lctitle` | String with every letter following an underscore (which is removed in the process) capitalized and rest, including the first letter, in lower case.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | `print_it`       | `printIt`  |
| `undercase` or `utitle`       | Lower case string with underscores inserted before every upper case letter in the original string and any number not at the end of string. Logically, this is the reverse of `camelcase`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | `PrintIt`        | `print_it` |
| `schemify`                    | String with all underscores replaced with dashes, resulting in more Lispers/Schemers-pleasing name.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | `print_it`       | `print-it` |
| `strip:[prefix]`              | String without the given prefix or the original string if it doesn't start with this prefix. Note that square brackets should be used literally, e.g. `%rename("strip:[wx]")`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | `wxPrint`        | `Print`    |
| `rstrip:[suffix]`             | String without the given suffix or the original string if it doesn't end with this suffix. Note that square brackets should be used literally, e.g. `%rename("rstrip:[Cls]")`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | `PrintCls`       | `Print`    |
| `regex:/pattern/subst/`       | String after (Perl-like) regex substitution operation. This function allows to apply arbitrary regular expressions to the identifier names. The *pattern* part is a regular expression in Perl syntax (as supported by the [Perl Compatible Regular Expressions (PCRE)](http://www.pcre.org/)) library and the *subst* string can contain back-references of the form `\N` where `N` is a digit from 0 to 9, or one of the following escape sequences: `\l`, `\L`, `\u`, `\U` or `\E`. The back-references are replaced with the contents of the corresponding capture group while the escape sequences perform the case conversion in the substitution string: `\l` and `\L` convert to the lower case, while `\u` and `\U`convert to the upper case. The difference between the elements of each pair is that `\l`and `\u` change the case of the next character only, while `\L` and `\U` do it for all the remaining characters or until `\E` is encountered. Finally please notice that backslashes need to be escaped in C strings, so in practice `"\\"` must be used in all these escape sequences. For example, to remove any alphabetic prefix before an underscore and capitalize the remaining part you could use the following directive:`%rename("regex:/(\\w+)_(.*)/\\u\\2/")` | `prefix_print`   | `Print`    |
| `command:cmd`                 | Output of an external command `cmd` with the string passed to it as input. Notice that this function is extremely slow compared to all the other ones as it involves spawning a separate process and using it for many declarations is not recommended. The *cmd* is not enclosed in square brackets but must be terminated with a triple `'<'` sign, e.g. `%rename("command:tr -d aeiou <<<")`(nonsensical example removing all vowels)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `Print`          | `Prnt`     |

The most general function of all of the above ones (not counting `command` which is even more powerful in principle but which should generally be avoided because of performance considerations) is the`regex` one. Here are some more examples of its use:

> 所有上述功能中最常用的功能（不包括 `command`，原则上功能更强大，但由于性能方面的考虑通常应该避免这种功能）是 `regex`。以下是其使用的更多示例：

```
// Strip the wx prefix from all identifiers except those starting with wxEVT
%rename("%(regex:/wx(?!EVT)(.*)/\\1/)s") ""; // wxSomeWidget -> SomeWidget
                                             // wxEVT_PAINT -> wxEVT_PAINT

// Apply a rule for renaming the enum elements to avoid the common prefixes
// which are redundant in C#/Java
%rename("%(regex:/^([A-Z][a-z]+)+_(.*)/\\2/)s", %$isenumitem) ""; // Colour_Red -> Red

// Remove all "Set/Get" prefixes.
%rename("%(regex:/^(Set|Get)(.*)/\\2/)s") ""; // SetValue -> Value
                                              // GetValue -> Value
```

As before, everything that was said above about `%rename` also applies to `%ignore`. In fact, the latter is just a special case of the former and ignoring an identifier is the same as renaming it to the special`"$ignore"` value. So the following snippets

> 和以前一样，关于 `%rename` 的上述内容也适用于 `%ignore`。事实上，后者只是前者的特例，忽略标识符与将其重命名为特殊的 `$ignore` 值相同。所以下面的片段

```
%ignore print;
```

and

> 以及

```
%rename("$ignore") print;
```

are exactly equivalent and `%rename` can be used to selectively ignore multiple declarations using the previously described matching possibilities.

> 完全等价，`%rename` 可用于使用前面描述的匹配可能性选择性地忽略多个声明。

#### 5.4.7.3 限制全局重命名规则

As explained in the previous sections, it is possible to either rename individual declarations or apply a rename rule to all of them at once. In practice, the latter is however rarely appropriate as there are always some exceptions to the general rules. To deal with them, the scope of an unnamed `%rename` can be limited using subsequent `match`parameters. They can be applied to any of the attributes associated by SWIG with the declarations appearing in its input. For example:

> 如前面部分所述，可以重命名单个声明，也可以一次将重命名规则应用于所有声明。实际上，后者很少适用，因为一般规则总有一些例外。为了处理它们，可以使用后续的 `match` 参数来限制未命名的 `%rename` 的范围。它们可以应用于 SWIG 关联的任何属性，并在其输入中显示声明。例如：

```
%rename("foo", match$name="bar") "";
```

can be used to achieve the same effect as the simpler

> 可以用更简单的方式达到同样的效果

```
%rename("foo") bar;
```

and so is not very interesting on its own. However `match` can also be applied to the declaration type, for example `match="class"` restricts the match to class declarations only (in C++) and `match="enumitem"`restricts it to the enum elements. SWIG also provides convenience macros for such match expressions, for example

> 所以它本身并不是很有趣。但是 `match` 也可以应用于声明类型，例如 `match ="class"` 仅限于匹配类声明（在 C++ 中）和 `match ="enumitem"` 将它限制为枚举元素。SWIG 还为这种匹配表达式提供了便利的宏，例如

```
%rename("%(title)s", %$isenumitem) "";
```

will capitalize the names of all the enum elements but not change the case of the other declarations. Similarly, `%$isclass`, `%$isfunction`,`%$isconstructor`, `%$isunion`, `%$istemplate`, and `%$isvariable`can be used. Many other checks are possible and this documentation is not exhaustive, see the "%rename predicates" section in `swig.swg` for the full list of supported match expressions.

In addition to literally matching some string with `match` you can also use `regexmatch` or `notregexmatch` to match a string against a regular expression. For example, to ignore all functions having "Old" as a suffix you could use

> 将大写所有枚举元素的名称，但不改变其他声明的大小写。类似地，可以使用 `%$isclass`、`%$isfunction`、`%$isconstructor`、`%$isunion`、`%$istemplate` 和 `%$isvariable`。许多其他检查都是可能的，本文档并非详尽无遗，请参阅 `swig.swg` 中的 `%rename` 谓词部分，以获取支持的匹配表达式的完整列表。
>
> 除了将一些字符串与 `match` 字面匹配之外，你还可以使用 `regexmatch` 或 `notregexmatch` 来匹配正则表达式的字符串。例如，要忽略所有具有 `Old` 作为后缀的函数，你可以使用

```
%rename("$ignore", regexmatch$name="Old$") "";
```

For simple cases like this, specifying the regular expression for the declaration name directly can be preferable and can also be done using `regextarget`:

> 对于像这样的简单情况，直接指定声明名称的正则表达式是可取的，也可以使用 `regextarget` 来完成：

```
%rename("$ignore", regextarget=1) "Old$";
```

Notice that the check is done only against the name of the declaration itself, if you need to match the full name of a C++ declaration you must use `fullname` attribute:

> 请注意，仅对声明本身的名称进行检查，如果需要匹配 C++ 声明的全名，则必须使用 `fullname` 属性：

```
%rename("$ignore", regextarget=1, fullname=1) "NameSpace::ClassName::.*Old$";
```

As for `notregexmatch`, it restricts the match only to the strings not matching the specified regular expression. So to rename all declarations to lower case except those consisting of capital letters only:

> 对于 `notregexmatch`，它仅将匹配限制为与指定正则表达式不匹配的字符串。因此，除了仅包含大写字母的声明外，将所有声明重命名为小写：

```
%rename("$(lower)s", notregexmatch$name="^[A-Z]+$") "";
```

Finally, variants of `%rename` and `%ignore` directives can be used to help wrap C++ overloaded functions and methods or C++ methods which use default arguments. This is described in the [Ambiguity resolution and renaming](http://www.swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_ambiguity_resolution_renaming) section in the C++ chapter.

> 最后，`%rename` 和 `%ignore` 指令的变体可用于帮助包装 C++ 重载函数和方法或使用默认参数的 C++ 方法。这在 C++ 章节的[消歧义和重命名](http://www.swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_ambiguity_resolution_renaming)部分中进行了描述。

#### 5.4.7.4 包装一些符号而忽略其他

Using the techniques described above it is possible to ignore everything in a header and then selectively wrap a few chosen methods or classes. For example, consider a header, `myheader.h`which has many classes in it and just the one class called `Star` is wanted within this header, the following approach could be taken:

> 使用上述技术，可以忽略头文件中的所有内容，然后有选择地包装一些选定的方法或类。例如，考虑一个头文件，`myheader.h`，其中包含许多类，并且在此头文件中只需要一个名为 `Star` 的类，可以采用以下方法：

```
%ignore ""; // Ignore everything

// Unignore chosen class 'Star'
%rename("%s") Star;

// As the ignore everything will include the constructor, destructor, methods etc
// in the class, these have to be explicitly unignored too:
%rename("%s") Star::Star;
%rename("%s") Star::~Star;
%rename("%s") Star::shine; // named method

%include "myheader.h"
```

Another approach which might be more suitable as it does not require naming all the methods in the chosen class is to begin by ignoring just the classes. This does not add an explicit ignore to any members of the class, so when the chosen class is unignored, all of its methods will be wrapped.

> 有另一种可能更合适的方法（因为它不需要命名所选类中的所有方法），首先忽略类。这不会向类的任何成员添加显式忽略，因此当所选类被取消时，它的所有方法都将被包装。

```
%rename($ignore, %$isclass) ""; // Only ignore all classes
%rename("%s") Star; // Unignore 'Star'
%include "myheader.h"
```

### 5.4.8 默认/可选参数

SWIG supports default arguments in both C and C++ code. For example:

> SWIG 支持 C 和 C++ 代码中的默认参数。例如：

```c
int plot(double x, double y, int color=WHITE);
```

In this case, SWIG generates wrapper code where the default arguments are optional in the target language. For example, this function could be used in Tcl as follows :

> 在这种情况下，SWIG 生成包装器代码，其中默认参数在目标语言中是可选的。例如，此函数可以在 Tcl 中使用，如下所示：

```tcl
% plot -3.4 7.5    # Use default value
% plot -3.4 7.5 10 # set color to 10 instead
```

Although the ANSI C standard does not allow default arguments, default arguments specified in a SWIG interface work with both C and C++.

**Note:** There is a subtle semantic issue concerning the use of default arguments and the SWIG generated wrapper code. When default arguments are used in C code, the default values are emitted into the wrappers and the function is invoked with a full set of arguments. This is different to when wrapping C++ where an overloaded wrapper method is generated for each defaulted argument. Please refer to the section on [default arguments](http://www.swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args) in the C++ chapter for further details.

> 尽管 ANSI C 标准不允许使用默认参数，但 SWIG 接口中指定的默认参数同时适用于 C 和 C++。
>
> 关于使用默认参数和 SWIG 生成的包装器代码存在一个微妙的语义问题。当在 C 代码中使用默认参数时，默认值将发送到包装器中，并使用一组完整的参数调用该函数。这与包装 C++ 时的不同之处在于为每个默认参数生成重载的包装器方法。有关更多详细信息，请参阅 C++ 一章中有关[默认参数](http://www.swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args)的部分。

### 5.4.9 函数指针与回调

Occasionally, a C library may include functions that expect to receive pointers to functions--possibly to serve as callbacks. SWIG provides full support for function pointers provided that the callback functions are defined in C and not in the target language. For example, consider a function like this:

> 有时，C 库可能包含期望接受函数指针的函数——可能用作回调函数。SWIG 提供对函数指针的完全支持，前提是回调函数是用 C 语言定义的，而不是用目标语言定义的。例如，考虑这样的函数：

```c
int binary_op(int a, int b, int (*op)(int, int));
```

When you first wrap something like this into an extension module, you may find the function to be impossible to use. For instance, in Python:

> 当你第一次将这样的东西包装到扩展模块中时，你可能会发现该功能无法使用。例如，在 Python 中：

```python
>>> def add(x, y):
...     return x+y
...
>>> binary_op(3, 4, add)
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
TypeError: Type error. Expected _p_f_int_int__int
>>>
```

The reason for this error is that SWIG doesn't know how to map a scripting language function into a C callback. However, existing C functions can be used as arguments provided you install them as constants. One way to do this is to use the `%constant` directive like this:

> 出现此错误的原因是 SWIG 不知道如何将脚本语言函数映射到 C 回调中。但是，如果将现有 C 函数作为常量安装，则可以将它们用作参数。一种方法是使用 `%constant` 指令，如下所示：

```
/* Function with a callback */
int binary_op(int a, int b, int (*op)(int, int));

/* Some callback functions */
%constant int add(int, int);
%constant int sub(int, int);
%constant int mul(int, int);
```

In this case, `add`, `sub`, and `mul` become function pointer constants in the target scripting language. This allows you to use them as follows:

> 在这种情况下，`add`、`sub` 和 `mul` 成为目标脚本语言中的函数指针常量。这允许你按如下方式使用它们：

```python
>>> binary_op(3, 4, add)
7
>>> binary_op(3, 4, mul)
12
>>>
```

Unfortunately, by declaring the callback functions as constants, they are no longer accessible as functions. For example:

> 不幸的是，通过将回调函数声明为常量，它们不再可以作为函数访问。例如：

```python
>>> add(3, 4)
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
TypeError: object is not callable: '_ff020efc_p_f_int_int__int'
>>>
```

If you want to make a function available as both a callback function and a function, you can use the `%callback` and `%nocallback`directives like this:

> 如果你想把一个函数作为回调函数和函数使用，你可以像这样使用 `%callback` 和 `%nocallback` 指令：

```
/* Function with a callback */
int binary_op(int a, int b, int (*op)(int, int));

/* Some callback functions */
%callback("%s_cb");
int add(int, int);
int sub(int, int);
int mul(int, int);
%nocallback;
```

The argument to `%callback` is a printf-style format string that specifies the naming convention for the callback constants (`%s` gets replaced by the function name). The callback mode remains in effect until it is explicitly disabled using `%nocallback`. When you do this, the interface now works as follows:

> `%callback` 的参数是一个 `printf` 样式的格式字符串，它指定了回调常量的命名约定（`%s` 被函数名替换）。回调模式保持有效，直到使用 `%nocallback` 显式禁用它。执行此操作时，接口现在的工作方式如下：

```python
>>> binary_op(3, 4, add_cb)
7
>>> binary_op(3, 4, mul_cb)
12
>>> add(3, 4)
7
>>> mul(3, 4)
12
```

Notice that when the function is used as a callback, special names such as `add_cb` are used instead. To call the function normally, just use the original function name such as `add()`.

SWIG provides a number of extensions to standard C printf formatting that may be useful in this context. For instance, the following variation installs the callbacks as all upper case constants such as `ADD`, `SUB`, and `MUL`:

> 请注意，当该函数用作回调时，将使用特殊名称，例如 `add_cb`。要正常调用函数，只需使用原始函数名称，例如 `add()`。
>
> SWIG 为标准 C `printf` 格式提供了许多扩展，在这种情况下可能很有用。例如，以下变体将回调安装为所有大写常量，例如 `ADD`、`SUB` 和 `MUL`：

```
/* Some callback functions */
%callback("%(uppercase)s");
int add(int, int);
int sub(int, int);
int mul(int, int);
%nocallback;
```

A format string of `"%(lowercase)s"` converts all characters to lower case. A string of `"%(title)s"` capitalizes the first character and converts the rest to lower case.

And now, a final note about function pointer support. Although SWIG does not normally allow callback functions to be written in the target language, this can be accomplished with the use of typemaps and other advanced SWIG features. See the [Typemaps chapter](http://www.swig.org/Doc3.0/Typemaps.html#Typemaps) for more about typemaps and individual target language chapters for more on callbacks and the 'director' feature.

> 格式字符串 `"%(lowercase)s"` 将所有字符转换为小写。一串 `"%(title)s"` 将第一个字符大写并将其余字符转换为小写字母。
>
> 现在，关于函数指针支持的最后一点。尽管 SWIG 通常不允许以目标语言编写回调函数，但这可以通过使用类型映射和其他高级 SWIG 功能来实现。有关字典图和单个目标语言章节的更多信息，请参阅[类型映射章节](http://www.swig.org/Doc3.0/Typemaps.html#Typemaps)，了解有关回调和导向器（director）功能的更多信息。

## 5.5 结构体与共用体

This section describes the behavior of SWIG when processing ANSI C structures and union declarations. Extensions to handle C++ are described in the next section.

If SWIG encounters the definition of a structure or union, it creates a set of accessor functions. Although SWIG does not need structure definitions to build an interface, providing definitions makes it possible to access structure members. The accessor functions generated by SWIG simply take a pointer to an object and allow access to an individual member. For example, the declaration :

> 本节描述了处理 ANSI C 结构体和共用体声明时 SWIG 的行为。处理 C++ 的扩展将在下一节中介绍。
>
> 如果 SWIG 遇到结构体和共用体的定义，它将创建一组访问器函数。虽然 SWIG 不需要结构体定义来构建接口，但提供定义可以使其访问结构体成员。SWIG 生成的访问器函数只需接受指向对象的指针，并允许访问单个成员。例如，声明：

```c
struct Vector {
  double x, y, z;
}
```

gets transformed into the following set of accessor functions :

> 变为以下一组访问函数：

```c
double Vector_x_get(struct Vector *obj) {
  return obj->x;
}
double Vector_y_get(struct Vector *obj) {
  return obj->y;
}
double Vector_z_get(struct Vector *obj) {
  return obj->z;
}
void Vector_x_set(struct Vector *obj, double value) {
  obj->x = value;
}
void Vector_y_set(struct Vector *obj, double value) {
  obj->y = value;
}
void Vector_z_set(struct Vector *obj, double value) {
  obj->z = value;
}
```

In addition, SWIG creates default constructor and destructor functions if none are defined in the interface. For example:

> 此外，如果接口中没有定义，SWIG 会创建默认构造函数和析构函数。例如：

```c
struct Vector *new_Vector() {
    return (Vector *) calloc(1, sizeof(struct Vector));
}
void delete_Vector(struct Vector *obj) {
    free(obj);
}
```

Using these low-level accessor functions, an object can be minimally manipulated from the target language using code like this:

> 使用这些低级访问器函数，可以使用以下代码从目标语言中对对象进行最低限度的操作：

```c
v = new_Vector()
Vector_x_set(v, 2)
Vector_y_set(v, 10)
Vector_z_set(v, -5)
...
delete_Vector(v)
```

However, most of SWIG's language modules also provide a high-level interface that is more convenient. Keep reading.

> 但是，SWIG 的大多数语言模块也提供了更方便的高级接口。继续阅读。

### 5.5.1 `typedef` 与结构体

SWIG supports the following construct which is quite common in C programs :

> SWIG 支持以下构造，这在 C 程序中很常见：

```c
typedef struct {
  double x, y, z;
} Vector;
```

When encountered, SWIG assumes that the name of the object is `Vector` and creates accessor functions like before. The only difference is that the use of `typedef` allows SWIG to drop the `struct` keyword on its generated code. For example:

> 当遇到时，SWIG 假定对象的名称是 `Vector` 并创建像之前一样的访问器函数。唯一的区别是使用 `typedef` 允许 SWIG 在其生成的代码上删除 `struct` 关键字。例如：

```c
double Vector_x_get(Vector *obj) {
  return obj->x;
}
```

If two different names are used like this :

> 如果两个不同的名字被如下使用：

```c
typedef struct vector_struct {
  double x, y, z;
} Vector;
```

the name `Vector` is used instead of `vector_struct` since this is more typical C programming style. If declarations defined later in the interface use the type `struct vector_struct`, SWIG knows that this is the same as `Vector` and it generates the appropriate type-checking code.

> 使用名称 `Vector` 代替 `vector_struct`，因为这是更典型的 C 编程风格。如果稍后在接口中定义的声明使用类型 `struct vector_struct`，则 SWIG 知道它与 `Vector` 相同，并且它生成适当的类型检查代码。

### 5.5.2 字符串与结构体

Structures involving character strings require some care. SWIG assumes that all members of type `char *` have been dynamically allocated using `malloc()` and that they are NULL-terminated ASCII strings. When such a member is modified, the previous contents will be released, and the new contents allocated. For example :

> 涉及字符串的结构体需要一些小心。SWIG 假定 `char *` 类型的所有成员都是使用 `malloc()` 动态分配的，并且它们是以 NULL 结尾的 ASCII 字符串。修改此类成员后，将释放先前的内容，并分配新内容。例如 ：

```
%module mymodule
...
struct Foo {
  char *name;
  ...
}
```

This results in the following accessor functions :

> 这导致了如下的访问器函数：

```c
char *Foo_name_get(Foo *obj) {
  return Foo->name;
}

char *Foo_name_set(Foo *obj, char *c) {
  if (obj->name)
    free(obj->name);
  obj->name = (char *) malloc(strlen(c)+1);
  strcpy(obj->name, c);
  return obj->name;
}
```

If this behavior differs from what you need in your applications, the SWIG "memberin" typemap can be used to change it. See the typemaps chapter for further details.

Note: If the `-c++` option is used, `new` and `delete` are used to perform memory allocation.

> 如果此行为与你在应用程序中所需的行为不同，则可以使用 SWIG `memberin` 类型映射来更改它。有关更多详细信息，请参阅类型映射章节。
>
> 注意：如果使用 `-c++` 选项，则使用 `new` 和 `delete` 来执行内存分配。

### 5.5.3 数组成员

Arrays may appear as the members of structures, but they will be read-only. SWIG will write an accessor function that returns the pointer to the first element of the array, but will not write a function to change the contents of the array itself. When this situation is detected, SWIG may generate a warning message such as the following :

> 数组可能显示为结构体的成员，但它们将是只读的。SWIG 将编写一个访问器函数，该函数返回指向数组第一个元素的指针，但不会编写一个函数来更改数组本身的内容。检测到这种情况时，SWIG 可能会生成一条警告消息，如下所示：

```
interface.i:116. Warning. Array member will be read-only
```

To eliminate the warning message, typemaps can be used, but this is discussed in a later chapter. In many cases, the warning message is harmless.

> 要消除警告消息，可以使用类型映射，但这将在后面的章节中讨论。在许多情况下，警告消息是无害的。

### 5.5.4 结构体数据成员

Occasionally, a structure will contain data members that are themselves structures. For example:

> 有时，结构体将包含本身就是结构体的数据成员。例如：

```c
typedef struct Foo {
  int x;
} Foo;

typedef struct Bar {
  int y;
  Foo f;           /* struct member */
} Bar;
```

When a structure member is wrapped, it is handled as a pointer, unless the `%naturalvar` directive is used where it is handled more like a C++ reference (see [C++ Member data](http://www.swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_member_data)). The accessors to the member variable as a pointer are effectively wrapped as follows:

> 当一个结构体成员被包装时，它被作为指针处理，除非使用 `%naturalvar` 指令处理它更像 C++ 引用（参见 [C++ 成员数据](http://www.swig.org/Doc3.0/SWIGPlus.html＃SWIGPlus_member_data)）。作为指针的成员变量的访问器函数有效地被包装成如下形式：

```c
Foo *Bar_f_get(Bar *b) {
  return &b->f;
}
void Bar_f_set(Bar *b, Foo *value) {
  b->f = *value;
}
```

The reasons for this are somewhat subtle but have to do with the problem of modifying and accessing data inside the data member. For example, suppose you wanted to modify the value of `f.x` of a `Bar`object like this:

> 其原因有些微妙，但与修改和访问数据成员内部数据的问题有关。例如，假设你想要修改 `Bar` 对象的 `f.x` 的值，如下所示：

```c
Bar *b;
b->f.x = 37;
```

Translating this assignment to function calls (as would be used inside the scripting language interface) results in the following code:

> 将此赋值转换为函数调用（将在脚本语言接口中使用）会产生以下代码：

```c
Bar *b;
Foo_x_set(Bar_f_get(b), 37);
```

In this code, if the `Bar_f_get()` function were to return a `Foo` instead of a `Foo *`, then the resulting modification would be applied to a *copy*of `f` and not the data member `f` itself. Clearly that's not what you want!

It should be noted that this transformation to pointers only occurs if SWIG knows that a data member is a structure or class. For instance, if you had a structure like this,

> 在这段代码中，如果 `Bar_f_get()` 函数返回一个 `Foo` 而不是 `Foo *`，那么得到的修改将应用于 `f` 的副本而不是数据成员 `f` 本身。显然，这不是你想要的！
>
> 应该注意，只有当 SWIG 知道数据成员是结构体或类时，才会发生对指针的转换。例如，如果你有这样的结构，

```c
struct Foo {
  WORD w;
};
```

and nothing was known about `WORD`, then SWIG will generate more normal accessor functions like this:

> 并且对 `WORD` 一无所知，那么 SWIG 将生成更多正常的访问器函数，如下所示：

```c
WORD Foo_w_get(Foo *f) {
    return f->w;
}

void Foo_w_set(FOO *f, WORD value) {
    f->w = value;
}
```

**Compatibility Note:** SWIG-1.3.11 and earlier releases transformed all non-primitive member datatypes to pointers. Starting in SWIG-1.3.12, this transformation *only* occurs if a datatype is known to be a structure, class, or union. This is unlikely to break existing code. However, if you need to tell SWIG that an undeclared datatype is really a struct, simply use a forward struct declaration such as `"struct Foo;"`.

> **注意兼容性：**SWIG-1.3.11 及更早版本将所有非原始成员数据类型转换为指针。从 SWIG-1.3.12 开始，如果已知数据类型是结构体、类或共用体则仅发生此转换。这不太可能破坏现有代码。但是，如果你需要告诉 SWIG 一个未声明的数据类型实际上是一个结构体，只需使用一个正向结构体声明，如 `struct Foo;`。

### 5.5.5 C 构造函数和析构函数

When wrapping structures, it is generally useful to have a mechanism for creating and destroying objects. If you don't do anything, SWIG will automatically generate functions for creating and destroying objects using `malloc()` and `free()`. Note: the use of `malloc()` only applies when SWIG is used on C code (i.e., when the `-c++` option is *not*supplied on the command line). C++ is handled differently.

If you don't want SWIG to generate default constructors for your interfaces, you can use the `%nodefaultctor` directive or the `-nodefaultctor` command line option. For example:

> 在包装结构体时，通常有一种用于创建和销毁对象的机制。如果你什么都不做，SWIG 会自动生成使用 `malloc()` 和 `free()` 创建和销毁对象的函数。注意：使用 `malloc()` 仅适用于在 C 代码上使用 SWIG（即，在命令行上*不*提供 `-c++` 选项）。C++ 的处理方式不同。
>
> 如果你不希望 SWIG 为你的接口生成默认构造函数，则可以使用 `%nodefaultctor` 指令或 `-nodefaultctor` 命令行选项。例如：

```
swig -nodefaultctor example.i
```

or

> 或

```
%module foo
...
%nodefaultctor;        // Don't create default constructors
... declarations ...
%clearnodefaultctor;   // Re-enable default constructors
```

If you need more precise control, `%nodefaultctor` can selectively target individual structure definitions. For example:

> 如果你需要更精确的控制，`%nodefaultctor` 可以选择性地定位单个结构体定义。例如：

```
%nodefaultctor Foo;      // No default constructor for Foo
...
struct Foo {             // No default constructor generated.
};

struct Bar {             // Default constructor generated.
};
```

Since ignoring the implicit or default destructors most of the time produces memory leaks, SWIG will always try to generate them. If needed, however, you can selectively disable the generation of the default/implicit destructor by using `%nodefaultdtor`

> 由于忽略隐式或默认析构函数大部分时间都会产生内存泄漏，因此 SWIG 将始终尝试生成它们。但是，如果需要，可以使用 `%nodefaultdtor` 选择性地禁用默认/隐式析构函数的生成

```
%nodefaultdtor Foo; // No default/implicit destructor for Foo
...
struct Foo {              // No default destructor is generated.
};

struct Bar {              // Default destructor generated.
};
```

**Compatibility note:** Prior to SWIG-1.3.7, SWIG did not generate default constructors or destructors unless you explicitly turned them on using `-make_default`. However, it appears that most users want to have constructor and destructor functions so it has now been enabled as the default behavior.

**Note:** There are also the `-nodefault` option and `%nodefault`directive, which disable both the default or implicit destructor generation. This could lead to memory leaks across the target languages, and it is highly recommended you don't use them.

> **注意兼容性：**在 SWIG-1.3.7 之前，SWIG 不会生成默认构造函数或析构函数，除非你使用 `-make_default` 明确打开它们。但是，似乎大多数用户都希望拥有构造函数和析构函数，因此现在已将其作为默认行为启用。
>
> **注意：**还有 `-nodefault` 选项和 `%nodefault` 指令，它们禁用默认或隐式析构函数生成。这可能会导致目标语言内存泄漏，强烈建议你不要使用它们。

### 5.5.6 向 C 结构体添加成员函数

Most languages provide a mechanism for creating classes and supporting object oriented programming. From a C standpoint, object oriented programming really just boils down to the process of attaching functions to structures. These functions normally operate on an instance of the structure (or object). Although there is a natural mapping of C++ to such a scheme, there is no direct mechanism for utilizing it with C code. However, SWIG provides a special `%extend`directive that makes it possible to attach methods to C structures for purposes of building an object oriented interface. Suppose you have a C header file with the following declaration :

> 大多数语言都提供了一种创建类和支持面向对象编程的机制。从 C 的角度来看，面向对象编程实际上归结为将函数附加到结构体的过程。这些函数通常在结构体（或对象）的实例上运行。虽然 C++ 有这样一种自然的映射方式，但是没有直接的机制可以将它与 C 代码一起使用。但是，SWIG 提供了一个特殊的 `%extend` 指令，可以将方法附加到 C 结构体以构建面向对象的接口。假设你有一个带有以下声明的 C 头文件：

```c
/* file : vector.h */
...
typedef struct Vector {
  double x, y, z;
} Vector;
```

You can make a `Vector` look a lot like a class by writing a SWIG interface like this:

> 你可以通过编写如下所示的 SWIG 接口使 `Vector` 看起来很像一个类：

```
// file : vector.i
%module mymodule
%{
#include "vector.h"
%}

%include "vector.h"          // Just grab original C header file
%extend Vector {             // Attach these functions to struct Vector
  Vector(double x, double y, double z) {
    Vector *v;
    v = (Vector *) malloc(sizeof(Vector));
    v->x = x;
    v->y = y;
    v->z = z;
    return v;
  }
  ~Vector() {
    free($self);
  }
  double magnitude() {
    return sqrt($self->x*$self->x+$self->y*$self->y+$self->z*$self->z);
  }
  void print() {
    printf("Vector [%g, %g, %g]\n", $self->x, $self->y, $self->z);
  }
};
```

Note the usage of the `$self` special variable. Its usage is identical to a C++ 'this' pointer and should be used whenever access to the struct instance is required. Also note that C++ constructor and destructor syntax has been used to simulate a constructor and destructor, even for C code. There is one subtle difference to a normal C++ constructor implementation though and that is although the constructor declaration is as per a normal C++ constructor, the newly constructed object must be returned **as if** the constructor declaration had a return value, a `Vector *` in this case.

Now, when used with proxy classes in Python, you can do things like this :

> 注意 `$self` 特殊变量的用法。它的用法与 C++ `this` 指针相同，只要需要访问结构体实例，就应该使用它。另请注意，C++ 构造函数和析构函数语法已用于模拟构造函数和析构函数，即使对于 C 代码也是如此。虽然正常的 C++ 构造函数实现有一个细微的区别，虽然构造函数声明是按照普通的 C++ 构造函数，但是必须返回新构造的对象，**好像**构造函数声明有一个返回值，在这种情况下是 `Vector *`。
>
> 现在，当与 Python 中的代理类一起使用时，你可以执行以下操作：

```python
>>> v = Vector(3, 4, 0)                 # Create a new vector
>>> print v.magnitude()                # Print magnitude
5.0
>>> v.print()                  # Print it out
[ 3, 4, 0 ]
>>> del v                      # Destroy it
```

The `%extend` directive can also be used inside the definition of the Vector structure. For example:

> `%extend` 指令也可以在 `Vector` 结构体的定义中使用。例如：

```
// file : vector.i
%module mymodule
%{
#include "vector.h"
%}

typedef struct Vector {
  double x, y, z;
  %extend {
    Vector(double x, double y, double z) { ... }
    ~Vector() { ... }
    ...
  }
} Vector;
```

Note that `%extend` can be used to access externally written functions provided they follow the naming convention used in this example :

> 请注意，`%extend` 可用于访问外部编写的函数，前提是它们遵循此示例中使用的命名约定：

```
/* File : vector.c */
/* Vector methods */
#include "vector.h"
Vector *new_Vector(double x, double y, double z) {
  Vector *v;
  v = (Vector *) malloc(sizeof(Vector));
  v->x = x;
  v->y = y;
  v->z = z;
  return v;
}
void delete_Vector(Vector *v) {
  free(v);
}

double Vector_magnitude(Vector *v) {
  return sqrt(v->x*v->x+v->y*v->y+v->z*v->z);
}

// File : vector.i
// Interface file
%module mymodule
%{
#include "vector.h"
%}

typedef struct Vector {
  double x, y, z;
  %extend {
    Vector(int, int, int); // This calls new_Vector()
    ~Vector();           // This calls delete_Vector()
    double magnitude();  // This will call Vector_magnitude()
    ...
  }
} Vector;
```

The name used for `%extend` should be the name of the struct and not the name of any `typedef` to the struct. For example:

> `%extend` 用的名字和结构体的名字相同，而非结构体的 `typedef`。例如：

```
typedef struct Integer {
  int value;
} Int;
%extend Integer { ...  } /* Correct name */
%extend Int { ...  } /* Incorrect name */

struct Float {
  float value;
};
typedef struct Float FloatValue;
%extend Float { ...  } /* Correct name */
%extend FloatValue { ...  } /* Incorrect name */
```

There is one exception to this rule and that is when the struct is anonymously named such as:

> 有一个例外，就是当结构体是匿名的时候，例如：

```
typedef struct {
  double value;
} Double;
%extend Double { ...  } /* Okay */
```

A little known feature of the `%extend` directive is that it can also be used to add synthesized attributes or to modify the behavior of existing data attributes. For example, suppose you wanted to make `magnitude` a read-only attribute of `Vector` instead of a method. To do this, you might write some code like this:

> `%extend` 指令的一个鲜为人知的功能是它还可以用于添加合成属性或修改现有数据属性的行为。例如，假设你想要 `magnitude` 是 `Vector` 的只读属性而不是方法。为此，你可以编写如下代码：

```
// Add a new attribute to Vector
%extend Vector {
    const double magnitude;
}
// Now supply the implementation of the Vector_magnitude_get function
%{
const double Vector_magnitude_get(Vector *v) {
  return (const double) sqrt(v->x*v->x+v->y*v->y+v->z*v->z);
}
%}
```

Now, for all practical purposes, `magnitude` will appear like an attribute of the object.

A similar technique can also be used to work with data members that you want to process. For example, consider this interface:

> 现在，出于所有实际目的，`magnitude` 将看起来像对象的属性。
>
> 类似的技术也可用于处理你要处理的数据成员。例如，考虑这个接口：

```c
typedef struct Person {
  char name[50];
  ...
} Person;
```

Say you wanted to ensure `name` was always upper case, you can rewrite the interface as follows to ensure this occurs whenever a name is read or written to:

> 假设你想确保 `name` 总是大写，你可以按如下方式重写接口，以确保每当读取或写入名称时都会发生这种情况：

```
typedef struct Person {
  %extend {
    char name[50];
  }
  ...
} Person;

%{
#include <string.h>
#include <ctype.h>

void make_upper(char *name) {
  char *c;
  for (c = name; *c; ++c)
    *c = (char)toupper((int)*c);
}

/* Specific implementation of set/get functions forcing capitalization */

char *Person_name_get(Person *p) {
  make_upper(p->name);
  return p->name;
}

void Person_name_set(Person *p, char *val) {
  strncpy(p->name, val, 50);
  make_upper(p->name);
}
%}
```

Finally, it should be stressed that even though `%extend` can be used to add new data members, these new members can not require the allocation of additional storage in the object (e.g., their values must be entirely synthesized from existing attributes of the structure or obtained elsewhere).

**Compatibility note:** The `%extend` directive is a new name for the `%addmethods` directive. Since `%addmethods` could be used to extend a structure with more than just methods, a more suitable directive name has been chosen.

> 最后，应该强调的是，即使 `%extend` 可以用于添加新的数据成员，这些新成员也不需要在对象中分配额外的存储（例如，它们的值必须完全从现有结构体的属性合成或在别处获得）。
>
> **注意兼容性：**`%extend` 指令是 `%addmethods` 指令的新名称。由于 `%addmethods` 可以用于扩展具有多个方法的结构，因此选择了更合适的指令名称。

### 5.5.7 嵌套结构体

Occasionally, a C program will involve structures like this :

> 有时候，C 程序会涉及这样的结构体：

```c
typedef struct Object {
  int objtype;
  union {
    int ivalue;
    double dvalue;
    char *strvalue;
    void *ptrvalue;
  } intRep;
} Object;
```

When SWIG encounters this, it performs a structure splitting operation that transforms the declaration into the equivalent of the following:

> 当 SWIG 遇到此问题时，它会执行结构体拆分操作，将声明转换为以下等效项：

```c
typedef union {
  int ivalue;
  double dvalue;
  char *strvalue;
  void *ptrvalue;
} Object_intRep;

typedef struct Object {
  int objType;
  Object_intRep intRep;
} Object;
```

SWIG will then create an `Object_intRep` structure for use inside the interface file. Accessor functions will be created for both structures. In this case, functions like this would be created :

> 然后，SWIG 将创建一个 `Object_intRep` 结构，以便在接口文件中使用。将为两个结构创建访问器功能。在这种情况下，将创建这样的函数：

```c
Object_intRep *Object_intRep_get(Object *o) {
  return (Object_intRep *) &o->intRep;
}
int Object_intRep_ivalue_get(Object_intRep *o) {
  return o->ivalue;
}
int Object_intRep_ivalue_set(Object_intRep *o, int value) {
  return (o->ivalue = value);
}
double Object_intRep_dvalue_get(Object_intRep *o) {
  return o->dvalue;
}
//... etc ...
```

Although this process is a little hairy, it works like you would expect in the target scripting language--especially when proxy classes are used. For instance, in Perl:

> 虽然这个过程有点惊险，但它的工作方式与目标脚本语言中的预期相同——尤其是在使用代理类时。例如，在 Perl 中：

```perl
# Perl5 script for accessing nested member
$o = CreateObject();                    # Create an object somehow
$o->{intRep}->{ivalue} = 7              # Change value of o.intRep.ivalue
```

If you have a lot of nested structure declarations, it is advisable to double-check them after running SWIG. Although, there is a good chance that they will work, you may have to modify the interface file in certain cases.

Finally, note that nesting is handled differently in C++ mode, see [Nested classes](http://www.swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_nested_classes).

> 如果你有很多嵌套结构体声明，建议在运行 SWIG 后仔细检查它们。虽然它们很可能会起作用，但在某些情况下可能需要修改接口文件。
>
> 最后，C++ 模式下嵌套的处理方式不同，详见[嵌套类](http://www.swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_nested_classes)。

### 5.5.8 其他关于包装结构体的事项

SWIG doesn't care if the declaration of a structure in a `.i` file exactly matches that used in the underlying C code (except in the case of nested structures). For this reason, there are no problems omitting problematic members or simply omitting the structure definition altogether. If you are happy passing pointers around, this can be done without ever giving SWIG a structure definition.

Starting with SWIG 1.3, a number of improvements have been made to SWIG's code generator. Specifically, even though structure access has been described in terms of high-level accessor functions such as this,

> SWIG 不关心 `.i` 文件中的结构体声明是否与底层 C 代码中使用的结构体完全匹配（嵌套结构体除外）。出于这个原因，省略有问题的成员或完全省略结构体定义是没有问题的。如果你很高兴传递指针，这可以在不给 SWIG 一个结构体定义的情况下完成。
>
> 从 SWIG 1.3 开始，对 SWIG 的代码生成器进行了许多改进。具体来说，即使已经根据诸如此类的高级访问器函数描述了结构体访问，

```c
double Vector_x_get(Vector *v) {
  return v->x;
}
```

most of the generated code is actually inlined directly into wrapper functions. Therefore, no function `Vector_x_get()` actually exists in the generated wrapper file. For example, when creating a Tcl module, the following function is generated instead:

> 大多数生成的代码实际上直接内联到包装器函数中。因此，生成的包装器文件中实际上不存在函数 `Vector_x_get()`。例如，在创建 Tcl 模块时，会生成以下函数：

```c
static int
_wrap_Vector_x_get(ClientData clientData, Tcl_Interp *interp,
                   int objc, Tcl_Obj *CONST objv[]) {
  struct Vector *arg1 ;
  double result ;

  if (SWIG_GetArgs(interp, objc, objv, "p:Vector_x_get self ", &arg0,
                   SWIGTYPE_p_Vector) == TCL_ERROR)
    return TCL_ERROR;
  result = (double ) (arg1->x);
  Tcl_SetObjResult(interp, Tcl_NewDoubleObj((double) result));
  return TCL_OK;
}
```

The only exception to this rule are methods defined with `%extend`. In this case, the added code is contained in a separate function.

Finally, it is important to note that most language modules may choose to build a more advanced interface. Although you may never use the low-level interface described here, most of SWIG's language modules use it in some way or another.

> 此规则的唯一例外是使用 `%extend` 定义的方法。在这种情况下，添加的代码包含在单独的函数中。
>
> 最后，需要注意的是，大多数语言模块可能会选择构建更高级的接口。虽然你可能永远不会使用此处描述的低级接口，但 SWIG 的大多数语言模块都会以某种方式使用它。

## 5.6 代码插入

Sometimes it is necessary to insert special code into the resulting wrapper file generated by SWIG. For example, you may want to include additional C code to perform initialization or other operations. There are four common ways to insert code, but it's useful to know how the output of SWIG is structured first.

> 有时需要在 SWIG 生成的结果包装器文件中插入特殊代码。例如，你可能希望包含其他 C 代码以执行初始化或其他操作。插入代码有四种常用方法，但首先了解如何构造 SWIG 的输出很有用。

### 5.6.1 SWIG 的输出

When SWIG creates its output C/C++ file, it is broken up into five sections corresponding to runtime code, headers, wrapper functions, and module initialization code (in that order).

* **Begin section**.

A placeholder for users to put code at the beginning of the C/C++ wrapper file. This is most often used to define preprocessor macros that are used in later sections.

* **Runtime code**.

This code is internal to SWIG and is used to include type-checking and other support functions that are used by the rest of the module.

* **Header section**.

This is user-defined support code that has been included by the `%{ ... %}` directive. Usually this consists of header files and other helper functions.

* **Wrapper code**.

These are the wrappers generated automatically by SWIG.

* **Module initialization**.

The function generated by SWIG to initialize the module upon loading.

> 当 SWIG 创建其输出 C/C++ 文件时，它将分为五个部分，分别对应于运行时代码、头文件、包装器函数和模块初始化代码（按此顺序）。
>
> * **开始部分**
>
> 用户的占位符，用于将代码放在 C/C++ 包装器文件的开头。这通常用于定义后续部分中使用的预处理器宏。
>
> * **运行时代码**
>
> 此代码是 SWIG 的内部代码，用于包含类型检查和其他模块使用的支持函数。
>
> * **头文件部分**
>
> 这是用户定义的支持代码，已由 `%{...%}` 指令包含。通常这包括头文件和其他辅助函数。
>
> * **包装器函数**
>
> 这些是 SWIG 自动生成的包装器。
>
> * **模块初始化**
>
> SWIG 生成的函数，用于在加载时初始化模块。

### 5.6.2 代码插入块

The `%insert` directive enables inserting blocks of code into a given section of the generated code. It can be used in one of two ways:

> `%insert` 指令允许将代码块插入生成代码的给定部分。它可以使用以下两种方式之一：

```
%insert("section") "filename"
%insert("section") %{ ... %}
```

The first will dump the contents of the file in the given `filename` into the named `section`. The second inserts the code between the braces into the named `section`. For example, the following adds code into the runtime section:

> 第一种将把给定 `filename` 中文件的内容转储到命名的 `section` 中。第二种将大括号之间的代码插入到命名的 `section` 中。例如，以下内容将代码添加到运行时部分：

```
%insert("runtime") %{
  ... code in runtime section ...
%}
```

There are the 5 sections, however, some target languages add in additional sections and some of these result in code being generated into a target language file instead of the C/C++ wrapper file. These are documented when available in the target language chapters. Macros named after the code sections are available as additional directives and these macro directives are normally used instead of `%insert`. For example, `%runtime` is used instead of `%insert("runtime")`. The valid sections and order of the sections in the generated C/C++ wrapper file is as shown:

> 有 5 个部分，但是，一些目标语言在附加部分中添加，其中一些导致代码生成到目标语言文件而不是 C/C++ 包装器文件。目标语言章节中提供了这些内容。以代码段命名的宏可用作附加指令，并且通常使用这些宏指令而不是 `%insert`。例如，使用 `%runtime` 而不是 `%insert("runtime")`。生成的 C/C++ 包装器文件中的有效部分和部分的顺序如下所示：

```
%begin %{
  ... code in begin section ...
%}

%runtime %{
  ... code in runtime section ...
%}

%header %{
  ... code in header section ...
%}

%wrapper %{
  ... code in wrapper section ...
%}

%init %{
  ... code in init section ...
%}
```

The bare `%{ ... %}` directive is a shortcut that is the same as `%header %{ ... %}`.

The `%begin` section is effectively empty as it just contains the SWIG banner by default. This section is provided as a way for users to insert code at the top of the wrapper file before any other code is generated. Everything in a code insertion block is copied verbatim into the output file and is not parsed by SWIG. Most SWIG input files have at least one such block to include header files and support C code. Additional code blocks may be placed anywhere in a SWIG file as needed.

> 简单的 `%{...%}` 指令是一个与 `%header%{...%}` 相同的快捷方式。
>
> `%begin` 部分实际上是空的，因为它默认只包含 SWIG 标签。提供此部分是为了让用户在生成任何其他代码之前在包装器文件的顶部插入代码。代码插入块中的所有内容都会逐字复制到输出文件中，并且不会被 SWIG 解析。大多数 SWIG 输入文件至少有一个这样的块，包括头文件和支持 C 代码。根据需要，可以将附加代码块放置在 SWIG 文件中的任何位置。

```
%module mymodule
%{
#include "my_header.h"
%}
... Declare functions here
%{

void some_extra_function() {
  ...
}
%}
```

A common use for code blocks is to write "helper" functions. These are functions that are used specifically for the purpose of building an interface, but which are generally not visible to the normal C program. For example :

> 代码块的一个常见用途是编写“辅助”函数。这些函数专门用于构建接口，但通常对普通 C 程序不可见。例如 ：

```
%{
/* Create a new vector */
static Vector *new_Vector() {
  return (Vector *) malloc(sizeof(Vector));
}

%}
// Now wrap it
Vector *new_Vector();
```

### 5.6.3 内联代码块

Since the process of writing helper functions is fairly common, there is a special inlined form of code block that is used as follows :

> 由于编写辅助函数的过程相当普遍，因此有一种特殊的内联形式的代码块，其使用方法如下：

```
%inline %{
/* Create a new vector */
Vector *new_Vector() {
  return (Vector *) malloc(sizeof(Vector));
}
%}
```

The `%inline` directive inserts all of the code that follows verbatim into the header portion of an interface file. The code is then parsed by both the SWIG preprocessor and parser. Thus, the above example creates a new command `new_Vector` using only one declaration. Since the code inside an `%inline %{ ... %}` block is given to both the C compiler and SWIG, it is illegal to include any SWIG directives inside a `%{ ... %}` block.

> `%inline` 指令将逐字输入的所有代码插入到接口文件的头文件部分中。然后由 SWIG 预处理器和解析器解析代码。因此，上面的示例仅使用一个声明创建一个新命令 `new_Vector`。由于 `%inline%{...%}` 块内的代码被赋予 C 编译器和 SWIG，因此在 `%{...%}` 块中包含任何 SWIG 指令是非法的。

### 5.6.4 初始化块

When code is included in the `%init` section, it is copied directly into the module initialization function. For example, if you needed to perform some extra initialization on module loading, you could write this:

> 当代码包含在 `%init` 部分中时，它会直接复制到模块初始化函数中。例如，如果你需要在模块加载时执行一些额外的初始化，你可以这样写：

```
%init %{
  init_variables();
%}
```

## 5.7 一个构建接口的策略

This section describes the general approach for building interfaces with SWIG. The specifics related to a particular scripting language are found in later chapters.

> 本节介绍使用 SWIG 构建接口的一般方法。与特定脚本语言相关的细节可在后面的章节中找到。

### 5.7.1 为 SWIG 准备 C 程序

SWIG doesn't require modifications to your C code, but if you feed it a collection of raw C header files or source code, the results might not be what you expect---in fact, they might be awful. Here's a series of steps you can follow to make an interface for a C program :

* Identify the functions that you want to wrap. It's probably not necessary to access every single function of a C program--thus, a little forethought can dramatically simplify the resulting scripting language interface. C header files are a particularly good source for finding things to wrap.
* Create a new interface file to describe the scripting language interface to your program.
* Copy the appropriate declarations into the interface file or use SWIG's `%include` directive to process an entire C source/header file.
* Make sure everything in the interface file uses ANSI C/C++ syntax.
* Make sure all necessary ``typedef`' declarations and type-information is available in the interface file. In particular, ensure that the type information is specified in the correct order as required by a C/C++ compiler. Most importantly, define a type before it is used! A C compiler will tell you if the full type information is not available if it is needed, whereas SWIG will usually not warn or error out as it is designed to work without full type information. However, if type information is not specified correctly, the wrappers can be sub-optimal and even result in uncompilable C/C++ code.
* If your program has a main() function, you may need to rename it (read on).
* Run SWIG and compile.

Although this may sound complicated, the process turns out to be fairly easy once you get the hang of it.

In the process of building an interface, SWIG may encounter syntax errors or other problems. The best way to deal with this is to simply copy the offending code into a separate interface file and edit it. However, the SWIG developers have worked very hard to improve the SWIG parser--you should report parsing errors to the [swig-devel mailing list](http://www.swig.org/mail.html) or to the [SWIG bug tracker](http://www.swig.org/bugs.html).

> SWIG 不需要修改你的 C 代码，但如果你提供原始 C 头文件或源代码的集合，结果可能不是你所期望的——实际上，它们可能很糟糕。以下是为 C 程序创建接口时可以遵循的一系列步骤：
>
> * 确定要包装的功能。可能没有必要访问 C 程序的每个单独的函数——因此，一点预先的考虑可以大大简化最终的脚本语言接口。查找要包装的东西的话，C 头文件是特别好的源头。
> * 创建一个新的接口文件来描述程序的脚本语言接口。
> * 将适当的声明复制到接口文件中，或使用 SWIG 的 `%include` 指令处理整个 C 源/头文件。
> * 确保接口文件中的所有内容都使用 ANSI C/C++ 语法。
> * 确保接口文件中提供了所有必需的 `typedef` 声明和类型信息。特别是，确保按照 C/C++ 编译器的要求以正确的顺序指定类型信息。最重要的是，在使用之前定义一个类型！如果需要，C 编译器将告诉你完整类型信息是否不可用，而 SWIG 通常不会发出警告或错误，因为它设计为在没有完整类型信息的情况下工作。但是，如果未正确指定类型信息，则包装器可能是次优的，甚至会导致无法编译的 C/C++ 代码。
> * 如果你的程序具有 `main()` 函数，则可能需要重命名（只读）。
> * 运行 SWIG 并编译。
>
> 虽然这可能听起来很复杂，但是一旦掌握了它，这个过程就相当容易了。
>
> 在构建接口的过程中，SWIG 可能会遇到语法错误或其他问题。处理此问题的最佳方法是将有问题的代码复制到单独的接口文件中并进行编辑。但是，SWIG 开发人员非常努力地改进 SWIG 解析器——你应该将解析错误报告给 [swig-devel 邮件列表](http://www.swig.org/mail.html)或 [SWIG bug tracker](http://www.swig.org/bugs.html)。

### 5.7.2 SWIG 接口文件

The preferred method of using SWIG is to generate a separate interface file. Suppose you have the following C header file :

> 使用 SWIG 的首选方法是生成单独的接口文件。假设你有以下 C 头文件：

```c
/* File : header.h */

#include <stdio.h>
#include <math.h>

extern int foo(double);
extern double bar(int, int);
extern void dump(FILE *f);
```

A typical SWIG interface file for this header file would look like the following :

> 此头文件的典型 SWIG 接口文件如下所示：

```
/* File : interface.i */
%module mymodule
%{
#include "header.h"
%}
extern int foo(double);
extern double bar(int, int);
extern void dump(FILE *f);
```

Of course, in this case, our header file is pretty simple so we could use a simpler approach and use an interface file like this:

> 当然，在这种情况下，我们的头文件非常简单，所以我们可以使用更简单的方法并使用这样的接口文件：

```
/* File : interface.i */
%module mymodule
%{
#include "header.h"
%}
%include "header.h"
```

The main advantage of this approach is minimal maintenance of an interface file for when the header file changes in the future. In more complex projects, an interface file containing numerous `%include` and `#include` statements like this is one of the most common approaches to interface file design due to lower maintenance overhead.

> 这种方法的主要优点是，当头文件将来发生变化时，接口文件的维护很少。在更复杂的项目中，包含许多 `%include` 和 `#include` 语句的接口文件是最常见的接口文件设计方法之一，因为维护成本较低。

### 5.7.3 为什么使用单独的接口文件？

Although SWIG can parse many header files, it is more common to write a special `.i` file defining the interface to a package. There are several reasons why you might want to do this:

* It is rarely necessary to access every single function in a large package. Many C functions might have little or no use in a scripted environment. Therefore, why wrap them?
* Separate interface files provide an opportunity to provide more precise rules about how an interface is to be constructed.
* Interface files can provide more structure and organization.
* SWIG can't parse certain definitions that appear in header files. Having a separate file allows you to eliminate or work around these problems.
* Interface files provide a more precise definition of what the interface is. Users wanting to extend the system can go to the interface file and immediately see what is available without having to dig it out of header files.

> 虽然 SWIG 可以解析许多头文件，但更常见的是编写一个特殊的 `.i` 文件来定义包的接口。你希望这样做的可能原因有以下几种：
>
> * 很少需要访问大型包中的每个功能。许多 C 函数可能在脚本环境中很少或没有用处。因此，为什么要包装它们？
> * 单独的接口文件提供了一个机会，可以提供有关如何构造接口的更精确的规则。
> * 接口文件可以提供更多的结构和组织。
> * SWIG 无法解析头文件中出现的某些定义。拥有单独的文件可以消除或解决这些问题。
> * 接口文件提供了更精确的接口定义。想要扩展系统的用户可以转到接口文件，并立即查看可用的内容，而无需将其从头文件中删除。

### 5.7.4 获得正确的头文件

Sometimes, it is necessary to use certain header files in order for the code generated by SWIG to compile properly. Make sure you include certain header files by using a `%{ %}` block like this:

> 有时，必须使用某些头文件才能使 SWIG 生成的代码正确编译。确保使用 `%{ %}` 块包含某些头文件，如下所示：

```
%module graphics
%{
#include <GL/gl.h>
#include <GL/glu.h>
%}

// Put the rest of the declarations here
...
```

### 5.7.5 怎么处理 `main()`

If your program defines a `main()` function, you may need to get rid of it or rename it in order to use a scripting language. Most scripting languages define their own `main()` procedure that is called instead. `main()` also makes no sense when working with dynamic loading. There are a few approaches to solving the `main()` conflict:

* Get rid of `main()` entirely.
* Rename `main()` to something else. You can do this by compiling your C program with an option like `-Dmain=oldmain`.
* Use conditional compilation to only include `main()` when not using a scripting language.

Getting rid of `main()` may cause potential initialization problems of a program. To handle this problem, you may consider writing a special function called `program_init()` that initializes your program upon startup. This function could then be called either from the scripting language as the first operation, or when the SWIG generated module is loaded.

As a general note, many C programs only use the `main()` function to parse command line options and to set parameters. However, by using a scripting language, you are probably trying to create a program that is more interactive. In many cases, the old `main()` program can be completely replaced by a Perl, Python, or Tcl script.

**Note:** In some cases, you might be inclined to create a scripting language wrapper for `main()`. If you do this, the compilation will probably work and your module might even load correctly. The only trouble is that when you call your `main()` wrapper, you will find that it actually invokes the `main()` of the scripting language interpreter itself! This behavior is a side effect of the symbol binding mechanism used in the dynamic linker. The bottom line: don't do this.

> 如果你的程序定义了一个 `main()` 函数，你可能需要删除它或重命名它以使用脚本语言。大多数脚本语言都定义了自己调用的 `main()` 程序。使用动态加载时，`main()` 也没有意义。解决 `main()` 冲突有几种方法：
>
> * 完全摆脱 `main()`。
> * 将 `main()` 重命名为其他内容。你可以通过使用 `-Dmain = oldmain` 等选项编译 C 程序来完成此操作。
> * 在不使用脚本语言时，使用条件编译仅包含 `main()`。
>
> 摆脱 `main()` 可能会导致程序潜在的初始化问题。要处理这个问题，你可以考虑编写一个名为 `program_init()` 的特殊函数，它在启动时初始化你的程序。然后可以从脚本语言中调用此函数作为第一个操作，或者在加载 SWIG 生成的模块时调用此函数。
>
> 总的来说，许多 C 程序只使用 `main()` 函数来解析命令行选项和设置参数。但是，通过使用脚本语言，你可能正在尝试创建更具交互性的程序。在许多情况下，旧的 `main()` 程序可以完全被 Perl、Python 或 Tcl 脚本替换。
>
> **注意：**在某些情况下，你可能倾向于为 `main()` 创建脚本语言包装器。如果你这样做，编译可能会工作，你的模块甚至可能正确加载。唯一的麻烦是当你调用你的 `main()` 包装器时，你会发现它实际上调用了脚本语言解释器本身的 `main()`！此行为是动态链接器中使用的符号绑定机制的副作用。底线：不要这样做。
