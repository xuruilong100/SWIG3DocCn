[TOC]

# 8 预处理

SWIG includes its own enhanced version of the C preprocessor. The preprocessor supports the standard preprocessor directives and macro expansion rules. However, a number of modifications and enhancements have been made. This chapter describes some of these modifications.

> SWIG 包含自己的增强型 C 预处理器。预处理器支持标准的预处理程序指令和宏扩展规则。但是，已经进行了许多修改和增强。本章介绍其中的一些修改。

## 8.1 文件包含

To include another file into a SWIG interface, use the `%include`directive like this:

> 要将另一个文件包含到 SWIG 接口文件中，请使用 `%include` 指令，如下所示：

```
%include "pointer.i"
```

Unlike, `#include`, `%include` includes each file once (and will not reload the file on subsequent `%include` declarations). Therefore, it is not necessary to use include-guards in SWIG interfaces.

By default, the `#include` is ignored unless you run SWIG with the `-includeall` option. The reason for ignoring traditional includes is that you often don't want SWIG to try and wrap everything included in standard header system headers and auxiliary files.

> 与 `#include` 不同，对于每个文件 `%include` 仅包含一次（并且不会在随后的 `%include` 声明中重新加载文件）。因此，没有必要在 SWIG 接口文件中使用包含保护。
>
> 默认情况下，除非你使用 `-includeall` 选项运行 SWIG，否则将忽略 `#include`。忽略传统包含的原因是，你通常不希望 SWIG 尝试包装标准库头文件、系统头文件和辅助文件中包含的所有内容。

## 8.2 文件导入

SWIG provides another file inclusion directive with the `%import` directive. For example:

> SWIG 提供了另一个文件包含指令——`%import`。例如：

```
%import "foo.i"
```

