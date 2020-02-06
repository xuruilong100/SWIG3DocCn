[TOC]

# 9 SWIG 库

To help build extension modules, SWIG is packaged with a library of support files that you can include in your own interfaces. These files often define new SWIG directives or provide utility functions that can be used to access parts of the standard C and C++ libraries. This chapter provides a reference to the current set of supported library files.

**Compatibility note:** Older versions of SWIG included a number of library files for manipulating pointers, arrays, and other structures. Most these files are now deprecated and have been removed from the distribution. Alternative libraries provide similar functionality. Please read this chapter carefully if you used the old libraries.

> 为了帮助构建扩展模块，SWIG 附带了支持文件库，你可以在自己的接口中包含这些支持文件。这些文件通常定义新的 SWIG 指令或提供实用函数，这些函数可用于访问标准 C 和 C++ 库的一部分。本章提供对当前支持的库文件集合的参考。
>
> **注意兼容性**：较早版本的 SWIG 包括许多用于处理指针、数组和其他结构体的库文件。现在大多数这些文件已被弃用，并已从发行版中删除。备用库提供类似的功能。如果你使用的是旧库，请仔细阅读本章。

## 9.1 `%include` 指令与库搜索路径

Library files are included using the `%include` directive. When searching for files, directories are searched in the following order:

1. The current directory
2. Directories specified with the `-I` command line option
3. `./swig_lib`
4. SWIG library install location as reported by `swig -swiglib`, for example `/usr/local/share/swig/1.3.30`
5. On Windows, a directory `Lib` relative to the location of `swig.exe` is also searched.

Within directories mentioned in points 3-5, SWIG first looks for a subdirectory corresponding to a target language (e.g., `python`, `tcl`, etc.). If found, SWIG will search the language specific directory first. This allows for language-specific implementations of library files.

You can ignore the installed SWIG library by setting the `SWIG_LIB`environment variable. Set the environment variable to hold an alternative library directory.

The directories that are searched are displayed when using `-verbose` commandline option.

> 使用 `%include` 指令包含库文件。搜索文件时，将按以下顺序搜索目录：
>
> 1. 当前目录
> 2. 用 `-I` 命令行选项指定的目录
> 3. `./swig_lib`
> 4. SWIG 库的安装位置（由 `swig -swiglib` 报告），例如 `/usr/local/share/swig/1.3.30`
> 5. 在 Windows 上，还将搜索相对于 `swig.exe` 位置的目录 `Lib`。
>
> 在第 3-5 点提到的目录中，SWIG 首先查找与目标语言相对应的子目录（例如，`python`、`tcl` 等）。如果找到，SWIG 将首先搜索特定语言的目录。这允许特定语言的库文件实现。
>
> 你可以通过设置 `SWIG_LIB` 环境变量来忽略已安装的 SWIG 库。设置环境变量以保存备用库目录。
>
> 使用 `-verbose` 命令行选项时，将显示搜索到的目录。

## 9.2 C 数组与指针

This section describes library modules for manipulating low-level C arrays and pointers. The primary use of these modules is in supporting C declarations that manipulate bare pointers such as `int *`, `double *`, or `void *`. The modules can be used to allocate memory, manufacture pointers, dereference memory, and wrap pointers as class-like objects. Since these functions provide direct access to memory, their use is potentially unsafe and you should exercise caution.

> 本节描述用于处理低级 C 数组和指针的库模块。这些模块的主要用途是支持 C 声明，这些声明可操纵诸如 `int *`、`double *` 或 `void *` 之类的裸指针。这些模块可用于分配内存、创建指针、解引用内存以及将指针包装为类对象。由于这些函数可直接访问内存，因此使用它们可能不安全，因此应谨慎行事。

### 9.2.1 `cpointer.i`

The `cpointer.i` module defines macros that can be used to used to generate wrappers around simple C pointers. The primary use of this module is in generating pointers to primitive datatypes such as `int` and `double`.

> `cpointer.i` 模块定义了可用于生成简单 C 指针的包装器的宏。这个模块的主要用途是生成指向原始数据类型，例如 `int` 和 `double` 的指针。

**`%pointer_functions(type, name)`**

Generates a collection of four functions for manipulating a pointer `type *`:

> 生成四个用于操纵指针 `type *` 的函数的集合：

```c
type *new_name()
```

Creates a new object of type `type` and returns a pointer to it. In C, the object is created using `calloc()`. In C++, `new` is used.

> 创建一个类型为 `type` 的新对象，并返回一个指向它的指针。在 C 语言中，使用 `calloc()` 创建对象。在 C++ 中，使用 `new`。

```c
type *copy_name(type value)
```

Creates a new object of type `type` and returns a pointer to it. An initial value is set by copying it from `value`. In C, the object is created using `calloc()`. In C++, `new` is used.

> 创建一个类型为 `type` 的新对象，并返回一个指向它的指针。通过从 `value` 中复制初始值来设置初始值。在 C 语言中，使用 `calloc()` 创建对象。在 C++ 中，使用 `new`。

```c
type *delete_name(type *obj)
```

Deletes an object type `type`.

> 删除一个 `type` 类型的对象。

```c
void name_assign(type *obj, type value)
```

Assigns `*obj = value`.

> 赋值 `*obj = value`。

```c
type name_value(type *obj)
```

Returns the value of `*obj`.

When using this macro, `type` may be any type and `name`must be a legal identifier in the target language. `name`should not correspond to any other name used in the interface file.

Here is a simple example of using `%pointer_functions()`:

> 返回 `*obj` 的值。
>
> 使用此宏时，`type` 可以是任何类型，而 `name` 必须是目标语言中的合法标识符。名称不应该与接口文件中使用的任何其他名称相对应。
>
> 这是一个使用 `%pointer_functions()` 的简单示例：

```
%module example
%include "cpointer.i"

/* Create some functions for working with "int *" */
%pointer_functions(int, intp);

/* A function that uses an "int *" */
void add(int x, int y, int *result);
```

Now, in Python:

> 现在，在 Python 中：

```python
>>> import example
>>> c = example.new_intp()     # Create an "int" for storing result
>>> example.add(3, 4, c)       # Call function
>>> example.intp_value(c)      # Dereference
7
>>> example.delete_intp(c)     # Delete
```

**`%pointer_class(type, name)`**

Wraps a pointer of `type *` inside a class-based interface. This interface is as follows:

> 在基于类的接口内包装 `type *` 指针。该接口如下：

```c++
struct name {
  name();                            // Create pointer object
  ~name();                           // Delete pointer object
  void assign(type value);           // Assign value
  type value();                      // Get value
  type *cast();                      // Cast the pointer to original type
  static name *frompointer(type *);  // Create class wrapper from existing
                                     // pointer
};
```

When using this macro, `type` is restricted to a simple type name like `int`, `float`, or `Foo`. Pointers and other complicated types are not allowed. `name` must be a valid identifier not already in use. When a pointer is wrapped as a class, the "class" may be transparently passed to any function that expects the pointer.

If the target language does not support proxy classes, the use of this macro will produce the example same functions as `%pointer_functions()` macro.

It should be noted that the class interface does introduce a new object or wrap a pointer inside a special structure. Instead, the raw pointer is used directly.

Here is the same example using a class instead:

