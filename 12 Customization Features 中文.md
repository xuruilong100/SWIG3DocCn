# 12 自定义功能

[TOC]

In many cases, it is desirable to change the default wrapping of particular declarations in an interface. For example, you might want to provide hooks for catching C++ exceptions, add assertions, or provide hints to the underlying code generator. This chapter describes some of these customization techniques. First, a discussion of exception handling is presented. Then, a more general-purpose customization mechanism known as "features" is described.

> 在许多情况下，用户希望更改接口文件中特定声明的默认包装。例如，你可能想要提供钩子来捕获 C++ 异常，添加断言或为底层代码生成器提供提示。本章介绍其中一些自定义技术。首先，对异常处理进行了讨论。然后，描述了一种称为“功能”的，更通用的自定义机制。

## 12.1 用 `%exception` 处理异常

The `%exception` directive allows you to define a general purpose exception handler. For example, you can specify the following:

> `%exception` 指令允许你定义通用异常处理程序。例如，你可以指定以下内容：

```
%exception {
  try {
    $action
  }
  catch (RangeError) {
    ... handle error ...
  }
}
```

How the exception is handled depends on the target language, for example, Python:

> 异常的处理方式取决于目标语言，例如在 Python 中：

```
%exception {
  try {
    $action
  }
  catch (RangeError) {
    PyErr_SetString(PyExc_IndexError, "index out-of-bounds");
    SWIG_fail;
  }
}
```

