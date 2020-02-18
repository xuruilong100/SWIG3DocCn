[TOC]

# 10 参数处理

In Chapter 3, SWIG's treatment of basic datatypes and pointers was described. In particular, primitive types such as `int` and `double` are mapped to corresponding types in the target language. For everything else, pointers are used to refer to structures, classes, arrays, and other user-defined datatypes. However, in certain applications it is desirable to change SWIG's handling of a specific datatype. For example, you might want to return multiple values through the arguments of a function. This chapter describes some of the techniques for doing this.

> 第 3 章介绍了 SWIG 对基本数据类型和指针的处理。特别地，原始类型（诸如 `int` 和 `double`）被映射到目标语言中的相应类型。对于其他所有类型，都使用指针来引用结构体、类、数组和其他用户定义的数据类型。但是，在某些应用程序中，希望更改 SWIG 对特定数据类型的处理。例如，你可能想通过函数的参数返回多个值。本章介绍了一些执行此类操作的技术。

## 10.1 `typemaps.i` 库

This section describes the `typemaps.i` library file--commonly used to change certain properties of argument conversion.

> 本节描述 `typemaps.i` 库文件——通常用于更改参数转换的某些属性。

### 10.1.1 引言

Suppose you had a C function like this:

> 假设你有如下 C 函数：

```c
void add(double a, double b, double *result) {
  *result = a + b;
}
```

From reading the source code, it is clear that the function is storing a value in the `double *result` parameter. However, since SWIG does not examine function bodies, it has no way to know that this is the underlying behavior.

One way to deal with this is to use the `typemaps.i` library file and write interface code like this:

> 通过阅读源代码，很明显该函数将值存储在 `double *result` 参数中。但是，由于 SWIG 不检查函数主体，因此没有办法知道函数的底层行为。
>
> 一种解决方法是使用 `typemaps.i` 库文件，并编写如下接口代码：

```
// Simple example using typemaps
%module example
%include "typemaps.i"

%apply double *OUTPUT { double *result };
%inline %{
extern void add(double a, double b, double *result);
%}
```

The `%apply` directive tells SWIG that you are going to apply a special type handling rule to a type. The `double *OUTPUT` specification is the name of a rule that defines how to return an output value from an argument of type `double *`. This rule gets applied to all of the datatypes listed in curly braces--in this case `double *result`.

When the resulting module is created, you can now use the function like this (shown for Python):

> `%apply` 命令告诉 SWIG 你将要对某类型应用特殊的类型处理规则。`double *OUTPUT` 规范是一个规则的名称，该规则定义了如何从类型为 `double *` 的参数返回输出值。该规则将应用于大括号中列出的所有数据类型，在这个例子中为 `double *result`。
>
> 结果模块创建后，你现在可以这样使用以下函数（针对 Python）：

```python
>>> a = add(3, 4)
>>> print a
7
>>>
```

In this case, you can see how the output value normally returned in the third argument has magically been transformed into a function return value. Clearly this makes the function much easier to use since it is no longer necessary to manufacture a special `double *` object and pass it to the function somehow.

Once a typemap has been applied to a type, it stays in effect for all future occurrences of the type and name. For example, you could write the following:

> 在这个例子中，你可以看到通常在第三个参数中返回的输出值如何神奇地转换为函数返回值。显然，这使函数更易于使用，因为不再需要制造特殊的 `double *` 对象，并将其以某种方式传递给函数。
>
> 一旦将类型映射应用于类型后，它对于以后所有出现的类型和名称都保持有效。例如，你可以编写以下内容：

```
%module example
%include "typemaps.i"

%apply double *OUTPUT { double *result };

%inline %{
extern void add(double a, double b, double *result);
extern void sub(double a, double b, double *result);
extern void mul(double a, double b, double *result);
extern void div(double a, double b, double *result);
%}
...
```

In this case, the `double *OUTPUT` rule is applied to all of the functions that follow.

Typemap transformations can even be extended to multiple return values. For example, consider this code:

> 在这个例子中，`double *OUTPUT` 规则将应用于随后的所有函数。
>
> 类型映射转换甚至可以扩展为多个返回值。例如，考虑以下代码：

```
%include "typemaps.i"
%apply int *OUTPUT { int *width, int *height };

// Returns a pair (width, height)
void getwinsize(int winid, int *width, int *height);
```

In this case, the function returns multiple values, allowing it to be used like this:

> 在这个例子中，该函数返回多个值，从而可以像这样使用它：

```python
>>> w, h = genwinsize(wid)
>>> print w
400
>>> print h
300
>>>
```