> 使用此宏时，`type` 仅限于一个简单的类型名称，例如 `int`、`float` 或 `Foo`。不允许使用指针和其他复杂类型。`name` 必须是尚未使用的有效标识符。当将指针包装为类时，类可以透明地传递给需要该指针的任何函数。
>
> 如果目标语言不支持代理类，则使用此宏将产生与 `%pointer_functions()` 宏相同的示例函数。
>
> 应当注意，类接口确实引入了新对象或将指针包装在特殊结构体内。相反，原始指针是直接使用的。
>
> 这是使用类的同一示例：

```
%module example
%include "cpointer.i"

/* Wrap a class interface around an "int *" */
%pointer_class(int, intp);

/* A function that uses an "int *" */
void add(int x, int y, int *result);
```

Now, in Python (using proxy classes)

> 现在，在 Python 中（使用代理类）

```python
>>> import example
>>> c = example.intp()         # Create an "int" for storing result
>>> example.add(3, 4, c)       # Call function
>>> c.value()                  # Dereference
7
```

Of the two macros, `%pointer_class` is probably the most convenient when working with simple pointers. This is because the pointers are access like objects and they can be easily garbage collected (destruction of the pointer object destroys the underlying object).

在两个宏中，`%pointer_class` 可能是使用简单指针时最方便的方法。这是因为指针可以像对象一样被访问，并且可以很容易地对其进行垃圾回收（指针对象的破坏会破坏基础对象）。

**`%pointer_cast(type1, type2, name)`**

Creates a casting function that converts `type1` to `type2`. The name of the function is `name`. For example:

> 创建一个转换函数，将 `type1` 转换为 `type2`。函数的名称是 `name`。例如：

```
%pointer_cast(int *, unsigned int *, int_to_uint);
```

In this example, the function `int_to_uint()` would be used to cast types in the target language.

**Note:** None of these macros can be used to safely work with strings (`char *` or `char **`).

**Note:** When working with simple pointers, typemaps can often be used to provide more seamless operation.

> 在此示例中，函数 `int_to_uint()` 将用于在目标语言中转换类型。
>
> **注意**：这些宏均不能安全地处理字符串（`char *` 或 `char **`）。
>
> **注意**：当使用简单的指针时，类型映射通常可以用来提供更无缝的操作。

### 9.2.2 `carrays.i`

This module defines macros that assist in wrapping ordinary C pointers as arrays. The module does not provide any safety or an extra layer of wrapping--it merely provides functionality for creating, destroying, and modifying the contents of raw C array data.

> 该模块定义了有助于将普通 C 指针包装为数组的宏。该模块不提供任何安全性或额外的包装层——它仅提供用于创建、销毁和修改原始 C 数组数据内容的功能。

**`%array_functions(type, name)`**

Creates four functions.

> 创建四个函数。

```c
type *new_name(int nelements)
```

Creates a new array of objects of type `type`. In C, the array is allocated using`calloc()`. In C++, `new []` is used.

> 创建一个类型为 `type` 的对象的数组。在 C 语言中，使用 `calloc()` 分配数组。在 C++ 中，使用 `new []`。

```c
type *delete_name(type *ary)
```

Deletes an array. In C, `free()` is used. In C++, `delete []` is used.

> 删除数组。在 C 中使用 `free()`。在 C++ 中使用 `delete []`

```c
type name_getitem(type *ary, int index)
```

Returns the value `ary[index]`.

> 返回值 `ary[index]`。

```c
void name_setitem(type *ary, int index, type value)
```

Assigns `ary[index] = value`.

When using this macro, `type` may be any type and `name` must be a legal identifier in the target language. `name` should not correspond to any other name used in the interface file.

Here is an example of `%array_functions()`. Suppose you had a function like this:

> 赋值 `ary[index] = value`。
>
> 使用此宏时，`type` 可以是任何类型，而 `name` 必须是目标语言中的合法标识符。名称不应该与接口文件中使用的任何其他名称相对应。
>
> 这是 `%array_functions()` 的示例。假设你具有如下函数：

```c
void print_array(double x[10]) {
  int i;
  for (i = 0; i < 10; i++) {
    printf("[%d] = %g\n", i, x[i]);
  }
}
```

To wrap it, you might write this:

> 要包装它，你可以这样写：

```
%module example

%include "carrays.i"
%array_functions(double, doubleArray);

void print_array(double x[10]);
```

Now, in a scripting language, you might write this:

> 现在，在脚本语言中，你可以这样写：

```python
a = new_doubleArray(10)             # Create an array
for i in range(0, 10):
    doubleArray_setitem(a, i, 2*i)  # Set a value
print_array(a)                      # Pass to C
delete_doubleArray(a)               # Destroy array
```

**`%array_class(type, name)`**

Wraps a pointer of `type *` inside a class-based interface. This interface is as follows:

> 在基于类的接口内包装 `type *` 指针。该接口如下：

```c++
struct name {
  name(int nelements);                  // Create an array
  ~name();                              // Delete array
  type getitem(int index);              // Return item
  void setitem(int index, type value);  // Set item
  type *cast();                         // Cast to original type
  static name *frompointer(type *);     // Create class wrapper from
                                        // existing pointer
};
```

When using this macro, `type` is restricted to a simple type name like `int` or `float`. Pointers and other complicated types are not allowed. `name` must be a valid identifier not already in use. When a pointer is wrapped as a class, it can be transparently passed to any function that expects the pointer.

When combined with proxy classes, the `%array_class()` macro can be especially useful. For example:

> 使用此宏时，`type` 仅限于一个简单的类型名称，例如 `int` 或 `float`。不允许使用指针和其他复杂类型。名称必须是尚未使用的有效标识符。将指针包装为类时，可以将其透明地传递给任何需要该指针的函数。
>
> 当与代理类结合使用时，`%array_class()` 宏可能会特别有用。例如：

```
%module example
%include "carrays.i"
%array_class(double, doubleArray);

void print_array(double x[10]);
```

Allows you to do this:

> 允许你这样做：

```python
import example
c = example.doubleArray(10)  # Create double[10]
for i in range(0, 10):
    c[i] = 2*i               # Assign values
example.print_array(c)       # Pass to C
```

**Note:** These macros do not encapsulate C arrays inside a special data structure or proxy. There is no bounds checking or safety of any kind. If you want this, you should consider using a special array object rather than a bare pointer.

**Note:** `%array_functions()` and `%array_class()` should not be used with types of `char` or `char *`.

> **注意**：这些宏不会将 C 数组封装在特殊的数据结构或代理中。没有边界检查或任何形式的安全性。如果需要，应该考虑使用特殊的数组对象而不是裸指针。
>
> **注意**：`%array_functions()` 和 `%array_class()` 不能和类型 `char` 或 `char *` 共同使用。

### 9.2.3 `cmalloc.i`

This module defines macros for wrapping the low-level C memory allocation functions `malloc()`, `calloc()`, `realloc()`, and `free()`.

> 该模块定义了用于包装低级 C 内存分配函数 `malloc()`、`calloc()`、`realloc()` 和 `free()` 的宏。

**`%malloc(type [, name=type])`**

Creates a wrapper around `malloc()` with the following prototype:

> 创建以下原型的 `malloc()` 包装器：

```c
type *malloc_name(int nbytes = sizeof(type));
```

If `type` is `void`, then the size parameter `nbytes` is required. The `name` parameter only needs to be specified when wrapping a type that is not a valid identifier (e.g., "`int *`", "`double **`", etc.).

> 如果 `type` 是 `void`，则需要 `size` 参数 `nbytes`。仅在包装不是有效标识符的类型时（例如，`int *`、`double **` 等），才需要指定 `name` 参数。