When defined, the code enclosed in braces is inserted directly into the low-level wrapper functions. The special variable `$action` is one of a few [`%exception` special variables](http://swig.org/Doc3.0/Customization.html#Customization_exception_special_variables) supported and gets replaced with the actual operation to be performed (a function call, method invocation, attribute access, etc.). An exception handler remains in effect until it is explicitly deleted. This is done by using either `%exception` or `%noexception` with no code. For example:

> 一旦定义，用大括号括起来的代码直接插入到低级包装器函数中。特殊变量 `$action` 是受支持的少数 [`%exception` 特殊变量](http://swig.org/Doc3.0/Customization.html#Customization_exception_special_variables)之一，并被替换为要执行的实际操作（函数调用、方法调用、属性访问等）。异常处理程序将一直有效，直到被明确删除。这可以通过 `%exception` 或 `%noexception` 来完成，无须编写代码。例如：

```
%exception;   // Deletes any previously defined handler
```

**Compatibility note:** Previous versions of SWIG used a special directive `%except` for exception handling. That directive is deprecated--`%exception` provides the same functionality, but is substantially more flexible.

> **注意兼容性**：早期版本的 SWIG 使用特殊指令 `%except` 进行异常处理。该指令已被弃用——`%exception` 提供相同的功能，但实质上更为灵活。

### 12.1.1 C 代码中的异常处理

C has no formal exception handling mechanism so there are several approaches that might be used. A somewhat common technique is to simply set a special error code. For example:

> C 没有正式的异常处理机制，因此可以使用几种方法来实现。某种常见的技术是简单地设置特殊的错误代码。例如：

```c
/* File : except.c */

static char error_message[256];
static int error_status = 0;

void throw_exception(char *msg) {
  strncpy(error_message, msg, 256);
  error_status = 1;
}

void clear_exception() {
  error_status = 0;
}
char *check_exception() {
  if (error_status)
    return error_message;
  else
    return NULL;
}
```

To use these functions, functions simply call `throw_exception()` to indicate an error occurred. For example :

> 要使用这些函数，函数只需调用 `throw_exception()` 来指示发生了错误。例如 ：

```c
double inv(double x) {
  if (x != 0)
    return 1.0/x;
  else {
    throw_exception("Division by zero");
    return 0;
  }
}
```

To catch the exception, you can write a simple exception handler such as the following (shown for Perl5) :

> 要捕获异常，你可以编写一个简单的异常处理程序，例如以下代码（针对 Perl5）：

```
%exception {
  char *err;
  clear_exception();
  $action
  if ((err = check_exception())) {
    croak(err);
  }
}
```

In this case, when an error occurs, it is translated into a Perl error. Each target language has its own approach to creating a runtime error/exception in and for Perl it is the `croak` method shown above.

> 在这种情况下，一旦错误发生，它将转换为 Perl 错误。每种目标语言都有自己的方法来在其中创建运行时错误/异常，对于 Perl 来说，这是上面显示的 `croak` 方法。

### 12.1.2 用 `longjmp()` 处理异常

Exception handling can also be added to C code using the `<setjmp.h>` library. Here is a minimalistic implementation that relies on the C preprocessor :

> 也可以使用 `<setjmp.h>` 库将异常处理添加到 C 代码中。这是一个依赖 C 预处理器的简约实现：

```c
/* File : except.c
   Just the declaration of a few global variables we're going to use */

#include <setjmp.h>
jmp_buf exception_buffer;
int exception_status;

/* File : except.h */
#include <setjmp.h>
extern jmp_buf exception_buffer;
extern int exception_status;

#define try if ((exception_status = setjmp(exception_buffer)) == 0)
#define catch(val) else if (exception_status == val)
#define throw(val) longjmp(exception_buffer, val)
#define finally else

/* Exception codes */

#define RangeError     1
#define DivisionByZero 2
#define OutOfMemory    3
```

Now, within a C program, you can do the following :

> 现在，在 C 程序中，你可以这样做：

```c
double inv(double x) {
  if (x)
    return 1.0/x;
  else
    throw(DivisionByZero);
}
```

Finally, to create a SWIG exception handler, write the following :

> 最终，如下可以创建一个 SWIG 异常处理器：

```
%{
#include "except.h"
%}

%exception {
  try {
    $action
  } catch(RangeError) {
    croak("Range Error");
  } catch(DivisionByZero) {
    croak("Division by zero");
  } catch(OutOfMemory) {
    croak("Out of memory");
  } finally {
    croak("Unknown exception");
  }
}
```

Note: This implementation is only intended to illustrate the general idea. To make it work better, you'll need to modify it to handle nested `try` declarations.

> 注意：此实现仅用于说明一般想法。为了使其更好地工作，你需要对其进行修改以处理嵌套的 `try` 声明。

### 12.1.3 处理 C++ 异常

Handling C++ exceptions is also straightforward. For example:

> 处理 C++ 异常也很简单。例如：

```
%exception {
  try {
    $action
  } catch(RangeError) {
    croak("Range Error");
  } catch(DivisionByZero) {
    croak("Division by zero");
  } catch(OutOfMemory) {
    croak("Out of memory");
  } catch(...) {
    croak("Unknown exception");
  }
}
```

The exception types need to be declared as classes elsewhere, possibly in a header file :

> 异常类型需要在其他地方声明为类，可能是在头文件中：

```c++
class RangeError {};
class DivisionByZero {};
class OutOfMemory {};
```

### 12.1.4 变量的异常处理器

By default all variables will ignore `%exception`, so it is effectively turned off for all variables wrappers. This applies to global variables, member variables and static member variables. The approach is certainly a logical one when wrapping variables in C. However, in C++, it is quite possible for an exception to be thrown while the variable is being assigned. To ensure `%exception` is used when wrapping variables, it needs to be 'turned on' using the `%allowexception` feature. Note that `%allowexception` is just a macro for `%feature("allowexcept")`, that is, it is a feature called `allowexcept`. Any variable which has this feature attached to it, will then use the `%exception` feature, but of course, only if there is a `%exception` attached to the variable in the first place. The `%allowexception` feature works like any other feature and so can be used globally or for selective variables.

> 默认情况下，所有变量都将忽略 `%exception`，因此对于所有变量包装器均将其关闭。这适用于全局变量、成员变量和静态成员变量。在用 C 包装变量时，这种方法当然是合乎逻辑的。但是，在 C++ 中，很可能在分配变量时引发异常。为了确保在包装变量时使用 `%exception`，需要使用 `%allowexception` 功能将其 `turned on`。请注意，`%allowexception` 只是 `%feature("allowexcept")` 的宏，也就是说，它是一个名为`allowexcept` 的功能。任何具有此功能的变量都将使用 `%exception` 功能，但是，前提当然是首先要在变量上附加 `%exception`。`%allowexception` 功能与任何其他功能一样工作，因此可以全局使用或用于选择性变量。

```
%allowexception;                // turn on globally
%allowexception Klass::MyVar;   // turn on for a specific variable

%noallowexception Klass::MyVar; // turn off for a specific variable
%noallowexception;              // turn off globally
```

### 12.1.5 定义不同的异常处理器

By default, the `%exception` directive creates an exception handler that is used for all wrapper functions that follow it. Unless there is a well-defined (and simple) error handling mechanism in place, defining one universal exception handler may be unwieldy and result in excessive code bloat since the handler is inlined into each wrapper function.

To fix this, you can be more selective about how you use the `%exception` directive. One approach is to only place it around critical pieces of code. For example:

> 默认情况下，`%exception` 指令创建一个异常处理程序，该异常处理程序用于其后的所有包装函数。除非有一个定义明确（且简单）的错误处理机制，否则定义一个通用异常处理程序可能会很麻烦，并且由于该处理程序被内联到每个包装器函数中，因此会导致代码过于膨胀。
>
> 为了解决这个问题，你可以更具选择性地使用 `%exception` 指令。一种方法是仅将其放置在关键的代码周围。例如：

```
%exception {
  ... your exception handler ...
}
/* Define critical operations that can throw exceptions here */

%exception;

/* Define non-critical operations that don't throw exceptions */
```

More precise control over exception handling can be obtained by attaching an exception handler to specific declaration name. For example:

> 通过将异常处理程序附加到特定的声明名称，可以获得对异常处理更精确的控制。例如：

```
%exception allocate {
  try {
    $action
  }
  catch (MemoryError) {
    croak("Out of memory");
  }
}
```

In this case, the exception handler is only attached to declarations named `allocate`. This would include both global and member functions. The names supplied to `%exception` follow the same rules as for `%rename` described in the section on [Ambiguity resolution and renaming](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_ambiguity_resolution_renaming). For example, if you wanted to define an exception handler for a specific class, you might write this:

> 在这种情况下，异常处理程序仅附加到名为 `allocate` 的声明。这将包括全局和成员函数。提供给 `%exception` 的名称遵循与[消歧义和重命名](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_ambiguity_resolution_renaming)一节中所述的 `%rename` 相同的规则。例如，如果你想为特定的类定义异常处理程序，则可以这样编写：

```
%exception Object::allocate {
  try {
    $action
  }
  catch (MemoryError) {
    croak("Out of memory");
  }
}
```

When a class prefix is supplied, the exception handler is applied to the corresponding declaration in the specified class as well as for identically named functions appearing in derived classes.

`%exception` can even be used to pinpoint a precise declaration when overloading is used. For example:

> 提供类前缀时，异常处理程序将应用于指定类中的相应声明，以及派生类中出现的名称相同的函数。
>
> 使用重载时，甚至可以使用 `%exception` 来精确定位声明。例如：

```
%exception Object::allocate(int) {
  try {
    $action
  }
  catch (MemoryError) {
    croak("Out of memory");
  }
}
```

Attaching exceptions to specific declarations is a good way to reduce code bloat. It can also be a useful way to attach exceptions to specific parts of a header file. For example:

> 将异常附加到特定声明是减少代码膨胀的好方法。将异常附加到头文件的特定部分也是一种有用的方法。例如：

```
%module example
%{
#include "someheader.h"
%}

// Define a few exception handlers for specific declarations
%exception Object::allocate(int) {
  try {
    $action
  }
  catch (MemoryError) {
    croak("Out of memory");
  }
}

%exception Object::getitem {
  try {
    $action
  }
  catch (RangeError) {
    croak("Index out of range");
  }
}
...
// Read a raw header file
%include "someheader.h"
```

**Compatibility note:** The `%exception` directive replaces the functionality provided by the deprecated `except` typemap. The typemap would allow exceptions to be thrown in the target language based on the return type of a function and was intended to be a mechanism for pinpointing specific declarations. However, it never really worked that well and the new %exception directive is much better.

> **注意兼容性**：`%exception` 指令替换了不推荐使用的 `except` 类型映射提供的功能。类型映射将允许根据函数的返回类型以目标语言抛出异常，并且该映射旨在成为一种精确定位特定声明的机制。但是，它从来没有真正奏效过，新的 `%exception` 指令要好得多。

### 12.1.6 `%exception` 的特殊变量

The `%exception` directive supports a few special variables which are placeholders for code substitution. The following table shows the available special variables and details what the special variables are replaced with.

> `%exception` 指令支持一些特殊变量，它们是代码替换的占位符。下表显示了可用的特殊变量，并详细说明了用哪些特殊变量替换的变量。

| `$action`             | The actual operation to be performed (a function call, method invocation, variable access, etc.)                              |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `$name`               | The C/C++ symbol name for the function.                                                                                       |
| `$symname`            | The symbol name used internally by SWIG                                                                                       |
| `$overname`           | The extra mangling used in the symbol name for overloaded method. Expands to nothing if the wrapped method is not overloaded. |
| `$wrapname`           | The language specific wrapper name (usually a C function name exported from the shared object/dll)                            |
| `$decl`               | The fully qualified C/C++ declaration of the method being wrapped without the return type                                     |
| `$fulldecl`           | The fully qualified C/C++ declaration of the method being wrapped including the return type                                   |
| `$parentclassname`    | The parent class name (if any) for a method.                                                                                  |
| `$parentclasssymname` | The target language parent class name (if any) for a method.                                                                  |

The special variables are often used in situations where method calls are logged. Exactly which form of the method call needs logging is up to individual requirements, but the example code below shows all the possible expansions, plus how an exception message could be tailored to show the C++ method declaration:

> 特殊变量通常用于记录方法调用的情况。究竟哪种形式的方法调用需要记录取决于个人要求，但是下面的示例代码显示了所有可能的扩展，以及如何定制异常消息以显示 C++ 方法声明：

```
%exception Special::something {
  log("symname: $symname");
  log("overname: $overname");
  log("wrapname: $wrapname");
  log("decl: $decl");
  log("fulldecl: $fulldecl");
  try {
    $action
  }
  catch (MemoryError) {
      croak("Out of memory in $decl");
  }
}
void log(const char *message);
struct Special {
  void something(const char *c);
  void something(int i);
};
```

Below shows the expansions for the 1st of the overloaded `something` wrapper methods for Perl:

> 下面显示了 Perl 重载的 `something` 包装器方法的第一种扩展：

```c
log("symname: Special_something");
log("overname: __SWIG_0");
log("wrapname: _wrap_Special_something__SWIG_0");
log("decl: Special::something(char const *)");
log("fulldecl: void Special::something(char const *)");
try {
  (arg1)->something((char const *)arg2);
}
catch (MemoryError) {
  croak("Out of memory in Special::something(char const *)");
}
```

### 12.1.7 使用 SWIG 异常库

The `exception.i` library file provides support for creating language independent exceptions in your interfaces. To use it, simply put an "`%include exception.i`" in your interface file. This provides a function `SWIG_exception()` that can be used to raise common scripting language exceptions in a portable manner. For example :

> `exception.i` 库文件支持在接口文件中创建独立于语言的异常。要使用它，只需在接口文件中引入 `%include exception.i`。这提供了一个函数 `SWIG_exception()`，该函数可用于以可移植的方式引发常见的脚本语言异常。例如 ：

```
// Language independent exception handler
%include exception.i

%exception {
  try {
    $action
  } catch(RangeError) {
    SWIG_exception(SWIG_ValueError, "Range Error");
  } catch(DivisionByZero) {
    SWIG_exception(SWIG_DivisionByZero, "Division by zero");
  } catch(OutOfMemory) {
    SWIG_exception(SWIG_MemoryError, "Out of memory");
  } catch(...) {
    SWIG_exception(SWIG_RuntimeError, "Unknown exception");
  }
}
```

As arguments, `SWIG_exception()` takes an error type code (an integer) and an error message string. The currently supported error types are :

> 作为参数，`SWIG_exception()` 采用错误类型代码（整数）和错误消息字符串。当前支持的错误类型是：

```
SWIG_UnknownError
SWIG_IOError
SWIG_RuntimeError
SWIG_IndexError
SWIG_TypeError
SWIG_DivisionByZero
SWIG_OverflowError
SWIG_SyntaxError
SWIG_ValueError
SWIG_SystemError
SWIG_AttributeError
SWIG_MemoryError
SWIG_NullReferenceError
```

The `SWIG_exception()` function can also be used in typemaps.

> `SWIG_exception()` 函数也可以用于类型映射。

## 12.2 对象所有权与 `%newobject`

A common problem in some applications is managing proper ownership of objects. For example, consider a function like this:

> 在某些应用程序中，一个常见的问题是正确地管理对象的所有权。例如，考虑如下函数：

```c
Foo *blah() {
  Foo *f = new Foo();
  return f;
}
```

If you wrap the function `blah()`, SWIG has no idea that the return value is a newly allocated object. As a result, the resulting extension module may produce a memory leak (SWIG is conservative and will never delete objects unless it knows for certain that the returned object was newly created).

To fix this, you can provide an extra hint to the code generator using the `%newobject` directive. For example:

> 如果包装函数 `blah()`，SWIG 不知道返回值是新分配的对象。结果，生成的扩展模块可能会产生内存泄漏（SWIG 是保守的，除非确定可以肯定返回的对象是新创建的，否则从不删除对象）。
>
> 为了解决这个问题，你可以使用 `%newobject` 指令为代码生成器提供额外的提示。例如：

```
%newobject blah;
Foo *blah();
```

`%newobject` works exactly like `%rename` and `%exception`. In other words, you can attach it to class members and parameterized declarations as before. For example:

> `%newobject` 的工作方式与 `%rename` 和 `%exception` 完全相同。换句话说，你可以像以前一样将其附加到类成员和参数化声明中。例如：

```
%newobject ::blah();                   // Only applies to global blah
%newobject Object::blah(int, double);  // Only blah(int, double) in Object
%newobject *::copy;                    // Copy method in all classes
...
```

When `%newobject` is supplied, many language modules will arrange to take ownership of the return value. This allows the value to be automatically garbage-collected when it is no longer in use. However, this depends entirely on the target language (a language module may also choose to ignore the `%newobject` directive).

Closely related to `%newobject` is a special typemap. The `newfree` typemap can be used to deallocate a newly allocated return value. It is only available on methods for which `%newobject` has been applied and is commonly used to clean-up string results. For example:

> 当提供 `%newobject` 时，许多语言模块将安排获取返回值的所有权。这样就可以在不再使用该值时自动对其进行垃圾回收。但是，这完全取决于目标语言（语言模块也可以选择忽略 `%newobject` 指令）。
>
> 与 `%newobject` 密切相关的是一个特殊的类型映射。`newfree` 类型映射可用于释放新分配的返回值。它仅在应用了 `%newobject` 的方法上可用，并且通常用于清理字符串结果。例如：

```
%typemap(newfree) char * "free($1);";
...
%newobject strdup;
...
char *strdup(const char *s);
```

In this case, the result of the function is a string in the target language. Since this string is a copy of the original result, the data returned by `strdup()` is no longer needed. The "newfree" typemap in the example simply releases this memory.

As a complement to the `%newobject`, from SWIG 1.3.28, you can use the `%delobject` directive. For example, if you have two methods, one to create objects and one to destroy them, you can use:

> 在这种情况下，函数的结果是目标语言中的字符串。由于该字符串是原始结果的副本，因此不再需要 `strdup()` 返回的数据。示例中的 `newfree` 类型映射仅释放该内存。
>
> 作为对 SWIG 1.3.28 中 `%newobject` 的补充，你可以使用 `%delobject` 指令。例如，如果你有两种方法，一种用于创建对象，另一种用于销毁它们，则可以使用：

```
%newobject create_foo;
%delobject destroy_foo;
...
Foo *create_foo();
void destroy_foo(Foo *foo);
```

or in a member method as:

> 或者在成员方法中作为：

```
%delobject Foo::destroy;

class Foo {
public:
  void destroy() { delete this;}

private:
  ~Foo();
};
```

`%delobject` instructs SWIG that the first argument passed to the method will be destroyed, and therefore, the target language should not attempt to deallocate it twice. This is similar to use the `DISOWN` typemap in the first method argument, and in fact, it also depends on the target language on implementing the 'disown' mechanism properly.

The use of `%newobject` is also integrated with reference counting and is covered in the [C++ reference counted objects](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_ref_unref) section.

**Compatibility note:** Previous versions of SWIG had a special `%new` directive. However, unlike `%newobject`, it only applied to the next declaration. For example:

> `%delobject` 指示 SWIG 传递给该方法的第一个参数将被销毁，因此，目标语言不应尝试对其进行两次分配。这类似于在第一个方法参数中使用 `DISOWN` 类型映射，实际上，它还取决于目标语言是否正确实现了 `disown` 机制。
>
> `%newobject` 的使用也与引用计数集成在一起，并在 [C++ 引用计数对象](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_ref_unref)章节中进行了介绍。
>
> **注意兼容性**：早期版本的 SWIG 具有特殊的 `%new` 指令。但是，与 `%newobject` 不同，它仅适用于下一个声明。例如：

```
%new char *strdup(const char *s);
```

For now this is still supported but is deprecated.

**How to shoot yourself in the foot:** The `%newobject` directive is not a declaration modifier like the old`%new` directive. Don't write code like this:

> 目前，仍支持此功能，但已弃用。
>
> **如何射击自己的脚**：`%newobject` 指令与旧的 `%new` 指令不同，它不是一个声明修饰符。不要写这样的代码：

```
%newobject
char *strdup(const char *s);
```

The results might not be what you expect.

> 结果可能不是你所期望的。

## 12.3 功能与 `%feature` 指令

Both `%exception` and `%newobject` are examples of a more general purpose customization mechanism known as "features." A feature is simply a user-definable property that is attached to specific declarations. Features are attached using the `%feature` directive. For example:

> `%exception` 和 `%newobject` 都是更通用的自定义机制（称为“功能”）的示例。功能只是附加到特定声明的用户可定义的属性。功能使用 `%feature` 指令附加。例如：

```
%feature("except") Object::allocate {
  try {
    $action
  }
  catch (MemoryError) {
    croak("Out of memory");
  }
}

%feature("new", "1") *::copy;
```

In fact, the `%exception` and `%newobject` directives are really nothing more than macros involving `%feature`:

> 实际上 `%exception` 和 `%newobject` 就是包含 `%feature` 的宏：

```
#define %exception %feature("except")
#define %newobject %feature("new", "1")
```

The name matching rules outlined in the [Ambiguity resolution and renaming](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_ambiguity_resolution_renaming) section applies to all `%feature` directives. In fact the `%rename` directive is just a special form of `%feature`. The matching rules mean that features are very flexible and can be applied with pinpoint accuracy to specific declarations if needed. Additionally, if no declaration name is given, a global feature is said to be defined. This feature is then attached to *every* declaration that follows. This is how global exception handlers are defined. For example:

> [消歧义析和重命名](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_ambiguity_resolution_renaming)章节中概述的名称匹配规则适用于所有 `%feature` 指令。实际上，`%rename` 指令只是 `%feature` 的一种特殊形式。匹配规则意味着功能非常灵活，如果需要，可以精确地将其应用于特定声明。此外，如果未给出声明名称，则称已定义了全局功能。然后，此功能将附加到随后的*每个*声明。这就是定义全局异常处理程序的方式。例如：

```
/* Define a global exception handler */
%feature("except") {
  try {
    $action
  }
  ...
}

... bunch of declarations ...
```

The `%feature` directive can be used with different syntax. The following are all equivalent:

> `%feature` 指令有不同的语法。以下都是等价的：

```
%feature("except") Object::method { $action };
%feature("except") Object::method %{ $action %};
%feature("except") Object::method " $action ";
%feature("except", "$action") Object::method;
```

The syntax in the first variation will generate the `{ }` delimiters used whereas the other variations will not.

> 第一个变体中的语法将生成使用的 `{ }` 分隔符，而其他变体则不。

### 12.3.1 功能属性

The `%feature` directive also accepts XML style attributes in the same way that typemaps do. Any number of attributes can be specified. The following is the generic syntax for features:

> `%feature` 指令也以与类型映射相同的方式接受 XML 样式的属性。可以指定任意数量的属性。以下是功能的通用语法：

```
%feature("name", "value", attribute1="AttributeValue1") symbol;
%feature("name", attribute1="AttributeValue1") symbol {value};
%feature("name", attribute1="AttributeValue1") symbol %{value%};
%feature("name", attribute1="AttributeValue1") symbol "value";
```

More than one attribute can be specified using a comma separated list. The Java module is an example that uses attributes in `%feature("except")`. The `throws` attribute specifies the name of a Java class to add to a proxy method's throws clause. In the following example, `MyExceptionClass` is the name of the Java class for adding to the throws clause.

> 可以使用逗号分隔的列表来指定多个属性。Java 模块是使用 `%feature("except")` 中的属性的示例。`throws` 属性指定要添加到代理方法 `throw` 子句中 Java 类的名称。在下面的示例中，`MyExceptionClass` 是要添加到 `throw` 子句中的 Java 类的名称。

```
%feature("except", throws="MyExceptionClass") Object::method {
  try {
    $action
  } catch (...) {
    ... code to throw a MyExceptionClass Java exception ...
  }
};
```

Further details can be obtained from the [Java exception handling](http://swig.org/Doc3.0/Java.html#Java_exception_handling) section.

> 更多细节可以从 [Java 异常处理](http://swig.org/Doc3.0/Java.html#Java_exception_handling)章节获得。

### 12.3.2 功能标志

Feature flags are used to enable or disable a particular feature. Feature flags are a common but simple usage of `%feature` and the feature value should be either `1` to enable or `0` to disable the feature.

> 功能标志用于启用或禁用特定功能。功能标志是 `%feature` 常见但简单的用法，值应为 `1` 则启用，为 `0` 则禁用功能。

```
%feature("featurename")          // enables feature
%feature("featurename", "1")     // enables feature
%feature("featurename", "x")     // enables feature
%feature("featurename", "0")     // disables feature
%feature("featurename", "")      // clears feature
```

Actually any value other than zero will enable the feature. Note that if the value is omitted completely, the default value becomes `1`, thereby enabling the feature. A feature is cleared by specifying no value, see [Clearing features](http://swig.org/Doc3.0/Customization.html#Customization_clearing_features). The `%immutable` directive described in the [Creating read-only variables](http://swig.org/Doc3.0/SWIG.html#SWIG_readonly_variables) section, is just a macro for `%feature("immutable")`, and can be used to demonstrates feature flags:

> 实际上，除 `0` 以外的任何值都将启用该功能。请注意，如果该值被完全省略，则默认值为 `1`，从而启用该功能。通过不指定任何值来清除功能，请参阅[清除功能](http://swig.org/Doc3.0/Customization.html#Customization_clearing_features)章节。在[创建只读变量](http://swig.org/Doc3.0/SWIG.html#SWIG_readonly_variables)章节中描述的 `%immutable` 指令只是 `%feature("immutable")` 的宏，可以用来演示功能标记：

```
                                // features are disabled by default
int red;                        // mutable

%feature("immutable");          // global enable
int orange;                     // immutable

%feature("immutable", "0");     // global disable
int yellow;                     // mutable

%feature("immutable", "1");     // another form of global enable
int green;                      // immutable

%feature("immutable", "");      // clears the global feature
int blue;                       // mutable
```

Note that features are disabled by default and must be explicitly enabled either globally or by specifying a targeted declaration. The above intersperses SWIG directives with C code. Of course you can target features explicitly, so the above could also be rewritten as:

> 请注意，默认情况下禁用功能，必须在全局范围内或通过指定目标声明来显式启用功能。上面的代码将 SWIG 指令插入 C 代码中。当然，你可以明确地定位功能，因此上面的内容也可以重写为：

```
%feature("immutable", "1") orange;
%feature("immutable", "1") green;
int red;                        // mutable
int orange;                     // immutable
int yellow;                     // mutable
int green;                      // immutable
int blue;                       // mutable
```

The above approach allows for the C declarations to be separated from the SWIG directives for when the C declarations are parsed from a C header file. The logic above can of course be inverted and rewritten as:

> 当从 C 头文件解析 C 声明时，上述方法允许将 C 声明与 SWIG 指令分开。上面的逻辑当然可以颠倒并重写为：

```
%feature("immutable", "1");
%feature("immutable", "0") red;
%feature("immutable", "0") yellow;
%feature("immutable", "0") blue;
int red;                        // mutable
int orange;                     // immutable
int yellow;                     // mutable
int green;                      // immutable
int blue;                       // mutable
```

As hinted above for `%immutable`, most feature flags can also be specified via alternative syntax. The alternative syntax is just a macro in the `swig.swg` Library file. The following shows the alternative syntax for the imaginary `featurename` feature:

> 就像上面对 `%immutable` 的提示一样，大多数功能标志也可以通过其他语法来指定。替代语法只是 `swig.swg` 库文件中的宏。下面显示了虚构的 `featurename` 功能的替代语法：

```
%featurename       // equivalent to %feature("featurename", "1") ie enables feature
%nofeaturename     // equivalent to %feature("featurename", "0") ie disables feature
%clearfeaturename  // equivalent to %feature("featurename", "")  ie clears feature
```

The concept of clearing features is discussed next.

> 接下来介绍清除功能的概念。

### 12.3.3 清除功能

A feature stays in effect until it is explicitly cleared. A feature is cleared by supplying a `%feature` directive with no value. For example `%feature("name", "")`. A cleared feature means that any feature exactly matching any previously defined feature is no longer used in the name matching rules. So if a feature is cleared, it might mean that another name matching rule will apply. To clarify, let's consider the `except` feature again (`%exception`):

> 在明确将其清除之前，功能一直有效。通过提供无值的 `%feature` 指令可以清除功能。例如 `%feature("name", "")`。清除的功能意味着名称匹配规则中不再使用与先前定义的功能完全匹配的任何功能。因此，如果一项功能被清除，则可能意味着将应用另一个名称匹配规则。为了澄清，让我们再次考虑 `except` 功能（`%exception`）：

```
// Define global exception handler
%feature("except") {
  try {
    $action
  } catch (...) {
    croak("Unknown C++ exception");
  }
}

// Define exception handler for all clone methods to log the method calls
%feature("except") *::clone() {
  try {
    logger.info("$action");
    $action
  } catch (...) {
    croak("Unknown C++ exception");
  }
}

... initial set of class declarations with clone methods ...

// clear the previously defined feature
%feature("except", "") *::clone();

... final set of class declarations with clone methods ...
```

In the above scenario, the initial set of clone methods will log all method invocations from the target language. This specific feature is cleared for the final set of clone methods. However, these clone methods will still have an exception handler (without logging) as the next best feature match for them is the global exception handler.

Note that clearing a feature is not always the same as disabling it. Clearing the feature above with `%feature("except", "") *::clone()` is not the same as specifying `%feature("except", "0") *::clone()`. The former will disable the feature for clone methods - the feature is still a better match than the global feature. If on the other hand, no global exception handler had been defined at all, then clearing the feature would be the same as disabling it as no other feature would have matched.

Note that the feature must match exactly for it to be cleared by any previously defined feature. For example the following attempt to clear the initial feature will not work:

> 在上述情况下，初始的克隆方法将记录来自目标语言的所有方法调用。最后一组克隆方法将清除此特定功能。但是，这些克隆方法仍将具有异常处理程序（不进行日志记录），因为它们的下一个最佳功能匹配是全局异常处理程序。
>
> 请注意，清除功能并不总是与禁用功能相同。用 `%feature("except", "0") *::clone()` 清除上面的功能与指定 `%feature("except", "0") *::clone()` 不同。前者将禁用克隆方法的功能，该功能仍然比全局功能更好。另一方面，如果根本没有定义全局异常处理程序，则清除该功能与禁用该功能相同，因为没有其他功能可以匹配。
>
> 请注意，该功能必须完全匹配才能被任何先前定义的功能清除。例如，以下清除初始功能的尝试将无效：

```
%feature("except") clone() { logger.info("$action"); $action }
%feature("except", "") *::clone();
```

but this will:

> 但是这可以：

```
%feature("except") clone() { logger.info("$action"); $action }
%feature("except", "") clone();
```

SWIG provides macros for disabling and clearing features. Many of these can be found in the `swig.swg`library file. The typical pattern is to define three macros; one to define the feature itself, one to disable the feature and one to clear the feature. The three macros below show this for the "except" feature:

> SWIG 提供了用于禁用和清除功能的宏。其中许多可以在 `swig.swg` 库文件中找到。典型的模式是定义三个宏。一种是定义功能本身，一种是禁用功能，另一种是清除功能。下面的三个宏针对 `except` 功能显示了这一点：

```
#define %exception      %feature("except")
#define %noexception    %feature("except", "0")
#define %clearexception %feature("except", "")
```

### 12.3.4 功能与默认参数

SWIG treats methods with default arguments as separate overloaded methods as detailed in the [default arguments](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args) section. Any `%feature` targeting a method with default arguments will apply to all the extra overloaded methods that SWIG generates if the default arguments are specified in the feature. If the default arguments are not specified in the feature, then the feature will match that exact wrapper method only and not the extra overloaded methods that SWIG generates. For example:

> SWIG 将具有默认参数的方法视为单独的重载方法，如[默认参数](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args)章节中所述。如果在功能中指定了默认参数，则以默认参数为目标的任何 `%feature` 都将应用于 SWIG 生成的所有额外重载方法。如果未在功能中指定默认参数，则功能将仅与该完全包装方法匹配，而不与 SWIG 生成的额外重载方法匹配。例如：

```
%feature("except") hello(int i=0, double d=0.0) { ... }
void hello(int i=0, double d=0.0);
```

will apply the feature to all three wrapper methods, that is:

> 将把功能应用于所有三个包装器方法，也就是：

```c++
void hello(int i, double d);
void hello(int i);
void hello();
```

If the default arguments are not specified in the feature:

> 如果功能中没有指定默认参数：

```
%feature("except") hello(int i, double d) { ... }
void hello(int i=0, double d=0.0);
```

then the feature will only apply to this wrapper method:

> 那么功能只应用这个包装器方法：

```c++
void hello(int i, double d);
```

and not these wrapper methods:

> 而不是这些方法：

```c++
void hello(int i);
void hello();
```

If [`compactdefaultargs`](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args) are being used, then the difference between specifying or not specifying default arguments in a feature is not applicable as just one wrapper is generated.

**Compatibility note:** The different behaviour of features specified with or without default arguments was introduced in SWIG-1.3.23 when the approach to wrapping methods with default arguments was changed.

> 如果使用 [`compactdefaultargs`](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args)，则仅在生成一个包装器时，在功能中指定或不指定默认参数之间的区别不适用。
>
> **注意兼容性**：当更改使用默认参数包装方法的方法时，在 SWIG-1.3.23 中引入了使用或不使用默认参数指定的功能的不同行为。

### 12.3.5 功能示例

As has been shown earlier, the intended use for the `%feature` directive is as a highly flexible customization mechanism that can be used to annotate declarations with additional information for use by specific target language modules. Another example is in the Python module. You might use `%feature`to rewrite proxy/shadow class code as follows:

> 如前所述，`%feature` 指令的预期用途是一种高度灵活的自定义机制，可用于为声明加上附加信息以供特定目标语言模块使用。另一个示例在 Python 模块中。你可以使用 `%feature` 来重写代理/影子类代码，如下所示：

```
%module example
%rename(bar_id) bar(int, double);

// Rewrite bar() to allow some nice overloading

%feature("shadow") Foo::bar(int) %{
def bar(*args):
    if len(args) == 3:
        return apply(examplec.Foo_bar_id, args)
    return apply(examplec.Foo_bar, args)
%}

class Foo {
public:
  int bar(int x);
  int bar(int x, double y);
}
```

Further details of `%feature` usage is described in the documentation for specific language modules.

> `%feature` 用法的更多详细信息在特定语言模块的文档中进行了描述。