It should also be noted that although the `%apply` directive is used to associate typemap rules to datatypes, you can also use the rule names directly in arguments. For example, you could write this:

> 还应该注意，尽管已经使用 `%apply` 命令将类型映射规则与数据类型相关联，但是你也可以直接在参数中使用规则名称。例如，你可以这样编写接口文件：

```
// Simple example using typemaps
%module example
%include "typemaps.i"

%{
extern void add(double a, double b, double *OUTPUT);
%}
extern void add(double a, double b, double *OUTPUT);
```

Typemaps stay in effect until they are explicitly deleted or redefined to something else. To clear a typemap, the `%clear` directive should be used. For example:

> 类型映射将一直有效，直到将其明确删除或重新定义为其他类型为止。要清除类型映射，应使用 `%clear` 命令。例如：

```
%clear double *result;      // Remove all typemaps for double *result
```

### 10.1.2 输入参数

The following typemaps instruct SWIG that a pointer really only holds a single input value:

> 以下类型映射告诉 SWIG，指针实际上仅保存一个输入值：

```c++
int *INPUT
short *INPUT
long *INPUT
unsigned int *INPUT
unsigned short *INPUT
unsigned long *INPUT
double *INPUT
float *INPUT
```

When used, it allows values to be passed instead of pointers. For example, consider this function:

> 使用时，它允许传递值而不是指针。例如，考虑以下函数：

```c
double add(double *a, double *b) {
  return *a + *b;
}
```

Now, consider this SWIG interface:

> 现在，考虑编写 SWIG 接口文件：

```
%module example
%include "typemaps.i"
...
%{
extern double add(double *, double *);
%}
extern double add(double *INPUT, double *INPUT);
```

When the function is used in the scripting language interpreter, it will work like this:

> 在脚本语言解释器中使用该函数时，它将像下面这样工作：

```python
result = add(3, 4)
```

### 10.1.3 输出参数

The following typemap rules tell SWIG that pointer is the output value of a function. When used, you do not need to supply the argument when calling the function. Instead, one or more output values are returned.

> 以下类型映射规则告诉 SWIG，指针是函数的输出值。使用时，在调用函数时不需要提供参数。而是返回一个或多个输出值。

```c
int *OUTPUT
short *OUTPUT
long *OUTPUT
unsigned int *OUTPUT
unsigned short *OUTPUT
unsigned long *OUTPUT
double *OUTPUT
float *OUTPUT
```

These methods can be used as shown in an earlier example. For example, if you have this C function :

> 可以如先前示例中所示使用这些方法。例如，如果你具有以下 C 函数：

```c
void add(double a, double b, double *c) {
  *c = a + b;
}
```

A SWIG interface file might look like this :

> SWIG 接口文件可能如下：

```
%module example
%include "typemaps.i"
...
%inline %{
extern void add(double a, double b, double *OUTPUT);
%}
```

In this case, only a single output value is returned, but this is not a restriction. An arbitrary number of output values can be returned by applying the output rules to more than one argument (as shown previously).

If the function also returns a value, it is returned along with the argument. For example, if you had this:

> 在这个例子中，仅返回单个输出值，但这不是限制。通过将输出规则应用于多个参数（如上所示），可以返回任意数量的输出值。
>
> 如果函数还返回值，则将其与参数一起返回。例如，如果你有：

```c
extern int foo(double a, double b, double *OUTPUT);
```

The function will return two values like this:

> 函数将返回两个值：

```python
iresult, dresult = foo(3.5, 2)
```

### 10.1.4 输入 / 输出参数

When a pointer serves as both an input and output value you can use the following typemaps :

> 当指针既用作输入值又用作输出值时，可以使用以下类型映射：

```c
int *INOUT
short *INOUT
long *INOUT
unsigned int *INOUT
unsigned short *INOUT
unsigned long *INOUT
double *INOUT
float *INOUT
```

A C function that uses this might be something like this:

> 有这样一个 C 函数：

```c
void negate(double *x) {
  *x = -(*x);
}
```

To make `x` function as both and input and output value, declare the function like this in an interface file :

> 要使 `x` 既作为函数的输入值又作为输出值，请在接口文件中声明如下函数：

```
%module example
%include "typemaps.i"
...
%{
extern void negate(double *);
%}
extern void negate(double *INOUT);
```

Now within a script, you can simply call the function normally :

> 现在，在脚本中可以轻松地调用函数：

```python
a = negate(3);         # a = -3 after calling this
```