**`%calloc(type [, name=type])`**

Creates a wrapper around `calloc()` with the following prototype:

> 创建以下原型的 `calloc()` 包装器：

```c
type *calloc_name(int nobj =1, int sz = sizeof(type));
```

If `type` is `void`, then the size parameter `sz` is required.

**`%realloc(type [, name=type])`**

Creates a wrapper around `realloc()` with the following prototype:

> 创建以下原型的 `realloc()` 包装器：

```c
type *realloc_name(type *ptr, int nitems);
```

Note: unlike the C `realloc()`, the wrapper generated by this macro implicitly includes the size of the corresponding type. For example, `realloc_int(p, 100)` reallocates `p` so that it holds 100 integers.

> **注意**：与 C 中的 `realloc()` 不同，此宏生成的包装器隐式包含相应类型的大小。例如，`realloc_int(p, 100)` 重新分配 `p`，使其包含 100 个整数。

**`%free(type [, name=type])`**

Creates a wrapper around `free()` with the following prototype:

> 创建以下原型的 `free()` 包装器：

```c
void free_name(type *ptr);
```

**`%sizeof(type [, name=type])`**

Creates the constant:

> 创建常量：

```
%constant int sizeof_name = sizeof(type);
```

**%allocators(type [, name=type])**

Generates wrappers for all five of the above operations.

Here is a simple example that illustrates the use of these macros:

> 为上述所有五个操作生成包装器。
>
> 这里有一个简单的示例，说明了这些宏的用法：

```
// SWIG interface
%module example
%include "cmalloc.i"

%malloc(int);
%free(int);

%malloc(int *, intp);
%free(int *, intp);

%allocators(double);
```

Now, in a script:

> 现在，在脚本中：

```python
>>> from example import *
>>> a = malloc_int()
>>> a
'_000efa70_p_int'
>>> free_int(a)
>>> b = malloc_intp()
>>> b
'_000efb20_p_p_int'
>>> free_intp(b)
>>> c = calloc_double(50)
>>> c
'_000fab98_p_double'
>>> c = realloc_double(100000)
>>> free_double(c)
>>> print sizeof_double
8
>>>
```

### 9.2.4 `cdata.i`

The `cdata.i` module defines functions for converting raw C data to and from strings in the target language. The primary applications of this module would be packing/unpacking of binary data structures---for instance, if you needed to extract data from a buffer. The target language must support strings with embedded binary data in order for this to work.

> `cdata.i` 模块定义了用于将原始 C 数据与目标语言的字符串进行相互转换的函数。该模块的主要应用是打包、解包二进制数据结构。例如，如果你需要从缓冲区提取数据。目标语言必须支持带有嵌入式二进制数据的字符串，这样才能起作用。

**`const char *cdata(void *ptr, size_t nbytes)`**

Converts `nbytes` of data at `ptr` into a string. `ptr` can be any pointer.

> 将 `ptr` 中 `nbytes` 大小的数据转换成字符串。`ptr` 可以是任何指针。

**`void memmove(void *ptr, const char *s)`**

Copies all of the string data in `s` into the memory pointed to by `ptr`. The string may contain embedded NULL bytes. This is actually a wrapper to the standard C library `memmove` function, which is declared as **`void memmove(void *ptr, const void *src, size_t n)`**. The `src` and length `n` parameters are extracted from the language specific string `s` in the underlying wrapper code.

One use of these functions is packing and unpacking data from memory. Here is a short example:

> 将 `s` 中的所有字符串数据复制到 `ptr` 指向的内存中。该字符串可能包含嵌入的 NULL 字节。这实际上是标准 C 库 `memmove` 函数的包装，该函数被声明为 **`void memmove(void *ptr，const void *src，size_t n)`**。`src` 和 `length` 参数是从基础包装代码中特定于语言的字符串 `s` 中提取的。
>
> 这些函数的一种用途是从内存打包和拆包数据。这是一个简短的示例：

```
// SWIG interface
%module example
%include "carrays.i"
%include "cdata.i"

%array_class(int, intArray);
```

Python example:

> Python 示例：

```python
>>> a = intArray(10)
>>> for i in range(0, 10):
...    a[i] = i
>>> b = cdata(a, 40)
>>> b
'\x00\x00\x00\x00\x00\x00\x00\x01\x00\x00\x00\x02\x00\x00\x00\x03\x00\x00\x00\x04
\x00\x00\x00\x05\x00\x00\x00\x06\x00\x00\x00\x07\x00\x00\x00\x08\x00\x00\x00\t'
>>> c = intArray(10)
>>> memmove(c, b)
>>> print c[4]
4
>>>
```

Since the size of data is not always known, the following macro is also defined:

> 由于并不总是知道数据大小，因此还定义了以下宏：

**`%cdata(type [, name=type])`**

Generates the following function for extracting C data for a given type.

> 生成以下函数以提取给定类型的 C 数据。

```c
char *cdata_name(type* ptr, int nitems)
```

`nitems` is the number of items of the given type to extract.

**Note:** These functions provide direct access to memory and can be used to overwrite data. Clearly they are unsafe.

> `nitems` 是要提取的给定类型的项目数。
>
> **注意**：这些函数提供对内存的直接访问，并可用于覆盖数据。显然，它们是不安全的。

## 9.3 C 字符串处理

A common problem when working with C programs is dealing with functions that manipulate raw character data using `char *`. In part, problems arise because there are different interpretations of`char *`---it could be a NULL-terminated string or it could point to binary data. Moreover, functions that manipulate raw strings may mutate data, perform implicit memory allocations, or utilize fixed-sized buffers.

The problems (and perils) of using `char *` are well-known. However, SWIG is not in the business of enforcing morality. The modules in this section provide basic functionality for manipulating raw C strings.

> 使用 C 程序时，一个常见的问题是使用 `char *` 处理原始字符数据的函数。部分地出现问题是因为对 `char *` 有不同的解释——它可以是以 NULL 结尾的字符串，也可以指向二进制数据。此外，操纵原始字符串的函数可能会改变数据，执行隐式内存分配或利用固定大小的缓冲区。
>
> 使用 `char *` 的问题（和危险）是众所周知的。但是，SWIG 并不涉及道德操守。本节中的模块提供用于处理原始 C 字符串的基本函数。

### 9.3.1 默认字符串处理

Suppose you have a C function with this prototype:

> 假设你有一个带有此原型的 C 函数：

```c
char *foo(char *s);
```

The default wrapping behavior for this function is to set `s` to a raw `char *` that refers to the internal string data in the target language. In other words, if you were using a language like Tcl, and you wrote this,

> 该函数的默认包装行为是将 `s` 设置为原始 `char *`，以目标语言引用内部字符串数据。换句话说，如果你使用的是 Tcl 这样的语言，并且你是这样写的，

```
% foo Hello
```

then `s` would point to the representation of "Hello" inside the Tcl interpreter. When returning a `char *`, SWIG assumes that it is a NULL-terminated string and makes a copy of it. This gives the target language its own copy of the result.

There are obvious problems with the default behavior. First, since a `char *` argument points to data inside the target language, it is **NOT** safe for a function to modify this data (doing so may corrupt the interpreter and lead to a crash). Furthermore, the default behavior does not work well with binary data. Instead, strings are assumed to be NULL-terminated.

