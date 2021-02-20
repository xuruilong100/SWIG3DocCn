[TOC]

# 14 可变数量参数

**(a.k.a, "The horror. The horror.")**

This chapter describes the problem of wrapping functions that take a variable number of arguments. For instance, generating wrappers for the C `printf()` family of functions.

This topic is sufficiently advanced to merit its own chapter. In fact, support for varargs is an often requested feature that was first added in SWIG-1.3.12. Most other wrapper generation tools have wisely chosen to avoid this issue.

> 本章描述包装带有可变数量参数函数的问题。例如，为 C 的 `printf()` 系列函数生成包装器。
>
> 这个主题已经足够高级，值得一读。实际上，对 `varargs` 的支持是一个经常要求的功能，它在 SWIG-1.3.12 中首次添加。大多数其他包装器生成工具明智地选择了避免此问题。

## 14.1 引言

Some C and C++ programs may include functions that accept a variable number of arguments. For example, most programmers are familiar with functions from the C library such as the following:

> 某些 C 和 C++ 程序可能包含接受可变数量参数的函数。例如，大多数程序员熟悉 C 库中的函数，例如：

```c
int printf(const char *fmt, ...)
int fprintf(FILE *, const char *fmt, ...);
int sprintf(char *s, const char *fmt, ...);
```

Although there is probably little practical purpose in wrapping these specific C library functions in a scripting language (what would be the point?), a library may include its own set of special functions based on a similar API. For example:

> 尽管用脚本语言包装这些特定的 C 库函数可能没有什么实际目的（这样做的目的是什么？），但是库可能基于相似的 API 包括自己的一组特殊函数。例如：

```c
int traceprintf(const char *fmt, ...);
```

In this case, you may want to have some kind of access from the target language.

Before describing the SWIG implementation, it is important to discuss the common uses of varargs that you are likely to encounter in real programs. Obviously, there are the `printf()` style output functions as shown. Closely related to this would be `scanf()` style input functions that accept a format string and a list of pointers into which return values are placed. However, variable length arguments are also sometimes used to write functions that accept a NULL-terminated list of pointers. A good example of this would be a function like this:

> 在这种情况下，你可能希望从目标语言进行某种访问。
>
> 在描述 SWIG 实现之前，重要的是讨论在实际程序中可能遇到的可变参数的常见用法。显然，有 `printf()` 样式的输出函数。与之密切相关的是 `scanf()` 风格的输入函数，它们接受格式字符串和放置返回值的指针列表。但是，有时也使用变长参数来编写接受以 `NULL` 结尾的指针列表的函数。一个很好的例子就是这样的函数：

```c
int execlp(const char *path, const char *arg1, ...);
...

/* Example */
execlp("ls", "ls", "-l", NULL);
```

In addition, varargs is sometimes used to fake default arguments in older C libraries. For instance, the low level `open()` system call is often declared as a varargs function so that it will accept two or three arguments:

> 此外，有时使用可变参数伪造老旧 C 库中的默认参数。例如，底层的 `open()` 系统调用通常被声明为可变参数函数，以便它将接受两个或三个参数：

```c
int open(const char *path, int oflag, ...);
...

/* Examples */
f = open("foo", O_RDONLY);
g = open("bar", O_WRONLY | O_CREAT, 0644);
```

Finally, to implement a varargs function, recall that you have to use the C library functions defined in `<stdarg.h>`. For example:

> 最后，要实现可变参数函数，请记住，你必须使用在 `<stdarg.h>` 中定义的 C 库函数。例如：

```c
List make_list(const char *s, ...) {
  va_list ap;
  List    x;
  ...
  va_start(ap, s);
  while (s) {
    x.append(s);
    s = va_arg(ap, const char *);
  }
  va_end(ap);
  return x;
}
```

## 14.2 问题

Generating wrappers for a variable length argument function presents a number of special challenges. Although C provides support for implementing functions that receive variable length arguments, there are no functions that can go in the other direction. Specifically, you can't write a function that dynamically creates a list of arguments and which invokes a varargs function on your behalf.

Although it is possible to write functions that accept the special type `va_list`, this is something entirely different. You can't take a `va_list` structure and pass it in place of the variable length arguments to another varargs function. It just doesn't work.

The reason this doesn't work has to do with the way that function calls get compiled. For example, suppose that your program has a function call like this:

> 为可变长度参数函数生成包装器提出了许多特殊的挑战。尽管 C 支持实现接收可变长度参数的函数，但是没有函数可以以另一种形式实现。具体来说，你不能编写一个函数来动态创建一个参数列表，并以此代表你调用可变参数函数。
>
> 尽管可以编写接受特殊类型 `va_list` 的函数，但这是完全不同的。你不能采用 `va_list` 结构体，并将其代替可变长度参数传递给另一个可变参数函数。就是行不通。
>
> 这不起作用的原因与函数调用的编译方式有关。例如，假设你的程序具有如下函数调用：

```c
printf("Hello %s. Your number is %d\n", name, num);
```