One subtle point of the `INOUT` rule is that many scripting languages enforce mutability constraints on primitive objects (meaning that simple objects like integers and strings aren't supposed to change). Because of this, you can't just modify the object's value in place as the underlying C function does in this example. Therefore, the `INOUT` rule returns the modified value as a new object rather than directly overwriting the value of the original input object.

**Compatibility note :** The `INOUT` rule used to be known as `BOTH` in earlier versions of SWIG. Backwards compatibility is preserved, but deprecated.

> `INOUT` 规则的一个细微之处是，许多脚本语言对原始对象实施了可变性约束（这意味着简单的对象，如整数和字符串，不应更改）。因此，你不能像在此示例中基础 C 函数那样就地修改对象的值。因此，`INOUT` 规则将修改后的值作为新对象返回，而不是直接覆盖原始输入对象的值。
>
> **注意兼容性**：在早期版本的 SWIG 中，`INOUT` 规则以前被称为 `BOTH`。已保留向后兼容性，但已弃用。

### 10.1.5 使用不同的名称

As previously shown, the `%apply` directive can be used to apply the `INPUT`, `OUTPUT`, and `INOUT` typemaps to different argument names. For example:

> 如前所示，`%apply` 命令可用于将 `INPUT`、`OUTPUT` 和 `INOUT` 类型映射应用于不同的参数名称。例如：

```
// Make double *result an output value
%apply double *OUTPUT { double *result };

// Make Int32 *in an input value
%apply int *INPUT { Int32 *in };

// Make long *x inout
%apply long *INOUT {long *x};
```

To clear a rule, the `%clear` directive is used:

> 为了清理掉规则，要使用 `%clear` 命令：

```
%clear double *result;
%clear Int32 *in, long *x;
```

Typemap declarations are lexically scoped so a typemap takes effect from the point of definition to the end of the file or a matching `%clear` declaration.

> 类型映射声明在词法上是作用域的，因此类型映射从定义点开始到文件末尾，或匹配的 `%clear` 声明之前均生效。

## 10.2 对输入值施加约束

In addition to changing the handling of various input values, it is also possible to use typemaps to apply constraints. For example, maybe you want to insure that a value is positive, or that a pointer is non-NULL. This can be accomplished including the `constraints.i` library file.

> 除了更改对各种输入值的处理之外，还可以使用类型映射来应用约束。例如，也许你想确保一个值是正数，或者一个指针是非 NULL 的。这可以通过包含 `constraints.i` 库文件来完成。

### 10.2.1 简单约束的例子

The constraints library is best illustrated by the following interface file :

> 以下接口文件是对 `constraints.i` 库最好的说明：

```
// Interface file with constraints
%module example
%include "constraints.i"

double exp(double x);
double log(double POSITIVE);         // Allow only positive values
double sqrt(double NONNEGATIVE);     // Non-negative values only
double inv(double NONZERO);          // Non-zero values
void   free(void *NONNULL);          // Non-NULL pointers only
```

The behavior of this file is exactly as you would expect. If any of the arguments violate the constraint condition, a scripting language exception will be raised. As a result, it is possible to catch bad values, prevent mysterious program crashes and so on.

> 该文件的行为与你期望的完全一样。如果任何参数违反约束条件，将引发脚本语言异常。最终，可能会捕获错误的值，防止神秘的程序崩溃，等等。

### 10.2.2 约束方法

The following constraints are currently available

> 下列约束是当前可用的：

```
POSITIVE                     Any number > 0 (not zero)
NEGATIVE                     Any number < 0 (not zero)
NONNEGATIVE                  Any number >= 0
NONPOSITIVE                  Any number <= 0
NONZERO                      Nonzero number
NONNULL                      Non-NULL pointer (pointers only).
```

### 10.2.3 对新的数据类型应用约束

The constraints library only supports the primitive C datatypes, but it is easy to apply it to new datatypes using `%apply`. For example :

> 约束库仅支持原始 C 数据类型，但使用 `%apply` 可以很容易地将其应用于新的数据类型。例如 ：

```
// Apply a constraint to a Real variable
%apply Number POSITIVE { Real in };

// Apply a constraint to a pointer type
%apply Pointer NONNULL { Vector * };
```

The special types of "Number" and "Pointer" can be applied to any numeric and pointer variable type respectively. To later remove a constraint, the `%clear` directive can be used :

> 特殊类型的 `Number` 和 `Pointer` 可以分别应用于任何数字和指针变量类型。为了以后删除约束，可以使用 `%clear` 命令：

```
%clear Real in;
%clear Vector *;
```