The purpose of `%import` is to collect certain information from another SWIG interface file or a header file without actually generating any wrapper code. Such information generally includes type declarations (e.g., `typedef`) as well as C++ classes that might be used as base-classes for class declarations in the interface. The use of `%import` is also important when SWIG is used to generate extensions as a collection of related modules. This is an advanced topic and is described in later in the [Working with Modules](http://swig.org/Doc3.0/Modules.html#Modules)chapter.

The `-importall` directive tells SWIG to follow all `#include`statements as imports. This might be useful if you want to extract type definitions from system header files without generating any wrappers.

> `%import` 的目的是从另一个 SWIG 接口文件或头文件中收集某些信息，而无需实际生成任何包装代码。此类信息通常包括类型声明（例如 `typedef`）以及可用作接口中类声明的 C++ 基类。当使用 SWIG 生成扩展作为相关模块的集合时，使用 `%import` 也很重要。这是一个高级主题，稍后在[使用模块](http://swig.org/Doc3.0/Modules.html#Modules)一章中进行介绍。
>
> `-importall` 指令告诉 SWIG 遵循所有的 `#include` 声明作为导入。如果要从系统头文件中提取类型定义而不生成任何包装，这可能很有用。

## 8.3 条件编译

SWIG fully supports the use of `#if`, `#ifdef`, `#ifndef`, `#else`, `#endif` to conditionally include parts of an interface. The following symbols are predefined by SWIG when it is parsing the interface:

> SWIG 完全支持使用 `#if`、`#ifdef`、`#ifndef`、`#else` 和 `#endif` 来有条件地包含接口的各个部分。SWIG 在解析接口时预定义了以下符号：

```
SWIG                            Always defined when SWIG is processing a file
SWIGIMPORTED                    Defined when SWIG is importing a file with %import
SWIG_VERSION                    Hexadecimal (binary-coded decimal) number containing SWIG version,
                                such as 0x010311 (corresponding to SWIG-1.3.11).
SWIGALLEGROCL                   Defined when using Allegro CL
SWIGCFFI                        Defined when using CFFI
SWIGCHICKEN                     Defined when using CHICKEN
SWIGCLISP                       Defined when using CLISP
SWIGCSHARP                      Defined when using C#
SWIGGUILE                       Defined when using Guile
SWIGJAVA                        Defined when using Java
SWIGJAVASCRIPT                  Defined when using Javascript
SWIG_JAVASCRIPT_JSC             Defined when using Javascript for JavascriptCore
SWIG_JAVASCRIPT_V8              Defined when using Javascript for v8 or node.js
SWIGLUA                         Defined when using Lua
SWIGMODULA3                     Defined when using Modula-3
SWIGMZSCHEME                    Defined when using Mzscheme
SWIGOCAML                       Defined when using Ocaml
SWIGOCTAVE                      Defined when using Octave
SWIGPERL                        Defined when using Perl
SWIGPHP                         Defined when using PHP5 or PHP7
SWIGPHP5                        Defined when using PHP5
SWIGPHP7                        Defined when using PHP7
SWIGPIKE                        Defined when using Pike
SWIGPYTHON                      Defined when using Python
SWIGR                           Defined when using R
SWIGRUBY                        Defined when using Ruby
SWIGSCILAB                      Defined when using Scilab
SWIGSEXP                        Defined when using S-expressions
SWIGTCL                         Defined when using Tcl
SWIGXML                         Defined when using XML
```

In addition, SWIG defines the following set of standard C/C++ macros:

> 此外，SWIG 定义了以下标准 C/C++ 宏集合：

```
__LINE__                        Current line number
__FILE__                        Current file name
__STDC__                        Defined to indicate ANSI C
__cplusplus                     Defined when -c++ option used
```

Interface files can look at these symbols as necessary to change the way in which an interface is generated or to mix SWIG directives with C code. These symbols are also defined within the C code generated by SWIG (except for the symbol `SWIG` which is only defined within the SWIG compiler).

> 接口文件可以根据需要查看这些符号，以更改生成接口的方式或将 SWIG 指令与 C 代码混合。这些符号也在 SWIG 生成的 C 代码中定义（符号 `SWIG` 仅在 SWIG 编译器中定义）。

## 8.4 宏扩展

Traditional preprocessor macros can be used in SWIG interfaces. Be aware that the `#define` statement is also used to try and detect constants. Therefore, if you have something like this in your file,

> 传统的预处理器宏可以在 SWIG 接口中使用。注意，`#define` 语句也用于尝试和检测常量。因此，如果文件中包含类似的内容，

```c++
#ifndef _FOO_H 1
#define _FOO_H 1
...
#endif
```

you may get some extra constants such as `_FOO_H` showing up in the scripting interface.

More complex macros can be defined in the standard way. For example:

> 你可能会在脚本接口中看到一些额外的常量，例如 `_FOO_H` 。
>
> 可以用标准方式定义更复杂的宏。例如：

```c++
#define EXTERN extern
#ifdef __STDC__
#define _ANSI(args)   (args)
#else
#define _ANSI(args) ()
#endif
```

The following operators can appear in macro definitions:

* `#x`

Converts macro argument `x` to a string surrounded by double quotes ("x").

* `x ## y`

Concatenates x and y together to form `xy`.

* `x`

If `x` is a string surrounded by double quotes, do nothing. Otherwise, turn into a string like `#x`. This is a non-standard SWIG extension.

> 以下运算符可以出现在宏定义中：
>
> * `#x`
>
> 将宏参数 `x` 转换为双引号（`"x"`）包围的字符串。
>
> * `x ## y`
>
> 将 `x` 和 `y` 串联在一起形成 `xy`。
>
> * `x`
>
> 如果 `x` 是用双引号引起来的字符串，则不执行任何操作。否则，将其转换为类似于 `#x` 的字符串。这是非标准的 SWIG 扩展。

## 8.5 SWIG 宏

SWIG provides an enhanced macro capability with the `%define`and `%enddef` directives. For example:

> SWIG 通过 `%define` 和 `%enddef` 指令提供增强的宏功能。例如：

```
%define ARRAYHELPER(type, name)
%inline %{
type *new_ ## name (int nitems) {
  return (type *) malloc(sizeof(type)*nitems);
}
void delete_ ## name(type *t) {
  free(t);
}
type name ## _get(type *t, int index) {
  return t[index];
}
void name ## _set(type *t, int index, type val) {
  t[index] = val;
}
%}
%enddef

ARRAYHELPER(int, IntArray)
ARRAYHELPER(double, DoubleArray)
```

The primary purpose of `%define` is to define large macros of code. Unlike normal C preprocessor macros, it is not necessary to terminate each line with a continuation character (\)--the macro definition extends to the first occurrence of `%enddef`. Furthermore, when such macros are expanded, they are reparsed through the C preprocessor. Thus, SWIG macros can contain all other preprocessor directives except for nested `%define` statements.

The SWIG macro capability is a very quick and easy way to generate large amounts of code. In fact, many of SWIG's advanced features and libraries are built using this mechanism (such as C++ template support).

> `%define` 的主要目的是定义大型的代码宏。与普通的 C 预处理器宏不同，它不必以连续字符（`\`）结束每一行——宏定义扩展到第一次出现 `%enddef` 的地方。此外，扩展此类宏时，将通过 C 预处理器对其进行重新解析。因此，除了嵌套的 `%define` 语句以外，SWIG 宏可以包含的所有其他预处理程序指令。
>
> SWIG 宏功能是一种非常快速简便地生成大量代码的方法。实际上，SWIG 的许多高级功能和库都是使用此机制构建的（例如 C++ 模板支持）。

## 8.6 C99 和 GNU 扩展

SWIG-1.3.12 and newer releases support variadic preprocessor macros. For example:

> SWIG-1.3.12 和更高版本支持可变参数预处理器宏。例如：

```c++
#define DEBUGF(fmt, ...)   fprintf(stderr, fmt, __VA_ARGS__)
```

When used, any extra arguments to `...` are placed into the special variable `__VA_ARGS__`. This also works with special SWIG macros defined using `%define`.

SWIG allows a variable number of arguments to be empty. However, this often results in an extra comma (, ) and syntax error in the resulting expansion. For example:

> 使用时，会将任何 `...` 的额外参数放入特殊变量 `__VA_ARGS__` 中。这也适用于使用 `%define` 定义的特殊 SWIG 宏。
>
> SWIG 允许可变数量的参数为空。但是，这通常在结果扩展中导致额外的逗号（`,`）和语法错误。例如：

```
DEBUGF("hello");   --> fprintf(stderr, "hello", );
```

To get rid of the extra comma, use `##` like this:

> 要消除多余的逗号，请使用 `##`，如下所示：

```
#define DEBUGF(fmt, ...)   fprintf(stderr, fmt, ##__VA_ARGS__)
```

SWIG also supports GNU-style variadic macros. For example:

> SWIG 还支持 GNU 风格的可变参数宏。例如：

```c++
#define DEBUGF(fmt, args...)  fprintf(stdout, fmt, args)
```

**Comment:** It's not entirely clear how variadic macros might be useful to interface building. However, they are used internally to implement a number of SWIG directives and are provided to make SWIG more compatible with C99 code.

> **注释**：尚不完全清楚可变参数宏对建立接口的有用性。但是，它们在内部用于实现许多 SWIG 指令，并用于使 SWIG 与 C99 代码更兼容。

## 8.7 预处理与分隔符

The preprocessor handles `{ }`, `" "` and `%{ %}` delimiters differently.

> 预处理器以不同的方式处理 `{}`、`" "` 和 `%{%}` 分隔符。

### 8.7.1 预处理与 `%{...%}`、`"..."` 分隔符

The SWIG preprocessor does not process any text enclosed in a code block %{ ... %}. Therefore, if you write code like this,

> SWIG 预处理程序不处理代码块 `%{...%}` 中包含的任何文本。因此，如果你编写这样的代码，

```
%{
#ifdef NEED_BLAH
int blah() {
  ...
}
#endif
%}
```

the contents of the `%{ ... %}` block are copied without modification to the output (including all preprocessor directives).

> `%{...%}` 块的内容被复制并输出（包括所有预处理指令）而不做修改。

### 8.7.2 预处理与 `{...}` 分隔符

SWIG always runs the preprocessor on text appearing inside `{ ... }`. However, sometimes it is desirable to make a preprocessor directive pass through to the output file. For example:

> SWIG 始终对出现在 `{...}` 中的文本运行预处理器。但是，有时希望使预处理程序指令传递到输出文件。例如：

```
%extend Foo {
  void bar() {
    #ifdef DEBUG
      printf("I'm in bar\n");
    #endif
  }
}
```

By default, SWIG will interpret the `#ifdef DEBUG` statement. However, if you really wanted that code to actually go into the wrapper file, prefix the preprocessor directives with `%` like this:

> 默认情况下，SWIG 将解释 `#ifdef DEBUG` 语句。但是，如果你真的希望该代码真正进入包装文件，请在预处理器指令前加上 `%` 前缀，如下所示：

```
%extend Foo {
  void bar() {
    %#ifdef DEBUG
      printf("I'm in bar\n");
    %#endif
  }
}
```

SWIG will strip the extra `%` and leave the preprocessor directive in the code.

> SWIG 将剥离多余的 `%` 并将预处理器指令保留在代码中。

## 8.8 预处理器与类型映射

[Typemaps](http://swig.org/Doc3.0/Typemaps.html#Typemaps) support a special attribute called `noblock` where the `{...}` delimiters can be used, but the delimiters are not actually generated into the code. The effect is then similar to using `" "` or `%{ %}` delimiters but the code **is** run through the preprocessor. For example:

> [类型映射](http://swig.org/Doc3.0/Typemaps.html#Typemaps)支持名为 `noblock` 的特殊属性，其中可以使用 `{...}` 分隔符，但实际上不会将分隔符生成为代码。这样，其效果类似于使用 `" "` 或 `%{ %}` 分隔符，但是代码通过预处理器运行。例如：

```
#define SWIG_macro(CAST) (CAST)$input
%typemap(in) Int {$1= SWIG_macro(int);}
```

might generate

> 也许会生成

```c++
{
    arg1=(int)jarg1;
}
```

whereas

> 然而

```
#define SWIG_macro(CAST) (CAST)$input
%typemap(in, noblock=1) Int {$1= SWIG_macro(int);}
```

might generate

> 也许会生成

```c++
arg1=(int)jarg1;
```

and

> 以及

```
#define SWIG_macro(CAST) (CAST)$input
%typemap(in) Int %{$1=SWIG_macro(int);%}
```

would generate

> 也许会生成

```c++
arg1=SWIG_macro(int);
```

## 8.9 查看预处理器的输出

Like many compilers, SWIG supports a `-E` command line option to display the output from the preprocessor. When the `-E` switch is used, SWIG will not generate any wrappers. Instead the results after the preprocessor has run are displayed. This might be useful as an aid to debugging and viewing the results of macro expansions.

> 像许多编译器一样，SWIG 支持 `-E` 命令行选项来显示预处理器的输出。使用 `-E` 开关时，SWIG 将不会生成任何包装。而是显示预处理器运行后的结果。这可能有助于调试和查看宏扩展的结果。

## 8.10 `#error` 与 `#warning` 指令

SWIG supports the commonly used `#warning` and `#error` preprocessor directives. The `#warning` directive will cause SWIG to issue a warning then continue processing. The `#error` directive will cause SWIG to exit with a fatal error. Example usage:

> SWIG 支持常用的 `#warning` 和 `#error` 预处理器指令。`#warning` 指令将使 SWIG 发出警告，然后继续处理。`#error` 指令将导致 SWIG 退出并出现致命错误。用法示例：

```
#error "This is a fatal error message"
#warning "This is a warning message"
```

The `#error` behaviour can be made to work like `#warning` if the `-cpperraswarn` commandline option is used. Alternatively, the `#pragma` directive can be used to the same effect, for example:

> 如果使用了 `-cpperraswarn` 命令行选项，则可以使 `#error` 行为像 `#warning` 一样工作。另外，也可以使用 `#pragma` 指令达到相同的效果，例如：

```c++
/* Modified behaviour: #error does not cause SWIG to exit with error */
#pragma SWIG cpperraswarn=1
/* Normal behaviour: #error does cause SWIG to exit with error */
#pragma SWIG cpperraswarn=0
```