> 那么 `s` 将指向 Tcl 解释器中 `Hello` 的表示。当返回 `char *` 时，SWIG 假定它是一个以 NULL 结尾的字符串，并对其进行复制。这为目标语言提供了自己的结果副本。
>
> 默认行为存在明显的问题。首先，由于 `char *` 参数指向目标语言内部的数据，因此函数修改该数据是不安全的（这样做可能会破坏解释器并导致崩溃）。此外，默认行为不适用于二进制数据。而是假定字符串以 NULL 终止。

### 9.3.2 传递二进制数据

If you have a function that expects binary data,

> 如果你的函数需要二进制数据，

```c
size_t parity(char *str, size_t len, size_t initial);
```

you can wrap the parameters `(char *str, size_t len)` as a single argument using a typemap. Just do this:

> 你可以使用类型映射将参数 `(char * str, size_t len)` 包装为单个参数。即这样做：

```
%apply (char *STRING, size_t LENGTH) { (char *str, size_t len) };
...
size_t parity(char *str, size_t len, size_t initial);
```

Now, in the target language, you can use binary string data like this:

> 现在，在目标语言中，你可以使用像这样的二进制字符串数据：

```python
>>> s = "H\x00\x15eg\x09\x20"
>>> parity(s, 0)
```

In the wrapper function, the passed string will be expanded to a pointer and length parameter. The `(char *STRING, int LENGTH)` multi-argument typemap is also available in addition to `(char *STRING, size_t LENGTH)`.

> 在包装器函数中，传递的字符串将扩展为指针和长度参数。除了 `(char * STRING, size_t LENGTH)` 外，`(char * STRING, int LENGTH)` 多参数类型映射也是可用的。

### 9.3.3 使用 `%newobject` 释放内存

If you have a function that allocates memory like this,

> 如果你有一个这样分配内存的函数，

```c
char *foo() {
  char *result = (char *) malloc(...);
  ...
  return result;
}
```

then the SWIG generated wrappers will have a memory leak--the returned data will be copied into a string object and the old contents ignored.

To fix the memory leak, use the `%newobject` directive.

> 那么 SWIG 生成的包装器将发生内存泄漏，返回的数据将被复制到字符串对象中，而旧内容将被忽略。
>
> 要解决内存泄漏，请使用 `%newobject` 指令。

```
%newobject foo;
...
char *foo();
```