When the compiler looks at this, it knows that you are calling `printf()` with exactly three arguments. Furthermore, it knows that the number of arguments as well are their types and sizes is *never* going to change during program execution. Therefore, this gets turned to machine code that sets up a three-argument stack frame followed by a call to `printf()`.

In contrast, suppose you attempted to make some kind of wrapper around `printf()` using code like this:

> 当编译器查看此内容时，它知道你正使用三个参数调用 `printf()`。此外，它知道参数的数量以及它们的类型和大小在程序执行期间永远不会改变。因此，这变成了设置三个参数的堆栈框架并随后调用 `printf()` 的机器代码。
>
> 相比之下，假设你尝试使用如下代码对 `printf()` 进行某种包装：

```c
int wrap_printf(const char *fmt, ...) {
  va_list ap;
  va_start(ap, fmt);
  ...
  printf(fmt, ap);
  ...
  va_end(ap);
};
```

Although this code might compile, it won't do what you expect. This is because the call to `printf()` is compiled as a procedure call involving only two arguments. However, clearly a two-argument configuration of the call stack is completely wrong if your intent is to pass an arbitrary number of arguments to the real `printf()`. Needless to say, it won't work.

Unfortunately, the situation just described is exactly the problem faced by wrapper generation tools. In general, the number of passed arguments will not be known until run-time. To make matters even worse, you won't know the types and sizes of arguments until run-time as well. Needless to say, there is no obvious way to make the C compiler generate code for a function call involving an unknown number of arguments of unknown types.

In theory, it *is* possible to write a wrapper that does the right thing. However, this involves knowing the underlying ABI for the target platform and language as well as writing special purpose code that manually constructed the call stack before making a procedure call. Unfortunately, both of these tasks require the use of inline assembly code. Clearly, that's the kind of solution you would much rather avoid.

With this nastiness in mind, SWIG provides a number of solutions to the varargs wrapping problem. Most of these solutions are compromises that provide limited varargs support without having to resort to assembly language. However, SWIG can also support real varargs wrapping (with stack-frame manipulation) if you are willing to get hands dirty. Keep reading.

> 尽管此代码可能会编译，但不会执行你期望的操作。这是因为对 `printf()` 的调用被编译为仅涉及两个参数的过程调用。但是，如果你的意图是将任意数量的参数传递给实际的 `printf()`，则调用堆栈的两个参数的配置显然是完全错误的。不用说，它不会起作用。
>
> 不幸的是，刚才描述的情况正是包装器生成工具所面临的问题。通常，直到运行时才知道传递的参数的数量。更糟糕的是，直到运行时，你才知道参数的类型和大小。不用说，没有明显的方法可以使 C 编译器为函数调用生成代码，该函数调用涉及数量未知的未知类型的参数。
>
> 从理论上讲，有可能写一个做正确事情的包装器。但是，这涉及了解目标平台和语言的底层 ABI，以及编写在进行过程调用之前手动构建调用堆栈的特殊用途代码。不幸的是，这两个任务都需要使用内联汇编代码。显然，这是你宁愿避免的解决方案。
>
> 考虑到这种麻烦，SWIG 为可变参数包装问题提供了许多解决方案。这些解决方案中的大多数都是折衷方案，可提供有限的可变参数支持，而不必诉诸汇编语言。但是，如果你愿意弄脏手的话，SWIG 还可以支持真正的可变参数包装（使用堆栈框架操作）。继续阅读。

## 14.3 默认可变参数支持

When variable length arguments appear in an interface, the default behavior is to drop the variable argument list entirely, replacing them with a single NULL pointer. For example, if you had this function,

> 当可变长度参数出现在接口中时，默认行为是完全删除可变参数列表，并用单个空指针替换它们。例如，如果你具有如下函数，

```c
void traceprintf(const char *fmt, ...);
```

it would be wrapped as if it had been declared as follows:

> 它就按照如下声明被包装：

```c
void traceprintf(const char *fmt);
```

When the function is called inside the wrappers, it is called as follows:

> 当在包装器中被调用时，它被如下调用：

```c
traceprintf(arg1, NULL);
```

Arguably, this approach seems to defeat the whole point of variable length arguments. However, this actually provides enough support for many simple kinds of varargs functions to still be useful, however it does come with a caveat. For instance, you could make function calls like this (in Python):

> 可以说，这种方法似乎抹杀了可变长度参数的全部特点。但是，这实际上为许多简单类型的可变参数函数提供了足够的支持，使其仍然有用，但是附上一个忠告。例如，你可以这样进行函数调用（在 Python 中）：

```python
>>> traceprintf("Hello World")
>>> traceprintf("Hello %s. Your number is %d\n" % (name, num))
>>> traceprintf("Your result is 90%%.")
```

Notice how string formatting is being done in Python instead of C. The caveat is the strings passed must be safe to use in C though. For example if `name` was to contain a `%` it should be double escaped in order to avoid unpredictable behaviour:

> 请注意，如何在 Python 而不是 C 中完成字符串格式化。需要注意的是，尽管如此，传递的字符串必须可以在 C 中安全使用。例如，如果 `name` 包含 `%`，则应对其进行两次转义，以避免不可预测的行为：

