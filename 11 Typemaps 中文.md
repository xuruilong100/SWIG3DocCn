[TOC]

# 11 类型映射

## 11.1 引言

Chances are, you are reading this chapter for one of two reasons; you either want to customize SWIG's behavior or you overheard someone mumbling some incomprehensible drivel about "typemaps" and you asked yourself "typemaps, what are those?" That said, let's start with a short disclaimer that "typemaps" are an advanced customization feature that provide direct access to SWIG's low-level code generator. Not only that, they are an integral part of the SWIG C++ type system (a non-trivial topic of its own). Typemaps are generally *not* a required part of using SWIG. Therefore, you might want to re-read the earlier chapters if you have found your way to this chapter with only a vague idea of what SWIG already does by default.

> 你正在阅读本章的原因可能有两个：你想自定义 SWIG 的行为，或是无意中听到有人抱怨“typemaps”一词，并问自己“typemaps 是什么？”就是说，让我们从一个简短的免责声明开始，即“typemaps”是一种高级定制功能，可以直接访问 SWIG 的低级代码生成器。不仅如此，它们还是 SWIG C++ 类型系统（其自身的重要内容）的组成部分。通常，*不是*使用 SWIG 的必需部分。因此，如果你对本章的方法一无所知，那么你可能想重新阅读前面的章节，而对于 SWIG 默认情况下已经做的事情却含糊其辞。

### 11.1.1 类型转换

One of the most important problems in wrapper code generation is the conversion or marshalling of datatypes between programming languages. Specifically, for every C/C++ declaration, SWIG must somehow generate wrapper code that allows values to be passed back and forth between languages. Since every programming language represents data differently, this is not a simple of matter of simply linking code together with the C linker. Instead, SWIG has to know something about how data is represented in each language and how it can be manipulated.

To illustrate, suppose you had a simple C function like this:

> 包装代码生成中最重要的问题之一是编程语言之间数据类型的转换或编组。具体来说，对于每个 C/C++ 声明，SWIG 必须以某种方式生成包装器代码，该包装器代码允许在语言之间来回传递值。由于每种编程语言表示数据的方式都不相同，因此简单地将代码与 C 链接器链接在一起并不是一件容易的事。相反，SWIG 必须了解有关每种语言如何表示数据以及如何对其进行操纵的知识。
>
> 为了说明这一点，假设你有一个简单的 C 函数，如下所示：

```c
int factorial(int n);
```

To access this function from Python, a pair of Python API functions are used to convert integer values. For example:

> 要从 Python 访问此函数，一对 Python API 函数用于转换整数值。例如：

```c
long PyInt_AsLong(PyObject *obj);      /* Python --> C */
PyObject *PyInt_FromLong(long x);      /* C --> Python */
```

The first function is used to convert the input argument from a Python integer object to C `long`. The second function is used to convert a value from C back into a Python integer object.

Inside the wrapper function, you might see these functions used like this:

> 第一个函数用于将输入参数从 Python 整数对象转换为 C 中的 `long`。 第二个函数用于将值从 C 转换回 Python 整数对象。
>
> 在包装器函数中，你可能会看到这些函数的用法如下：

```c
PyObject *wrap_factorial(PyObject *self, PyObject *args) {
  int       arg1;
  int       result;
  PyObject *obj1;
  PyObject *resultobj;

  if (!PyArg_ParseTuple("O:factorial", &obj1)) return NULL;
  arg1 = PyInt_AsLong(obj1);
  result = factorial(arg1);
  resultobj = PyInt_FromLong(result);
  return resultobj;
}
```

Every target language supported by SWIG has functions that work in a similar manner. For example, in Perl, the following functions are used:

> SWIG 支持的每种目标语言都具有以类似方式工作的功能。例如，在 Perl 中，使用以下功能：

```c
IV SvIV(SV *sv);                     /* Perl --> C */
void sv_setiv(SV *sv, IV val);       /* C --> Perl */
```

In Tcl:

```c
int Tcl_GetLongFromObj(Tcl_Interp *interp, Tcl_Obj *obj, long *value);
Tcl_Obj *Tcl_NewIntObj(long value);
```

The precise details are not so important. What is important is that all of the underlying type conversion is handled by collections of utility functions and short bits of C code like this--you simply have to read the extension documentation for your favorite language to know how it works (an exercise left to the reader).

> 确切的细节不是那么重要。重要的是，所有底层类型转换都由实用程序函数和类似这样的 C 代码的短代码集合处理——你只需阅读自己喜欢的语言的扩展文档以了解其工作原理（留给读者的练习）。

### 11.1.2 类型映射

Since type handling is so central to wrapper code generation, SWIG allows it to be completely defined (or redefined) by the user. To do this, a special `%typemap` directive is used. For example:

> 由于类型处理对于包装代码生成非常重要，因此 SWIG 允许用户完全定义（或重新定义）它。为此，使用特殊的 `%typemap` 指令。例如：

```
/* Convert from Python --> C */
%typemap(in) int {
  $1 = PyInt_AsLong($input);
}

/* Convert from C --> Python */
%typemap(out) int {
  $result = PyInt_FromLong($1);
}
```

At first glance, this code will look a little confusing. However, there is really not much to it. The first typemap (the "in" typemap) is used to convert a value from the target language to C. The second typemap (the "out" typemap) is used to convert in the other direction. The content of each typemap is a small fragment of code that is inserted directly into the SWIG generated wrapper functions. The code is usually C or C++ code which will be generated into the C/C++ wrapper functions. Note that this isn't always the case as some target language modules allow target language code within the typemaps which gets generated into target language specific files. Within this code, a number of special variables prefixed with a `$` are expanded. These are really just placeholders for C/C++ variables that are generated in the course of creating the wrapper function. In this case, `$input` refers to an input object that needs to be converted to C/C++ and `$result` refers to an object that is going to be returned by a wrapper function. `$1` refers to a C/C++ variable that has the same type as specified in the typemap declaration (an `int` in this example).

A short example might make this a little more clear. If you were wrapping a function like this:

> 乍看之下，这段代码看起来有些混乱。但是，实际上并没有太多。第一个类型映射（`in` 类型映射）用于将值从目标语言转换为 C。第二个类型映射（`out` 类型映射）用于向另一个方向转换。每个类型映射的内容都是一小段代码，直接插入 SWIG 生成的包装器函数中。该代码通常是 C 或 C++ 代码，它们将生成到 C/C++ 包装器函数中。请注意，并非总是如此，因为某些目标语言模块允许类型映射中的目标语言代码生成到目标语言特定的文件中。在此代码中，将扩展许多带有 `$` 前缀的特殊变量。这些实际上只是 C/C++ 变量的占位符，这些变量是在创建包装器函数的过程中生成的。在这种情况下，`$input` 是指需要转换为 C/C++ 的输入对象，而 `$result` 是指将由包装器函数返回的对象。`$1` 指的是一个 C/C++ 变量，其类型与类型映射声明中指定的类型相同（本例中为 `int`）。
>
> 一个简短的示例可能会使这一点更加清楚。如果要包装这样的函数：

```c
int gcd(int x, int y);
```

A wrapper function would look approximately like this:

> 包装器函数大致如下所示：

```c
PyObject *wrap_gcd(PyObject *self, PyObject *args) {
  int arg1;
  int arg2;
  int result;
  PyObject *obj1;
  PyObject *obj2;
  PyObject *resultobj;

  if (!PyArg_ParseTuple("OO:gcd", &obj1, &obj2)) return NULL;

  /* "in" typemap, argument 1 */
  {
    arg1 = PyInt_AsLong(obj1);
  }

  /* "in" typemap, argument 2 */
  {
    arg2 = PyInt_AsLong(obj2);
  }

  result = gcd(arg1, arg2);

  /* "out" typemap, return value */
  {
    resultobj = PyInt_FromLong(result);
  }

  return resultobj;
}
```

In this code, you can see how the typemap code has been inserted into the function. You can also see how the special $ variables have been expanded to match certain variable names inside the wrapper function. This is really the whole idea behind typemaps--they simply let you insert arbitrary code into different parts of the generated wrapper functions. Because arbitrary code can be inserted, it possible to completely change the way in which values are converted.

> 在此代码中，你可以看到如何将类型映射代码插入到函数中。你还可以看到特殊的 `$` 变量是如何扩展的，以匹配包装器函数中的某些变量名称。这实际上就是类型映射背后的全部思想，它们只是让你将任意代码插入生成的包装器函数的不同部分。由于可以插入任意代码，因此可以完全改变值转换的方式。

### 11.1.3 模式匹配

As the name implies, the purpose of a typemap is to "map" C datatypes to types in the target language. Once a typemap is defined for a C datatype, it is applied to all future occurrences of that type in the input file. For example:

> 顾名思义，类型映射的目的是将 C 数据类型“映射”为目标语言中的类型。一旦为 C 数据类型定义类型映射，它将应用于输入文件中出现的所有该类型。例如：

```
/* Convert from Perl --> C */
%typemap(in) int {
  $1 = SvIV($input);
}

...
int factorial(int n);
int gcd(int x, int y);
int count(char *s, char *t, int max);
```

The matching of typemaps to C datatypes is more than a simple textual match. In fact, typemaps are fully built into the underlying type system. Therefore, typemaps are unaffected by `typedef`, namespaces, and other declarations that might hide the underlying type. For example, you could have code like this:

> 类型映射与 C 数据类型的匹配不仅仅是简单的文本匹配。实际上，类型映射完全内置在基础类型系统中。因此，类型映射不受 `typedef`、命名空间和其他可能隐藏基础类型的声明的影响。例如，你可能具有以下代码：

```
/* Convert from Ruby--> C */
%typemap(in) int {
  $1 = NUM2INT($input);
}
...
typedef int Integer;
namespace foo {
  typedef Integer Number;
};

int foo(int x);
int bar(Integer y);
int spam(foo::Number a, foo::Number b);
```

In this case, the typemap is still applied to the proper arguments even though typenames don't always match the text "int". This ability to track types is a critical part of SWIG--in fact, all of the target language modules work merely define a set of typemaps for the basic types. Yet, it is never necessary to write new typemaps for typenames introduced by `typedef`.

In addition to tracking typenames, typemaps may also be specialized to match against a specific argument name. For example, you could write a typemap like this:

> 在这种情况下，即使类型名并不总是与文本 `int` 匹配，也仍然将类型映射应用于适当的参数。这种跟踪类型的能力是 SWIG 的重要组成部分——实际上，所有目标语言模块都只能为基本类型定义一组类型映射。但是，从来没有必要为 `typedef` 引入的类型名编写新的类型映射。
>
> 除了跟踪类型名称之外，类型映射还可以专门用于与特定的参数名称匹配。例如，你可以编写这样的类型映射：

```
%typemap(in) double nonnegative {
  $1 = PyFloat_AsDouble($input);
  if ($1 < 0) {
    PyErr_SetString(PyExc_ValueError, "argument must be nonnegative.");
    SWIG_fail;
  }
}

...
double sin(double x);
double cos(double x);
double sqrt(double nonnegative);

typedef double Real;
double log(Real nonnegative);
...
```

For certain tasks such as input argument conversion, typemaps can be defined for sequences of consecutive arguments. For example:

> 对于某些任务，例如输入参数转换，可以为连续参数序列定义类型映射。例如：

```
%typemap(in) (char *str, int len) {
  $1 = PyString_AsString($input);   /* char *str */
  $2 = PyString_Size($input);       /* int len   */
}
...
int count(char *str, int len, char c);
```

In this case, a single input object is expanded into a pair of C arguments. This example also provides a hint to the unusual variable naming scheme involving `$1`, `$2`, and so forth.

> 在这种情况下，单个输入对象将扩展为一对 C 参数。这个例子也暗示了涉及不寻常的变量命名方案，包括 `$1`、`$2` 等等。

### 11.1.4 复用类型映射

Typemaps are normally defined for specific type and argument name patterns. However, typemaps can also be copied and reused. One way to do this is to use assignment like this:

> 类型映射通常为特定的类型和参数名称模式而定义。但是，类型映射也可以复制和重用。一种方法是使用赋值：

```
%typemap(in) Integer = int;
%typemap(in) (char *buffer, int size) = (char *str, int len);
```

A more general form of copying is found in the `%apply` directive like this:

> 在 `%apply` 指令中可以找到更通用的复制形式，如下所示：

```
%typemap(in) int {
  /* Convert an integer argument */
  ...
}
%typemap(out) int {
  /* Return an integer value */
  ...
}

/* Apply all of the integer typemaps to size_t */
%apply int { size_t };
```

`%apply` merely takes *all* of the typemaps that are defined for one type and applies them to other types. Note: you can include a comma separated set of types in the `{...}` part of `%apply`.

It should be noted that it is not necessary to copy typemaps for types that are related by `typedef`. For example, if you have this,

> `%apply` 仅接受为一种类型定义的*所有*类型映射，并将它们应用于其他类型。注意：你可以在 `%apply` 的 `{...}` 部分中包含一组用逗号分隔的类型。
>
> 应该注意的是，没有必要为 `typedef` 相关的类型复制类型映射。例如，如果你有这个，

```c++
typedef int size_t;
```

then SWIG already knows that the `int` typemaps apply. You don't have to do anything.

> 那么 SWIG 已经知道了 `int` 的类型映射。你不必做任何事情。

### 11.1.5 类型映射能干什么？

The primary use of typemaps is for defining wrapper generation behavior at the level of individual C/C++ datatypes. There are currently six general categories of problems that typemaps address:

> 类型映射的主要用途是在单个 C/C++ 数据类型级别上定义包装器生成行为。当前，类型映射解决了六大类问题：

**Argument handling**

> **参数处理**

```c
int foo(int x, double y, char *s);
```

* Input argument conversion ("in" typemap).
* Input argument type checking for types used in overloaded methods ("typecheck" typemap).
* Output argument handling ("argout" typemap).
* Input argument value checking ("check" typemap).
* Input argument initialization ("arginit" typemap).
* Default arguments ("default" typemap).
* Input argument resource management ("freearg" typemap).

> * 输入参数转换（`in` 类型映射）。
> * 重载方法中的输入参数类型检查（`typecheck` 类型映射）。
> * 输出参数处理（`argout` 类型映射）。
> * 输入参数值检查（`检查` 类型映射）。
> * 输入参数初始化（`arginit` 类型映射）。
> * 默认参数（`默认` 类型映射）。
> * 输入参数资源管理（`freearg` 类型映射）。

**Return value handling**

> **返回值处理**

```c
int foo(int x, double y, char *s);
```

* Function return value conversion ("out" typemap).
* Return value resource management ("ret" typemap).
* Resource management for newly allocated objects ("newfree" typemap).

> * 函数返回值转换（`out` 类型映射）。
> * 返回值资源管理（`ret` 类型映射）。
> * 新分配对象的资源管理（`newfree` 类型映射）。

**Exception handling**

> **异常处理**

```c
int foo(int x, double y, char *s) throw(MemoryError, IndexError);
```

* Handling of C++ exception specifications. ("throw" typemap).

> * 处理 C++ 异常规范。（`throw` 类型映射）。

**Global variables**

> **全局变量**

```c
int foo;
```

* Assignment of a global variable. ("varin" typemap).
* Reading a global variable. ("varout" typemap).

> * 分配全局变量。（`varin` 类型映射）。
> * 读取全局变量。（`varout` 类型映射）。

**Member variables**

> **成员变量**

```c
struct Foo {
  int x[20];
};
```

* Assignment of data to a class/structure member. ("memberin" typemap).

> * 将数据分配给类或结构体成员。（`memberin` 类型映射）。

**Constant creation**

> **创建常量**

```
#define FOO 3
%constant int BAR = 42;
enum { ALE, LAGER, STOUT };
```

* Creation of constant values. ("consttab" or "constcode" typemap).

Details of each of these typemaps will be covered shortly. Also, certain language modules may define additional typemaps that expand upon this list. For example, the Java module defines a variety of typemaps for controlling additional aspects of the Java bindings. Consult language specific documentation for further details.

> * 创建常数值。（`consttab` 或 `constcode` 类型映射）。
>
> 每个类型映射的详细内容很快会提到。同样，某些语言模块可能会定义其他类型映射以扩展此列表上。例如，Java 模块定义了各种类型映射来控制 Java 绑定的其他方面。请查阅特定于语言的文档以获取更多详细信息。

### 11.1.6 类型映射不能干什么？

Typemaps can't be used to define properties that apply to C/C++ declarations as a whole. For example, suppose you had a declaration like this,

> 类型映射不能用于定义整体上适用于 C/C++ 声明的属性。例如，假设你有一个这样的声明，

```c++
Foo *make_Foo(int n);
```