This will release the result if the appropriate target language support is available. SWIG provides the appropriate "newfree" typemap for `char *` so that the memory is released, however, you may need to provide your own "newfree" typemap for other types. See [Object ownership and %newobject](http://swig.org/Doc3.0/Customization.html#Customization_ownership) for more details.

> 如果目标语言有适当的支持可用，结果将被释放。SWIG 为 `char *` 提供了适当的 `newfree` 类型映射，以便释放内存，但是，你可能需要为其他类型提供自己的 `newfree` 类型映射。有关更多详细信息，请参见[对象所有权和 `%newobject`](http://swig.org/Doc3.0/Customization.html#Customization_ownership)。

### 9.3.4 `cstring.i`

The `cstring.i` library file provides a collection of macros for dealing with functions that either mutate string arguments or which try to output string data through their arguments. An example of such a function might be this rather questionable implementation:

> `cstring.i` 库文件提供了一个宏集合，用于处理使字符串参数发生改变或试图通过其参数输出字符串数据的函数。这个函数的一个示例可能是这种相当可疑的实现：

```c
void get_path(char *s) {
  // Potential buffer overflow---uh, oh.
  sprintf(s, "%s/%s", base_directory, sub_directory);
}
...
// Somewhere else in the C program
{
  char path[1024];
  ...
  get_path(path);
  ...
}
```

(Off topic rant: If your program really has functions like this, you would be well-advised to replace them with safer alternatives involving bounds checking).

The macros defined in this module all expand to various combinations of typemaps. Therefore, the same pattern matching rules and ideas apply.

> （不在主题范围内：如果你的程序确实具有这样的功能，建议你使用涉及边界检查的更安全替代方法来替换它们）。
>
> 此模块中定义的宏全部扩展为类型映射的各种组合。因此，适用相同的模式匹配规则和思想。

**`%cstring_bounded_output(parm, maxsize)`**

Turns parameter `parm` into an output value. The output string is assumed to be NULL-terminated and smaller than `maxsize` characters. Here is an example:

> 将参数 `parm` 转换为输出值。假设输出字符串以 NULL 结尾，并且小于 `maxsize` 个字符。这是一个例子：

```
%cstring_bounded_output(char *path, 1024);
...
void get_path(char *path);
```

In the target language:

> 在目标语言中：

```python
>>> get_path()
/home/beazley/packages/Foo/Bar
>>>
```

Internally, the wrapper function allocates a small buffer (on the stack) of the requested size and passes it as the pointer value. Data stored in the buffer is then returned as a function return value. If the function already returns a value, then the return value and the output string are returned together (multiple return values). **If more than maxsize bytes are written, your program will crash with a buffer overflow!**

> 在内部，包装器函数会分配一个请求大小的小缓冲区（在堆栈上），并将其作为指针值传递。然后将存储在缓冲区中的数据作为函数返回值返回。如果函数已经返回一个值，则将返回值和输出字符串一起返回（多个返回值）。**如果写入的字节数超过最大字节数，则程序将因缓冲区溢出而崩溃！**

**`%cstring_chunk_output(parm, chunksize)`**

Turns parameter `parm` into an output value. The output string is always `chunksize` and may contain binary data. Here is an example:

> 将参数 `parm` 转换为输出值。输出字符串始终为 `chunksize`，并且可能包含二进制数据。这是一个例子：

```
%cstring_chunk_output(char *packet, PACKETSIZE);
...
void get_packet(char *packet);
```

In the target language:

> 在目标语言中：

```python
>>> get_packet()
'\xa9Y:\xf6\xd7\xe1\x87\xdbH;y\x97\x7f\xd3\x99\x14V\xec\x06\xea\xa2\x88'
>>>
```

This macro is essentially identical to `%cstring_bounded_output`. The only difference is that the result is always `chunksize` characters. Furthermore, the result can contain binary data. **If more than maxsizebytes are written, your program will crash with a buffer overflow!**

> 这个宏在本质上与 `%cstring_bounded_output` 相同。唯一的区别是结果始终是 `chunksize` 个字符。此外，结果可以包含二进制数据。**如果写入的字节数超过 `maxsizebytes`，你的程序将因缓冲区溢出而崩溃！**

**`%cstring_bounded_mutable(parm, maxsize)`**

Turns parameter `parm` into a mutable string argument. The input string is assumed to be NULL-terminated and smaller than `maxsize` characters. The output string is also assumed to be NULL-terminated and less than `maxsize` characters.

> 将参数 `parm` 转换为可变的字符串参数。假设输入字符串以 NULL 终止，并且小于 `maxsize` 字符。还假定输出字符串以 NULL 终止并且小于 `maxsize` 个字符。

```
%cstring_bounded_mutable(char *ustr, 1024);
...
void make_upper(char *ustr);
```

In the target language:

> 在目标语言中：

```python
>>> make_upper("hello world")
'HELLO WORLD'
>>>
```

Internally, this macro is almost exactly the same as`%cstring_bounded_output`. The only difference is that the parameter accepts an input value that is used to initialize the internal buffer. It is important to emphasize that this function does not mutate the string value passed---instead it makes a copy of the input value, mutates it, and returns it as a result. **If more than maxsize bytes are written, your program will crash with a buffer overflow!**

> 在内部，此宏与 `%cstring_bounded_output` 几乎完全相同。唯一的区别是该参数接受用于初始化内部缓冲区的输入值。需要强调的是，此函数不会使传递的字符串值发生突变，而是复制输入值，对其进行突变并作为结果返回。**如果写入的字节数超过最大字节数，则程序将因缓冲区溢出而崩溃！**

**`%cstring_mutable(parm [, expansion])`**

Turns parameter `parm` into a mutable string argument. The input string is assumed to be NULL-terminated. An optional parameter `expansion` specifies the number of extra characters by which the string might grow when it is modified. The output string is assumed to be NULL-terminated and less than the size of the input string plus any expansion characters.

> 将参数 `parm` 转换为可变的字符串参数。假定输入字符串以 NULL 终止。可选参数 `expansion` 指定修改字符串时字符串可能增长的额外字符数。假定输出字符串以 NULL 终止，并且小于输入字符串的大小加上任何扩展字符。

```
%cstring_mutable(char *ustr);
...
void make_upper(char *ustr);

%cstring_mutable(char *hstr, HEADER_SIZE);
...
void attach_header(char *hstr);
```

In the target language:

> 在目标语言中：

```python
>>> make_upper("hello world")
'HELLO WORLD'
>>> attach_header("Hello world")
'header: Hello world'
>>>
```

This macro differs from `%cstring_bounded_mutable()` in that a buffer is dynamically allocated (on the heap using `malloc/new`). This buffer is always large enough to store a copy of the input value plus any expansion bytes that might have been requested. It is important to emphasize that this function does not directly mutate the string value passed---instead it makes a copy of the input value, mutates it, and returns it as a result. **If the function expands the result by more than expansion extra bytes, then the program will crash with a buffer overflow!**

> 这个宏与 `%cstring_bounded_mutable()` 的不同之处在于动态地分配了一个缓冲区（在堆上使用 `malloc`、`new`）。此缓冲区始终足够大，可以存储输入值的副本以及可能已请求的任何扩展字节。需要强调的是，此函数不会直接更改传递的字符串值，而是复制输入值，对其进行更改并返回结果。**如果函数将结果扩展多于扩展多余字节，则程序将因缓冲区溢出而崩溃！**

**`%cstring_output_maxsize(parm, maxparm)`**

This macro is used to handle bounded character output functions where both a `char *` and a maximum length parameter are provided. As input, a user simply supplies the maximum length. The return value is assumed to be a NULL-terminated string.

> 该宏用于处理有限制的字符输出函数，其中提供了 `char *` 和最大长度参数。作为输入，用户只需提供最大长度即可。返回值假定为以 NULL 结尾的字符串。

```
%cstring_output_maxsize(char *path, int maxpath);
...
void get_path(char *path, int maxpath);
```

In the target language:

> 在目标语言中：

```python
>>> get_path(1024)
'/home/beazley/Packages/Foo/Bar'
>>>
```

This macro provides a safer alternative for functions that need to write string data into a buffer. User supplied buffer size is used to dynamically allocate memory on heap. Results are placed into that buffer and returned as a string object.

> 对于需要将字符串数据写入缓冲区的函数，此宏提供了更安全的选择。用户提供的缓冲区大小用于在堆上动态分配内存。结果放入该缓冲区中，并作为字符串对象返回。

**`%cstring_output_withsize(parm, maxparm)`**

This macro is used to handle bounded character output functions where both a `char *` and a pointer `int *` are passed. Initially, the `int *` parameter points to a value containing the maximum size. On return, this value is assumed to contain the actual number of bytes. As input, a user simply supplies the maximum length. The output value is a string that may contain binary data.

> 这个宏用于处理有限制的字符输出函数，其中既传递了 `char *` 指针，又传递了指针 `int *`。最初，`int *` 参数指向包含最大大小的值。返回时，假定该值包含实际字节数。作为输入，用户只需提供最大长度即可。输出值是一个可能包含二进制数据的字符串。

```
%cstring_output_withsize(char *data, int *maxdata);
...
void get_data(char *data, int *maxdata);
```

In the target language:

> 在目标语言中：

```python
>>> get_data(1024)
'x627388912'
>>> get_data(1024)
'xyzzy'
>>>
```

This macro is a somewhat more powerful version of `%cstring_output_chunk()`. Memory is dynamically allocated and can be arbitrary large. Furthermore, a function can control how much data is actually returned by changing the value of the `maxparm` argument.

> 这个宏是 `%cstring_output_chunk()` 的更强大的版本。内存是动态分配的，可以任意大。此外，一个函数可以通过更改 `maxparm` 参数的值来控制实际返回多少数据。

**`%cstring_output_allocate(parm, release)`**

This macro is used to return strings that are allocated within the program and returned in a parameter of type `char **`. For example:

> 该宏用于返回在程序内分配的字符串，并以 `char **` 类型的参数返回。例如：

```c
void foo(char **s) {
  *s = (char *) malloc(64);
  sprintf(*s, "Hello world\n");
}
```

The returned string is assumed to be NULL-terminated. `release` specifies how the allocated memory is to be released (if applicable). Here is an example:

> 假定返回的字符串以 NULL 终止。`release` 指定如何释放分配的内存（如果适用）。这是一个例子：

```
%cstring_output_allocate(char **s, free(*$1));
...
void foo(char **s);
```

In the target language:

> 在目标语言中：

```python
>>> foo()
'Hello world\n'
>>>
```

**`%cstring_output_allocate_size(parm, szparm, release)`**

This macro is used to return strings that are allocated within the program and returned in two parameters of type `char **` and `int *`. For example:

> 该宏用于返回在程序中分配的字符串，并以 `char **` 和 `int *` 类型的两个参数返回。例如：

```c
void foo(char **s, int *sz) {
  *s = (char *) malloc(64);
  *sz = 64;
  // Write some binary data
  ...
}
```

The returned string may contain binary data. `release`specifies how the allocated memory is to be released (if applicable). Here is an example:

> 返回的字符串可能包含二进制数据。`release` 指定释放分配的内存的方式（如果适用）。这是一个例子：

```
%cstring_output_allocate_size(char **s, int *slen, free(*$1));
...
void foo(char **s, int *slen);
```

In the target language:

> 在目标语言中：

```python
>>> foo()
'\xa9Y:\xf6\xd7\xe1\x87\xdbH;y\x97\x7f\xd3\x99\x14V\xec\x06\xea\xa2\x88'
>>>
```

This is the safest and most reliable way to return binary string data in SWIG. If you have functions that conform to another prototype, you might consider wrapping them with a helper function. For example, if you had this:

> 这是在 SWIG 中返回二进制字符串数据的最安全、最可靠的方法。如果你具有符合另一个原型的函数，则可以考虑使用辅助函数将它们包装起来。例如，如果你有：

```c
char *get_data(int *len);
```

You could wrap it with a function like this:

> 你可以用这样一个函数包装它：

```c
void my_get_data(char **result, int *len) {
  *result = get_data(len);
}
```

**Comments:**

* Support for the `cstring.i` module depends on the target language. Not all SWIG modules currently support this library.
* Reliable handling of raw C strings is a delicate topic. There are many ways to accomplish this in SWIG. This library provides support for a few common techniques.
* If used in C++, this library uses `new` and `delete []` for memory allocation. If using ANSI C, the library uses `malloc()` and `free()`.
* Rather than manipulating `char *` directly, you might consider using a special string structure or class instead.

> **评论：**
>
> * 对 `cstring.i` 模块的支持取决于目标语言。当前，并非所有的 SWIG 模块都支持该库。
> * 可靠地处理原始 C 字符串是一个微妙的话题。在 SWIG 中有许多方法可以完成此操作。该库提供对一些常用技术的支持。
> * 如果在 C++ 中使用，该库使用 `new` 和 `delete []` 进行内存分配。如果使用 ANSI C，该库将使用 `malloc()` 和 `free()`。
> * 你可以考虑使用特殊的字符串结构或类，而不是直接操作 `char *`。

## 9.4 STL/C++ 库

The library modules in this section provide access to parts of the standard C++ library including the STL. SWIG support for the STL is an ongoing effort. Support is quite comprehensive for some language modules but some of the lesser used modules do not have quite as much library code written.

The following table shows which C++ classes are supported and the equivalent SWIG interface library file for the C++ library.

> 本节中的库模块提供对标准 C++ 库（包括 STL）的某些部分的访问。SWIG 对 STL 的支持是一项持续的工作。对某些语言模块的支持非常全面，但是一些使用较少的模块没有编写太多的库代码。
>
> 下表显示了支持的 C++ 类以及 C++ 库的等效 SWIG 接口库文件。

| **C++ class**     | **C++ Library file** | **SWIG Interface library file** |
| ----------------- | -------------------- | ------------------------------- |
| `std::auto_ptr`   | `memory`             | `std_auto_ptr.i`                |
| `std::deque`      | `deque`              | `std_deque.i`                   |
| `std::list`       | `list`               | `std_list.i`                    |
| `std::map`        | `map`                | `std_map.i`                     |
| `std::pair`       | `utility`            | `std_pair.i`                    |
| `std::set`        | `set`                | `std_set.i`                     |
| `std::string`     | `string`             | `std_string.i`                  |
| `std::vector`     | `vector`             | `std_vector.i`                  |
| `std::array`      | `array` (C++11)      | `std_array.i`                   |
| `std::shared_ptr` | `shared_ptr` (C++11) | `std_shared_ptr.i`              |

The list is by no means complete; some language modules support a subset of the above and some support additional STL classes. Please look for the library files in the appropriate language library directory.

> 清单绝不完整； 一些语言模块支持上述内容的子集，而某些语言模块支持其他 STL 类。请在适当的语言库目录中查找库文件。

### 9.4.1 `std::string`

The `std_string.i` library provides typemaps for converting C++ `std::string` objects to and from strings in the target scripting language. For example:

> `std_string.i` 库提供了用于将 C++ `std::string` 对象与目标脚本语言中的字符串进行相互转换的类型映射。例如：

```
%module example
%include "std_string.i"

std::string foo();
void        bar(const std::string &x);
```

In the target language:

> 在脚本语言中：

```python
x = foo();                # Returns a string object
bar("Hello World");       # Pass string as std::string
```

A common problem that people encounter is that of classes/structures containing a `std::string`. This can be overcome by defining a typemap. For example:

> 人们遇到的一个常见问题是包含 `std::string` 的类或结构体的问题。这可以通过定义类型映射来克服。例如：

```
%module example
%include "std_string.i"

%apply const std::string& {std::string* foo};

struct my_struct
{
  std::string foo;
};
```

In the target language:

> 在脚本语言中：

```python
x = my_struct()
x.foo="Hello World"       # assign with string
print x.foo               # print as string
```

This module only supports types `std::string` and `const std::string &`. Pointers and non-const references are left unmodified and returned as SWIG pointers.

This library file is fully aware of C++ namespaces. If you export `std::string` or rename it with a typedef, make sure you include those declarations in your interface. For example:

> 该模块仅支持类型 `std::string` 和 `const std::string&`。指针和非常引用保持不变，并作为 SWIG 指针返回。
>
> 该库文件完全了解 C++ 命名空间。如果导出 `std::string` 或使用 `typedef` 重命名，请确保在接口中包含这些声明。例如：

```
%module example
%include "std_string.i"

using namespace std;
typedef std::string String;
...
void foo(string s, const String &t);     // std_string typemaps still applied
```

### 9.4.2 `std::vector`

The `std_vector.i` library provides support for the C++ `std::vector` class in the STL. Using this library involves the use of the `%template` directive. All you need to do is to instantiate different versions of `vector` for the types that you want to use. For example:

> `std_vector.i` 库为 STL 中的 C++ `std::vector` 类提供了支持。使用这个库需要使用 `%template` 指令。你要做的就是为要使用的类型实例化不同版本的 `vector`。例如：

```
%module example
%include "std_vector.i"

namespace std {
  %template(vectori) vector<int>;
  %template(vectord) vector<double>;
};
```

When a template `vector<X>` is instantiated a number of things happen:

* A class that exposes the C++ API is created in the target language. This can be used to create objects, invoke methods, etc. This class is currently a subset of the real STL vector class.
* Input typemaps are defined for `vector<X>`, `const vector<X> &`, and `const vector<X> *`. For each of these, a pointer `vector<X> *` may be passed or a native list object in the target language.
* An output typemap is defined for `vector<X>`. In this case, the values in the vector are expanded into a list object in the target language.
* For all other variations of the type, the wrappers expect to receive a `vector<X> *` object in the usual manner.
* An exception handler for `std::out_of_range` is defined.
* Optionally, special methods for indexing, item retrieval, slicing, and element assignment may be defined. This depends on the target language.

To illustrate the use of this library, consider the following functions:

> 当模板 `vector<X>` 被实例化时，会发生很多事情：
>
> * 在目标语言中创建暴露 C++ API 的类。它可以用于创建对象，调用方法等。此类当前是真实 STL `vector` 类的子集。
> * 输入类型映射是为 `vector<X>`、`const vector<X>&` 和 `const vector<X> *` 定义的。对于其中的每一个，都可以传递指针 `vector<X> *` 或目标语言中的原生列表对象。
> * 为 `vector<X>` 定义了一个输出类型映射。在这种情况下，向量中的值将扩展为目标语言中的列表对象。
> * 对于所有其他类型的包装，包装器希望以通常的方式接收一个 `vector<X> *` 对象。
> * 为 `std::out_of_range` 定义了一个异常处理。
> * 可选地，可以定义用于索引，项目检索，切片和元素分配的特殊方法。这取决于目标语言。
>
> 为了说明此库的用法，请考虑以下函数：

```c++
/* File : example.h */

#include <vector>
#include <algorithm>
#include <functional>
#include <numeric>

double average(std::vector<int> v) {
  return std::accumulate(v.begin(), v.end(), 0.0)/v.size();
}

std::vector<double> half(const std::vector<double>& v) {
  std::vector<double> w(v);
  for (unsigned int i=0; i<w.size(); i++)
    w[i] /= 2.0;
  return w;
}

void halve_in_place(std::vector<double>& v) {
  std::transform(v.begin(), v.end(), v.begin(),
                 std::bind2nd(std::divides<double>(), 2.0));
}
```

To wrap with SWIG, you might write the following:

> 用 SWIG 包装，你可以这样写：

```
%module example
%{
#include "example.h"
%}

%include "std_vector.i"
// Instantiate templates used by example
namespace std {
  %template(IntVector) vector<int>;
  %template(DoubleVector) vector<double>;
}

// Include the header file with above prototypes
%include "example.h"
```

Now, to illustrate the behavior in the scripting interpreter, consider this Python example:

> 现在，为了说明脚本解释器中的行为，请考虑以下 Python 示例：

```python
>>> from example import *
>>> iv = IntVector(4)         # Create an vector<int>
>>> for i in range(0, 4):
...      iv[i] = i
>>> average(iv)               # Call method
1.5
>>> average([0, 1, 2, 3])        # Call with list
1.5
>>> half([1, 2, 3])             # Half a list
(0.5, 1.0, 1.5)
>>> halve_in_place([1, 2, 3])   # Oops
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
TypeError: Type error. Expected _p_std__vectorTdouble_t
>>> dv = DoubleVector(4)
>>> for i in range(0, 4):
...       dv[i] = i
>>> halve_in_place(dv)       # Ok
>>> for i in dv:
...       print i
...
0.0
0.5
1.0
1.5
>>> dv[20] = 4.5
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
  File "example.py", line 81, in __setitem__
    def __setitem__(*args): return apply(examplec.DoubleVector___setitem__, args)
IndexError: vector index out of range
>>>
```

This library module is fully aware of C++ namespaces. If you use vectors with other names, make sure you include the appropriate `using` or typedef directives. For example:

> 该库模块完全了解 C++ 命名空间。如果你使用带有其他名称的向量，请确保包括适当的 `using` 或 `typedef` 指令。例如：

```
%include "std_vector.i"

namespace std {
  %template(IntVector) vector<int>;
}

using namespace std;
typedef std::vector Vector;

void foo(vector<int> *x, const Vector &x);
```

**Note:** This module makes use of several advanced SWIG features including templatized typemaps and template partial specialization. If you are trying to wrap other C++ code with templates, you might look at the code contained in `std_vector.i`. Alternatively, you can show them the code if you want to make their head explode.

**Note:** This module is defined for all SWIG target languages. However argument conversion details and the public API exposed to the interpreter vary.

> **注意**：该模块利用了几种高级 SWIG 功能，包括模板化的类型映射和模板的偏特化。如果你尝试使用模板包装其他 C++ 代码，则可以查看 `std_vector.i` 中包含的代码。或者，如果你想让他们的脑袋爆炸，则可以向他们显示代码。
>
> **注意**：该模块是为所有 SWIG 目标语言定义的。但是，参数转换的详细信息和解释器公开的公共 API 有所不同。

### 9.4.3 STL 异常

Many of the STL wrapper functions add parameter checking and will throw a language dependent error/exception should the values not be valid. The classic example is array bounds checking. The library wrappers are written to throw a C++ exception in the case of error. The C++ exception in turn gets converted into an appropriate error/exception for the target language. By and large this handling should not need customising, however, customisation can easily be achieved by supplying appropriate "throws" typemaps. For example:

> 许多 STL 包装器函数都会添加参数检查，如果值无效，则会抛出与语言有关的错误或异常。经典示例是数组边界检查。库包装程序被编写为在发生错误的情况下引发 C++ 异常。反过来，C++ 异常会转换为目标语言的相应错误或异常。总的来说，这种处理不需要定制，但是，可以通过提供适当的 `throw` 类型映射来轻松实现定制。例如：

```
%module example
%include "std_vector.i"
%typemap(throws) std::out_of_range {
  // custom exception handler
}
%template(VectInt) std::vector<int>;
```

The custom exception handler might, for example, log the exception then convert it into a specific error/exception for the target language.

When using the STL it is advisable to add in an exception handler to catch all STL exceptions. The `%exception` directive can be used by placing the following code before any other methods or libraries to be wrapped:

> 自定义异常处理程序可能是记录异常，然后将其转换为目标语言的特定错误或异常。
>
> 使用 STL 时，建议添加一个异常处理程序以捕获所有 STL 异常。通过将以下代码放在任何其他要包装的方法或库之前，可以使用 `%exception` 伪指令：

```
%include "exception.i"

%exception {
  try {
    $action
  } catch (const std::exception& e) {
    SWIG_exception(SWIG_RuntimeError, e.what());
  }
}
```

Any thrown STL exceptions will then be gracefully handled instead of causing a crash.

> 然后将妥善处理所有引发的 STL 异常，而不会导致崩溃。

### 9.4.4 `shared_ptr` 智能指针

Some target languages have support for handling the `shared_ptr` reference counted smart pointer. This smart pointer is available in the standard C++11 library as `std::shared_ptr`. It was also in TR1 as `std::tr1::shared_ptr` before it was fully standardized. Support for the widely used `boost::shared_ptr` is also available.

In order to use `std::shared_ptr`, the `std_shared_ptr.i` library file should be included:

> 一些目标语言支持处理 `shared_ptr` 引用计数智能指针。该智能指针在标准 C++11 库中以 `std::shared_ptr` 的形式提供。在 TR1 中，它在完全标准化之前还以 `std::tr1::shared_ptr` 的形式出现。也支持广泛使用的 `boost::shared_ptr`。
>
> 为了使用 `std::shared_ptr`，应该包含 `std_shared_ptr.i` 库文件：

```
%include <std_shared_ptr.i>
```

The pre-standard `std::tr1::shared_ptr` can be used by including the following macro before including the `std_shared_ptr.i` library file:

> 通过在包含 `std_shared_ptr.i` 库文件之前包含以下宏，可以使用标准的 `std::tr1::shared_ptr`：

```
#define SWIG_SHARED_PTR_SUBNAMESPACE tr1
%include <std_shared_ptr.i>
```

In order to use `boost::shared_ptr`, the `boost_shared_ptr.i` library file should be included:

> 为了使用 `boost::shared_ptr`，应该包含 `boost_shared_ptr.i` 库文件：

```
%include <boost_shared_ptr.i>
```

You can only use one of these variants of shared_ptr in your interface file at a time. and all three variants must be used in conjunction with the `%shared_ptr(T)` macro, where `T` is the underlying pointer type equating to usage `shared_ptr<T>`. The type `T` must be non-primitive. A simple example demonstrates usage:

> 你只能在接口文件中使用 `shared_ptr` 的这些变体中的一个。并且所有三个变体都必须与 `%shared_ptr(T)` 宏结合使用，其中 `T` 是基础指针类型，等同于用法 `shared_ptr<T>`。类型 `T` 必须是非原始的。一个简单的示例演示用法：

```
%module example
%include <boost_shared_ptr.i>
%shared_ptr(IntValue)

%inline %{
#include <boost/shared_ptr.hpp>

struct IntValue {
  int value;
  IntValue(int v) : value(v) {}
};

static int extractValue(const IntValue &t) {
  return t.value;
}

static int extractValueSmart(boost::shared_ptr<IntValue> t) {
  return t->value;
}
%}
```

Note that the `%shared_ptr(IntValue)` declaration occurs after the inclusion of the `boost_shared_ptr.i` library which provides the macro and, very importantly, before any usage or declaration of the type, `IntValue`. The `%shared_ptr` macro provides, a few things for handling this smart pointer, but mostly a number of typemaps. These typemaps override the default typemaps so that the underlying proxy class is stored and passed around as a pointer to a `shared_ptr` instead of a plain pointer to the underlying type. This approach means that any instantiation of the type can be passed to methods taking the type by value, reference, pointer or as a smart pointer. The interested reader might want to look at the generated code, however, usage is simple and no different handling is required from the target language. For example, a simple use case of the above code from Java would be:

> 请注意，`%shared_ptr(IntValue)` 声明是在包含提供宏的 `boost_shared_ptr.i` 库之后出现的，并且非常重要的是，在使用或声明类型 `IntValue` 之前出现。`%shared_ptr` 宏提供了一些用于处理此智能指针的内容，但主要是许多类型映射。这些类型映射会覆盖默认的类型映射，以便存储和传递底层代理类，作为指向 `shared_ptr` 的指针，而不是指向底层类型的普通指针。这种方法意味着该类型的任何实例都可以通过值、引用、指针或作为智能指针传递给采用该类型的方法。有兴趣的读者可能想看一下生成的代码，但是，用法很简单，不需要与目标语言进行不同的处理。例如，上述来自 Java 的代码的简单用例将是：

```java
IntValue iv = new IntValue(1234);
int val1 = example.extractValue(iv);
int val2 = example.extractValueSmart(iv);
System.out.println(val1 + " " + val2);
```

This shared_ptr library works quite differently to SWIG's normal, but somewhat limited, [smart pointer handling](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_smart_pointers). The shared_ptr library does not generate extra wrappers, just for smart pointer handling, in addition to the proxy class. The normal proxy class including inheritance relationships is generated as usual. The only real change introduced by the `%shared_ptr` macro is that the proxy class stores a pointer to the shared_ptr instance instead of a raw pointer to the instance. A proxy class derived from a base which is being wrapped with shared_ptr can and **must** be wrapped as a shared_ptr too. In other words all classes in an inheritance hierarchy must all be used with the `%shared_ptr` macro. For example the following code can be used with the base class shown earlier:

> 这个 `shared_ptr` 库的工作方式与 SWIG 的正常方法有所不同，但有点局限，[智能指针处理](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_smart_pointers)。除了代理类之外，`shared_ptr` 库不会生成其他包装，仅用于智能指针处理。照常生成包括继承关系在内的普通代理类。`%shared_ptr` 宏引入的唯一真正的变化是代理类存储指向 `shared_ptr` 实例的指针，而不是指向该实例的原始指针。可以（必须）也从必须由 `shared_ptr` 包装的基派生的代理类也包装为 `shared_ptr`。换句话说，继承层次结构中的所有类都必须与 `%shared_ptr` 宏一起使用。例如，以下代码可与前面显示的基类一起使用：

```
%shared_ptr(DerivedIntValue)
%inline %{
struct DerivedIntValue : IntValue {
  DerivedIntValue(int value) : IntValue(value) {}
  ...
};
%}
```

A `shared_ptr` of the derived class can now be passed to a method where the base is expected in the target language, just as it can in C++:

> 现在可以将派生类的 `shared_ptr` 传递给目标语言（如 C++ 中一样）的方法：

```c++
DerivedIntValue div = new DerivedIntValue(5678);
int val3 = example.extractValue(div);
int val4 = example.extractValueSmart(div);
```

If the `%shared_ptr` macro is omitted for any class in the inheritance hierarchy, SWIG will warn about this and the generated code may or may not result in a C++ compilation error. For example, the following input:

> 如果继承层次结构中的任何类都省略了 `%shared_ptr` 宏，SWIG 将对此发出警告，并且所生成的代码可能会或可能不会导致 C++ 编译错误。例如，以下输入：

```
%include "boost_shared_ptr.i"
%shared_ptr(Parent);

%inline %{
  #include <boost/shared_ptr.hpp>
  struct GrandParent {
    virtual ~GrandParent() {}
  };

  struct Parent : GrandParent {
    virtual ~Parent() {}
  };

  struct Child : Parent {
    virtual ~Child() {}
  };
%}
```

warns about the missing smart pointer information:

> 警告缺少智能指针的信息：

```
example.i:12: Warning 520: Base class 'GrandParent' of 'Parent' is not similarly marked as a smart pointer.
example.i:16: Warning 520: Derived class 'Child' of 'Parent' is not similarly marked as a smart pointer.
```

Adding the missing `%shared_ptr` macros will fix this:

> 添加缺少的 `%shared_ptr` 宏将解决此问题：

```
%include "boost_shared_ptr.i"
%shared_ptr(GrandParent);
%shared_ptr(Parent);
%shared_ptr(Child);

... as before ...
```

**Note:** There is somewhat limited support for `%shared_ptr` and the director feature and the degrees of success varies among the different target languages. Please help to improve this support by providing patches with improvements.

> **注意**：对 `%shared_ptr` 和 director 功能的支持有限，成功的程度因不同的目标语言而异。请通过提供具有改进功能的补丁来帮助改善此支持。

### 9.4.5 `auto_ptr` 智能指针

While `std::auto_ptr` is deprecated in C++11, some existing code may still be using it, so SWIG provides limited support for this class: `std_auto_ptr.i` defines the typemaps which apply to the functions returning objects of this type. Any other use of `std_auto_ptr.i` is not directly supported.

A typical example of use would be

> 尽管在 C++11中不推荐使用 `std::auto_ptr`，但是某些现有代码可能仍在使用它，因此 SWIG 对此类提供了有限的支持：`std_auto_ptr.i` 定义了适用于返回此类型对象的函数的类型映射。不直接支持对 `std_auto_ptr.i` 的任何其他使用。
>
> 使用的典型示例是

```
%include <std_auto_ptr.i>

%auto_ptr(Klass)
%inline %{
class Klass {
public:
  // Factory function creating objects of this class:
  static std::auto_ptr<Klass> Create(int value) {
    return std::auto_ptr<Klass>(new Klass(value));
  }

  int getValue() const { return m_value; }

private:
  DerivedIntValue(int value) : m_value(value) {}
  int m_value;
};
%}
```

The returned objects can be used naturally from the target language, e.g. from C#:

> 返回的对象可以自然地从目标语言使用，例如 来自 C#：

```csharp
Klass k = Klass.Create(17);
int value = k.getValue();
```

## 9.5 实用函数库

### 9.5.1 `exception.i`

The `exception.i` library provides a language-independent function for raising a run-time exception in the target language. This library is largely used by the SWIG library writers. If possible, use the error handling scheme available to your target language as there is greater flexibility in what errors/exceptions can be thrown.

> `exception.i` 库提供了一种独立于语言的函数，用于以目标语言引发运行时异常。SWIG 库作者主要使用此库。如果可能，请使用适用于你的目标语言的错误处理方案，因为在引发什么错误或异常方面具有更大的灵活性。

**`SWIG_exception(int code, const char *message)`**

Raises an exception in the target language. `code` is one of the following symbolic constants:

> 在目标语言中引发异常。`code` 是以下符号常量之一：

```
SWIG_MemoryError
SWIG_IOError
SWIG_RuntimeError
SWIG_IndexError
SWIG_TypeError
SWIG_DivisionByZero
SWIG_OverflowError
SWIG_SyntaxError
SWIG_ValueError
SWIG_SystemError
```

`message` is a string indicating more information about the problem.

The primary use of this module is in writing language-independent exception handlers. For example:

> `message` 是一个字符串，指示有关该问题的更多信息。
>
> 该模块的主要用途是编写独立于语言的异常处理程序。例如：

```
%include "exception.i"
%exception std::vector::getitem {
  try {
    $action
  } catch (std::out_of_range& e) {
    SWIG_exception(SWIG_IndexError, const_cast<char*>(e.what()));
  }
}
```