```python
>>> traceprintf("Your result is 90%.\n")  # unpredictable behaviour
>>> traceprintf("Your result is 90%%.\n") # good
```

Read on for further solutions.

> 继续阅读以了解进一步的解决方案。

## 14.4 用 `%varargs` 替换参数

Instead of dropping the variable length arguments, an alternative approach is to replace `(...)` with a set of suitable arguments. SWIG provides a special `%varargs` directive that can be used to do this. For example,

> 除了删除可变长度参数之外，另一种方法是用一组合适的参数替换 `(...)`。SWIG 提供了一个特殊的 `%varargs` 指令，可用于执行此操作。例如，

```c
%varargs(int mode = 0) open;
...
int open(const char *path, int oflags, ...);
```

is equivalent to this:

> 等价于：

```c
int open(const char *path, int oflags, int mode = 0);
```

In this case, `%varargs` is simply providing more specific information about the extra arguments that might be passed to a function. If the arguments to a varargs function are of uniform type, `%varargs` can also accept a numerical argument count as follows:

> 在这种情况下，`%varargs` 只是提供有关可能传递给函数的额外参数的更具体的信息。如果可变参数函数的参数是统一类型，则 `%varargs` 也可以接受数字参数计数，如下所示：

```c
%varargs(3, char *str = NULL) execlp;
...
int execlp(const char *path, const char *arg, ...);
```

and is effectively seen as:

> 这样看起来更清晰：

```c
int execlp(const char *path, const char *arg, 
           char *str1 = NULL, 
           char *str2 = NULL, 
           char *str3 = NULL);
```

This would wrap `execlp()` as a function that accepted up to 3 optional arguments. Depending on the application, this may be more than enough for practical purposes.

The handling of [default arguments](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args) can be changed via the `compactdefaultargs` feature. If this feature is used, for example

> 这会将 `execlp()` 包装为一个函数，该函数最多接受 3 个可选参数。根据实际应用，这可能绰绰有余。
>
> [默认参数](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args)的处理可以通过 `compactdefaultargs` 特性进行更改。例如，如果使用此功能

```c
%feature("compactdefaultargs") execlp;
%varargs(3, char *str = NULL) execlp;
...
int execlp(const char *path, const char *arg, ...);
```