and you wanted to tell SWIG that `make_Foo(int n)` returned a newly allocated object (for the purposes of providing better memory management). Clearly, this property of `make_Foo(int n)` is *not* a property that would be associated with the datatype `Foo *` by itself. Therefore, a completely different SWIG customization mechanism (`%feature`) is used for this purpose. Consult the [Customization Features](http://swig.org/Doc3.0/Customization.html#Customization) chapter for more information about that.

Typemaps also can't be used to rearrange or transform the order of arguments. For example, if you had a function like this:

> 并且你想告诉 SWIG `make_Foo(int n)` 返回了一个新分配的对象（目的是提供更好的内存管理）。显然，`make_Foo(int n)` 的此属性*不是*本身将与数据类型 `Foo *` 相关联的属性。因此，为此目的要使用完全不同的 SWIG 定制机制（`%feature`）。有关更多信息，请参考[自定义功能](http://swig.org/Doc3.0/Customization.html#Customization)章节。
>
> 类型映射也不能用于重新排列或转换参数的顺序。例如，如果你具有如下函数：

```c
void foo(int, char *);
```

you can't use typemaps to interchange the arguments, allowing you to call the function like this:

> 你不能使用类型映射来交换参数，允许你能这样调用函数：

```python
foo("hello", 3)          # Reversed arguments
```

If you want to change the calling conventions of a function, write a helper function instead. For example:

> 如果要更改函数的调用约定，请编写辅助函数。例如：

```
%rename(foo) wrap_foo;
%inline %{
void wrap_foo(char *s, int x) {
  foo(x, s);
}
%}
```

### 11.1.7 与面向切面编程的相似之处

SWIG has parallels to [Aspect Oriented Software Development (AOP)](http://en.wikipedia.org/wiki/Aspect-oriented_programming). The [AOP terminology](http://en.wikipedia.org/wiki/Aspect-oriented_programming#Terminology) with respect to SWIG typemaps can be viewed as follows:

* **Cross-cutting concerns**: The cross-cutting concerns are the modularization of the functionality that the typemaps implement, which is primarily marshalling of types from/to the target language and C/C++.
* **Advice**: The typemap body contains code which is executed whenever the marshalling is required.
* **Pointcut**: The pointcuts are the positions in the wrapper code that the typemap code is generated into.
* **Aspect**: Aspects are the combination of the pointcut and the advice, hence each typemap is an aspect.

SWIG can also be viewed as has having a second set of aspects based around [`%feature`](http://swig.org/Doc3.0/Customization.html#Customization). Features such as `%exception` are also cross-cutting concerns as they encapsulate code that can be used to add logging or exception handling to any function.

> SWIG 与[面向切面的软件开发（AOP）](http://en.wikipedia.org/wiki/Aspect-oriented_programming)相似。与 SWIG 类型映射有关的 [AOP 术语](http://en.wikipedia.org/wiki/Aspect-oriented_programming#Terminology)如下：
>
> * **横切关注点**：横切关注点是类型映射实现的功能的模块化，主要是将目标语言和 C/C++ 之间的类型进行编组。
> * **通知**：类型映射主体包含在需要编组时执行的代码。
> * **切入点**：切入点是包装器代码中生成类型映射代码的位置。
> * **切面**：切面是切入点和通知的组合，因此每个类型映射都是一个切面。
>
> 也可以将 SWIG 视为具有基于 [`%feature`](http://swig.org/Doc3.0/Customization.html#Customization) 的第二组切面。诸如 `%exception` 之类的功能也是横切关注点，因为它们封装了可用于向任何函数添加日志记录或异常处理的代码。

### 11.1.8 本章的剩余部分

The rest of this chapter provides detailed information for people who want to write new typemaps. This information is of particular importance to anyone who intends to write a new SWIG target language module. Power users can also use this information to write application specific type conversion rules.

Since typemaps are strongly tied to the underlying C++ type system, subsequent sections assume that you are reasonably familiar with the basic details of values, pointers, references, arrays, type qualifiers (e.g., `const`), structures, namespaces, templates, and memory management in C/C++. If not, you would be well-advised to consult a copy of "The C Programming Language" by Kernighan and Ritchie or "The C++ Programming Language" by Stroustrup before going any further.

> 本章的剩余部分为想要编写新的类型映射的人提供了详细的信息。对于打算为 SWIG 编写新目标语言模块的人来说，这些信息都特别重要。高级用户还可以使用这些信息来编写应用程序特定的类型转换规则。
>
> 由于类型映射与底层 C++ 类型系统紧密相关，因此后续章节假定你对值、指针、引用、数组、类型限定符（例如 `const`）、结构体、命名空间、模板和 C/C++ 中的内存管理相当熟悉。如果不是这样，建议你先阅读 Kernighan 和 Ritchie 撰写的《The C Programming Language》或 Stroustrup 撰写的《The C++ Programming Language》。

## 11.2 类型映射详述

This section describes the behavior of the `%typemap` directive itself.

> 本节描述了 `%typemap` 指令本身的行为。

### 11.2.1 定义一个类型映射

New typemaps are defined using the `%typemap` declaration. The general form of this declaration is as follows (parts enclosed in `[...]` are optional):

新的类型映射使用 `%typemap` 声明定义。该声明的一般形式如下（`[...]` 中的部分是可选的）：

```
%typemap(method [, modifiers]) typelist code ;
```

*method* is a simply a name that specifies what kind of typemap is being defined. It is usually a name like `"in"`, `"out"`, or `"argout"`. The purpose of these methods is described later.

*modifiers* is an optional comma separated list of `name="value"` values. These are sometimes to attach extra information to a typemap and is often target-language dependent. They are also known as typemap attributes.

*typelist* is a list of the C++ type patterns that the typemap will match. The general form of this list is as follows:

> *method* 是一个简单的名称，用于指定要定义的类型映射。通常，它的名称类似于 `in`、`out` 或 `argout`。这些方法的目的将在后面说明。
>
> *modifiers* 是一个可选的逗号分隔列表，其中包含 `name="value"` 值。这些有时会在类型映射上附加额外的信息，并且通常取决于目标语言。它们也称为类型映射属性。
>
> *typelist* 是类型映射将匹配的 C++ 类型模式的列表。此列表的一般形式如下：

```
typelist    :  typepattern [, typepattern, typepattern, ... ] ;

typepattern :  type [ (parms) ]
            |  type name [ (parms) ]
            |  ( typelist ) [ (parms) ]
```

Each type pattern is either a simple type, a simple type and argument name, or a list of types in the case of multi-argument typemaps. In addition, each type pattern can be parameterized with a list of temporary variables (parms). The purpose of these variables will be explained shortly.

*code* specifies the code used in the typemap. Usually this is C/C++ code, but in the statically typed target languages, such as Java and C#, this can contain target language code for certain typemaps. It can take any one of the following forms:

> 每个类型模式可以是简单类型，简单类型和参数名称，或者在多参数类型映射的情况下是类型列表。此外，可以使用一系列临时变量（参数）对每个类型模式进行参数化。这些变量的目的将在稍后说明。
>
> *code* 指定类型映射中使用的代码。通常这是 C/C++ 代码，但是在静态类型的目标语言（例如 Java 和 C#）中，它可以包含某些类型映射的目标语言代码。可以采用以下任何一种形式：

```
code       : { ... }
           | " ... "
           | %{ ... %}
```

Note that the preprocessor will expand code within the `{}` delimiters, but not in the last two styles of delimiters, see [Preprocessor and Typemaps](http://swig.org/Doc3.0/Preprocessor.html#Preprocessor_delimiters). Here are some examples of valid typemap specifications:

> 请注意，预处理器将在 `{}` 分隔符内扩展代码，但不会在最后两种分隔符样式中扩展代码，请参阅[《预处理器与类型映射》](http://swig.org/Doc3.0/Preprocessor.html#Preprocessor_delimiters)。以下是有效的类型映射规范的一些示例：

```
/* Simple typemap declarations */
%typemap(in) int {
  $1 = PyInt_AsLong($input);
}
%typemap(in) int "$1 = PyInt_AsLong($input);";
%typemap(in) int %{
  $1 = PyInt_AsLong($input);
%}

/* Typemap with extra argument name */
%typemap(in) int nonnegative {
  ...
}

/* Multiple types in one typemap */
%typemap(in) int, short, long {
  $1 = SvIV($input);
}

/* Typemap with modifiers */
%typemap(in, doc="integer") int "$1 = scm_to_int($input);";

/* Typemap applied to patterns of multiple arguments */
%typemap(in) (char *str, int len),
             (char *buffer, int size)
{
  $1 = PyString_AsString($input);
  $2 = PyString_Size($input);
}

/* Typemap with extra pattern parameters */
%typemap(in, numinputs=0) int *output (int temp),
                          long *output (long temp)
{
  $1 = &temp;
}
```

Admittedly, it's not the most readable syntax at first glance. However, the purpose of the individual pieces will become clear.

> 乍一看，这并不是最易读的语法。但是，各个部分的目的将变得清楚。

### 11.2.2 类型映射作用范围

Once defined, a typemap remains in effect for all of the declarations that follow. A typemap may be redefined for different sections of an input file. For example:

> 定义后，类型映射对于随后的所有声明都有效。可以为输入文件的不同部分重新定义类型映射。例如：

```
// typemap1
%typemap(in) int {
...
}

int fact(int);                    // typemap1
int gcd(int x, int y);            // typemap1

// typemap2
%typemap(in) int {
...
}

int isprime(int);                 // typemap2
```

One exception to the typemap scoping rules pertains to the `%extend` declaration. `%extend` is used to attach new declarations to a class or structure definition. Because of this, all of the declarations in an `%extend` block are subject to the typemap rules that are in effect at the point where the class itself is defined. For example:

> 类型映射范围规则的一个例外与 `%extend` 声明有关。`%extend` 用于将新的声明附加到类或结构体定义上。因此，`%extend` 块中的所有声明都将受到类型映射规则的约束，该规则在定义类本身时生效。例如：

```
class Foo {
  ...
};

%typemap(in) int {
 ...
}

%extend Foo {
  int blah(int x);    // typemap has no effect.  Declaration is attached to Foo which
                      // appears before the %typemap declaration.
};
```

### 11.2.3 复制类型映射

A typemap is copied by using assignment. For example:

> 使用赋值复制类型映射。例如：

```
%typemap(in) Integer = int;
```

or this:

> 或者

```
%typemap(in) Integer, Number, int32_t = int;
```

Types are often managed by a collection of different typemaps. For example:

> 类型通常由不同类型映射的集合来管理。例如：

```
%typemap(in)     int { ... }
%typemap(out)    int { ... }
%typemap(varin)  int { ... }
%typemap(varout) int { ... }
```

To copy all of these typemaps to a new type, use `%apply`. For example:

> 要将所有这些类型映射复制到一个新的类型，请使用 `%apply`。例如：

```
%apply int { Integer };            // Copy all int typemaps to Integer
%apply int { Integer, Number };    // Copy all int typemaps to both Integer and Number
```

The patterns for `%apply` follow the same rules as for `%typemap`. For example:

> `%apply` 的模式遵循与 `%typemap` 相同的规则。例如：

```
%apply int *output { Integer *output };                    // Typemap with name
%apply (char *buf, int len) { (char *buffer, int size) };  // Multiple arguments
```

### 11.2.4 删除类型映射

A typemap can be deleted by simply defining no code. For example:

> 不需要定义代码即可删除类型映射。例如：

```
%typemap(in) int;               // Clears typemap for int
%typemap(in) int, long, short;  // Clears typemap for int, long, short
%typemap(in) int *output;
```

The `%clear` directive clears all typemaps for a given type. For example:

> `%clear` 指令清除给定类型的所有类型映射。例如：

```
%clear int;                     // Removes all types for int
%clear int *output, long *output;
```

**Note:** Since SWIG's default behavior is defined by typemaps, clearing a fundamental type like `int` will make that type unusable unless you also define a new set of typemaps immediately after the clear operation.

> **注意**：由于 SWIG 的默认行为是由类型映射定义的，因此除非清除操作之后立即定义了一组新的类型映射，否则清除基本类型（如 `int`）将使该类型不可用。

### 11.2.5 类型映射的位置

Typemap declarations can be declared in the global scope, within a C++ namespace, and within a C++ class. For example:

> 可以在全局范围、C++ 命名空间和 C++ 类中声明类型映射声明。例如：

```
%typemap(in) int {
  ...
}

namespace std {
  class string;
  %typemap(in) string {
    ...
  }
}

class Bar {
public:
  typedef const int & const_reference;
  %typemap(out) const_reference {
    ...
  }
};
```

When a typemap appears inside a namespace or class, it stays in effect until the end of the SWIG input (just like before). However, the typemap takes the local scope into account. Therefore, this code

> 当类型映射出现在命名空间或类中时，它一直有效直到 SWIG 输入文件的结束（就像之前一样）。但是，类型映射将局部范围考虑在内。因此，此代码

```
namespace std {
  class string;
  %typemap(in) string {
    ...
  }
}
```

is really defining a typemap for the type `std::string`. You could have code like this:

> 确实为 `std::string` 类型定义了一个类型映射。你可能会有这样的代码：

```
namespace std {
  class string;
  %typemap(in) string {          /* std::string */
    ...
  }
}

namespace Foo {
  class string;
  %typemap(in) string {          /* Foo::string */
    ...
  }
}
```

In this case, there are two completely distinct typemaps that apply to two completely different types (`std::string` and `Foo::string`).

It should be noted that for scoping to work, SWIG has to know that `string` is a typename defined within a particular namespace. In this example, this is done using the forward class declaration `class string`.

> 在这种情况下，有两个完全不同的类型映射适用于两个完全不同的类型（`std::string` 和 `Foo::string`）。
>
> 应当注意，为使作用域有效，SWIG 必须知道 `string` 是在特定名称空间内定义的类型名。在此示例中，这是使用正向类声明 `class string` 完成的。

## 11.3 模式匹配规则

The section describes the pattern matching rules by which C/C++ datatypes are associated with typemaps. The matching rules can be observed in practice by using the debugging options also described.

> 本节描述了模式匹配规则，通过这些规则，C/C++ 数据类型与类型映射相关联。实际中，可以通过使用调试选项来观察匹配规则。

### 11.3.1 基本匹配规则

Typemaps are matched using both a type and a name (typically the name of a argument). For a given `TYPE NAME` pair, the following rules are applied, in order, to find a match. The first typemap found is used.

* Typemaps that exactly match `TYPE` and `NAME`.
* Typemaps that exactly match `TYPE` only.
* If `TYPE` is a C++ template of type `T<TPARMS>`, where `TPARMS` are the template parameters, the type is stripped of the template parameters and the following checks are then made:
  * Typemaps that exactly match `T` and `NAME`.
  * Typemaps that exactly match `T` only.

If `TYPE` includes qualifiers (const, volatile, etc.), each qualifier is stripped one at a time to form a new stripped type and the matching rules above are repeated on the stripped type. The left-most qualifier is stripped first, resulting in the right-most (or top-level) qualifier being stripped last. For example `int const*const` is first stripped to `int *const` then `int *`.

If `TYPE` is an array. The following transformation is made:

* Replace all dimensions to `[ANY]` and look for a generic array typemap.

To illustrate, suppose that you had a function like this:

> 使用类型和名称（通常是参数名称）来匹配类型映射。对于给定的 `TYPE NAME` 配对，将应用以下规则来查找匹配项。使用找到的第一个类型映射。
>
> * 与 `TYPE` 和 `NAME` 完全匹配的类型映射。
> * 仅与 `TYPE` 完全匹配的类型映射。
> * 如果 `TYPE` 是 `T<TPARMS>` 类型的 C++ 模板，其中 `TPARMS` 是模板参数，则将类型的模板参数剥离，然后进行以下检查：
>   * 与 `T` 和 `NAME` 完全匹配的类型映射。
>   * 仅与 `T` 完全匹配的类型映射。
>
> 如果 `TYPE` 包含限定符（`const`、`volatile` 等），则每次剥离一个限定符以形成新的剥离类型，并在剥离类型上重复上述匹配规则。最左边的限定符首先被剥离，最右边的（或顶级）限定符最后被剥离。例如，首先将 `int const * const` 剥离为 `int * const`，然后剥离为 `int *`。
>
> 如果 `TYPE` 是一个数组。进行以下转换：
>
> * 将所有尺寸替换为 `[ANY]`，并查找通用数组类型映射
>
> 为了说明这一点，假设你具有如下函数：

```c
int foo(const char *s);
```

To find a typemap for the argument `const char *s`, SWIG will search for the following typemaps:

> 要为参数 `const char *s` 查找类型映射，SWIG 将搜索以下类型映射：

```
const char *s           Exact type and name match
const char *            Exact type match
char *s                 Type and name match (qualifier stripped)
char *                  Type match (qualifier stripped)
```

When more than one typemap rule might be defined, only the first match found is actually used. Here is an example that shows how some of the basic rules are applied:

> 当可能定义多个类型映射规则时，实际上仅使用找到的第一个匹配项。下面是一个示例，显示了如何应用一些基本规则：

```
%typemap(in) int *x {
  ... typemap 1
}

%typemap(in) int * {
  ... typemap 2
}

%typemap(in) const int *z {
  ... typemap 3
}

%typemap(in) int [4] {
  ... typemap 4
}

%typemap(in) int [ANY] {
  ... typemap 5
}

void A(int *x);        // int *x rule       (typemap 1)
void B(int *y);        // int * rule        (typemap 2)
void C(const int *x);  // int *x rule       (typemap 1)
void D(const int *z);  // const int *z rule (typemap 3)
void E(int x[4]);      // int [4] rule      (typemap 4)
void F(int x[1000]);   // int [ANY] rule    (typemap 5)
```

**Compatibility note:** SWIG-2.0.0 introduced stripping the qualifiers one step at a time. Prior versions stripped all qualifiers in one step.

> **注意兼容性**：SWIG-2.0.0 引入了一次删除一个限定符。先前的版本一次就消除了所有限定符。

### 11.3.2 `typedef` 还原匹配

If no match is found using the rules in the previous section, SWIG applies a typedef reduction to the type and repeats the typemap search for the reduced type. To illustrate, suppose you had code like this:

> 如果使用上一节中的规则未找到匹配项，则 SWIG 将 `typedef` 还原，然后对还原后的类型重复进行类型映射搜索。为了说明这一点，假设你有如下代码：

```
%typemap(in) int {
  ... typemap 1
}

typedef int Integer;
void blah(Integer x);
```

To find the typemap for `Integer x`, SWIG will first search for the following typemaps:

> 为了找到 `Integer x` 的类型映射，SWIG 将首先搜索以下类型映射：

```
Integer x
Integer
```

Finding no match, it then applies a reduction `Integer -> int` to the type and repeats the search.

> 如果找不到匹配项，则对类型应用还原 `Integer -> int`，并重复搜索。

```
int x
int      --> match: typemap 1
```

Even though two types might be the same via typedef, SWIG allows typemaps to be defined for each typename independently. This allows for interesting customization possibilities based solely on the typename itself. For example, you could write code like this:

> 即使两个类型通过 `typedef` 可能是相同的，SWIG 仍允许为每个类型名分别定义类型映射。这允许仅基于类型名称本身进行有趣的自定义。例如，你可以编写如下代码：

```
typedef double  pdouble;     // Positive double

// typemap 1
%typemap(in) double {
  ... get a double ...
}
// typemap 2
%typemap(in) pdouble {
  ... get a positive double ...
}
double sin(double x);           // typemap 1
pdouble sqrt(pdouble x);        // typemap 2
```

When reducing the type, only one typedef reduction is applied at a time. The search process continues to apply reductions until a match is found or until no more reductions can be made.

For complicated types, the reduction process can generate a long list of patterns. Consider the following:

> 还原类型时，一次仅还原一次 `typedef`。搜索过程将继续应用还原直到找到匹配项，或无法再进行还原。
>
> 对于复杂类型，还原过程可以生成一长串模式。考虑以下：

```c
typedef int Integer;
typedef Integer Row4[4];
void foo(Row4 rows[10]);
```

To find a match for the `Row4 rows[10]` argument, SWIG would check the following patterns, stopping only when it found a match:

> 为了找到 `Row4 rows [10]` 参数的匹配项，SWIG 将检查以下模式，仅在找到匹配项时停止：

```
Row4 rows[10]
Row4 [10]
Row4 rows[ANY]
Row4 [ANY]

# Reduce Row4 --> Integer[4]
Integer rows[10][4]
Integer [10][4]
Integer rows[ANY][ANY]
Integer [ANY][ANY]

# Reduce Integer --> int
int rows[10][4]
int [10][4]
int rows[ANY][ANY]
int [ANY][ANY]
```

For parameterized types like templates, the situation is even more complicated. Suppose you had some declarations like this:

> 对于像模板这样的参数化类型，情况甚至更加复杂。假设你有一些这样的声明：

```c++
typedef int Integer;
typedef foo<Integer, Integer> fooii;
void blah(fooii *x);
```

In this case, the following typemap patterns are searched for the argument `fooii *x`:

> 在这种情况下，将在以下类型映射模式中搜索参数 `fooii *x`：

```
fooii *x
fooii *

# Reduce fooii --> foo<Integer, Integer>
foo<Integer, Integer> *x
foo<Integer, Integer> *

# Reduce Integer -> int
foo<int, Integer> *x
foo<int, Integer> *

# Reduce Integer -> int
foo<int, int> *x
foo<int, int> *
```

Typemap reductions are always applied to the left-most type that appears. Only when no reductions can be made to the left-most type are reductions made to other parts of the type. This behavior means that you could define a typemap for `foo<int, Integer>`, but a typemap for `foo<Integer, int>` would never be matched. Admittedly, this is rather esoteric--there's little practical reason to write a typemap quite like that. Of course, you could rely on this to confuse your coworkers even more.

As a point of clarification, it is worth emphasizing that typedef matching is a typedef **reduction** process only, that is, SWIG does not search for every single possible typedef. Given a type in a declaration, it will only reduce the type, it won't build it up looking for typedefs. For example, given the type `Struct`, the typemap below will not be used for the `aStruct` parameter, because `Struct` is fully reduced:

> 还原类型映射始终应用于出现在最左侧的类型。仅当无法对最左边的类型进行还原时，才对类型的其他部分进行还原。这种行为意味着你可以为 `foo<int, Integer>` 定义一个类型映射，但是 `foo<Integer, int>` 的类型映射不会被匹配。诚然，这是相当不常见的——几乎没有实际的理由来编写类似的类型映射。当然，你可以用它使你的同事更加困惑。
>
> 需要澄清的一点是，值得强调的是 `typedef` 匹配仅是 `typedef` 的“还原”过程，也就是说，SWIG 不会搜索每个可能的 `typedef`。给定声明中的类型，它只会还原类型，而不会在寻找 `typedef` 时建立它。例如，给定类型为 `Struct`，由于 `Struct` 已被完全还原，因此以下类型映射将不会用于 `aStruct` 参数：

```
struct Struct {...};
typedef Struct StructTypedef;

%typemap(in) StructTypedef {
  ...
}

void go(Struct aStruct);
```

### 11.3.3 默认类型映射匹配规则

If the basic pattern matching rules result in no match being made, even after typedef reductions, the default typemap matching rules are used to look for a suitable typemap match. These rules match a generic typemap based on the reserved `SWIGTYPE` base type. For example pointers will use `SWIGTYPE *`and references will use `SWIGTYPE &`. More precisely, the rules are based on the C++ class template partial specialization matching rules used by C++ compilers when looking for an appropriate partial template specialization. This means that a match is chosen from the most specialized set of generic typemap types available. For example, when looking for a match to `int const *`, the rules will prefer to match `SWIGTYPE const *` if available before matching `SWIGTYPE *`, before matching `SWIGTYPE`.

Most SWIG language modules use typemaps to define the default behavior of the C primitive types. This is entirely straightforward. For example, a set of typemaps for primitives marshalled by value or const reference are written like this:

> 如果即使在还原 `typedef` 之后基本模式匹配规则最终没有匹配，将使用默认的类型映射匹配规则来寻找合适的匹配。这些规则匹配基于保留的 `SWIGTYPE` 基本类型的通用类型映射。例如，指针将使用 `SWIGTYPE *`，而引用将使用 `SWIGTYPE &`。更准确地说，这些规则基于 C++ 类模板偏特化匹配规则，这些匹配规则由 C++ 编译器在寻找合适的模板偏特化时使用。这意味着从可用的最特定的通用类型映射类型集合中选择一个匹配项。例如，当寻找与 `int const *` 的匹配项时，规则将优先匹配 `SWIGTYPE const *`（如果有的话），然后再匹配 `SWIGTYPE *`，再匹配 `SWIGTYPE`。
>
> 大多数 SWIG 语言模块都使用类型映射来定义 C 基本类型的默认行为。这是非常简单的。例如，按值或常引用编组的原始类型的一组类型映射如下所示：

```
%typemap(in) int           "... convert to int ...";
%typemap(in) short         "... convert to short ...";
%typemap(in) float         "... convert to float ...";
...
%typemap(in) const int &   "... convert ...";
%typemap(in) const short & "... convert ...";
%typemap(in) const float & "... convert ...";
...
```

Since typemap matching follows all `typedef` declarations, any sort of type that is mapped to a primitive type by value or const reference through `typedef` will be picked up by one of these primitive typemaps. Most language modules also define typemaps for char pointers and char arrays to handle strings, so these non-default types will also be used in preference as the basic typemap matching rules provide a better match than the default typemap matching rules.

Below is a list of the typical default types supplied by language modules, showing what the "in" typemap would look like:

> 由于类型映射匹配遵循所有的 `typedef` 声明，因此通过 `typedef` 通过值或常引用映射到原始类型的任何类型的类型都将被这些原始类型映射之一所拾取。大多数语言模块还为 `char` 指针和 `char` 数组定义了类型映射以处理字符串，因此这些非默认类型也将优先使用，因为基本的类型映射匹配规则比默认的类型映射匹配规则提供了更好的匹配。
>
> 下面是语言模块提供的典型默认类型的列表，显示了 `in` 类型映射的样子：

```
%typemap(in) SWIGTYPE &            { ... default reference handling ...                       };
%typemap(in) SWIGTYPE *            { ... default pointer handling ...                         };
%typemap(in) SWIGTYPE *const       { ... default pointer const handling ...                   };
%typemap(in) SWIGTYPE *const&      { ... default pointer const reference handling ...         };
%typemap(in) SWIGTYPE[ANY]         { ... 1D fixed size arrays handlling ...                   };
%typemap(in) SWIGTYPE []           { ... unknown sized array handling ...                     };
%typemap(in) enum SWIGTYPE         { ... default handling for enum values ...                 };
%typemap(in) const enum SWIGTYPE & { ... default handling for const enum reference values ... };
%typemap(in) SWIGTYPE (CLASS::*)   { ... default pointer member handling ...                  };
%typemap(in) SWIGTYPE              { ... simple default handling ...                          };
```

If you wanted to change SWIG's default handling for simple pointers, you would simply redefine the rule for `SWIGTYPE *`. Note, the simple default typemap rule is used to match against simple types that don't match any other rules:

> 如果你想更改 SWIG 对简单指针的默认处理，只需简单地为 `SWIGTYPE *` 重新定义规则。请注意，简单的默认类型映射规则用于与不匹配任何其他规则的简单类型进行匹配：

```
%typemap(in) SWIGTYPE              { ... simple default handling ...                          }
```

This typemap is important because it is the rule that gets triggered when call or return by value is used. For instance, if you have a declaration like this:

> 此类型映射很重要，因为使用调用或按值返回时会触发该规则。例如，如果你有这样的声明：

```c
double dot_product(Vector a, Vector b);
```

The `Vector` type will usually just get matched against `SWIGTYPE`. The default implementation of `SWIGTYPE`is to convert the value into pointers ([as described in this earlier section](http://swig.org/Doc3.0/SWIG.html#SWIG_nn22)).

By redefining `SWIGTYPE` it may be possible to implement other behavior. For example, if you cleared all typemaps for `SWIGTYPE`, SWIG simply won't wrap any unknown datatype (which might be useful for debugging). Alternatively, you might modify SWIGTYPE to marshal objects into strings instead of converting them to pointers.

Let's consider an example where the following typemaps are defined and SWIG is looking for the best match for the enum shown below:

> `Vector` 类型通常只会与 `SWIGTYPE` 相匹配。`SWIGTYPE` 的默认实现是将值转换为指针（[如本之前的章节所述](http://swig.org/Doc3.0/SWIG.html#SWIG_nn22)）。
>
> 通过重新定义 `SWIGTYPE`，可以实现其他行为。例如，如果你清除了 `SWIGTYPE` 的所有类型映射，则 SWIG 不会包装任何未知的数据类型（这可能对调试很有用）。或者，你可以修改 `SWIGTYPE` 以将对象编组为字符串，而不是将它们转换为指针。
>
> 让我们考虑一个示例，其中定义了以下类型映射，并且 SWIG 正在为以下所示的枚举寻找最佳匹配：

```
%typemap(in) const Hello &          { ... }
%typemap(in) const enum SWIGTYPE &  { ... }
%typemap(in) enum SWIGTYPE &        { ... }
%typemap(in) SWIGTYPE &             { ... }
%typemap(in) SWIGTYPE               { ... }

enum Hello {};
const Hello &hi;
```

The typemap at the top of the list will be chosen, not because it is defined first, but because it is the closest match for the type being wrapped. If any of the typemaps in the above list were not defined, then the next one on the list would have precedence.

The best way to explore the default typemaps is to look at the ones already defined for a particular language module. Typemap definitions are usually found in the SWIG library in a file such as `java.swg`, `csharp.swg` etc. However, for many of the target languages the typemaps are hidden behind complicated macros, so the best way to view the default typemaps, or any typemaps for that matter, is to look at the preprocessed output by running `swig -E` on any interface file. Finally the best way to view the typemap matching rules in action is via the [debugging typemap pattern matching](http://swig.org/Doc3.0/Typemaps.html#Typemaps_debugging_search) options covered later on.

**Compatibility note:** The default typemap matching rules were modified in SWIG-2.0.0 from a slightly simpler scheme to match the current C++ class template partial specialization matching rules.

> 将选择列表顶部的类型映射，不仅是因为首先定义了它，而且是因为它与被包装的类型最匹配。如果上面列表中的任何类型映射未定义，则列表中的下一个优先。
>
> 探索默认类型映射的最佳方法是查看已为特定语言模块定义的映射。类型映射定义通常可以在 SWIG 库的 java.swg、csharp.swg 等文件中找到。但是，对于许多目标语言而言，类型映射都隐藏在复杂的宏后面，因此，查看默认类型映射或任何与此相关的类型映射的最佳方法是在任何接口文件上运行 `swig -E` 来查看预处理后的输出。最后，查看正在使用的类型映射匹配规则的最佳方法是通过稍后介绍的[调试类型映射匹配模式](http://swig.org/Doc3.0/Typemaps.html#Typemaps_debugging_search) 选项。
>
> **注意兼容性**：默认的类型映射匹配规则是在 SWIG-2.0.0 中从稍微简单的方案中修改的，以匹配当前的 C++ 类模板偏特化匹配规则。

### 11.3.4 多参数类型映射

When multi-argument typemaps are specified, they take precedence over any typemaps specified for a single type. For example:

> 指定多参数类型映射时，它们优先于为单个类型指定的任何类型映射。例如：

```
%typemap(in) (char *buffer, int len) {
  // typemap 1
}

%typemap(in) char *buffer {
  // typemap 2
}

void foo(char *buffer, int len, int count); // (char *buffer, int len)
void bar(char *buffer, int blah);           // char *buffer
```

Multi-argument typemaps are also more restrictive in the way that they are matched. Currently, the first argument follows the matching rules described in the previous section, but all subsequent arguments must match exactly.

> 多参数类型映射在匹配方式上也有更多限制。当前，第一个参数遵循上一节中描述的匹配规则，但是所有后续参数必须完全匹配。

### 11.3.5 匹配规则对比 C++ 模板

For those intimately familiar with C++ templates, a comparison of the typemap matching rules and template type deduction is interesting. The two areas considered are firstly the default typemaps and their similarities to partial template specialization and secondly, non-default typemaps and their similarities to full template specialization.

For default (SWIGTYPE) typemaps the rules are inspired by C++ class template partial specialization. For example, given partial specialization for `T const&` :

> 对于那些熟悉 C++ 模板的人来说，比较类型映射匹配规则和模板类型推导是很有趣的。首先考虑的两个方面是默认类型映射及其与模板偏特化的相似性，其次是非默认类型映射及其与模板完全化的相似性。
>
> 对于默认（`SWIGTYPE`）类型映射，规则受 C++ 类模板偏特化的启发。例如，给定 `T const&` 的偏特化：

```c++
template <typename T> struct X             { void a(); };
template <typename T> struct X< T const& > { void b(); };
```

The full (unspecialized) template is matched with most types, such as:

> 完全（非偏）模板与大多数类型匹配，例如：

```c++
X< int & >            x1;  x1.a();
```

and the following all match the `T const&` partial specialization:

> 以及以下所有匹配 `T const&` 的偏特化的代码：

```c++
X< int *const& >      x2;  x2.b();
X< int const*const& > x3;  x3.b();
X< int const& >       x4;  x4.b();
```

Now, given just these two default typemaps, where T is analogous to SWIGTYPE:

> 现在，仅给出这两个默认类型映射，其中 `T` 类似于 `SWIGTYPE`：

```
%typemap(...) SWIGTYPE        { ... }
%typemap(...) SWIGTYPE const& { ... }
```

The generic default typemap `SWIGTYPE` is used with most types, such as

> 通用默认类型映射 `SWIGTYPE` 用于大多数类型，例如

```c++
int &
```

and the following all match the `SWIGTYPE const&` typemap, just like the partial template matching:

> 并且以下所有内容都匹配 `SWIGTYPE const&` 类型映射，就像部分模板匹配一样：

```c++
int *const&
int const*const&
int const&
```

Note that the template and typemap matching rules are not identical for all default typemaps though, for example, with arrays.

For non-default typemaps, one might expect SWIG to follow the fully specialized template rules. This is nearly the case, but not quite. Consider a very similar example to the earlier partially specialized template but this time there is a fully specialized template:

> 请注意，模板和类型映射匹配规则对于所有默认类型映射都不相同，例如，对于数组。
>
> 对于非默认类型映射，可能希望 SWIG 遵循完全特化的模板规则。这几乎是事实，但事实并非如此。考虑一个与早期的偏特化模板非常相似的示例，但是这次有一个完全专用的模板：

```c++
template <typename T> struct Y       { void a(); };
template <> struct Y< int const & >  { void b(); };
```

Only the one type matches the specialized template exactly:

> 只有一种类型与专用模板完全匹配：

```c++
Y< int & >             y1;  y1.a();
Y< int *const& >       y2;  y2.a();
Y< int const *const& > y3;  y3.a();
Y< int const& >        y4;  y4.b(); // fully specialized match
```

Given typemaps with the same types used for the template declared above, where T is again analogous to SWIGTYPE:

> 给定具有与上面声明的模板相同类型的类型映射，其中 `T` 再次类似于 `SWIGTYPE`：

```
%typemap(...) SWIGTYPE        { ... }
%typemap(...) int const&      { ... }
```

The comparison between non-default typemaps and fully specialized single parameter templates turns out to be the same, as just the one type will match the non-default typemap:

> 事实证明，非默认类型映射和完全特化的单参数模板之间的比较是相同的，因为只有一种类型会匹配非默认类型映射：

```c++
int &
int *const&
int const*const&
int const&        // matches non-default typemap int const&
```

However, if a non-const type is used instead:

> 但是，如果改用非常量类型：

```
%typemap(...) SWIGTYPE        { ... }
%typemap(...) int &           { ... }
```

then there is a clear difference to template matching as both the const and non-const types match the typemap:

> 那么模板匹配有明显的区别，因为 `const` 和非 `const` 类型都与类型映射匹配：

```c++
int &             // matches non-default typemap int &
int *const&
int const*const&
int const&        // matches non-default typemap int &
```

There are other subtle differences such as typedef handling, but at least it should be clear that the typemap matching rules are similar to those for specialized template handling.

> 还有其他一些细微的差异，例如 `typedef` 处理，但至少应该清楚的是，类型映射匹配规则类似于专门模板处理的规则。

### 11.3.6 调试类型映射模式匹配

There are two useful debug command line options available for debugging typemaps, `-debug-tmsearch`and `-debug-tmused`.

The `-debug-tmsearch` option is a verbose option for debugging typemap searches. This can be very useful for watching the pattern matching process in action and for debugging which typemaps are used. The option displays all the typemaps and types that are looked for until a successful pattern match is made. As the display includes searches for each and every type needed for wrapping, the amount of information displayed can be large. Normally you would manually search through the displayed information for the particular type that you are interested in.

For example, consider some of the code used in the [Typedef reductions](http://swig.org/Doc3.0/Typemaps.html#Typemaps_typedef_reductions) section already covered:

> 有两个有用的调试命令行选项可用于调试类型映射：`-debug-tmsearch` 和 `-debug-tmused`。
>
> `-debug-tmsearch` 选项是用于调试类型映射搜索的详细选项。这对于观察实际的模式匹配过程以及调试使用哪种类型映射非常有用。该选项显示在成功进行模式匹配之前要查找的所有类型映射和类型。由于显示内容包括对包装所需的每种类型的搜索，因此显示的信息量可能很大。通常，你将在显示的信息中手动搜索感兴趣的特定类型。
>
> 例如，考虑已经讨论过的[还原 `typedef`](http://swig.org/Doc3.0/Typemaps.html#Typemaps_typedef_reductions)章节中使用的一些代码：

```c++
typedef int Integer;
typedef Integer Row4[4];
void foo(Row4 rows[10]);
```

A sample of the debugging output is shown below for the "in" typemap:

> 下面显示了 `in` 类型映射的调试输出示例：

```
swig -perl -debug-tmsearch example.i
...
example.h:3: Searching for a suitable 'in' typemap for: Row4 rows[10]
  Looking for: Row4 rows[10]
  Looking for: Row4 [10]
  Looking for: Row4 rows[ANY]
  Looking for: Row4 [ANY]
  Looking for: Integer rows[10][4]
  Looking for: Integer [10][4]
  Looking for: Integer rows[ANY][ANY]
  Looking for: Integer [ANY][ANY]
  Looking for: int rows[10][4]
  Looking for: int [10][4]
  Looking for: int rows[ANY][ANY]
  Looking for: int [ANY][ANY]
  Looking for: SWIGTYPE rows[ANY][ANY]
  Looking for: SWIGTYPE [ANY][ANY]
  Looking for: SWIGTYPE rows[ANY][]
  Looking for: SWIGTYPE [ANY][]
  Looking for: SWIGTYPE *rows[ANY]
  Looking for: SWIGTYPE *[ANY]
  Looking for: SWIGTYPE rows[ANY]
  Looking for: SWIGTYPE [ANY]
  Looking for: SWIGTYPE rows[]
  Looking for: SWIGTYPE []
  Using: %typemap(in) SWIGTYPE []
...
```

showing that the best default match supplied by SWIG is the `SWIGTYPE []` typemap. As the example shows, the successful match displays the used typemap source including typemap method, type and optional name in one of these simplified formats:

* `Using: %typemap(method) type name`
* `Using: %typemap(method) type name = type2 name2`
* `Using: %apply type2 name2 { type name }`

This information might meet your debugging needs, however, you might want to analyze further. If you next invoke SWIG with the `-E` option to display the preprocessed output, and search for the particular typemap used, you'll find the full typemap contents (example shown below for Python):

> 表明 SWIG 提供的最佳默认匹配项是 `SWIGTYPE []` 类型映射。如示例所示，成功匹配以下列简化格式之一显示使用的类型映射，包括类型映射方法，类型和可选名称：
>
> * `Using: %typemap(method) type name`
> * `Using: %typemap(method) type name = type2 name2`
> * `Using: %apply type2 name2 { type name }`

> 此信息可能满足你的调试需求，但是，你可能需要进一步分析。如果接下来使用 `-E` 选项调用 SWIG 以显示预处理后的输出，并搜索所使用的特定类型映射，则将找到完整的类型映射内容（以下示例显示在 Python 中）：

```
%typemap(in, noblock=1) SWIGTYPE [] (void *argp = 0, int res = 0) {
  res = SWIG_ConvertPtr($input, &argp, $descriptor, $disown |  0 );
  if (!SWIG_IsOK(res)) {
    SWIG_exception_fail(SWIG_ArgError(res), "in method '" "$symname" "', argument "
                       "$argnum"" of type '" "$type""'");
  }
  $1 = ($ltype)(argp);
}
```

The generated code for the `foo` wrapper will then contain the snippets of the typemap with the special variables expanded. The rest of this chapter will need reading though to fully understand all of this, however, the relevant parts of the generated code for the above typemap can be seen below:

> 然后，为 `foo` 包装程序生成的代码将包含带有特殊变量扩展的类型映射的代码段。本章的其余部分虽然需要阅读才能完全理解所有这些内容，但是，可以在下面看到上述类型映射的生成代码的相关部分：

```c++
SWIGINTERN PyObject *_wrap_foo(PyObject *SWIGUNUSEDPARM(self), PyObject *args) {
...
  void *argp1 = 0 ;
  int res1 = 0 ;
...
  res1 = SWIG_ConvertPtr(obj0, &argp1, SWIGTYPE_p_a_4__int, 0 |  0 );
  if (!SWIG_IsOK(res1)) {
    SWIG_exception_fail(SWIG_ArgError(res1), "in method '" "foo" "', argument "
                       "1"" of type '" "int [10][4]""'");
  }
  arg1 = (int (*)[4])(argp1);
...
}
```

Searches for multi-argument typemaps are not mentioned unless a matching multi-argument typemap does actually exist. For example, the output for the code in the [earlier multi-arguments section](http://swig.org/Doc3.0/Typemaps.html#Typemaps_multi_argument_typemaps_patterns) is as follows:

> 除非确实存在匹配的多参数类型映射，否则不涉及搜索多参数类型映射。例如，[较早的多参数章节](http://swig.org/Doc3.0/Typemaps.html#Typemaps_multi_argument_typemaps_patterns)中的代码输出如下：

```
...
example.h:39: Searching for a suitable 'in' typemap for: char *buffer
  Looking for: char *buffer
  Multi-argument typemap found...
  Using: %typemap(in) (char *buffer, int len)
...
```

The second option for debugging is `-debug-tmused` and this displays the typemaps used. This option is a less verbose version of the `-debug-tmsearch` option as it only displays each successfully found typemap on a separate single line. The output displays the type, and name if present, the typemap method in brackets and then the actual typemap used in the same simplified format output by the `-debug-tmsearch`option. Below is the output for the example code at the start of this section on debugging.

> 调试的第二个选项是 `-debug-tmused`，它显示了使用的类型映射。这个选项是 `-debug-tmsearch` 选项的一个不太冗长的版本，因为它只在单独的一行上显示每个成功找到的类型映射。输出将显示类型和名称（如果存在），括号中的类型映射方法，然后显示由 `-debug-tmsearch` 选项以相同简化格式输出的实际类型映射。以下是本节开始时有关调试的示例代码的输出。

```
$ swig -perl -debug-tmused example.i
example.h:3: Typemap for Row4 rows[10] (in) : %typemap(in) SWIGTYPE []
example.h:3: Typemap for Row4 rows[10] (typecheck) : %typemap(typecheck) SWIGTYPE *
example.h:3: Typemap for Row4 rows[10] (freearg) : %typemap(freearg) SWIGTYPE []
example.h:3: Typemap for void foo (out) : %typemap(out) void
```

Now, consider the following interface file:

> 现在，考虑下面的接口文件：

```
%module example

%{
void set_value(const char* val) {}
%}

%typemap(check) char *NON_NULL {
  if (!$1) {
    /* ... error handling ... */
  }
}

// use default pointer handling instead of strings
%apply SWIGTYPE * { const char* val, const char* another_value }

%typemap(check) const char* val = char* NON_NULL;

%typemap(arginit, noblock=1) const char* val {
  $1 = "";
}

void set_value(const char* val);
```

and the output debug:

> 输出调试结果：

```
swig -perl5 -debug-tmused example.i
example.i:21: Typemap for char const *val (arginit) : %typemap(arginit) char const *val
example.i:21: Typemap for char const *val (in) : %apply SWIGTYPE * { char const *val }
example.i:21: Typemap for char const *val (typecheck) : %apply SWIGTYPE * { char const *val }
example.i:21: Typemap for char const *val (check) : %typemap(check) char const *val = char *NON_NULL
example.i:21: Typemap for char const *val (freearg) : %apply SWIGTYPE * { char const *val }
example.i:21: Typemap for void set_value (out) : %typemap(out) void
```

The following observations about what is displayed can be noted (the same applies for `-debug-tmsearch`):

* The relevant typemap is shown, but for typemap copying, the appropriate `%typemap` or `%apply` is displayed, for example, the "check" and "in" typemaps.
* The typemap modifiers are not shown, eg the `noblock=1` modifier in the "arginit" typemap.
* The exact `%apply` statement might look different to what is in the actual code. For example, the `const char* another_value` is not shown as it is not relevant here. Also the types may be displayed slightly differently - `char const *` and not `const char*`.

> 可以注意到以下有关显示内容的观察结果（`-debug-tmsearch` 同样适用）：
>
> * 显示了相关的类型映射，但是对于复制类型映射，将显示适当的 `%typemap` 或 `%apply`，例如，`check` 和 `in` 类型映射。
> * 类型映射修饰符未显示，例如，`arginit` 类型映射中的 `noblock = 1` 修饰符。
> * 确切的 `%apply` 语句可能看起来与实际代码不同。例如，未显示 `const char * another_value`，因为此处不相关。同样，类型的显示可能略有不同——`char const *` 而不是 `const char *`。

## 11.4 代码生成规则

This section describes rules by which typemap code is inserted into the generated wrapper code.

> 本节描述将类型映射代码插入到生成的包装器代码中的规则。

### 11.4.1 作用域

When a typemap is defined like this:

> 当类型映射定义如下：

```
%typemap(in) int {
  $1 = PyInt_AsLong($input);
}
```

the typemap code is inserted into the wrapper function using a new block scope. In other words, the wrapper code will look like this:

> 使用新的块作用域将类型映射代码插入包装器函数。换句话说，包装器代码将如下所示：

```c++
wrap_whatever() {
  ...
  // Typemap code
  {
    arg1 = PyInt_AsLong(obj1);
  }
  ...
}
```

Because the typemap code is enclosed in its own block, it is legal to declare temporary variables for use during typemap execution. For example:

> 因为类型映射代码包含在其自己的块中，所以声明临时变量供在类型映射执行期间使用是合法的。例如：

```
%typemap(in) short {
  long temp;          /* Temporary value */
  if (Tcl_GetLongFromObj(interp, $input, &temp) != TCL_OK) {
    return TCL_ERROR;
  }
  $1 = (short) temp;
}
```

Of course, any variables that you declare inside a typemap are destroyed as soon as the typemap code has executed (they are not visible to other parts of the wrapper function or other typemaps that might use the same variable names).

Occasionally, typemap code will be specified using a few alternative forms. For example:

> 当然，你在类型映射中声明的任何变量都将在该类型映射代码执行后立即销毁（它们对于包装器函数的其他部分或其他可能使用相同变量名的类型映射不可见）。
>
> 有时，会使用一些其他形式来指定类型映射代码。例如：

```
%typemap(in) int "$1 = PyInt_AsLong($input);";
%typemap(in) int %{
$1 = PyInt_AsLong($input);
%}
%typemap(in, noblock=1) int {
$1 = PyInt_AsLong($input);
}
```

These three forms are mainly used for cosmetics--the specified code is not enclosed inside a block scope when it is emitted. This sometimes results in a less complicated looking wrapper function. Note that only the third of the three typemaps have the typemap code passed through the SWIG preprocessor.

> 这三种形式主要用于化妆——发出特定代码时，指定代码未包含在块作用域内。有时这会导致看起来不太复杂的包装器函数。请注意，三个类型映射中只有三分之一具有通过 SWIG 预处理程序传递的类型映射代码。

### 11.4.2 声明新的局部变量

Sometimes it is useful to declare a new local variable that exists within the scope of the entire wrapper function. A good example of this might be an application in which you wanted to marshal strings. Suppose you had a C++ function like this

> 有时，声明存在于整个包装器函数范围内的新局部变量很有用。一个很好的例子就是你想在其中封送字符串的应用程序。假设你有一个这样的 C++ 函数

```c++
int foo(std::string *s);
```

and you wanted to pass a native string in the target language as an argument. For instance, in Perl, you wanted the function to work like this:

> 并且你想传递目标语言中的本地字符串作为参数。例如，在 Perl 中，你希望函数像这样工作：

```perl
$x = foo("Hello World");
```

To do this, you can't just pass a raw Perl string as the `std::string *` argument. Instead, you have to create a temporary `std::string` object, copy the Perl string data into it, and then pass a pointer to the object. To do this, simply specify the typemap with an extra parameter like this:

> 为此，你不能仅将原始 Perl 字符串作为 `std::string *` 参数传递。相反，你必须创建一个临时的 `std::string` 对象，将 Perl 字符串数据复制到其中，然后将指针传递给该对象。为此，只需使用如下额外参数指定类型映射：

```
%typemap(in) std::string * (std::string temp) {
  unsigned int len;
  char        *s;
  s = SvPV($input, len);         /* Extract string data */
  temp.assign(s, len);           /* Assign to temp */
  $1 = &temp;                   /* Set argument to point to temp */
}
```

In this case, `temp` becomes a local variable in the scope of the entire wrapper function. For example:

> 在这种情况下，`temp` 成为整个包装器函数范围内的局部变量。例如：

```c++
wrap_foo() {
  std::string temp;   // <--- Declaration of temp goes here
  ...

  /* Typemap code */
  {
    ...
    temp.assign(s, len);
    ...
  }
  ...
}
```

When you set `temp` to a value, it persists for the duration of the wrapper function and gets cleaned up automatically on exit.

It is perfectly safe to use more than one typemap involving local variables in the same declaration. For example, you could declare a function as :

> 当你将 `temp` 设置为一个值时，它将在包装器函数的整个过程中持续存在，并在退出时自动清除。
>
> 在同一声明中使用多个涉及局部变量的类型映射是绝对安全的。例如，你可以将一个函数声明为：

```c++
void foo(std::string *x, std::string *y, std::string *z);
```

This is safely handled because SWIG actually renames all local variable references by appending an argument number suffix. Therefore, the generated code would actually look like this:

> 这是安全处理的，因为 SWIG 实际上通过附加参数编号后缀来重命名所有局部变量引用。因此，生成的代码实际上将如下所示：

```c++
wrap_foo() {
  int *arg1;    /* Actual arguments */
  int *arg2;
  int *arg3;
  std::string temp1;    /* Locals declared in the typemap */
  std::string temp2;
  std::string temp3;
  ...
  {
    char *s;
    unsigned int len;
    ...
    temp1.assign(s, len);
    arg1 = *temp1;
  }
  {
    char *s;
    unsigned int len;
    ...
    temp2.assign(s, len);
    arg2 = &temp2;
  }
  {
    char *s;
    unsigned int len;
    ...
    temp3.assign(s, len);
    arg3 = &temp3;
  }
  ...
}
```

There is an exception: if the variable name starts with the `_global_` prefix, the argument number is not appended. Such variables can be used throughout the generated wrapper function. For example, the above typemap could be rewritten to use `_global_temp` instead of `temp` and the generated code would then contain a single `_global_temp` variable instead of `temp1`, `temp2` and `temp3`:

> 有一个例外：如果变量名以 `_global_` 前缀开头，则不附加参数编号。此类变量可在整个生成的包装器函数中使用。例如，上面的类型映射可以重写为使用 `_global_temp` 而不是 `temp`，然后生成的代码将包含单个 `_global_temp` 变量而不是 `temp1`，`temp2` 和 `temp3`：

```
%typemap(in) std::string * (std::string _global_temp) {
 ... as above ...
}
```

Some typemaps do not recognize local variables (or they may simply not apply). At this time, only typemaps that apply to argument conversion support this (input typemaps such as the "in" typemap).

**Note:**

When declaring a typemap for multiple types, each type must have its own local variable declaration.

> 一些类型映射不能识别局部变量（或者它们可能根本不适用）。目前，仅适用于参数转换的类型映射支持此功能（输入类型映射，例如 `in` 类型映射）。
>
> **注意**：
>
> 当声明多个类型的类型映射时，每个类型必须具有自己的局部变量声明。

```
%typemap(in) const std::string *, std::string * (std::string temp) // NO!
// only std::string * has a local variable
// const std::string * does not (oops)
....

%typemap(in) const std::string * (std::string temp), std::string * (std::string temp) // Correct
....
```

### 11.4.3 特殊变量

Within all typemaps, the following special variables are expanded. This is by no means a complete list as some target languages have additional special variables which are documented in the language specific chapters.

> 下列特殊变量是对所有类型映射的扩展。这绝不是一个完整的列表，因为某些目标语言具有额外的特殊变量，这些特殊变量记录在目标语言的特定章节中。

| Variable         | Meaning                                                                                                                                                        |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$n`             | A C local variable corresponding to type *n* in the typemap pattern.                                                                                           |
| `$argnum`          | Argument number. Only available in typemaps related to argument conversion                                                                                     |
| `$n_name`        | Argument name                                                                                                                                                  |
| $*n*_type        | Real C datatype of type *n*.                                                                                                                                   |
| $*n*_ltype       | ltype of type *n*                                                                                                                                              |
| $*n*_mangle      | Mangled form of type *n*. For example `_p_Foo`                                                                                                                 |
| $*n*_descriptor  | Type descriptor structure for type *n*. For example`SWIGTYPE_p_Foo`. This is primarily used when interacting with the run-time type checker (described later). |
| $**n*_type       | Real C datatype of type *n* with one pointer removed.                                                                                                          |
| $**n*_ltype      | ltype of type *n* with one pointer removed.                                                                                                                    |
| $**n*_mangle     | Mangled form of type *n* with one pointer removed.                                                                                                             |
| $**n*_descriptor | Type descriptor structure for type *n* with one pointer removed.                                                                                               |
| $&*n*_type       | Real C datatype of type *n* with one pointer added.                                                                                                            |
| $&*n*_ltype      | ltype of type *n* with one pointer added.                                                                                                                      |
| $&*n*_mangle     | Mangled form of type *n* with one pointer added.                                                                                                               |
| $&*n*_descriptor | Type descriptor structure for type *n* with one pointer added.                                                                                                 |
| $*n*_basetype    | Base typename with all pointers and qualifiers stripped.                                                                                                       |

Within the table, `$n` refers to a specific type within the typemap specification. For example, if you write this

> 在表中，`$n` 表示类型映射规范中的特定类型。例如，如果你编写此

```
%typemap(in) int *INPUT {

}
```

then `$1` refers to `int *INPUT`. If you have a typemap like this,

> 那么 `$1` 指向 `int *INPUT`。如果你有如下类型映射，

```
%typemap(in) (int argc, char *argv[]) {
  ...
}
```

then `$1` refers to `int argc` and $2 refers to `char *argv[]`.

Substitutions related to types and names always fill in values from the actual code that was matched. This is useful when a typemap might match multiple C datatype. For example:

> 那么 `$1` 指向 `int argc`，`$2` 指向 `char *argv[]`。
>
> 与类型和名称相关的替换总是填充匹配的实际代码中的值。当一个类型映射可能匹配多个 C 数据类型时，这很有用。例如：

```
%typemap(in)  int, short, long {
  $1 = ($1_ltype) PyInt_AsLong($input);
}
```

In this case, `$1_ltype` is replaced with the datatype that is actually matched.

When typemap code is emitted, the C/C++ datatype of the special variables `$1` and `$2` is always an "ltype." An "ltype" is simply a type that can legally appear on the left-hand side of a C assignment operation. Here are a few examples of types and ltypes:

> 在这种情况下，将 `$1_ltype` 替换为实际匹配的数据类型。
>
> 当发出类型映射代码时，特殊变量 `$1` 和 `$2` 的 C/C++ 数据类型始终是 `ltype`。`ltype` 只是可以合法出现在 C 赋值操作左侧的类型。以下是一些类型和 `ltypes` 的示例：

```
type              ltype
------            ----------------
int               int
const int         int
const int *       int *
int [4]           int *
int [4][5]        int (*)[5]
```

In most cases a ltype is simply the C datatype with qualifiers stripped off. In addition, arrays are converted into pointers.

Variables such as `$&1_type` and `$*1_type` are used to safely modify the type by removing or adding pointers. Although not needed in most typemaps, these substitutions are sometimes needed to properly work with typemaps that convert values between pointers and values.

If necessary, type related substitutions can also be used when declaring locals. For example:

> 在大多数情况下，`ltype` 只是带有限定符的 C 数据类型。另外，数组被转换为指针。
>
> 诸如 `$&1_type` 和 `$*1_type` 之类的变量用于通过删除或添加指针来安全地修改类型。尽管在大多数类型映射中不需要，但是有时有时需要这些替换才能正确处理在指针和值之间转换值的类型映射。
>
> 如有必要，在声明本地变量时也可以使用类型相关的替换。例如：

```
%typemap(in) int * ($*1_type temp) {
  temp = PyInt_AsLong($input);
  $1 = &temp;
}
```

There is one word of caution about declaring local variables in this manner. If you declare a local variable using a type substitution such as `$1_ltype temp`, it won't work like you expect for arrays and certain kinds of pointers. For example, if you wrote this,

> 以这种方式声明局部变量有一个警告。如果你使用诸如 `$1_ltype temp` 之类的类型替换声明局部变量，它将无法像你期望的那样使用数组和某些类型的指针。例如，如果你编写了此代码，

```
%typemap(in) int [10][20] {
  $1_ltype temp;
}
```

then the declaration of `temp` will be expanded as

> 那么 `temp` 的声明将被扩展为

```c
int (*)[20] temp;
```

This is illegal C syntax and won't compile. There is currently no straightforward way to work around this problem in SWIG due to the way that typemap code is expanded and processed. However, one possible workaround is to simply pick an alternative type such as `void *` and use casts to get the correct type when needed. For example:

> 这是非法的 C 语法，不会编译。由于类型映射代码的扩展和处理方式，当前在 SWIG 中没有解决此问题的简单方法。然而，一种可能的解决方法是简单地选择一种替代类型，例如 `void *`，并在需要时使用强制类型转换来获取正确的类型。例如：

```
%typemap(in) int [10][20] {
  void *temp;
  ...
  (($1_ltype) temp)[i][j] = x;    /* set a value */
  ...
}
```

Another approach, which only works for arrays is to use the `$1_basetype` substitution. For example:

> 另一种只对数组有效的方法是使用 `$1_basetype` 替换。例如：

```
%typemap(in) int [10][20] {
  $1_basetype temp[10][20];
  ...
  temp[i][j] = x;    /* set a value */
  ...
}
```

### 11.4.4 特殊变量宏

Special variable macros are like macro functions in that they take one or more input arguments which are used for the macro expansion. They look like macro/function calls but use the special variable `$` prefix to the macro name. Note that unlike normal macros, the expansion is not done by the preprocessor, it is done during the SWIG parsing/compilation stages. The following special variable macros are available across all language modules.

> 特殊变量宏就像宏函数一样，它们采用一个或多个用于宏扩展的输入参数。它们看起来像是宏/函数调用，但是在宏名称中使用特殊变量 `$` 前缀。请注意，与普通宏不同，扩展不是由预处理器完成的，而是在 SWIG 解析/编译阶段完成的。以下特殊变量宏可在所有语言模块中使用。

#### 11.4.4.1 `$descriptor(type)`

This macro expands into the type descriptor structure for any C/C++ type specified in `type`. It behaves like the `$1_descriptor` special variable described above except that the type to expand is taken from the macro argument rather than inferred from the typemap type. For example, `$descriptor(std::vector<int> *)` will expand into `SWIGTYPE_p_std__vectorT_int_t`. This macro is mostly used in the scripting target languages and is demonstrated later in the [Run-time type checker usage](http://swig.org/Doc3.0/Typemaps.html#Typemaps_runtime_type_checker_usage) section.

> 这个宏扩展为 `type` 中指定的任何 C/C++ 类型的类型描述符结构。它的行为类似于上述的 `$1_descriptor` 特殊变量，不同之处在于要扩展的类型是从宏参数中获取的，而不是从类型映射类型中推断出来的。例如，`$descriptor(std::vector<int> *)` 将扩展为 `SWIGTYPE_p_std__vectorT_int_t`。该宏主要用于脚本目标语言，稍后在[运行时类型检查器用法](http://swig.org/Doc3.0/Typemaps.html#Typemaps_runtime_type_checker_usage)章节中进行演示。

#### 11.4.4.2 `$typemap(method, typepattern)`

This macro uses the [pattern matching rules](http://swig.org/Doc3.0/Typemaps.html#Typemaps_pattern_matching) described earlier to lookup and then substitute the special variable macro with the code in the matched typemap. The typemap to search for is specified by the arguments, where `method` is the typemap method name and `typepattern` is a type pattern as per the `%typemap` specification in the [Defining a typemap](http://swig.org/Doc3.0/Typemaps.html#Typemaps_defining) section.

The special variables within the matched typemap are expanded into those for the matched typemap type, not the typemap within which the macro is called. In practice, there is little use for this macro in the scripting target languages. It is mostly used in the target languages that are statically typed as a way to obtain the target language type given the C/C++ type and more commonly only when the C++ type is a template parameter.

The example below is for C# only and uses some typemap method names documented in the C# chapter, but it shows some of the possible syntax variations.

> 此宏使用前面描述的[模式匹配规则](http://swig.org/Doc3.0/Typemaps.html#Typemaps_pattern_matching)查找，然后用匹配的类型映射中的代码替换特殊变量宏。根据参数指定要搜索的类型映射，其中 `method` 是类型映射方法名称，而 `typepattern` 是类型模式，正如[定义类型映射](http://swig.org/Doc3.0/Typemaps.html#Typemaps_defining)章节的 `%typemap` 规则。
>
> 匹配的类型映射中的特殊变量将扩展为匹配的类型映射类型的特殊变量，而不是其中调用宏的类型映射。实际上，在脚本目标语言中，此宏很少使用。它通常用在目标语言中，这些目标语言是静态类型化的，以便在给定 C/C++ 类型的情况下获取目标语言类型，并且更常见的情况是仅在 C++ 类型是模板参数时使用。
>
> 下面的示例仅适用于 C#，并使用了 C# 一章中记录的某些类型映射方法名称，但它显示了一些可能的语法变体。

```
%typemap(cstype) unsigned long    "uint"
%typemap(cstype) unsigned long bb "bool"
%typemap(cscode) BarClass %{
  void foo($typemap(cstype, unsigned long aa) var1,
           $typemap(cstype, unsigned long bb) var2,
           $typemap(cstype, (unsigned long bb)) var3,
           $typemap(cstype, unsigned long) var4)
  {
    // do something
  }
%}
```

The result is the following expansion

> 结果是下列扩展

```
%typemap(cstype) unsigned long    "uint"
%typemap(cstype) unsigned long bb "bool"
%typemap(cscode) BarClass %{
  void foo(uint var1,
           bool var2,
           bool var3,
           uint var4)
  {
    // do something
  }
%}
```

### 11.4.5 特殊变量与类型映射属性

As of SWIG-3.0.7 typemap attributes will also expand special variables and special variable macros.

Example usage showing the expansion in the 'out' attribute (C# specific) as well as the main typemap body:

> 从 SWIG-3.0.7 开始，类型映射属性还将扩展特殊变量和特殊变量宏。
>
> 用法示例显示 `out` 属性（特定于 C#）以及主要的类型映射主体中的扩展：

```
%typemap(ctype, out="$*1_ltype") unsigned int& "$*1_ltype"
```

is equivalent to the following as `$*1_ltype` expands to `unsigned int`:

> 与下面的等价，相当于 `$*1_ltype` 扩展 `unsigned int`：

```
%typemap(ctype, out="unsigned int") unsigned int& "unsigned int"
```

### 11.4.6 特殊变量联合特殊变量宏

Special variables can also be used within special variable macros. The special variables are expanded before they are used in the special variable macros.

Consider the following C# typemaps:

> 特殊变量也可以在特殊变量宏中使用。在特殊变量宏中使用特殊变量之前，先对其进行扩展。
>
> 考虑以下 C# 类型映射：

```
%typemap(cstype) unsigned int "uint"
%typemap(cstype, out="$typemap(cstype, $*1_ltype)") unsigned int& "$typemap(cstype, $*1_ltype)"
```

Special variables are expanded first and hence the above is equivalent to:

> 特殊变量首先被扩展，因此以上等效于：

```
%typemap(cstype) unsigned int "uint"
%typemap(cstype, out="$typemap(cstype, unsigned int)") unsigned int& "$typemap(cstype, unsigned int)"
```

which then expands to:

> 然后扩展：

```
%typemap(cstype) unsigned int "uint"
%typemap(cstype, out="uint") unsigned int& "uint"
```

## 11.5 通用类型映射方法

The set of typemaps recognized by a language module may vary. However, the following typemap methods are nearly universal:

> 语言模块识别的类型映射集可能有所不同。但是，以下类型映射方法几乎是通用的：

### 11.5.1 `in` 类型映射

The "in" typemap is used to convert function arguments from the target language to C. For example:

> `in` 类型映射用于将函数参数从目标语言转换为 C 语言。例如：

```
%typemap(in) int {
  $1 = PyInt_AsLong($input);
}
```

The following special variables are available:

> 以下特殊变量可用：

```
$input            - Input object holding value to be converted.
$symname          - Name of function/method being wrapped
```

This is probably the most commonly redefined typemap because it can be used to implement customized conversions.

In addition, the "in" typemap allows the number of converted arguments to be specified. The `numinputs` attributes facilitates this. For example:

> 这可能是最常见的重新定义的类型映射，因为它可用于实现自定义转换。
>
> 另外，`in` 类型映射允许指定转换参数的数量。`numinputs` 属性有助于实现这一点。例如：

```
// Ignored argument.
%typemap(in, numinputs=0) int *out (int temp) {
  $1 = &temp;
}
```

At this time, only zero or one arguments may be converted. When `numinputs` is set to 0, the argument is effectively ignored and cannot be supplied from the target language. The argument is still required when making the C/C++ call and the above typemap shows the value used is instead obtained from a locally declared variable called `temp`. Usually `numinputs` is not specified, whereupon the default value is 1, that is, there is a one to one mapping of the number of arguments when used from the target language to the C/C++ call. [Multi-argument typemaps](http://swig.org/Doc3.0/Typemaps.html#Typemaps_multi_argument_typemaps) provide a similar concept where the number of arguments mapped from the target language to C/C++ can be changed for multiple adjacent C/C++ arguments.

**Compatibility note:** Specifying `numinputs=0` is the same as the old "ignore" typemap.

> 此时，只能转换零个或一个参数。当 `numinputs` 设置为 0 时，该参数将被有效忽略，并且无法从目标语言中提供。进行 C/C++ 调用时，仍然需要该参数，并且上面的类型映射显示所使用的值是从本地声明的名为 `temp` 的变量中获取的。通常不指定 `numinputs`，因此默认值为 1，即从目标语言到 C/C++ 调用使用时，参数数量是一对一的映射。[多参数类型映射](http://swig.org/Doc3.0/Typemaps.html#Typemaps_multi_argument_typemaps)提供了类似的概念，其中可以为多个相邻的 C更改从目标语言映射到 C/C++ 的参数数量 / C++ 参数。
>
> **注意兼容性**：指定 `numinputs = 0` 与旧的 `ignore` 类型映射相同。

### 11.5.2 `typecheck` 类型映射

The "typecheck" typemap is used to support overloaded functions and methods. It merely checks an argument to see whether or not it matches a specific type. For example:

> `typecheck` 类型映射用于支持重载的函数和方法。它仅检查参数以查看其是否与特定类型匹配。例如：

```
%typemap(typecheck, precedence=SWIG_TYPECHECK_INTEGER) int {
  $1 = PyInt_Check($input) ? 1 : 0;
}
```

For typechecking, the `$1` variable is always a simple integer that is set to 1 or 0 depending on whether or not the input argument is the correct type. Set to 1 if the input argument is the correct type otherwise set to 0.

If you define new "in" typemaps *and* your program uses overloaded methods, you should also define a collection of "typecheck" typemaps. More details about this follow in the [Typemaps and overloading](http://swig.org/Doc3.0/Typemaps.html#Typemaps_overloading)section.

> 对于类型检查，`$1` 变量始终是一个简单整数，根据输入参数是否为正确的类型将其设置为 1 或 0。如果输入参数是正确的类型，则设置为 1，否则设置为 0。
>
> 如果你定义新的 `in` 类型映射*，并且你的程序使用重载方法，则还应该定义 `typecheck` 类型映射的集合。有关此问题的更多详细信息，请参见[类型映射与重载](http://swig.org/Doc3.0/Typemaps.html#Typemaps_overloading)章节。

### 11.5.3 `out` 类型映射

The "out" typemap is used to convert function/method return values from C into the target language. For example:

> `out` 类型映射用于将函数/方法的返回值从 C 转换为目标语言。例如：

```
%typemap(out) int {
  $result = PyInt_FromLong($1);
}
```

The following special variables are available.

> 以下特殊变量可用：

```
$result           - Result object returned to target language.
$symname          - Name of function/method being wrapped
```

The "out" typemap supports an optional attribute flag called "optimal". This is for code optimisation and is detailed in the [Optimal code generation when returning by value](http://swig.org/Doc3.0/Typemaps.html#Typemaps_optimal) section.

> `out` 类型映射支持名为 `optimal` 的可选属性标志。这是用于代码优化的，在[按值返回时的最佳代码生成](http://swig.org/Doc3.0/Typemaps.html#Typemaps_optimal)部分中进行了详细说明。

### 11.5.4 `arginit` 类型映射

The "arginit" typemap is used to set the initial value of a function argument--before any conversion has occurred. This is not normally necessary, but might be useful in highly specialized applications. For example:

> 在进行任何转换之前，`arginit` 类型映射用于设置函数参数的初始值。通常这不是必需的，但在高度专业化的应用程序中可能很有用。例如：

```
// Set argument to NULL before any conversion occurs
%typemap(arginit) int *data {
  $1 = NULL;
}
```

### 11.5.5 `default` 类型映射

The "default" typemap is used to turn an argument into a default argument. For example:

> `default` 类型映射用于将参数转换为默认参数。例如：

```
%typemap(default) int flags {
  $1 = DEFAULT_FLAGS;
}
...
int foo(int x, int y, int flags);
```

The primary use of this typemap is to either change the wrapping of default arguments or specify a default argument in a language where they aren't supported (like C). Target languages that do not support optional arguments, such as Java and C#, effectively ignore the value specified by this typemap as all arguments must be given.

Once a default typemap has been applied to an argument, all arguments that follow must have default values. See the [Default/optional arguments](http://swig.org/Doc3.0/SWIG.html#SWIG_default_args) section for further information on default argument wrapping.

> 此类型映射的主要用途是更改默认参数的包装，或为不支持默认参数的语言（例如 C）指定默认参数。不支持可选参数的目标语言（例如 Java 和 C#）实际上会忽略此类型映射所指定的值，因为必须提供所有参数。
>
> 将默认类型映射应用于参数后，后面的所有参数都必须具有默认值。有关默认参数包装的更多信息，请参见[默认/可选参数](http://swig.org/Doc3.0/SWIG.html#SWIG_default_args)部分。

### 11.5.6 `check` 类型映射

The "check" typemap is used to supply value checking code during argument conversion. The typemap is applied *after* arguments have been converted. For example:

> `check` 类型映射用于在参数转换期间提供值检查代码。类型参数是在参数转换之后应用的。例如：

```
%typemap(check) int positive {
  if ($1 <= 0) {
    SWIG_exception(SWIG_ValueError, "Expected positive value.");
  }
}
```

### 11.5.7 `"argout"` 类型映射

The "argout" typemap is used to return values from arguments. This is most commonly used to write wrappers for C/C++ functions that need to return multiple values. The "argout" typemap is almost always combined with an "in" typemap---possibly to ignore the input value. For example:

> `argout` 类型映射用于从参数返回值。这最常用于为需要返回多个值的 C/C++ 函数编写包装器。`argout` 类型映射几乎总是与 `in` 类型映射结合使用——可能会忽略输入值。例如：

```
/* Set the input argument to point to a temporary variable */
%typemap(in, numinputs=0) int *out (int temp) {
  $1 = &temp;
}

%typemap(argout) int *out {
  // Append output value $1 to $result
  ...
}
```

The following special variables are available.

> 可以使用下列特殊变量

```
$result           - Result object returned to target language.
$input            - The original input object passed.
$symname          - Name of function/method being wrapped
```

The code supplied to the "argout" typemap is always placed after the "out" typemap. If multiple return values are used, the extra return values are often appended to return value of the function.

See the `typemaps.i` library file for examples.

> 提供给 `argout` 类型映射的代码始终放置在 `out` 类型映射之后。如果使用多个返回值，则通常会将多余的返回值附加到函数的返回值上。
>
> 有关示例，请参见 `typemaps.i` 库文件。

### 11.5.8 `"freearg"` 类型映射

The "freearg" typemap is used to cleanup argument data. It is only used when an argument might have allocated resources that need to be cleaned up when the wrapper function exits. The "freearg" typemap usually cleans up argument resources allocated by the "in" typemap. For example:

> `freearg` 类型映射用于清除参数数据。仅当参数可能分配了包装器函数退出时需要清除的资源时，才使用它。通常，`freearg` 类型映射会清除 `in` 类型映射分配的参数资源。例如：

```
// Get a list of integers
%typemap(in) int *items {
  int nitems = Length($input);
  $1 = (int *) malloc(sizeof(int)*nitems);
}
// Free the list
%typemap(freearg) int *items {
  free($1);
}
```

The "freearg" typemap inserted at the end of the wrapper function, just before control is returned back to the target language. This code is also placed into a special variable `$cleanup` that may be used in other typemaps whenever a wrapper function needs to abort prematurely.

> 在控件返回到目标语言之前，将 `freearg` 类型映射插入包装器函数的末尾。这段代码也被放入一个特殊的变量 `$cleanup` 中，只要包装器函数需要提前中止，该变量就可以在其他类型映射中使用。

### 11.5.9 `newfree` 类型映射

The "newfree" typemap is used in conjunction with the `%newobject` directive and is used to deallocate memory used by the return result of a function. For example:

> `newfree` 类型映射与 `%newobject` 指令一起使用，用于释放函数返回结果使用的内存。例如：

```
%typemap(newfree) string * {
  delete $1;
}
%typemap(out) string * {
  $result = PyString_FromString($1->c_str());
}
...

%newobject foo;
...
string *foo();
```

See [Object ownership and %newobject](http://swig.org/Doc3.0/Customization.html#Customization_ownership) for further details.

> 更多细节请查看[对象所有权和 `%newobject`](http://swig.org/Doc3.0/Customization.html#Customization_ownership)章节。

### 11.5.10 `ret` 类型映射

The "ret" typemap is not used very often, but can be useful for anything associated with the return type, such as resource management, return value error checking, etc. Usually this can all be done in the "out" typemap, but sometimes it is handy to use the "out" typemap code untouched and add to the generated code using the code in the "ret" typemap. One such case is memory clean up. For example, a `stringheap_t` type is defined indicating that the returned memory must be deleted and a `string_t` type is defined indicating that the returned memory must not be deleted.

> `ret` 类型映射不是很经常使用，但是对于与返回类型相关的任何事情（例如资源管理，返回值错误检查等）都很有用。通常都可以在 `out` 类型映射中完成，但是有时方便地使用未修改的 `out` 类型映射代码，并使用 `ret` 类型映射中的代码添加到生成的代码中。一种这样的情况是内存清理。例如，定义了 `stringheap_t` 类型，指示必须删除返回的内存，定义 `string_t` 类型，指示必须删除返回的内存。

```
%typemap(ret) stringheap_t %{
  free($1);
%}

typedef char * string_t;
typedef char * stringheap_t;

string_t MakeString1();
stringheap_t MakeString2();
```

The "ret" typemap above will only be used for `MakeString2`, but both functions will use the default "out" typemap for `char *` provided by SWIG. The code above would ensure the appropriate memory is freed in all target languages as the need to provide custom "out" typemaps (which involve target language specific code) is not necessary.

This approach is an alternative to using the "newfree" typemap and `%newobject` as there is no need to list all the functions that require the memory cleanup, it is purely done on types.

> 上面的 `ret` 类型映射将仅用于 `MakeString2`，但是两个函数都将使用 SWIG 提供的 `char *` 的默认`out` 类型映射。上面的代码将确保在所有目标语言中释放适当的内存，因为不需要提供自定义的 `out` 类型映射（涉及目标语言特定的代码）。
>
> 这种方法是使用 `newfree` 类型映射和 `%newobject` 的一种替代方法，因为不需要列出所有需要内存清理的功能，它完全是在类型上完成的。

### 11.5.11 `memberin` 类型映射

The "memberin" typemap is used to copy data from *an already converted input value* into a structure member. It is typically used to handle array members and other special cases. For example:

> `memberin` 类型映射用于将数据从*已经转换的输入值*复制到结构成员中。它通常用于处理数组成员和其他特殊情况。例如：

```
%typemap(memberin) int [4] {
  memmove($1, $input, 4*sizeof(int));
}
```

It is rarely necessary to write "memberin" typemaps---SWIG already provides a default implementation for arrays, strings, and other objects.

> 几乎没有必要编写 `memberin` 类型映射——SWIG 已经为数组、字符串和其他对象提供了默认实现。

### 11.5.12 `varin` 类型映射

The "varin" typemap is used to convert objects in the target language to C for the purposes of assigning to a C/C++ global variable. This is implementation specific.

> `varin` 类型映射用于将目标语言中的对象转换为 C，以分配给 C/C++ 全局变量。这是特定于实现的。

### 11.5.13 `varout` 类型映射

The "varout" typemap is used to convert a C/C++ object to an object in the target language when reading a C/C++ global variable. This is implementation specific.

> 读取 C/C++ 全局变量时，`varout` 类型映射用于将 C/C++ 对象转换为目标语言中的对象。这是特定于实现的。

### 11.5.14 `throws` 类型映射

The "throws" typemap is only used when SWIG parses a C++ method with an exception specification or has the `%catches` feature attached to the method. It provides a default mechanism for handling C++ methods that have declared the exceptions they will throw. The purpose of this typemap is to convert a C++ exception into an error or exception in the target language. It is slightly different to the other typemaps as it is based around the exception type rather than the type of a parameter or variable. For example:

> 仅当 SWIG 解析具有异常规范的 C++ 方法或将 `%catches` 功能附加到该方法时，才使用 `throw` 类型映射。它提供了一种默认机制来处理声明了将要抛出的异常的 C++ 方法。此类型映射的目的是将 C++ 异常转换为目标语言中的错误或异常。它与其他类型映射略有不同，因为它基于异常类型而不是参数或变量的类型。例如：

```
%typemap(throws) const char * %{
  PyErr_SetString(PyExc_RuntimeError, $1);
  SWIG_fail;
%}
void bar() throw (const char *);
```

As can be seen from the generated code below, SWIG generates an exception handler with the catch block comprising the "throws" typemap content.

> 从下面的生成代码中可以看出，SWIG 生成带有 `catch` 块的异常处理程序，该 `catch` 块包含 `throw` 类型映射内容。

```c
...
try {
  bar();
}
catch(char const *_e) {
  PyErr_SetString(PyExc_RuntimeError, _e);
  SWIG_fail;
}
...
```

Note that if your methods do not have an exception specification yet they do throw exceptions, SWIG cannot know how to deal with them. For a neat way to handle these, see the [Exception handling with %exception](http://swig.org/Doc3.0/Customization.html#Customization_exception) section.

> 请注意，如果你的方法没有异常规范，但它们确实会引发异常，则 SWIG 无法知道如何处理它们。有关处理这些错误的巧妙方法，请参阅[使用 `%exception` 处理异常](http://swig.org/Doc3.0/Customization.html#Customization_exception)部分。

## 11.6 一些类型映射示例

This section contains a few examples. Consult language module documentation for more examples.

> 本节包含一些示例。有关更多示例，请查阅语言模块的文档。

### 11.6.1 数组的类型映射

A common use of typemaps is to provide support for C arrays appearing both as arguments to functions and as structure members.

For example, suppose you had a function like this:

> 类型映射的一种常见用法是为 C 数组提供支持，这些 C 数组既作为函数的参数出现，又作为结构体成员出现。
>
> 例如，假设你具有如下函数：

```c
void set_vector(int type, float value[4]);
```

If you wanted to handle `float value[4]` as a list of floats, you might write a typemap similar to this:

> 如果你想将 `float value[4]` 作为一列浮点数表处理，则可以编写类似于以下内容的类型映射：

```
%typemap(in) float value[4] (float temp[4]) {
  int i;
  if (!PySequence_Check($input)) {
    PyErr_SetString(PyExc_ValueError, "Expected a sequence");
    SWIG_fail;
  }
  if (PySequence_Length($input) != 4) {
    PyErr_SetString(PyExc_ValueError, "Size mismatch. Expected 4 elements");
    SWIG_fail;
  }
  for (i = 0; i < 4; i++) {
    PyObject *o = PySequence_GetItem($input, i);
    if (PyNumber_Check(o)) {
      temp[i] = (float) PyFloat_AsDouble(o);
    } else {
      PyErr_SetString(PyExc_ValueError, "Sequence elements must be numbers");
      SWIG_fail;
    }
  }
  $1 = temp;
}
```

In this example, the variable `temp` allocates a small array on the C stack. The typemap then populates this array and passes it to the underlying C function.

When used from Python, the typemap allows the following type of function call:

> 在这个例子中，变量 `temp` 在 C 栈上分配了一个小数组。然后，类型映射将填充此数组，并将其传递给基础 C 函数。
>
> 当从 Python 使用时，类型映射允许以下类型的函数调用：

```python
>>> set_vector(type, [ 1, 2.5, 5, 20 ])
```

If you wanted to generalize the typemap to apply to arrays of all dimensions you might write this:

> 如果要泛化类型映射以应用于所有维度的数组，则可以这样编写：

```
%typemap(in) float value[ANY] (float temp[$1_dim0]) {
  int i;
  if (!PySequence_Check($input)) {
    PyErr_SetString(PyExc_ValueError, "Expected a sequence");
    SWIG_fail;
  }
  if (PySequence_Length($input) != $1_dim0) {
    PyErr_SetString(PyExc_ValueError, "Size mismatch. Expected $1_dim0 elements");
    SWIG_fail;
  }
  for (i = 0; i < $1_dim0; i++) {
    PyObject *o = PySequence_GetItem($input, i);
    if (PyNumber_Check(o)) {
      temp[i] = (float) PyFloat_AsDouble(o);
    } else {
      PyErr_SetString(PyExc_ValueError, "Sequence elements must be numbers");
      SWIG_fail;
    }
  }
  $1 = temp;
}
```

In this example, the special variable `$1_dim0` is expanded with the actual array dimensions. Multidimensional arrays can be matched in a similar manner. For example:

> 在这个例子中，特殊变量 `$1_dim0` 被扩展为实际的数组维度。多维数组可以类似的方式进行匹配。例如：

```
%typemap(in) float matrix[ANY][ANY] (float temp[$1_dim0][$1_dim1]) {
  ... convert a 2d array ...
}
```

For large arrays, it may be impractical to allocate storage on the stack using a temporary variable as shown. To work with heap allocated data, the following technique can be used.

> 对于大型数组，使用所示的临时变量在堆栈上分配存储可能不切实际。要使用堆分配的数据，可以使用以下技术。

```
%typemap(in) float value[ANY] {
  int i;
  if (!PySequence_Check($input)) {
    PyErr_SetString(PyExc_ValueError, "Expected a sequence");
    SWIG_fail;
  }
  if (PySequence_Length($input) != $1_dim0) {
    PyErr_SetString(PyExc_ValueError, "Size mismatch. Expected $1_dim0 elements");
    SWIG_fail;
  }
  $1 = (float *) malloc($1_dim0*sizeof(float));
  for (i = 0; i < $1_dim0; i++) {
    PyObject *o = PySequence_GetItem($input, i);
    if (PyNumber_Check(o)) {
      $1[i] = (float) PyFloat_AsDouble(o);
    } else {
      free($1);
      PyErr_SetString(PyExc_ValueError, "Sequence elements must be numbers");
      SWIG_fail;
    }
  }
}
%typemap(freearg) float value[ANY] {
  if ($1) free($1);
}
```

In this case, an array is allocated using `malloc`. The `freearg` typemap is then used to release the argument after the function has been called.

Another common use of array typemaps is to provide support for array structure members. Due to subtle differences between pointers and arrays in C, you can't just "assign" to a array structure member. Instead, you have to explicitly copy elements into the array. For example, suppose you had a structure like this:

> 在这种情况下，使用 `malloc` 分配数组。然后，在调用函数后，使用 `freearg` 类型映射释放参数。
>
> 数组类型映射的另一个常见用途是为数组结构成员提供支持。由于 C 语言中的指针和数组之间存在细微的差异，因此你不能只是“分配”给数组结构成员。相反，你必须将元素显式复制到数组中。例如，假设你具有这样的结构体：

```c
struct SomeObject {
  float value[4];
  ...
};
```

When SWIG runs, it won't produce any code to set the `vec` member. You may even get a warning message like this:

> SWIG 运行时，不会产生任何代码来设置 `vec` 成员。你甚至可能收到以下警告消息：

```
$ swig -python  example.i
example.i:10: Warning 462: Unable to set variable of type float [4].
```

These warning messages indicate that SWIG does not know how you want to set the `vec` field.

To fix this, you can supply a special "memberin" typemap like this:

> 这些警告消息表明 SWIG 不知道你如何设置 `vec` 字段。
>
> 要解决此问题，可以提供一个特殊的 `memberin` 类型映射，如下所示：

```
%typemap(memberin) float [ANY] {
  int i;
  for (i = 0; i < $1_dim0; i++) {
      $1[i] = $input[i];
  }
}
```

The memberin typemap is used to set a structure member from data that has already been converted from the target language to C. In this case, `$input` is the local variable in which converted input data is stored. This typemap then copies this data into the structure.

When combined with the earlier typemaps for arrays, the combination of the "in" and "memberin" typemap allows the following usage:

> `memberin` 类型映射用于从已经从目标语言转换为 C 的数据中设置结构成员。在这种情况下，`$input` 是局部变量，用于存储转换后的输入数据。然后，此类型映射将此数据复制到结构体中。
>
> 当与早期的数组类型映射结合使用时，`in` 和 `memberin` 类型映射的组合允许以下用法：

```python
>>> s = SomeObject()
>>> s.x = [1, 2.5, 5, 10]
```

Related to structure member input, it may be desirable to return structure members as a new kind of object. For example, in this example, you will get very odd program behavior where the structure member can be set nicely, but reading the member simply returns a pointer:

> 与结构体成员输入有关，可能希望将结构成员作为一种新的对象返回。例如，在此示例中，你将获得非常奇怪的程序行为，可以很好地设置结构成员，但是读取成员仅返回一个指针：

```python
>>> s = SomeObject()
>>> s.x = [1, 2.5, 5, 10]
>>> print s.x
_1008fea8_p_float
>>>
```

To fix this, you can write an "out" typemap. For example:

> 要修正的话，你可以使用 `out` 类型映射。例如：

```
%typemap(out) float [ANY] {
  int i;
  $result = PyList_New($1_dim0);
  for (i = 0; i < $1_dim0; i++) {
    PyObject *o = PyFloat_FromDouble((double) $1[i]);
    PyList_SetItem($result, i, o);
  }
}
```

Now, you will find that member access is quite nice:

> 现在，你可以发现成员访问变的相当正常：

```
>>> s = SomeObject()
>>> s.x = [1, 2.5, 5, 10]
>>> print s.x
[ 1, 2.5, 5, 10]
```

**Compatibility Note:** SWIG1.1 used to provide a special "memberout" typemap. However, it was mostly useless and has since been eliminated. To return structure members, simply use the "out" typemap.

> **注意兼容性**：SWIG1.1 过去提供特殊的 `memberout` 类型映射。但是，它几乎没有用，因此已被淘汰。要返回结构成员，只需使用 `out` 类型映射。

### 11.6.2 用类型映射的实现限制

One particularly interesting application of typemaps is the implementation of argument constraints. This can be done with the "check" typemap. When used, this allows you to provide code for checking the values of function arguments. For example:

> 类型映射的一个有趣应用是实现参数限制。这可以用 `check` 类型映射做到。类型映射允许你提供代码以检查函数参数的值。例如：

```
%module math

%typemap(check) double posdouble {
  if ($1 < 0) {
    croak("Expecting a positive number");
  }
}

...
double sqrt(double posdouble);
```

This provides a sanity check to your wrapper function. If a negative number is passed to this function, a Perl exception will be raised and your program terminated with an error message.

This kind of checking can be particularly useful when working with pointers. For example:

> 这为包装器函数提供了完整性检查。如果将负数传递给此函数，则会引发 Perl 异常，并且你的程序终止并显示错误消息。
>
> 在使用指针时，这种检查特别有用。例如：

```
%typemap(check) Vector * {
  if ($1 == 0) {
    PyErr_SetString(PyExc_TypeError, "NULL Pointer not allowed");
    SWIG_fail;
  }
}
```

will prevent any function involving a `Vector *` from accepting a NULL pointer. As a result, SWIG can often prevent a potential segmentation faults or other run-time problems by raising an exception rather than blindly passing values to the underlying C/C++ program.

> 会阻止任何涉及 `Vector *` 的函数接受空指针。最终，SWIG 通常可以通过引发异常，而不是将值盲目地传递给底层 C/C++ 程序来防止潜在的分段错误或其他运行时问题。

## 11.7 多目标语言的类型映射

The code within typemaps is usually language dependent, however, many target languages support the same typemaps. In order to distinguish typemaps across different languages, the preprocessor should be used. For example, the "in" typemap for Perl and Ruby could be written as:

> 类型映射中的代码通常取决于语言，但是，许多目标语言都支持相同的类型映射。为了区分不同语言之间的类型映射，应使用预处理器。例如，Perl 和 Ruby 的 `in` 类型映射可以写为：

```
#if defined(SWIGPERL)
  %typemap(in) int "$1 = ($1_ltype) SvIV($input);"
#elif defined(SWIGRUBY)
  %typemap(in) int "$1 = NUM2INT($input);"
#else
  #warning no "in" typemap defined
#endif
```

The full set of language specific macros is defined in the [Conditional Compilation](http://swig.org/Doc3.0/Preprocessor.html#Preprocessor_condition_compilation) section. The example above also shows a common approach of issuing a warning for an as yet unsupported language.

**Compatibility note:** In SWIG-1.1 different languages could be distinguished with the language name being put within the `%typemap` directive, for example, `%typemap(ruby, in) int "$1 = NUM2INT($input);"`.

> 在[条件编译](http://swig.org/Doc3.0/Preprocessor.html#Preprocessor_condition_compilation)章节中定义了特定于语言的完整宏集合。上面的示例还显示了针对尚不支持的语言发出警告的常见方法。
>
> **注意兼容性**：在 SWIG-1.1中，可以通过在 `%typemap` 指令中放入语言名称来区分不同的语言，例如，`%typemap(ruby, in) int "$1 = NUM2INT($input);"`。

## 11.8 返回值时的最优代码生成

The "out" typemap is the main typemap for return types. This typemap supports an optional attribute flag called "optimal", which is for reducing temporary variables and the amount of generated code, thereby giving the compiler the opportunity to use *return value optimization* for generating faster executing code. It only really makes a difference when returning objects by value and has some limitations on usage, as explained later on.

When a function returns an object by value, SWIG generates code that instantiates the default type on the stack then assigns the value returned by the function call to it. A copy of this object is then made on the heap and this is what is ultimately stored and used from the target language. This will be clearer considering an example. Consider running the following code through SWIG:

> `out` 类型映射是返回类型的主要类型映射。此类型映射支持一个称为 `optimal` 的可选属性标志，该标志用于减少临时变量和所生成的代码量，从而使编译器有机会使用*返回值优化*来生成执行速度更快的代码。如后面所述，只有在按值返回对象时，它才真正有所不同，并且在用法上有一些限制。
>
> 当函数按值返回对象时，SWIG 会生成代码，该代码实例化堆栈上的默认类型，然后将函数调用返回的值分配给它。然后在堆上创建此对象的副本，这是最终从目标语言存储和使用的对象。考虑一个例子，这将更加清楚。考虑通过 SWIG 运行以下代码：

```
%typemap(out) SWIGTYPE %{
  $result = new $1_ltype((const $1_ltype &)$1);
%}

%inline %{
#include <iostream>
using namespace std;

struct XX {
  XX() { cout << "XX()" << endl; }
  XX(int i) { cout << "XX(" << i << ")" << endl; }
  XX(const XX &other) { cout << "XX(const XX &)" << endl; }
  XX & operator =(const XX &other) { cout << "operator=(const XX &)" << endl; return *this; }
  ~XX() { cout << "~XX()" << endl; }
  static XX create() {
    return XX(0);
  }
};
%}
```

The "out" typemap shown is the default typemap for C# when returning objects by value. When making a call to `XX::create()` from C#, the output is as follows:

> 当按值返回对象时，显示的 `out` 类型映射是 C# 的默认类型映射。从 C# 调用 `XX::create()` 时，输出如下：

```csharp
XX()
XX(0)
operator=(const XX &)
~XX()
XX(const XX &)
~XX()
~XX()
```

Note that three objects are being created as well as an assignment. Wouldn't it be great if the `XX::create()` method was the only time a constructor was called? As the method returns by value, this is asking a lot and the code that SWIG generates by default makes it impossible for the compiler to use *return value optimisation (RVO)*. However, this is where the "optimal" attribute in the "out" typemap can help out. If the typemap code is kept the same and just the "optimal" attribute specified like this:

> 请注意，正在创建三个对象以及一个分配。如果唯一调用构造函数的方法是 `XX::create()` 方法，那不是很好吗？由于该方法按值返回，因此要求很多，而 SWIG 默认生成的代码使编译器无法使用*返回值优化（RVO）*。但是，这是 `out` 类型映射中的 `optimal` 属性可以提供帮助的地方。如果类型映射代码保持相同，并且仅指定 `optimal` 属性，如下所示：

```
%typemap(out, optimal="1") SWIGTYPE %{
  $result = new $1_ltype((const $1_ltype &)$1);
%}
```

then when the code is run again, the output is simply:

> 再次运行代码，输出很简单：

```csharp
XX(0)
~XX()
```

How the "optimal" attribute works is best explained using the generated code. Without "optimal", the generated code is:

> 使用生成的代码可以最好地解释 `optimal` 属性的工作方式。如果没有 `optimal`，则生成的代码为：

```csharp
SWIGEXPORT void * SWIGSTDCALL CSharp_XX_create() {
  void * jresult ;
  XX result;
  result = XX::create();
  jresult = new XX((const XX &)result);
  return jresult;
}
```

With the "optimal" attribute, the code is:

> 有了 `optimal` 属性，代码为：

```csharp
SWIGEXPORT void * SWIGSTDCALL CSharp_XX_create() {
  void * jresult ;
  jresult = new XX((const XX &)XX::create());
  return jresult;
}
```

The major difference is the `result` temporary variable holding the value returned from `XX::create()` is no longer generated and instead the copy constructor call is made directly from the value returned by `XX::create()`. With modern compilers implementing RVO, the copy is not actually done, in fact the object is never created on the stack in `XX::create()` at all, it is simply created directly on the heap. In the first instance, the `$1` special variable in the typemap is expanded into `result`. In the second instance, `$1`is expanded into `XX::create()` and this is essentially what the "optimal" attribute is telling SWIG to do.

The "optimal" attribute optimisation is not turned on by default as it has a number of restrictions. Firstly, some code cannot be condensed into a simple call for passing into the copy constructor. One common occurrence is when [%exception](http://swig.org/Doc3.0/Customization.html#Customization_exception) is used. Consider adding the following `%exception` to the example:

> 主要区别是 `result` 临时变量不再保存从 `XX::create()` 返回的值，而是直接从 `XX::create()` 返回的值进行复制构造函数调用。使用实现 RVO 的现代编译器，实际上并不会完成复制，实际上，该对象根本不会在 `XX::create()` 中的堆栈上创建，而只是在堆上直接创建。首先，将类型映射中的 `$1` 特殊变量扩展为 `result`。在第二种情况下，将 `$1` 扩展为 `XX::create()`，这实际上就是 `optimal` 属性告诉 SWIG 要做的事情。
>
> 默认情况下，`optimal` 属性优化未启用，因为它有许多限制。首先，某些代码不能被精简为传递给复制构造函数的简单调用。一种常见的情况是使用 [`%exception`](http://swig.org/Doc3.0/Customization.html#Customization_exception)。考虑在示例中添加以下 `%exception`：

```
%exception XX::create() %{
try {
  $action
} catch(const std::exception &e) {
  cout << e.what() << endl;
}
%}
```

SWIG can detect when the "optimal" attribute cannot be used and will ignore it and in this case will issue the following warning:

> SWIG 可以检测到何时无法使用 `optimal` 属性，并将其忽略，在这种情况下，将发出以下警告：

```
example.i:28: Warning 474: Method XX::create() usage of the optimal attribute ignored
example.i:14: Warning 474: in the out typemap as the following cannot be used to generate
optimal code:
try {
  result = XX::create();
} catch(const std::exception &e) {
  cout << e.what() << endl;
}
```

It should be clear that the above code cannot be used as the argument to the copy constructor call, that is, for the `$1` substitution.

Secondly, if the typemaps uses `$1` more than once, then multiple calls to the wrapped function will be made. Obviously that is not very optimal. In fact SWIG attempts to detect this and will issue a warning something like:

> 应该清楚的是，上面的代码不能用作复制构造函数调用的参数，即不能用于 `$1` 替换。
>
> 其次，如果类型映射多次使用 `$1`，则将多次调用包装器函数。显然，这不是很理想。实际上，SWIG 会尝试检测到这一点，并将发出类似以下的警告：

```
example.i:21: Warning 475: Multiple calls to XX::create() might be generated due to
example.i:7: Warning 475: optimal attribute usage in the out typemap.
```

However, it doesn't always get it right, for example when `$1` is within some commented out code.

> 但是，它并不总是正确，例如，当 `$1` 在某些注释掉的代码中时。

## 11.9 多参数类型映射

So far, the typemaps presented have focused on the problem of dealing with single values. For example, converting a single input object to a single argument in a function call. However, certain conversion problems are difficult to handle in this manner. As an example, consider the example at the very beginning of this chapter:

> 到目前为止，所提供的类型映射已集中在处理单个值的问题上。例如，在函数调用中将单个输入对象转换为单参数。但是，某些转换问题很难以这种方式处理。例如，请考虑本章开头的示例：

```c
int foo(int argc, char *argv[]);
```

Suppose that you wanted to wrap this function so that it accepted a single list of strings like this:

> 假设你想包装此函数，以使其接受单个字符串列表，如下所示：

```python
>>> foo(["ale", "lager", "stout"])
```

To do this, you not only need to map a list of strings to `char *argv[]`, but the value of `int argc` is implicitly determined by the length of the list. Using only simple typemaps, this type of conversion is possible, but extremely painful. Multi-argument typemaps help in this situation.

A multi-argument typemap is a conversion rule that specifies how to convert a *single* object in the target language to a set of consecutive function arguments in C/C++. For example, the following multi-argument maps perform the conversion described for the above example:

> 为此，你不仅需要将字符串列表映射到 `char *argv[]`，而且 `int argc` 的值由列表的长度隐式确定。仅使用简单的类型映射，这种类型的转换是可能的，但是非常痛苦。在这种情况下，多参数类型映射会有所帮助。
>
> 多参数类型映射是一种转换规则，它指定如何将目标语言中的*单个*对象转换为 C/C++ 中的一组连续函数参数。例如，以下多参数映射执行上述示例中描述的转换：

```
%typemap(in) (int argc, char *argv[]) {
  int i;
  if (!PyList_Check($input)) {
    PyErr_SetString(PyExc_ValueError, "Expecting a list");
    SWIG_fail;
  }
  $1 = PyList_Size($input);
  $2 = (char **) malloc(($1+1)*sizeof(char *));
  for (i = 0; i < $1; i++) {
    PyObject *s = PyList_GetItem($input, i);
    if (!PyString_Check(s)) {
      free($2);
      PyErr_SetString(PyExc_ValueError, "List items must be strings");
      SWIG_fail;
    }
    $2[i] = PyString_AsString(s);
  }
  $2[i] = 0;
}

%typemap(freearg) (int argc, char *argv[]) {
  if ($2) free($2);
}

/* Required for C++ method overloading */
%typecheck(SWIG_TYPECHECK_STRING_ARRAY) (int argc, char *argv[]) {
  $1 = PyList_Check($input) ? 1 : 0;
}
```

A multi-argument map is always specified by surrounding the arguments with parentheses as shown. For example:

> 如上所示，总是通过用括号将参数括起来来指定多参数映射。例如：

```
%typemap(in) (int argc, char *argv[]) { ... }
```

Within the typemap code, the variables `$1`, `$2`, and so forth refer to each type in the map. All of the usual substitutions apply--just use the appropriate `$1` or `$2` prefix on the variable name (e.g., `$2_type`, `$1_ltype`, etc.)

Multi-argument typemaps always have precedence over simple typemaps and SWIG always performs longest-match searching. Therefore, you will get the following behavior:

> 在类型映射代码中，变量 `$1`、`$2` 等引用映射中的每种类型。所有通常的替换都适用——只需在变量名称上使用适当的 `$1` 或 `$2` 前缀即可（例如 `$2_type`，`$1_ltype` 等）
>
> 多参数类型映射始终优先于简单类型映射，而 SWIG 始终执行最长匹配搜索。因此，你将得到以下行为：

```
%typemap(in) int argc                              { ... typemap 1 ... }
%typemap(in) (int argc, char *argv[])              { ... typemap 2 ... }
%typemap(in) (int argc, char *argv[], char *env[]) { ... typemap 3 ... }

int foo(int argc, char *argv[]);                   // Uses typemap 2
int bar(int argc, int x);                          // Uses typemap 1
int spam(int argc, char *argv[], char *env[]);     // Uses typemap 3
```

It should be stressed that multi-argument typemaps can appear anywhere in a function declaration and can appear more than once. For example, you could write this:

> 应该强调的是，多参数类型映射可以出现在函数声明中的任何位置，并且可以出现多次。例如，你可以这样编写：

```
%typemap(in) (int scount, char *swords[]) { ... }
%typemap(in) (int wcount, char *words[]) { ... }

void search_words(int scount, char *swords[], int wcount, char *words[], int maxcount);
```

Other directives such as `%apply` and `%clear` also work with multi-argument maps. For example:

> 其他指令，例如 `%apply` 和 `%clear` 也可以与多参数映射一起使用。例如：

```
%apply (int argc, char *argv[]) {
    (int scount, char *swords[]),
    (int wcount, char *words[])
};
...
%clear (int scount, char *swords[]), (int wcount, char *words[]);
...
```

Don't forget to also provide a suitable [typemap for overloaded functions](http://swig.org/Doc3.0/Typemaps.html#Typemaps_overloading), such as `%typecheck` shown for foo above. This is only required if the function is overloaded in C++.

Although multi-argument typemaps may seem like an exotic, little used feature, there are several situations where they make sense. First, suppose you wanted to wrap functions similar to the low-level `read()` and `write()` system calls. For example:

> 不要忘记提供合适的[重载函数的类型映射](http://swig.org/Doc3.0/Typemaps.html#Typemaps_overloading)，例如上面为 `foo` 显示的 `%typecheck`。仅当函数在 C++ 中重载时才需要。
>
> 尽管多参数类型映射可能看起来像是一种奇特的，很少使用的功能，但在某些情况下它们是有意义的。首先，假设你想包装类似于低级 `read()` 和 `write()` 系统调用的函数。例如：

```c++
typedef unsigned int size_t;

int read(int fd, void *rbuffer, size_t len);
int write(int fd, void *wbuffer, size_t len);
```

As is, the only way to use the functions would be to allocate memory and pass some kind of pointer as the second argument---a process that might require the use of a helper function. However, using multi-argument maps, the functions can be transformed into something more natural. For example, you might write typemaps like this:

> 如此这样，使用这些函数的唯一方法是分配内存并传递某种指针作为第二个参数，该过程可能需要使用辅助函数。但是，使用多参数映射可以将功能转换为更自然的功能。例如，你可以这样编写类型映射：

```
// typemap for an outgoing buffer
%typemap(in) (void *wbuffer, size_t len) {
  if (!PyString_Check($input)) {
    PyErr_SetString(PyExc_ValueError, "Expecting a string");
    SWIG_fail;
  }
  $1 = (void *) PyString_AsString($input);
  $2 = PyString_Size($input);
}

// typemap for an incoming buffer
%typemap(in) (void *rbuffer, size_t len) {
  if (!PyInt_Check($input)) {
    PyErr_SetString(PyExc_ValueError, "Expecting an integer");
    SWIG_fail;
  }
  $2 = PyInt_AsLong($input);
  if ($2 < 0) {
    PyErr_SetString(PyExc_ValueError, "Positive integer expected");
    SWIG_fail;
  }
  $1 = (void *) malloc($2);
}

// Return the buffer.  Discarding any previous return result
%typemap(argout) (void *rbuffer, size_t len) {
  Py_XDECREF($result);   /* Blow away any previous result */
  if (result < 0) {      /* Check for I/O error */
    free($1);
    PyErr_SetFromErrno(PyExc_IOError);
    return NULL;
  }
  $result = PyString_FromStringAndSize($1, result);
  free($1);
}
```

(note: In the above example, `$result` and `result` are two different variables. `result` is the real C datatype that was returned by the function. `$result` is the scripting language object being returned to the interpreter.).

Now, in a script, you can write code that simply passes buffers as strings like this:

> （注意：在上面的示例中，`$result` 和 `result` 是两个不同的变量。`result` 是函数返回的实际 C 数据类型。`$result` 是要返回到解释器的脚本语言对象 ）。
>
> 现在，在脚本中，你可以编写简单地将缓冲区作为字符串传递的代码，如下所示：

```python
>>> f = example.open("Makefile")
>>> example.read(f, 40)
'TOP        = ../..\nSWIG       = $(TOP)/.'
>>> example.read(f, 40)
'./swig\nSRCS       = example.c\nTARGET    '
>>> example.close(f)
0
>>> g = example.open("foo", example.O_WRONLY | example.O_CREAT, 0644)
>>> example.write(g, "Hello world\n")
12
>>> example.write(g, "This is a test\n")
15
>>> example.close(g)
0
>>>
```

A number of multi-argument typemap problems also arise in libraries that perform matrix-calculations--especially if they are mapped onto low-level Fortran or C code. For example, you might have a function like this:

> 在执行矩阵计算的库中，还会出现许多多参数类型映射问题，尤其是如果将它们映射到低级 Fortran 或 C 代码上。例如，你可能具有以下功能：

```c
int is_symmetric(double *mat, int rows, int columns);
```

In this case, you might want to pass some kind of higher-level object as an matrix. To do this, you could write a multi-argument typemap like this:

> 在这种情况下，你可能需要传递某种高级对象作为矩阵。为此，你可以编写一个如下所示的多参数类型映射：

```
%typemap(in) (double *mat, int rows, int columns) {
  MatrixObject *a;
  a = GetMatrixFromObject($input);     /* Get matrix somehow */

  /* Get matrix properties */
  $1 = GetPointer(a);
  $2 = GetRows(a);
  $3 = GetColumns(a);
}
```

This kind of technique can be used to hook into scripting-language matrix packages such as Numeric Python. However, it should also be stressed that some care is in order. For example, when crossing languages you may need to worry about issues such as row-major vs. column-major ordering (and perform conversions if needed). Note that multi-argument typemaps cannot deal with non-consecutive C/C++ arguments; a workaround such as a helper function re-ordering the arguments to make them consecutive will need to be written.

> 这种技术可用于连接脚本语言矩阵包，例如 Numeric Python。但是，还应该强调，一定要谨慎。例如，在使用多种语言时，你可能需要担心行优先与列优先的排序（并在需要时执行转换）。注意，多参数类型映射不能处理非连续的 C/C++ 参数。需要编写一种变通方法，例如帮助函数，将参数重新排序以使其连续。

## 11.10 类型映射警告

Warnings can be added to typemaps so that SWIG generates a warning message whenever the typemap is used. See the information in the [issuing warnings](http://swig.org/Doc3.0/Warnings.html#Warnings_nn5) section.

> 可以将警告添加到类型映射，以便每当使用类型映射时 SWIG 都会生成警告消息。请参阅[发布警告](http://swig.org/Doc3.0/Warnings.html#Warnings_nn5)章节中的信息。

## 11.11 类型映射片段

The primary purpose of fragments is to reduce code bloat that repeated use of typemap code can lead to. Fragments are snippets of code that can be thought of as code dependencies of a typemap. If a fragment is used by more than one typemap, then the snippet of code within the fragment is only generated once. Code bloat is typically reduced by moving typemap code into a support function and then placing the support function into a fragment.

For example, if you have a very long typemap

> 片段的主要目的是减少重复使用类型映射代码可能导致的代码膨胀。片段是代码片段，可以将其视为类型映射的代码依赖项。如果一个片段被多个类型映射使用，则该片段内的代码片段仅生成一次。通常可以通过将类型映射代码移入支持功能，然后将支持功能放入片段中来减少代码膨胀。
>
> 例如，如果你的类型映射很长

```
%typemap(in) MyClass * {
  MyClass *value = 0;

  ... many lines of marshalling code  ...

  $result = value;
}
```

the same marshalling code is often repeated in several typemaps, such as "in", "varin", "directorout", etc. SWIG copies the code for each argument that requires the typemap code, easily leading to code bloat in the generated code. To eliminate this, define a fragment that includes the common marshalling code:

> 相同的编组代码通常在多个类型映射中重复，例如 `in`、`varin`、`directorout` 等。SWIG 为需要该类型映射代码的每个参数复制代码，从而很容易导致所生成代码中的代码膨胀。为了消除这种情况，请定义一个包含通用编组代码的片段：

```
%fragment("AsMyClass", "header") {
  MyClass *AsMyClass(PyObject *obj) {
    MyClass *value = 0;

    ... many lines of marshalling code  ...

    return value;
  }
}

%typemap(in, fragment="AsMyClass") MyClass * {
  $result = AsMyClass($input);
}

%typemap(varin, fragment="AsMyClass") MyClass * {
  $result = AsMyClass($input);
}
```

When the "in" or "varin" typemaps for MyClass are required, the contents of the fragment called "AsMyClass" is added to the "header" section within the generated code, and then the typemap code is emitted. Hence, the method `AsMyClass` will be generated into the wrapper code before any typemap code that calls it.

To define a fragment you need a fragment name, a section name for generating the fragment code into, and the code itself. See [Code insertion blocks](http://swig.org/Doc3.0/SWIG.html#SWIG_nn42) for a full list of section names. Usually the section name used is "header". Different delimiters can be used:

> 当需要 MyClass 的 `in` 或 `varin` 类型映射时，将名为 `AsMyClass` 的片段的内容添加到生成的代码中的 `header` 部分，然后发出该类型映射代码。因此，方法 `AsMyClass` 将在调用它的任何类型映射代码之前生成到包装器代码中。
>
> 要定义一个片段，你需要一个片段名称，用于将片段代码生成到其中的段名称以及代码本身。有关部分名称的完整列表，请参见[代码插入块](http://swig.org/Doc3.0/SWIG.html#SWIG_nn42)。通常，使用的节名称是 `header`。可以使用不同的定界符：

```
%fragment("my_name", "header") %{ ... %}
%fragment("my_name", "header") { ... }
%fragment("my_name", "header") " ... "
```

and these follow the usual preprocessing rules mentioned in the [Preprocessing delimiters](http://swig.org/Doc3.0/Preprocessor.html#Preprocessor_delimiters) section. The following are some rules and guidelines for using fragments:

1. A fragment is added to the wrapping code only once. When using the `MyClass *` typemaps above and wrapping the method:

> 并且它们遵循[预处理分隔符](http://swig.org/Doc3.0/Preprocessor.html#Preprocessor_delimiters)章节中提到的常规预处理规则。以下是使用片段的一些规则和准则：
>
> 1. 一个片段仅被添加到包装代码一次。当使用上面的 `MyClass *` 类型映射并包装方法时：

```c++
void foo(MyClass *a, MyClass *b);
```

the generated code will look something like:

```c++
MyClass *AsMyClass(PyObject *obj) {
...
}

void _wrap_foo(...) {
....
arg1 = AsMyClass(obj1);
arg2 = AsMyClass(obj2);
...
foo(arg1, arg2);
}
```

even as there is duplicated typemap code to process both `a` and `b`, the `AsMyClass` method will be defined only once.

2. A fragment should only be defined once. If there is more than one definition, the first definition is the one used. All other definitions are silently ignored. For example, if you have

> 即使存在重复的类型映射代码来处理 `a` 和 `b`、`AsMyClass` 方法也只会定义一次。
>
> 2. 一个片段只能定义一次。如果有多个定义，则第一个定义是使用的定义。所有其他定义都被忽略。例如，如果你有

```
%fragment("AsMyClass", "header") { ...definition 1... }
....
%fragment("AsMyClass", "header") { ...definition 2... }
```

only the first definition is used. In this way you can override the default fragments in a SWIG library by defining your fragment before the library `%include`. Note that this behavior is the opposite to typemaps, where the last typemap defined/applied prevails. Fragments follow the first-in-first-out convention since they are intended to be global, while typemaps are intended to be locally specialized.

3. Fragment names cannot contain commas.

4. A fragment can use one or more additional fragments, for example:

> 仅使用第一个定义。这样，你可以通过在 `%include` 库之前定义片段来覆盖 SWIG 库中的默认片段。请注意，此行为与类型映射相反，后者以最后定义/应用的类型映射为准。片段遵循先进先出的约定，因为它们是全局的，而类型映射则是本地的。
>
> 3. 片段名称不能包含逗号。
> 4. 一个片段可以使用一个或多个其他片段，例如：

```
%fragment("<limits.h>", "header") {
%#include <limits.h>
}


%fragment("AsMyClass", "header", fragment="<limits.h>") {
MyClass *AsMyClass(PyObject *obj) {
MyClass *value = 0;

... some marshalling code  ...

if  (ival < CHAR_MIN /*defined in <limits.h>*/) {
...
} else {
...
}
...
return value;
}
}
```

in this case, when the "AsMyClass" fragment is emitted, it also triggers the inclusion of the "<limits.h>" fragment.

5. A fragment can have dependencies on a number of other fragments, for example:

> 在这种情况下，发出 `AsMyClass` 片段时，也会触发包含 `<limits.h>` 片段。
>
> 5. 一个片段可以依赖于许多其他片段，例如：

```
%fragment("bigfragment", "header", fragment="frag1", fragment="frag2", fragment="frag3") "";
```

When the "bigfragment" is used, the three dependent fragments "frag1", "frag2" and "frag3" are also pulled in. Note that as "bigframent" is empty (the empty string - ""), it does not add any code itself, but merely triggers the inclusion of the other fragments.

6. A typemap can also use more than one fragment, but since the syntax is different, you need to specify the dependent fragments in a comma separated list. Consider:

> 当使用 `bigfragment` 时，三个从属片段 `frag1`、`frag2` 和 `frag3` 也被拉入。请注意，由于 `bigframent` 为空（空字符串——`""`），因此不添加任何代码本身，但仅触发其他片段的包含。
>
> 6. 一个类型映射也可以使用多个片段，但是由于语法不同，你需要在逗号分隔的列表中指定从属片段。考虑：

```
%typemap(in, fragment="frag1, frag2, frag3") {...}
```

which is equivalent to:

> 等效于：

```
%typemap(in, fragment="bigfragment") {...}
```

when used with the "bigfragment" defined above.

7. Finally, you can force the inclusion of a fragment at any point in the generated code as follows:

> 与上面定义的 `bigfragment` 一起使用时。
>
> 7. 最后，你可以按以下步骤在生成的代码中的任何位置强制包含片段：

```
%fragment("bigfragment");
```

which is very useful inside a template class, for example.

Most readers will probably want to skip the next two sub-sections on advanced fragment usage unless a desire to really get to grips with some powerful but tricky macro and fragment usage that is used in parts of the SWIG typemap library.

> 例如，这在模板类内部非常有用。
>
> 除非希望真正掌握 SWIG 类型映射库的某些部分中使用的某些功能强大但棘手的宏和片段用法，否则大多数读者可能会希望跳过接下来的两节有关高级片段用法的小节。

### 11.11.1 片段类型特化

Fragments can be *type specialized*. The syntax is as follows:

> 片段可以*类型特化*。语法如下：

```
%fragment("name", "header") { ...a type independent fragment... }
%fragment("name"{type}, "header") { ...a type dependent fragment...  }
```

where `type` is a C/C++ type. Like typemaps, fragments can also be used inside templates, for example:

> 其中，`type` 是 C/C++ 类型。像类型映射一样，片段也可以在模板内部使用，例如：

```
template <class T>
struct A {
  %fragment("incode"{A<T>}, "header") {
    ... 'incode' specialized fragment ...
  }

  %typemap(in, fragment="incode"{A<T>}) {
    ... here we use the 'type specialized' fragment "incode"{A<T>} ...
  }
};
```

### 11.11.2 片段与自动类型映射特化

Since fragments can be type specialized, they can be elegantly used to specialize typemaps. For example, if you have something like:

> 由于片段可以是类型特化的，因此可以很好地用于特化类型映射。例如，如果你有以下内容：

```
%fragment("incode"{float}, "header") {
  float in_method_float(PyObject *obj) {
    ...
  }
}

%fragment("incode"{long}, "header") {
  float in_method_long(PyObject *obj) {
    ...
  }
}

// %my_typemaps macro definition
%define %my_typemaps(Type)
%typemap(in, fragment="incode"{Type}) Type {
  value = in_method_##Type(obj);
}
%enddef

%my_typemaps(float);
%my_typemaps(long);
```

then the proper `"incode"{float}` or `"incode"{long}` fragment will be used, and the `in_method_float`and `in_method_long` methods will be called whenever the `float` or `long` types are used as input parameters.

This feature is used a lot in the typemaps shipped in the SWIG library for some scripting languages. The interested (or very brave) reader can take a look at the fragments.swg file shipped with SWIG to see this in action.

> 那么将使用正确的 `"incode"{float}` 或 `"incode"{long}` 片段，并且每当使用 `float` 或 `long` 类型时，就会调用 `in_method_float` 和 `in_method_long` 方法 作为输入参数。
>
> SWIG 库附带的类型映射中的某些脚本语言经常使用此功能。有兴趣的（或非常勇敢的）读者可以查看 SWIG 附带的 `fragments.swg` 文件，以了解实际情况。

## 11.12 运行时类型检查器

Most scripting languages need type information at run-time. This type information can include how to construct types, how to garbage collect types, and the inheritance relationships between types. If the language interface does not provide its own type information storage, the generated SWIG code needs to provide it.

Requirements for the type system:

* Store inheritance and type equivalence information and be able to correctly re-create the type pointer.
* Share type information between modules.
* Modules can be loaded in any order, regardless of actual type dependency.
* Avoid the use of dynamically allocated memory, and library/system calls in general.
* Provide a reasonably fast implementation, minimizing the lookup time for all language modules.
* Custom, language specific information can be attached to types.
* Modules can be unloaded from the type system.

> 大多数脚本语言在运行时都需要类型信息。此类型信息可以包括如何构造类型，如何垃圾回收类型以及类型之间的继承关系。如果语言接口不提供自己的类型信息存储，则生成的 SWIG 代码需要提供它。
>
> 类型系统要求：
>
> * 存储继承和类型等效信息，并能够正确地重新创建类型指针。
> * 在模块之间共享类型信息。
> * 模块可以以任何顺序加载，而不管实际的类型依赖性如何。
> * 避免一般使用动态分配的内存和库/系统调用。
> * 提供合理快速的实施，以最小化所有语言模块的查找时间。
> * 自定义，特定于语言的信息可以附加到类型上。
> * 可以从类型系统中卸载模块。

### 11.12.1 实现

The run-time type checker is used by many, but not all, of SWIG's supported target languages. The run-time type checker features are not required and are thus not used for statically typed languages such as Java and C#. The scripting and scheme based languages rely on it and it forms a critical part of SWIG's operation for these languages.

When pointers, arrays, and objects are wrapped by SWIG, they are normally converted into typed pointer objects. For example, an instance of `Foo *` might be a string encoded like this:

> SWIG 支持的许多（但不是全部）目标语言都使用运行时类型检查器。运行时类型检查器功能不是必需的，因此不用于 Java 和 C# 之类的静态类型语言。基于脚本和方案的语言都依赖它，并且它构成了 SWIG 对这些语言的操作的关键部分。
>
> 当指针，数组和对象由 SWIG 包装时，它们通常会转换为类型化的指针对象。例如，`Foo *` 的实例可能是这样编码的字符串：

```
_108e688_p_Foo
```

At a basic level, the type checker simply restores some type-safety to extension modules. However, the type checker is also responsible for making sure that wrapped C++ classes are handled correctly---especially when inheritance is used. This is especially important when an extension module makes use of multiple inheritance. For example:

> 从根本上讲，类型检查器只是将一些类型安全性恢复到扩展模块。但是，类型检查器还负责确保正确处理包装的 C++ 类——尤其是在使用继承时。当扩展模块利用多重继承时，这一点尤其重要。例如：

```c++
class Foo {
public:
  int x;
};

class Bar {
public:
  int y;
};

class FooBar : public Foo, public Bar {
public:
  int z;
};
```

When the class `FooBar` is organized in memory, it contains the contents of the classes `Foo` and `Bar` as well as its own data members. For example:

> 当在内存中组织类 `FooBar` 时，它包含类 `Foo` 和 `Bar` 的内容以及它自己的数据成员。例如：

```
FooBar --> | -----------|  <-- Foo
           |   int x    |
           |------------|  <-- Bar
           |   int y    |
           |------------|
           |   int z    |
           |------------|
```

Because of the way that base class data is stacked together, the casting of a `Foobar *` to either of the base classes may change the actual value of the pointer. This means that it is generally not safe to represent pointers using a simple integer or a bare `void *`---type tags are needed to implement correct handling of pointer values (and to make adjustments when needed).

In the wrapper code generated for each language, pointers are handled through the use of special type descriptors and conversion functions. For example, if you look at the wrapper code for Python, you will see code similar to the following (simplified for brevity):

> 由于将基类数据堆叠在一起的方式，将 `Foobar *` 强制转换为任一基类都可能会更改指针的实际值。这意味着使用一个简单的整数或一个简单的 `void *`——来表示指针通常是不安全的——需要使用类型标记来实现对指针值的正确处理（并在需要时进行调整）。
>
> 在为每种语言生成的包装器代码中，通过使用特殊的类型描述符和转换函数来处理指针。例如，如果查看 Python 的包装器代码，你将看到类似于以下代码（为简洁起见简化）：

```c++
if (!SWIG_IsOK(SWIG_ConvertPtr(obj0, (void **) &arg1, SWIGTYPE_p_Foo, 0))) {
  SWIG_exception_fail(SWIG_TypeError, "in method 'GrabVal', expecting type Foo");
}
```

In this code, `SWIGTYPE_p_Foo` is the type descriptor that describes `Foo *`. The type descriptor is actually a pointer to a structure that contains information about the type name to use in the target language, a list of equivalent typenames (via typedef or inheritance), and pointer value handling information (if applicable). The `SWIG_ConvertPtr()` function is simply a utility function that takes a pointer object in the target language and a type-descriptor object and uses this information to generate a C++ pointer. The `SWIG_IsOK`macro checks the return value for errors and `SWIG_exception_fail` can be called to raise an exception in the target language. However, the exact name and calling conventions of the conversion function depends on the target language (see language specific chapters for details).

The actual type code is in swigrun.swg, and gets inserted near the top of the generated swig wrapper file. The phrase "a type X that can cast into a type Y" means that given a type X, it can be converted into a type Y. In other words, X is a derived class of Y or X is a typedef of Y. The structure to store type information looks like this:

> 在这段代码中，`SWIGTYPE_p_Foo` 是描述 `Foo *` 的类型描述符。类型描述符实际上是指向结构的指针，该结构包含有关要在目标语言中使用的类型名称的信息，等效类型名称的列表（通过 `typedef` 或继承）以及指针值处理信息（如果适用）。`SWIG_ConvertPtr()` 函数只是一个实用函数，它接受目标语言中的指针对象和类型描述符对象，并使用此信息生成 C++ 指针。`SWIG_IsOK` 宏检查错误的返回值，并且可以调用 `SWIG_exception_fail` 引发目标语言中的异常。但是，转换函数的确切名称和调用约定取决于目标语言（有关详细信息，请参见特定于语言的章节）。
>
> 实际的类型代码在 `swigrun.swg` 中，并插入到生成的 swig 包装文件顶部附近。短语“可以转换为 `Y` 类型的 `X` 类型”表示给定 `X` 类型，可以将其转换为 `Y` 类型。换句话说，`X` 是 `Y` 的派生类，或者 `X` 是 `Y` 的 `typedef`。存储类型信息的结构如下所示：

```c++
/* Structure to store information on one type */
typedef struct swig_type_info {
  const char *name;             /* mangled name of this type */
  const char *str;              /* human readable name for this type */
  swig_dycast_func dcast;       /* dynamic cast function down a hierarchy */
  struct swig_cast_info *cast;  /* Linked list of types that can cast into this type */
  void *clientdata;             /* Language specific type data */
} swig_type_info;

/* Structure to store a type and conversion function used for casting */
typedef struct swig_cast_info {
  swig_type_info *type;          /* pointer to type that is equivalent to this type */
  swig_converter_func converter; /* function to cast the void pointers */
  struct swig_cast_info *next;   /* pointer to next cast in linked list */
  struct swig_cast_info *prev;   /* pointer to the previous cast */
} swig_cast_info;
```

Each `swig_type_info` stores a linked list of types that it is equivalent to. Each entry in this doubly linked list stores a pointer back to another swig_type_info structure, along with a pointer to a conversion function. This conversion function is used to solve the above problem of the FooBar class, correctly returning a pointer to the type we want.

The basic problem we need to solve is verifying and building arguments passed to functions. So going back to the `SWIG_ConvertPtr()` function example from above, we are expecting a `Foo *` and need to check if `obj0` is in fact a `Foo *`. From before, `SWIGTYPE_p_Foo` is just a pointer to the `swig_type_info`structure describing `Foo *`. So we loop through the linked list of `swig_cast_info` structures attached to `SWIGTYPE_p_Foo`. If we see that the type of `obj0` is in the linked list, we pass the object through the associated conversion function and then return a positive. If we reach the end of the linked list without a match, then `obj0` can not be converted to a `Foo *` and an error is generated.

Another issue needing to be addressed is sharing type information between multiple modules. More explicitly, we need to have ONE `swig_type_info` for each type. If two modules both use the type, the second module loaded must lookup and use the swig_type_info structure from the module already loaded. Because no dynamic memory is used and the circular dependencies of the casting information, loading the type information is somewhat tricky, and not explained here. A complete description is in the `Lib/swiginit.swg` file (and near the top of any generated file).

Each module has one swig_module_info structure which looks like this:

> 每个 `swig_type_info` 都存储一个等效的类型的链表。这个双向链接列表中的每个条目都存储着一个指向另一个 `swig_type_info` 结构的指针，以及一个指向转换函数的指针。此转换函数用于解决 `FooBar` 类的上述问题，正确返回指向所需类型的指针。
>
> 我们需要解决的基本问题是验证和构建传递给函数的参数。因此，从上面回到 `SWIG_ConvertPtr()` 函数示例，我们期望的是 `Foo *`，并且需要检查 `obj0` 实际上是否为 `Foo *`。从前，`SWIGTYPE_p_Foo` 只是指向描述 `Foo *` 的 `swig_type_info` 结构的指针。因此，我们遍历附加到 `SWIGTYPE_p_Foo` 的 `swig_cast_info` 结构的链接列表。如果我们看到 `obj0` 的类型在链表中，则将对象传递给关联的转换函数，然后返回一个正数。如果我们到达链表的末尾但没有匹配项，则无法将 `obj0` 转换为 `Foo *` 并生成错误。
>
> 需要解决的另一个问题是在多个模块之间共享类型信息。更明确地说，我们需要为每种类型使用一个 `swig_type_info`。如果两个模块都使用该类型，则加载的第二个模块必须从已加载的模块中查找并使用 `swig_type_info` 结构。因为没有使用动态内存，而且转换信息的循环依赖关系，所以加载类型信息有些棘手，这里不再赘述。完整的描述在 `Lib/swiginit.swg` 文件中（并且在任何生成的文件的顶部附近）。
>
> 每个模块都有一个 `swig_module_info` 结构，如下所示：

```c++
/* Structure used to store module information
 * Each module generates one structure like this, and the runtime collects
 * all of these structures and stores them in a circularly linked list.*/
typedef struct swig_module_info {
  swig_type_info **types;         /* Array of pointers to swig_type_info structs in this module */
  int size;                       /* Number of types in this module */
  struct swig_module_info *next;  /* Pointer to next element in circularly linked list */
  swig_type_info **type_initial;  /* Array of initially generated type structures */
  swig_cast_info **cast_initial;  /* Array of initially generated casting structures */
  void *clientdata;               /* Language specific module data */
} swig_module_info;
```

Each module stores an array of pointers to `swig_type_info` structures and the number of types in this module. So when a second module is loaded, it finds the `swig_module_info` structure for the first module and searches the array of types. If any of its own types are in the first module and have already been loaded, it uses those `swig_type_info` structures rather than creating new ones. These `swig_module_info` structures are chained together in a circularly linked list.

> 每个模块存储一个指向 `swig_type_info` 结构体的指针数组以及该模块中类型的数量。因此，在加载第二个模块时，它将为第一个模块找到 `swig_module_info` 结构，并搜索类型数组。如果在第一个模块中有任何自己的类型并且已经被加载，则它使用那些 `swig_type_info` 结构体而不是创建新的结构。这些 `swig_module_info` 结构以循环链接列表的形式链接在一起。

### 11.12.2 使用

This section covers how to use these functions from typemaps. To learn how to call these functions from external files (not the generated _wrap.c file), see the [External access to the run-time system](http://swig.org/Doc3.0/Modules.html#Modules_external_run_time) section.

When pointers are converted in a typemap, the typemap code often looks similar to this:

> 本节介绍如何使用类型映射中的这些功能。要了解如何从外部文件（而不是生成的 `_wrap.c` 文件）中调用这些函数，请参见[对运行时系统的外部访问](http://swig.org/Doc3.0/Modules.html#Modules_external_run_time )章节。
>
> 在类型映射中转换指针时，类型映射代码通常看起来类似于以下内容：

```
%typemap(in) Foo * {
  if (!SWIG_IsOK(SWIG_ConvertPtr($input, (void **) &$1, $1_descriptor, 0))) {
    SWIG_exception_fail(SWIG_TypeError, "in method '$symname', expecting type Foo");
  }
}
```

The most critical part is the typemap is the use of the `$1_descriptor` special variable. When placed in a typemap, this is expanded into the `SWIGTYPE_*` type descriptor object above. As a general rule, you should always use `$1_descriptor` instead of trying to hard-code the type descriptor name directly.

There is another reason why you should always use the `$1_descriptor` variable. When this special variable is expanded, SWIG marks the corresponding type as "in use." When type-tables and type information is emitted in the wrapper file, descriptor information is only generated for those datatypes that were actually used in the interface. This greatly reduces the size of the type tables and improves efficiency.

Occasionally, you might need to write a typemap that needs to convert pointers of other types. To handle this, the special variable macro `$descriptor(type)` covered earlier can be used to generate the SWIG type descriptor name for any C datatype. For example:

> 最关键的部分是类型映射是 `$1_descriptor` 特殊变量的使用。当放置在类型映射中时，它会扩展到上面的 `SWIGTYPE_*` 类型描述符对象中。通常，应该始终使用 `$1_descriptor` 而不是尝试直接对类型描述符名称进行硬编码。
>
> 还有另一个原因，为什么你应该始终使用 `$1_descriptor` 变量。扩展此特殊变量后，SWIG 会将相应的类型标记为“使用中”。当在包装文件中发出类型表和类型信息时，仅为接口中实际使用的那些数据类型生成描述符信息。这大大减小了类型表的大小并提高了效率。
>
> 有时，你可能需要编写一个类型映射，该类型映射需要转换其他类型的指针。为了解决这个问题，前面介绍的特殊变量宏 `$descriptor(type)` 可用于为任何 C 数据类型生成 SWIG 类型描述符名称。例如：

```
%typemap(in) Foo * {
  if (!SWIG_IsOK(SWIG_ConvertPtr($input, (void **) &$1, $1_descriptor, 0))) {
    Bar *temp;
    if (!SWIG_IsOK(SWIG_ConvertPtr($input, (void **) &temp, $descriptor(Bar *), 0))) {
      SWIG_exception_fail(SWIG_TypeError, "in method '$symname', expecting type Foo or Bar");
    }
    $1 = (Foo *)temp;
  }
}
```

The primary use of `$descriptor(type)` is when writing typemaps for container objects and other complex data structures. There are some restrictions on the argument---namely it must be a fully defined C datatype. It can not be any of the special typemap variables.

In certain cases, SWIG may not generate type-descriptors like you expect. For example, if you are converting pointers in some non-standard way or working with an unusual combination of interface files and modules, you may find that SWIG omits information for a specific type descriptor. To fix this, you may need to use the `%types` directive. For example:

> `$descriptor(type)` 的主要用途是为容器对象和其他复杂数据结构编写类型映射时。参数有一些限制——即它必须是完全定义的 C 数据类型。它不能是任何特殊的类型映射变量。
>
> 在某些情况下，SWIG 可能不会生成你期望的类型描述符。例如，如果你以某种非标准的方式转换指针或使用接口文件和模块的异常组合，则可能会发现SWIG忽略了特定类型描述符的信息。为了解决这个问题，你可能需要使用 `%types` 指令。例如：

```
%types(int *, short *, long *, float *, double *);
```

When `%types` is used, SWIG generates type-descriptor information even if those datatypes never appear elsewhere in the interface file.

Further details about the run-time type checking can be found in the documentation for individual language modules. Reading the source code may also help. The file `Lib/swigrun.swg` in the SWIG library contains all of the source of the generated code for type-checking. This code is also included in every generated wrapped file so you probably just look at the output of SWIG to get a better sense for how types are managed.

> 当使用 `%types`时，SWIG 会生成类型描述符信息，即使这些数据类型从不出现在接口文件的其他位置。
>
> 有关运行时类型检查的更多详细信息，请参见各个语言模块的文档。阅读源代码也可能会有所帮助。SWIG 库中的 `Lib/swigrun.swg` 文件包含用于类型检查的生成代码的所有源。该代码也包含在每个生成的包装文件中，因此你可能只需查看 SWIG 的输出即可更好地了解如何管理类型。

## 11.13 类型映射与重载

This section does not apply to the statically typed languages like Java and C#, where overloading of the types is handled much like C++ by generating overloaded methods in the target language. In many of the other target languages, SWIG still fully supports C++ overloaded methods and functions. For example, if you have a collection of functions like this:

> 本章节不适用于 Java 和 C# 等静态类型的语言，在这些类型中，类型的重载与 C++ 一样，是通过在目标语言中生成重载的方法来处理的。在许多其他目标语言中，SWIG 仍完全支持 C++ 重载方法和函数。例如，如果你具有以下功能集合：

```c++
int foo(int x);
int foo(double x);
int foo(char *s, int y);
```

You can access the functions in a normal way from the scripting interpreter:

> 你可以从脚本解释器以常规方式访问函数：

```
# Python
foo(3)           # foo(int)
foo(3.5)         # foo(double)
foo("hello", 5)  # foo(char *, int)

# Tcl
foo 3            # foo(int)
foo 3.5          # foo(double)
foo hello 5      # foo(char *, int)
```

To implement overloading, SWIG generates a separate wrapper function for each overloaded method. For example, the above functions would produce something roughly like this:

> 为了实现重载，SWIG 为每个重载方法生成一个单独的包装器函数。例如，以上函数将产生大致如下所示的内容：

```c++
// wrapper pseudocode
_wrap_foo_0(argc, args[]) {       // foo(int)
  int arg1;
  int result;
  ...
  arg1 = FromInteger(args[0]);
  result = foo(arg1);
  return ToInteger(result);
}

_wrap_foo_1(argc, args[]) {       // foo(double)
  double arg1;
  int result;
  ...
  arg1 = FromDouble(args[0]);
  result = foo(arg1);
  return ToInteger(result);
}

_wrap_foo_2(argc, args[]) {       // foo(char *, int)
  char *arg1;
  int   arg2;
  int result;
  ...
  arg1 = FromString(args[0]);
  arg2 = FromInteger(args[1]);
  result = foo(arg1, arg2);
  return ToInteger(result);
}
```

Next, a dynamic dispatch function is generated:

> 接着生成动态调度函数：

```c++
_wrap_foo(argc, args[]) {
  if (argc == 1) {
    if (IsInteger(args[0])) {
      return _wrap_foo_0(argc, args);
    }
    if (IsDouble(args[0])) {
      return _wrap_foo_1(argc, args);
    }
  }
  if (argc == 2) {
    if (IsString(args[0]) && IsInteger(args[1])) {
      return _wrap_foo_2(argc, args);
    }
  }
  error("No matching function!\n");
}
```

The purpose of the dynamic dispatch function is to select the appropriate C++ function based on argument types---a task that must be performed at runtime in most of SWIG's target languages.

The generation of the dynamic dispatch function is a relatively tricky affair. Not only must input typemaps be taken into account (these typemaps can radically change the types of arguments accepted), but overloaded methods must also be sorted and checked in a very specific order to resolve potential ambiguity. A high-level overview of this ranking process is found in the "[SWIG and C++](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus)" chapter. What isn't mentioned in that chapter is the mechanism by which it is implemented---as a collection of typemaps.

To support dynamic dispatch, SWIG first defines a general purpose type hierarchy as follows:

> 动态调度函数的目的是根据参数类型选择适当的 C++ 函数，这是大多数 SWIG 目标语言都必须在运行时执行的任务。
>
> 动态调度函数的生成是一个比较棘手的事情。不仅必须考虑输入类型映射（这些类型映射可以从根本上改变接受的参数的类型），而且还必须以非常特定的顺序对重载方法进行排序和检查，以解决潜在的歧义。[SWIG 和 C++ ](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus)一章中提供了有关此排名过程的高级概述。在这一章中没有提到的是实现它的机制——作为类型映射的集合。
>
> 为了支持动态调度，SWIG 首先定义通用类型层次结构，如下所示：

```
Symbolic Name                   Precedence Value
------------------------------  ------------------
SWIG_TYPECHECK_POINTER           0
SWIG_TYPECHECK_VOIDPTR           10
SWIG_TYPECHECK_BOOL              15
SWIG_TYPECHECK_UINT8             20
SWIG_TYPECHECK_INT8              25
SWIG_TYPECHECK_UINT16            30
SWIG_TYPECHECK_INT16             35
SWIG_TYPECHECK_UINT32            40
SWIG_TYPECHECK_INT32             45
SWIG_TYPECHECK_UINT64            50
SWIG_TYPECHECK_INT64             55
SWIG_TYPECHECK_UINT128           60
SWIG_TYPECHECK_INT128            65
SWIG_TYPECHECK_INTEGER           70
SWIG_TYPECHECK_FLOAT             80
SWIG_TYPECHECK_DOUBLE            90
SWIG_TYPECHECK_COMPLEX           100
SWIG_TYPECHECK_UNICHAR           110
SWIG_TYPECHECK_UNISTRING         120
SWIG_TYPECHECK_CHAR              130
SWIG_TYPECHECK_STRING            140
SWIG_TYPECHECK_BOOL_ARRAY        1015
SWIG_TYPECHECK_INT8_ARRAY        1025
SWIG_TYPECHECK_INT16_ARRAY       1035
SWIG_TYPECHECK_INT32_ARRAY       1045
SWIG_TYPECHECK_INT64_ARRAY       1055
SWIG_TYPECHECK_INT128_ARRAY      1065
SWIG_TYPECHECK_FLOAT_ARRAY       1080
SWIG_TYPECHECK_DOUBLE_ARRAY      1090
SWIG_TYPECHECK_CHAR_ARRAY        1130
SWIG_TYPECHECK_STRING_ARRAY      1140
```

(These precedence levels are defined in `swig.swg`, a library file that's included by all target language modules.)

In this table, the precedence-level determines the order in which types are going to be checked. Low values are always checked before higher values. For example, integers are checked before floats, single values are checked before arrays, and so forth.

Using the above table as a guide, each target language defines a collection of "typecheck" typemaps. The following excerpt from the Python module illustrates this:

> （这些优先级在 `swig.swg` 中定义，`swig.swg` 是所有目标语言模块都包含的库文件。）
>
> 在此表中，优先级确定要检查的类型的顺序。始终先检查低值，然后再检查高值。例如，在浮点数之前检查整数，在数组之前检查单个值，依此类推。
>
> 使用上表作为指导，每种目标语言都定义了 `typecheck` 类型映射的集合。以下 Python 模块摘录说明了这一点：

```
/* Python type checking rules */
/* Note:  %typecheck(X) is a macro for %typemap(typecheck, precedence=X) */

%typecheck(SWIG_TYPECHECK_INTEGER)
  int, short, long,
  unsigned int, unsigned short, unsigned long,
  signed char, unsigned char,
  long long, unsigned long long,
  const int &, const short &, const long &,
  const unsigned int &, const unsigned short &, const unsigned long &,
  const long long &, const unsigned long long &,
  enum SWIGTYPE,
  bool, const bool &
{
  $1 = (PyInt_Check($input) || PyLong_Check($input)) ? 1 : 0;
}

%typecheck(SWIG_TYPECHECK_DOUBLE)
  float, double,
  const float &, const double &
{
  $1 = (PyFloat_Check($input) || PyInt_Check($input) || PyLong_Check($input)) ? 1 : 0;
}

%typecheck(SWIG_TYPECHECK_CHAR) char {
  $1 = (PyString_Check($input) && (PyString_Size($input) == 1)) ? 1 : 0;
}

%typecheck(SWIG_TYPECHECK_STRING) char * {
  $1 = PyString_Check($input) ? 1 : 0;
}

%typemap(typecheck, precedence=SWIG_TYPECHECK_POINTER, noblock=1) SWIGTYPE * {
  void *vptr = 0;
  int res = SWIG_ConvertPtr($input, &vptr, $1_descriptor, 0);
  $1 = SWIG_IsOK(res) ? 1 : 0;
}

%typecheck(SWIG_TYPECHECK_POINTER) PyObject * {
  $1 = ($input != 0);
}
```

It might take a bit of contemplation, but this code has merely organized all of the basic C++ types, provided some simple type-checking code, and assigned each type a precedence value.

Finally, to generate the dynamic dispatch function, SWIG uses the following algorithm:

* Overloaded methods are first sorted by the number of required arguments.
* Methods with the same number of arguments are then sorted by precedence values of argument types.
* Typecheck typemaps are then emitted to produce a dispatch function that checks arguments in the correct order.

If you haven't written any typemaps of your own, it is unnecessary to worry about the typechecking rules. However, if you have written new input typemaps, you might have to supply a typechecking rule as well. An easy way to do this is to simply copy one of the existing typechecking rules. Here is an example,

> 这可能需要一些考虑，但是此代码仅组织了所有基本 C++ 类型，提供了一些简单的类型检查代码，并为每种类型分配了优先级值。
>
> 最后，为了生成动态调度功能，SWIG 使用以下算法：
>
> * 重载的方法首先按所需参数的数量排序。
> * 然后，将具有相同数量参数的方法按参数类型的优先级值排序。
> * 然后发出Typecheck类型映射，以产生一个调度函数，该函数以正确的顺序检查参数。

> 如果你尚未编写任何类型映射，则不必担心类型检查规则。但是，如果你编写了新的输入类型映射，则可能还必须提供类型检查规则。一种简单的方法是简单地复制现有的类型检查规则之一。这是一个例子

```
// Typemap for a C++ string
%typemap(in) std::string {
  if (PyString_Check($input)) {
    $1 = std::string(PyString_AsString($input));
  } else {
    SWIG_exception(SWIG_TypeError, "string expected");
  }
}
// Copy the typecheck code for "char *".
%typemap(typecheck) std::string = char *;
```

The bottom line: If you are writing new typemaps and you are using overloaded methods, you will probably have to write new typecheck code or copy and modify existing typecheck code.

If you write a typecheck typemap and omit the precedence level, for example commenting it out as shown below:

> 底线：如果你正在编写新的类型映射，并且使用的是重载方法，则可能必须编写新的类型检查代码或复制和修改现有的类型检查代码。
>
> 如果编写类型检查类型映射并忽略优先级，例如将其注释掉，如下所示：

```
%typemap(typecheck /*, precedence=SWIG_TYPECHECK_INTEGER*/) int {
  $1 = PyInt_Check($input) ? 1 : 0;
}
```

then the type is given a precedence higher than any other known precedence level and a [warning](http://swig.org/Doc3.0/Warnings.html#Warnings) is issued:

> 然后为该类型赋予比其他任何已知优先级高的优先级，并发出[警告](http://swig.org/Doc3.0/Warnings.html#Warnings)：

```
example.i:18: Warning 467: Overloaded method foo(int) not supported (incomplete type checking rule - no precedence level in typecheck typemap for 'int').
```

**Notes:**

* Typecheck typemaps are not used for non-overloaded methods. Because of this, it is still always necessary to check types in any "in" typemaps.
* The dynamic dispatch process is only meant to be a heuristic. There are many corner cases where SWIG simply can't disambiguate types to the same degree as C++. The only way to resolve this ambiguity is to use the %rename directive to rename one of the overloaded methods (effectively eliminating overloading).
* Typechecking may be partial. For example, if working with arrays, the typecheck code might simply check the type of the first array element and use that to dispatch to the correct function. Subsequent "in" typemaps would then perform more extensive type-checking.
* Make sure you read the section on overloading in the "[SWIG and C++](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus)" chapter.

> **注意：**
>
> * Typecheck类型映射不适用于非重载方法。因此，仍然始终需要检查任何 `in` 类型映射中的类型。
* 动态调度过程仅是一种启发式方法。在许多特殊情况下，SWIG 不能完全消除类型与 C++ 相同的歧义。解决此歧义的唯一方法是使用 `%rename` 指令重命名其中一种重载方法（有效消除重载）。
* 类型检查可能是部分的。例如，如果使用数组，则类型检查代码可以简单地检查第一个数组元素的类型，然后使用它来分派给正确的函数。随后的 `in` 类型映射将执行更广泛的类型检查。
* 确保你已阅读 [SWIG 和 C++](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus) 一章中有关重载的部分。

## 11.14 `%apply` 和 `%clear` 详情

In order to implement certain kinds of program behavior, it is sometimes necessary to write sets of typemaps. For example, to support output arguments, one often writes a set of typemaps like this:

> 为了实现某些类型的程序行为，有时有必要编写类型映射集。例如，为了支持输出参数，通常会编写这样的一组类型映射：

```
%typemap(in, numinputs=0) int *OUTPUT (int temp) {
  $1 = &temp;
}
%typemap(argout) int *OUTPUT {
  // return value somehow
}
```

To make it easier to apply the typemap to different argument types and names, the `%apply` directive performs a copy of all typemaps from one type to another. For example, if you specify this,

> 为了更容易地将类型映射应用于不同的参数类型和名称，`%apply` 指令将所有类型映射从一种类型复制到另一种类型。例如，如果你指定此选项，

```
%apply int *OUTPUT { int *retvalue, int32 *output };
```

then all of the `int *OUTPUT` typemaps are copied to `int *retvalue` and `int32 *output`.

However, there is a subtle aspect of `%apply` that needs more description. Namely, `%apply` does not overwrite a typemap rule if it is already defined for the target datatype. This behavior allows you to do two things:

* You can specialize parts of a complex typemap rule by first defining a few typemaps and then using `%apply` to incorporate the remaining pieces.
* Sets of different typemaps can be applied to the same datatype using repeated `%apply` directives.

For example:

> 然后将所有 `int *OUTPUT` 类型映射复制到 `int *retvalue` 和 `int32 * output`。
>
> 但是，`%apply` 有一个细微的方面需要更多描述。就是说，如果此行为使你可以做两件事：
>
> * 你可以通过首先定义一些类型映射，然后使用 `%apply` 来合并其余部分来专门化复杂类型映射规则的各个部分。
> * 可以使用重复的 `%apply` 指令将不同类型映射的集合应用于相同的数据类型。
>
> 例如：

```
%typemap(in) int *INPUT (int temp) {
  temp = ... get value from $input ...;
  $1 = &temp;
}

%typemap(check) int *POSITIVE {
  if (*$1 <= 0) {
    SWIG_exception(SWIG_ValueError, "Expected a positive number!\n");
    return NULL;
  }
}

...
%apply int *INPUT     { int *invalue };
%apply int *POSITIVE  { int *invalue };
```

Since `%apply` does not overwrite or replace any existing rules, the only way to reset behavior is to use the `%clear` directive. `%clear` removes all typemap rules defined for a specific datatype. For example:

> 由于 `%apply` 不会覆盖或替换任何现有规则，因此重置行为的唯一方法是使用 `%clear` 伪指令。`%clear` 删除为特定数据类型定义的所有类型映射规则。例如：

```
%clear int *invalue;
```

## 11.15 在类型映射间传递数据

It is also important to note that the primary use of local variables is to create stack-allocated objects for temporary use inside a wrapper function (this is faster and less-prone to error than allocating data on the heap). In general, the variables are not intended to pass information between different types of typemaps. However, this can be done if you realize that local names have the argument number appended to them. For example, you could do this:

> 同样重要的是要注意，局部变量的主要用途是创建包装分配的对象，以便在包装器函数内部临时使用（与在堆上分配数据相比，此方法更快且更不容易出错）。通常，这些变量无意在不同类型的类型映射之间传递信息。但是，如果你意识到本地名称后面附加了参数编号，则可以这样做。例如，你可以这样做：

```
%typemap(in) int *(int temp) {
  temp = (int) PyInt_AsLong($input);
  $1 = &temp;
}

%typemap(argout) int * {
  PyObject *o = PyInt_FromLong(temp$argnum);
  ...
}
```

In this case, the `$argnum` variable is expanded into the argument number. Therefore, the code will reference the appropriate local such as `temp1` and `temp2`. It should be noted that there are plenty of opportunities to break the universe here and that accessing locals in this manner should probably be avoided. At the very least, you should make sure that the typemaps sharing information have exactly the same types and names.

> 在这种情况下，`$argnum` 变量将扩展为参数编号。因此，代码将引用适当的本地变量，例如 `temp1` 和 `temp2`。应当指出，这里有很多打破宇宙的机会，应该避免以这种方式访问本地人。至少，你应该确保共享信息的类型映射具有完全相同的类型和名称。

## 11.16 C++ `this` 指针

All the rules discussed for typemaps apply to C++ as well as C. However in addition C++ passes an extra parameter into every non-static class method -- the `this` pointer. Occasionally it can be useful to apply a typemap to this pointer (for example to check and make sure `this` is non-null before deferencing). Actually, C also has an the equivalent of the `this` pointer which is used when accessing variables in a C struct.

In order to customise the `this` pointer handling, target a variable named `self` in your typemaps. `self` is the name SWIG uses to refer to the extra parameter in wrapped functions.

For example, if wrapping for Java generation:

> 讨论类型映射的所有规则都适用于 C++ 和 C。但是，此外，C++ 向每个非静态类方法（`this` 指针）传递了一个额外的参数。有时候，将类型映射应用于此指针可能会很有用（例如，检查并确保在递延前确保 `this` 为非空）。实际上，C 还具有等效于 `this` 指针的指针，该指针在访问 C 结构体中的变量时使用。
>
> 为了自定义 `this` 指针处理，在类型映射中定位一个名为 `self` 的变量。`self` 是 SWIG 在包装器函数中用来引用附加参数的名称。
>
> 例如，如果包装为生成 Java：

```
%typemap(check) SWIGTYPE *self %{
if (!$1) {
  SWIG_JavaThrowException(jenv, SWIG_JavaNullPointerException,
    "invalid native object; delete() likely already called");
  return $null;
}
%}
```

In the above case, the `$1` variable is expanded into the argument name that SWIG is using as the `this`pointer. SWIG will then insert the check code before the actual C++ class method is called, and will raise an exception rather than crash the Java virtual machine. The generated code will look something like:

> 在上述情况下，将 `$1` 变量扩展为 SWIG 用作 `this` 指针的参数名称。然后，SWIG 将在调用实际的 C++ 类方法之前插入检查代码，并且将引发异常而不是使 Java 虚拟机崩溃。生成的代码如下所示：

```c++
if (!arg1) {
SWIG_JavaThrowException(jenv, SWIG_JavaNullPointerException,
  "invalid native object; delete() likely already called");
return ;
}
(arg1)->wrappedFunction(...);
```

Note that if you have a parameter named `self` then it will also match the typemap. One work around is to create an interface file that wraps the method, but gives the argument a name other than `self`.

> 请注意，如果你有一个名为 `self` 的参数，则它也将与类型映射匹配。一种解决方法是创建一个包装该方法的接口文件，但为自变量指定一个不同于 `self` 的名称。

## 11.17 到哪去找更多的信息？

The best place to find out more information about writing typemaps is to look in the SWIG library. Most language modules define all of their default behavior using typemaps. These are found in files such as`python.swg`, `perl5.swg`, `tcl8.swg` and so forth. The `typemaps.i` file in the library also contains numerous examples. You should look at these files to get a feel for how to define typemaps of your own. Some of the language modules support additional typemaps and further information is available in the individual chapters for each target language. There you may also find more hands-on practical examples.

> 要找到有关编写类型映射的更多信息的最佳位置是在 SWIG 库中查找。大多数语言模块都使用类型映射定义所有默认行为。这些可以在诸如 `python.swg`、`perl5.swg`、`tcl8.swg` 等文件中找到。库中的 `typemaps.i` 文件也包含许多示例。你应该查看这些文件，以了解如何定义自己的类型映射。一些语言模块支持其他类型映射，并且在每种章节的每种目标语言中都提供了更多信息。在这里你还可以找到更多动手的实际示例。