a call from the target language which does not provide the maximum number of arguments, such as,`execlp("a", "b", "c")` will generate C code which includes the missing default values, that is, `execlp("a", "b", "c", NULL, NULL)`. If `compactdefaultargs` is not used, then the generated code will be `execlp("a", "b", "c")`. The former is useful for helping providing a sentinel to terminate the argument list. However, this is not guaranteed, for example when a user passes a non-NULL value for all the parameters. When using `compactdefaultargs` it is possible to guarantee the NULL sentinel is passed through the, `numinputs=0` [`in` typemap attribute](http://swig.org/Doc3.0/Typemaps.html#Typemaps_nn26), naming the **last parameter**. For example,

> 来自目标语言的调用不提供参数的最大数量，例如 `execlp(a, b, c)` 会生成 C 代码，其中包含缺少的默认值，即 `execlp(a, b, c, NULL, NULL)`。如果未使用 `compactdefaultargs`，则生成的代码将为 `execlp(a, b, c)`。前者可用于帮助提供前哨以终止参数列表。但是，例如当用户为所有参数传递非 `NULL` 值时，就不能保证。当使用 `compactdefaultargs` 时，可以保证 `NULL` 标记通过 `numinputs = 0` [`in` 类型映射属性](http://swig.org/Doc3.0/Typemaps.html#Typemaps_nn26)传递， 命名“最后一个参数”。例如，

```c
%feature("compactdefaultargs") execlp;
%varargs(3, char *str = NULL) execlp;
%typemap(in, numinputs=0) char *str3 ""
...
int execlp(const char *path, const char *arg, ...);
```

Note that `str3` is the name of the last argument, as we have used `%varargs` with 3. Now `execlp("a", "b", "c", "d", "e")` will result in an error as one too many arguments has been passed, as now only 2 additional `str` arguments can be passed with the 3rd one always using the specified default `NULL`.

Argument replacement is most appropriate in cases where the types of the extra arguments are uniform and the maximum number of arguments are known. Argument replacement is not as useful when working with functions that accept mixed argument types such as `printf()`. Providing general purpose wrappers to such functions presents special problems (covered shortly).

> 注意，`str3` 是最后一个参数的名称，因为我们已经将 `%varargs` 与 3 一起使用了。现在 `execlp("a", "b", "c", "d", "e")` 由于传递了太多的参数而导致错误，因为现在两个 `str` 参数可以被传递，而第三个参数只能使用指定的默认值 `NULL`。
>
> 在额外参数的类型统一且已知最大参数数量的情况下，参数替换最合适。当使用诸如 `printf()` 之类接受混合参数类型的函数时，参数替换没有那么有用。为此类函数提供通用包装器会带来一些特殊问题（很快就会提到）。

## 14.5 可变参数和类型映射

Variable length arguments may be used in typemap specifications. For example:

> 可变长度参数可以在类型映射规范中使用。例如：

```
%typemap(in) (...) {
    // Get variable length arguments (somehow)
    ...
}

%typemap(in) (const char *fmt, ...) {
    // Multi-argument typemap
}
```

However, this immediately raises the question of what "type" is actually used to represent `(...)`. For lack of a better alternative, the type of `(...)` is set to `void *`. Since there is no way to dynamically pass arguments to a varargs function (as previously described), the `void *` argument value is intended to serve as a place holder for storing some kind of information about the extra arguments (if any). In addition, the default behavior of SWIG is to pass the `void *` value as an argument to the function. Therefore, you could use the pointer to hold a valid argument value if you wanted.

To illustrate, here is a safer version of wrapping `printf()` in Python:

> 但是，这立即引起了一个问题，即实际上使用什么“类型”来表示 `(...)`。由于缺少更好的选择，将 `(...)` 的类型设置为 `void *`。由于无法将参数动态传递给可变参数函数（如前所述），因此 `void *` 参数值旨在用作占位符，用于存储有关额外参数的某种信息（如果有的话）。另外，SWIG 的默认行为是将 `void *` 值作为参数传递给函数。因此，如果需要，可以使用指针保存有效的参数值。
>
> 为了说明这一点，这是在 Python 中包装 `printf()` 的一个更安全的版本：

```
%typemap(in) (const char *fmt, ...) {
    $1 = "%s";                                /* Fix format string to %s */
    $2 = (void *) PyString_AsString($input);  /* Get string argument */
};
...
int printf(const char *fmt, ...);
```

In this example, the format string is implicitly set to `"%s"`. This prevents a program from passing a bogus format string to the extension. Then, the passed input object is decoded and placed in the `void *` argument defined for the `(...)` argument. When the actual function call is made, the underlying wrapper code will look roughly like this:

> 在此示例中，格式字符串被隐式设置为 `%s`。这样可以防止程序将伪格式字符串传递给扩展。然后，对传递的输入对象进行解码，并将其放置在为 `(...)` 定义的 `void *` 参数中。进行实际的函数调用时，底层包装器代码将大致如下所示：

```c
wrap_printf() {
  char *arg1;
  void *arg2;
  int   result;

  arg1 = "%s";
  arg2 = (void *) PyString_AsString(arg2obj);
  ...
  result = printf(arg1, arg2);
  ...
}
```

Notice how both arguments are passed to the function and it does what you would expect.

The next example illustrates a more advanced kind of varargs typemap. Disclaimer: this requires special support in the target language module and is not guaranteed to work with all SWIG modules at this time. It also starts to illustrate some of the more fundamental problems with supporting varargs in more generality.

If a typemap is defined for any form of `(...)`, many SWIG modules will generate wrappers that accept a variable number of arguments as input and will make these arguments available in some form. The precise details of this depends on the language module being used (consult the appropriate chapter for more details). However, suppose that you wanted to create a Python wrapper for the `execlp()` function shown earlier. To do this using a typemap instead of using `%varargs`, you might first write a typemap like this:

> 请注意如何将两个参数都传递给函数，并且函数可以实现你所期望的。
>
> 下一个示例说明了更高级的可变参数类型映射。免责声明：这需要目标语言模块的特殊支持，并且目前不保证可以与所有 SWIG 模块一起使用。它还开始更广泛地说明支持可变参数的一些更基本的问题。
>
> 如果为任何形式的 `(...)` 定义了一个类型映射，则许多 SWIG 模块将生成包装器，这些包装器接受可变数量的参数作为输入，并使这些参数以某种形式可用。具体的细节取决于所使用的语言模块（有关更多细节，请咨询相应的章节）。但是，假设你想为前面显示的 `execlp()` 函数创建一个 Python 包装器。要使用类型映射而不是 `%varargs` 来做到这一点，你可以首先编写一个如下的类型映射：

```
%typemap(in) (...)(char *vargs[10]) {
  int i;
  int argc;
  for (i = 0; i < 10; i++) vargs[i] = 0;
  argc = PyTuple_Size(varargs);
  if (argc > 10) {
    PyErr_SetString(PyExc_ValueError, "Too many arguments");
    SWIG_fail;
  }
  for (i = 0; i < argc; i++) {
    PyObject *pyobj = PyTuple_GetItem(varargs, i);
    char *str = 0;
%#if PY_VERSION_HEX>=0x03000000
    PyObject *pystr;
    if (!PyUnicode_Check(pyobj)) {
      PyErr_SetString(PyExc_ValueError, "Expected a string");
      SWIG_fail;
    }
    pystr = PyUnicode_AsUTF8String(pyobj);
    str = strdup(PyBytes_AsString(pystr));
    Py_XDECREF(pystr);
%#else  
    if (!PyString_Check(pyobj)) {
      PyErr_SetString(PyExc_ValueError, "Expected a string");
      SWIG_fail;
    }
    str = PyString_AsString(pyobj);
%#endif
    vargs[i] = str;
  }
  $1 = (void *)vargs;
}

%typemap(freearg) (...) {
%#if PY_VERSION_HEX>=0x03000000
  int i;
  for (i = 0; i < 10; i++) {
    free(vargs$argnum[i]);
  }
%#endif
}
```

In the `in` typemap, the special variable `varargs` is a tuple holding all of the extra arguments passed (this is specific to the Python module). The typemap then pulls this apart and sticks the values into the array of strings `args`. Then, the array is assigned to `$1` (recall that this is the `void *` variable corresponding to `(...)`). However, this assignment is only half of the picture----clearly this alone is not enough to make the function work. The 'freearg' typemap cleans up memory allocated in the `in` typemap; this code is generated to be called after the `execlp` function is called. To patch everything up, you have to rewrite the underlying action code using the `%feature` directive like this:

> 在 `in` 类型映射中，特殊变量 `varargs` 是一个元组，其中包含所有传递的额外参数（特定于 Python 模块）。然后，类型映射将其拆开并将值粘贴到字符串 `args` 数组中。然后，将该数组分配给 `$1`（请记住，这是与`(...)` 对应的 `void *` 变量）。但是，此分配只是图片的一半，显然，仅此一项不足以使该函数正常工作。`freearg` 类型映射清理在 `in` 类型映射中分配的内存；调用 `execlp` 函数后，将生成此代码以供调用。为了解决问题，你必须使用 `%feature` 指令来重写底层动作代码，如下所示：

```
%feature("action") execlp {
  char **vargs = (char **) arg3;
  result = execlp(arg1, arg2, vargs[0], vargs[1], vargs[2], vargs[3], vargs[4],
                  vargs[5], vargs[6], vargs[7], vargs[8], vargs[9], NULL);
}

int execlp(const char *path, const char *arg, ...);
```

This patches everything up and creates a function that more or less works. However, don't try explaining this to your coworkers unless you know for certain that they've had several cups of coffee. If you really want to elevate your guru status and increase your job security, continue to the next section.

> 这会解决所有问题，并创建一个或多或少起作用的函数。但是，请勿尝试向你的同事解释这一点，除非你确定他们已经喝了几杯咖啡。如果你确实想提升专家的地位并提高工作安全性，请继续下一节。

## 14.6 用 `libffi` 包装可变参数

All of the previous examples have relied on features of SWIG that are portable and which don't rely upon any low-level machine-level details. In many ways, they have all dodged the real issue of variable length arguments by recasting a varargs function into some weaker variation with a fixed number of arguments of known types. In many cases, this works perfectly fine. However, if you want more generality than this, you need to bring out some bigger guns.

One way to do this is to use a special purpose library such as libffi (http://www.sourceware.org/libffi/). libffi is a library that allows you to dynamically construct call-stacks and invoke procedures in a relatively platform independent manner. Details about the library can be found in the libffi distribution and are not repeated here.

To illustrate the use of libffi, suppose that you *really* wanted to create a wrapper for `execlp()` that accepted *any* number of arguments. To do this, you might make a few adjustments to the previous example. For example:

> 前面的所有示例均依赖于 SWIG 的特性，这些特性具有可移植性，并且不依赖于任何低级的机器级详细信息。在许多方面，它们都通过使用固定数量的已知类型的参数将可变参数函数重铸为一些较弱的变体，从而避免了变长参数的实际问题。在许多情况下，这非常正常。但是，如果你想要比这更多的通用性，则需要一些更大的枪。
>
> 一种实现方法是使用专用库，例如 `libffi`（<http://www.sourceware.org/libffi/>）。`libffi` 是一个库，可让你以相对平台无关的方式动态构造调用栈和调用过程。有关库的详细信息可以在 `libffi` 发行版中找到，这里不再赘述。
>
> 为了说明 `libffi` 的用法，假设你确实想为 `execlp()` 创建一个包装器，该包装器接受任意数量的参数。为此，你可以对上一个示例进行一些调整。例如：

```
/* Take an arbitrary number of extra arguments and place into an array
   of strings */

%typemap(in) (...) {
  char **argv;
  int    argc;
  int    i;

  argc = PyTuple_Size(varargs);
  argv = (char **) malloc(sizeof(char *)*(argc+1));
  for (i = 0; i < argc; i++) {
    PyObject *o = PyTuple_GetItem(varargs, i);
    if (!PyString_Check(o)) {
      free(argv);
      PyErr_SetString(PyExc_ValueError, "Expected a string");
      SWIG_fail;
    }
    argv[i] = PyString_AsString(o);
  }
  argv[i] = NULL;
  $1 = (void *) argv;
}

/* Rewrite the function call, using libffi */    

%feature("action") execlp {
  int       i, vc;
  ffi_cif   cif;
  ffi_type  **types;
  void      **values;
  char      **args;

  vc = PyTuple_Size(varargs);
  types  = (ffi_type **) malloc((vc+3)*sizeof(ffi_type *));
  values = (void **) malloc((vc+3)*sizeof(void *));
  args   = (char **) arg3;

  /* Set up path parameter */
  types[0] = &ffi_type_pointer;
  values[0] = &arg1;
  
  /* Set up first argument */
  types[1] = &ffi_type_pointer;
  values[1] = &arg2;

  /* Set up rest of parameters */
  for (i = 0; i <= vc; i++) {
    types[2+i] = &ffi_type_pointer;
    values[2+i] = &args[i];
  }
  if (ffi_prep_cif(&cif, FFI_DEFAULT_ABI, vc+3,
                   &ffi_type_uint, types) == FFI_OK) {
    ffi_call(&cif, (void (*)()) execlp, &result, values);
  } else {
    free(types);
    free(values);
    free(arg3);
    PyErr_SetString(PyExc_RuntimeError, "Whoa!!!!!");
    SWIG_fail;
  }
  free(types);
  free(values);
  free(arg3);
}

/* Declare the function. Whew! */
int execlp(const char *path, const char *arg1, ...);
```

Looking at this example, you may start to wonder if SWIG is making life any easier. Given the amount of code involved, you might also wonder why you didn't just write a hand-crafted wrapper! Either that or you're wondering "why in the hell am I trying to wrap this varargs function in the first place?!?" Obviously, those are questions you'll have to answer for yourself.

As a more extreme example of libffi, here is some code that attempts to wrap `printf()`,

> 看这个例子，你可能会开始怀疑 SWIG 是否使生活变得更加轻松。考虑到所涉及的代码量，你可能还想知道为什么不只是编写手工包装器！ 要么，要么你想知道“为什么我非要首先包装这个可变参数函数不可？！？”显然，这些是你必须自己回答的问题。
>
> 作为 `libffi` 的一个更极端的示例，下面是一些代码，试图包装 `printf()`，

```
/* A wrapper for printf() using libffi */

%{
/* Structure for holding passed arguments after conversion */
  typedef struct {
    int type;
    union {
      int    ivalue;
      double dvalue;
      void   *pvalue;
    } val;
  } vtype;
  enum { VT_INT, VT_DOUBLE, VT_POINTER };
%}

%typemap(in) (const char *fmt, ...) {
  vtype *argv;
  int    argc;
  int    i;

  /* Format string */
  $1 = PyString_AsString($input);

  /* Variable length arguments */
  argc = PyTuple_Size(varargs);
  argv = (vtype *) malloc(argc*sizeof(vtype));
  for (i = 0; i < argc; i++) {
    PyObject *o = PyTuple_GetItem(varargs, i);
    if (PyInt_Check(o)) {
      argv[i].type = VT_INT;
      argv[i].val.ivalue = PyInt_AsLong(o);
    } else if (PyFloat_Check(o)) {
      argv[i].type = VT_DOUBLE;
      argv[i].val.dvalue = PyFloat_AsDouble(o);
    } else if (PyString_Check(o)) {
      argv[i].type = VT_POINTER;
      argv[i].val.pvalue = (void *) PyString_AsString(o);
    } else {
      free(argv);
      PyErr_SetString(PyExc_ValueError, "Unsupported argument type");
      return NULL;
    }
  }
  $2 = (void *) argv;
}

/* Rewrite the function call using libffi */    
%feature("action") printf {
  int       i, vc;
  ffi_cif   cif;
  ffi_type  **types;
  void      **values;
  vtype     *args;

  vc = PyTuple_Size(varargs);
  types  = (ffi_type **) malloc((vc+1)*sizeof(ffi_type *));
  values = (void **) malloc((vc+1)*sizeof(void *));
  args   = (vtype *) arg2;

  /* Set up fmt parameter */
  types[0] = &ffi_type_pointer;
  values[0] = &arg1;

  /* Set up rest of parameters */
  for (i = 0; i < vc; i++) {
    switch(args[i].type) {
    case VT_INT:
      types[1+i] = &ffi_type_uint;
      values[1+i] = &args[i].val.ivalue;
      break;
    case VT_DOUBLE:
      types[1+i] = &ffi_type_double;
      values[1+i] = &args[i].val.dvalue;
      break;
    case VT_POINTER:
      types[1+i] = &ffi_type_pointer;
      values[1+i] = &args[i].val.pvalue;
      break;
    default:
      abort();    /* Whoa! We're seriously hosed */
      break;   
    }
  }
  if (ffi_prep_cif(&cif, FFI_DEFAULT_ABI, vc+1,
                   &ffi_type_uint, types) == FFI_OK) {
    ffi_call(&cif, (void (*)()) printf, &result, values);
  } else {
    free(types);
    free(values);
    free(args);
    PyErr_SetString(PyExc_RuntimeError, "Whoa!!!!!");
    SWIG_fail;
  }
  free(types);
  free(values);
  free(args);
}

/* The function */
int printf(const char *fmt, ...);
```

Much to your amazement, it even seems to work if you try it:

> 让你惊奇的是，它看起来是可以运行的：

```python
>>> import example
>>> example.printf("Grade: %s   %d/60 = %0.2f%%\n", "Dave", 47, 47.0*100/60)
Grade: Dave   47/60 = 78.33%
>>>
```

Of course, there are still some limitations to consider:

> 当然，依然有一些限制要考虑：

```python
>>> example.printf("la de da de da %s", 42)
Segmentation fault (core dumped)
```

And, on this note, we leave further exploration of libffi to the reader as an exercise. Although Python has been used as an example, most of the techniques in this section can be extrapolated to other language modules with a bit of work. The only details you need to know is how the extra arguments are accessed in each target language. For example, in the Python module, we used the special `varargs` variable to get these arguments. Modules such as Tcl8 and Perl5 simply provide an argument number for the first extra argument. This can be used to index into an array of passed arguments to get values. Please consult the chapter on each language module for more details.

> 并且，在本节中，我们将 `libffi` 的进一步探索留给读者作为练习。尽管以 Python 为例，但本部分中的大多数技术都可以通过一些工作外推到其他语言模块中。你需要知道的唯一详细信息是如何在每种目标语言中访问额外的参数。例如，在 Python 模块中，我们使用了特殊的 `varargs` 变量来获取这些参数。诸如 Tcl8 和 Perl5 之类的模块只是为第一个额外的参数提供了一个参数编号。这可以用于索引传递的参数数组以获取值。有关更多详细信息，请查阅每个语言模块上的章节。

## 14.7 包装 `va_list`

Closely related to variable length argument wrapping, you may encounter functions that accept a parameter of type `va_list`. For example:

> 与可变长度参数包装密切相关，你可能会遇到接受类型为 `va_list` 的参数的函数。例如：

```c
int vprintf(const char *fmt, va_list ap);
```

As far as we know, there is no obvious way to wrap these functions with SWIG. This is because there is no documented way to assemble the proper `va_list` structure (there are no C library functions to do it and the contents of va_list are opaque). Not only that, the contents of a `va_list` structure are closely tied to the underlying call-stack. It's not clear that exporting a `va_list` would have any use or that it would work at all.

A workaround can be implemented by writing a simple varargs C wrapper and then using the techniques discussed earlier in this chapter for varargs. Below is a simple wrapper for `vprintf` renamed so that it can still be called as `vprintf` from your target language. The `%varargs` used in the example restricts the function to taking one string argument.

> 据我们所知，没有明显的方法用 SWIG 包装这些函数。这是因为没有成文的方法来组装适当的 `va_list` 结构体（没有 C 库函数可以执行此操作，并且 `va_list` 的内容是不透明的）。不仅如此，`va_list` 结构体的内容与底层调用堆栈紧密相关。目前尚不清楚导出 `va_list` 是否会有用或根本无法使用。
>
> 可以通过编写一个简单的可变参数 C 包装器，然后使用本章前面讨论的可变参数技术来实现一种解决方法。下面是一个重命名为 `vprintf` 的简单包装，以便你仍可以从目标语言中将其称为 `vprintf`。本示例中使用的 `%varargs` 将函数限制为采用一个字符串参数。

```
%{
int vprintf(const char *fmt, va_list ap);
%}

%varargs(const char *) my_vprintf;
%rename(vprintf) my_vprintf;

%inline %{
int my_vprintf(const char *fmt, ...) {
  va_list ap;
  int result;

  va_start(ap, fmt);
  result = vprintf(fmt, ap);
  va_end(ap);
  return result;
}
%}
```

## 14.8 C++ 的问题

Wrapping of C++ member functions that accept a variable number of arguments presents a number of challenges. By far, the easiest way to handle this is to use the `%varargs` directive. This is portable and it fully supports classes much like the `%rename` directive. For example:

> 包装接受可变数量参数的 C++ 成员函数提出了许多挑战。到目前为止，最简单的方法是使用 `%varargs` 指令。这是可移植的，它完全支持类，类似于 `%rename` 指令。例如：

```c++
%varargs (10, char * = NULL) Foo::bar;

class Foo {
public:
  virtual void bar(char *arg, ...);   // gets varargs above
};

class Spam: public Foo {
public:
  virtual void bar(char *arg, ...);   // gets varargs above
};
```

`%varargs` also works with constructors, operators, and any other C++ programming construct that accepts variable arguments.

Doing anything more advanced than this is likely to involve a serious world of pain. In order to use a library like libffi, you will need to know the underlying calling conventions and details of the C++ ABI. For instance, the details of how `this` is passed to member functions as well as any hidden arguments that might be used to pass additional information. These details are implementation specific and may differ between compilers and even different versions of the same compiler. Also, be aware that invoking a member function is further complicated if it is a virtual method. In this case, invocation might require a table lookup to obtain the proper function address (although you might be able to obtain an address by casting a bound pointer to a pointer to function as described in the C++ ARM section 18.3.4).

If you do decide to change the underlying action code, be aware that SWIG always places the `this` pointer in `arg1`. Other arguments are placed in `arg2`, `arg3`, and so forth. For example:

> `%varargs` 也可以与构造函数、运算符以及任何其他接受可变参数的 C++ 编程构造一起使用。
>
> 做任何比这更高级的事情都可能会带来严重的痛苦。为了使用 `libffi` 之类的库，你将需要了解底层的调用约定和 C++ ABI 的详细信息。例如，如何将 `this` 传递给成员函数的细节，以及可能用于传递其他信息的任何隐藏参数。这些细节是特定于实现的，并且在编译器之间，甚至同一编译器的不同版本之间可能有所不同。另外，请注意，如果成员函数是虚方法，则调用成员函数会更加复杂。在这种情况下，调用可能需要查找表以获得正确的函数地址（尽管你可以通过将绑定指针强制转换为指向函数的指针来获得地址，如 C++ ARM 第 18.3.4 节中所述）。
>
> 如果你确实决定更改基础操作代码，请注意 SWIG 始终将 `this` 指针放在 `arg1` 中。其他参数放在 `arg2`、`arg3` 等中。例如：

```
%feature("action") Foo::bar {
  ...
  result = arg1->bar(arg2, arg3, etc.);
  ...
}
```

Given the potential to shoot yourself in the foot, it is probably easier to reconsider your design or to provide an alternative interface using a helper function than it is to create a fully general wrapper to a varargs C++ member function.

> 鉴于有可能使自己陷入困境，重新创建设计或使用辅助函数提供替代接口可能比为可变参数 C++ 成员函数创建完全通用的包装容易。

## 14.9 讨论

This chapter has provided a number of techniques that can be used to address the problem of variable length argument wrapping. If you care about portability and ease of use, the `%varargs` directive is probably the easiest way to tackle the problem. However, using typemaps, it is possible to do some very advanced kinds of wrapping.

One point of discussion concerns the structure of the libffi examples in the previous section. Looking at that code, it is not at all clear that this is the easiest way to solve the problem. However, there are a number of subtle aspects of the solution to consider--mostly concerning the way in which the problem has been decomposed. First, the example is structured in a way that tries to maintain separation between wrapper-specific information and the declaration of the function itself. The idea here is that you might structure your interface like this:

> 本章提供了许多可用于解决可变长度参数包装问题的技术。如果你关心可移植性和易用性，那么 `%varargs` 指令可能是解决该问题的最简单方法。但是，使用类型映射，可以进行一些非常高级的包装。
>
> 讨论的重点是上一节中 `libffi` 示例的结构。查看该代码，根本不清楚这是解决问题的最简单方法。但是，该解决方案有许多微妙的方面需要考虑，主要是解决问题的方式。首先，该示例的结构试图保持包装专用信息和函数本身的声明之间的分隔。这里的想法是，你可以像这样构造你的接口：

```
%typemap(const char *fmt, ...) {
  ...
}
%feature("action") traceprintf {
  ...
}

/* Include some header file with traceprintf in it */
%include "someheader.h"
```

Second, careful scrutiny will reveal that the typemaps involving `(...)` have nothing whatsoever to do with the libffi library. In fact, they are generic with respect to the way in which the function is actually called. This decoupling means that it will be much easier to consider other library alternatives for making the function call. For instance, if libffi wasn't supported on a certain platform, you might be able to use something else instead. You could use conditional compilation to control this:

> 其次，仔细检查将发现涉及 `(...)` 的类型映射与 `libffi` 库没有任何关系。实际上，它们在函数实际调用方式方面是通用的。这种解耦意味着考虑其他库替代方法来进行函数调用会容易得多。例如，如果某个平台上不支持 `libffi`，则可以使用其他功能。你可以使用条件编译来控制此操作：

```
#ifdef USE_LIBFFI
%feature("action") printf {
  ...
}
#endif
#ifdef USE_OTHERFFI
%feature("action") printf {
...
}
#endif
```

Finally, even though you might be inclined to just write a hand-written wrapper for varargs functions, the techniques used in the previous section have the advantage of being compatible with all other features of SWIG such as exception handling.

As a final word, some C programmers seem to have the assumption that the wrapping of variable length argument functions is an easily solved problem. However, this section has hopefully dispelled some of these myths. All things being equal, you are better off avoiding variable length arguments if you can. If you can't avoid them, please consider some of the simple solutions first. If you can't live with a simple solution, proceed with caution. At the very least, make sure you carefully read the section "A7.3.2 Function Calls" in Kernighan and Ritchie and make sure you fully understand the parameter passing conventions used for varargs. Also, be aware of the platform dependencies and reliability issues that this will introduce. Good luck.

> 最后，即使你可能只想为可变参数函数编写手写包装器，上一节中使用的技术仍具有与 SWIG 的所有其他功能（例如异常处理）兼容的优点。
>
> 最后，一些 C 程序员似乎假设可变长度参数函数的包装是一个容易解决的问题。但是，本节有望消除其中的一些神话。在所有条件都相同的情况下，最好尽可能避免使用可变长度参数。如果你无法避免它们，请首先考虑一些简单的解决方案。如果你不能使用简单的解决方案，请谨慎操作。至少，请确保你仔细阅读了 Kernighan 和 Ritchie 中的“A7.3.2函数调用”章节，并确保你完全了解用于可变参数的参数传递约定。另外，请注意将要引入的平台依赖性和可靠性问题。祝你好运。
