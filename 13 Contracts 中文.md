[TOC]

# 13 约定

A common problem that arises when wrapping C libraries is that of maintaining reliability and checking for errors. The fact of the matter is that many C programs are notorious for not providing error checks. Not only that, when you expose the internals of an application as a library, it often becomes possible to crash it simply by providing bad inputs or using it in a way that wasn't intended.

This chapter describes SWIG's support for software contracts. In the context of SWIG, a contract can be viewed as a runtime constraint that is attached to a declaration. For example, you can easily attach argument checking rules, check the output values of a function and more. When one of the rules is violated by a script, a runtime exception is generated rather than having the program continue to execute.

> 包装 C 库时出现的常见问题是维护可靠性和检查错误。事实是，许多 C 程序因不提供错误检查而臭名昭著。不仅如此，当你将应用程序的内部结构公开为一个库时，通常可能会因提供错误的输入或以非预期的方式使用它而使其崩溃。
>
> 本章介绍 SWIG 对软件约定的支持。在 SWIG 的上下文中，约定可以视为附加到声明的运行时约束。例如，你可以轻松地附加参数检查规则，检查函数的输出值等等。如果脚本违反了其中一个规则，则会生成运行时异常，而不是让程序继续执行。

## 13.1 `%contract` 指令

Contracts are added to a declaration using the %contract directive. Here is a simple example:

> 使用 `%contract` 指令将约定添加到声明中。这是一个简单的示例：

```
%contract sqrt(double x) {
require:
  x >= 0;
ensure:
  sqrt >= 0;
}

...
double sqrt(double);
```

In this case, a contract is being added to the `sqrt()` function. The `%contract` directive must always appear before the declaration in question. Within the contract there are two sections, both of which are optional. The `require:` section specifies conditions that must hold before the function is called. Typically, this is used to check argument values. The `ensure:` section specifies conditions that must hold after the function is called. This is often used to check return values or the state of the program. In both cases, the conditions that must hold must be specified as boolean expressions.

In the above example, we're simply making sure that sqrt() returns a non-negative number (if it didn't, then it would be broken in some way).

Once a contract has been specified, it modifies the behavior of the resulting module. For example:

> 在这种情况下，将约定添加到 `sqrt()` 函数中。`%contract` 指令必须始终出现在相关声明之前。约定中有两个部分，两者都是可选的。`require:` 部分指定了在调用函数之前必须满足的条件。通常，它用于检查参数值。`ensure:` 部分指定了在调用函数后必须满足的条件。通常用于检查返回值或程序状态。在这两种情况下，必须满足的条件都必须指定为布尔表达式。
>
> 在上面的示例中，我们只是确保 `sqrt()` 返回一个非负数（如果不返回，那么它将以某种方式被破坏）。
>
> 指定约定后，它将修改结果模块的行为。例如：

```python
>>> example.sqrt(2)
1.4142135623730951
>>> example.sqrt(-2)
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
RuntimeError: Contract violation: require: (arg1>=0)
>>>
```

## 13.2 `%contract` 与类

The `%contract` directive can also be applied to class methods and constructors. For example:

> `%contract` 指令也可以应用于类方法和构造函数。例如：

```
%contract Foo::bar(int x, int y) {
require:
  x > 0;
ensure:
  bar > 0;
}

%contract Foo::Foo(int a) {
require:
  a > 0;
}

class Foo {
public:
  Foo(int);
  int bar(int, int);
};
```

The way in which `%contract` is applied is exactly the same as the `%feature` directive. Thus, any contract that you specified for a base class will also be attached to inherited methods. For example:

> `%contract` 的应用方式与 `%feature` 指令完全相同。因此，你为基类指定的任何约定也将附加到继承的方法。例如：

```c++
class Spam : public Foo {
public:
  int bar(int, int);    // Gets contract defined for Foo::bar(int, int)
};
```

In addition to this, separate contracts can be applied to both the base class and a derived class. For example:

> 除此之外，可以将单独的约定同时应用于基类和派生类。例如：

```
%contract Foo::bar(int x, int) {
require:
  x > 0;
}

%contract Spam::bar(int, int y) {
require:
  y > 0;
}

class Foo {
public:
  int bar(int, int);   // Gets Foo::bar contract.
};

class Spam : public Foo {
public:
  int bar(int, int);   // Gets Foo::bar and Spam::bar contract
};
```

When more than one contract is applied, the conditions specified in a "require:" section are combined together using a logical-AND operation. In other words conditions specified for the base class and conditions specified for the derived class all must hold. In the above example, this means that both the arguments to `Spam::bar` must be positive.

> 当应用多个约定时，使用逻辑与运算将 `require:` 部分中指定的条件组合在一起。换句话说，为基类指定的条件和为派生类指定的条件都必须成立。在上面的示例中，这意味着 `Spam::bar` 的两个参数都必须为正。

## 13.3 约定集成与 `%aggregate_check`

Consider an interface file that contains the following code:

> 考虑包含以下代码的接口文件：

```c++
#define  UP     1
#define  DOWN   2
#define  RIGHT  3
#define  LEFT   4

void move(SomeObject *, int direction, int distance);
```

One thing you might want to do is impose a constraint on the direction parameter to make sure it's one of a few accepted values. To do that, SWIG provides an easy to use macro %aggregate_check() that works like this:

> 你可能想做的一件事是对 `direction` 参数施加约束，以确保它是少数几个可接受的值之一。为此，SWIG 提供了一个易于使用的宏 `%aggregate_check()`，其工作方式如下：

```
%aggregate_check(int, check_direction, UP, DOWN, LEFT, RIGHT);
```

This merely defines a utility function of the form

> 只是定义一个工具函数

```c
int check_direction(int x);
```

That checks the argument x to see if it is one of the values listed. This utility function can be used in contracts. For example:

> 这将检查参数 `x` 以查看其是否为列出的值之一。可以在约定中使用此实用程序函数。例如：

```
%aggregate_check(int, check_direction, UP, DOWN, RIGHT, LEFT);

%contract move(SomeObject *, int direction, in) {
require:
  check_direction(direction);
}

#define  UP     1
#define  DOWN   2
#define  RIGHT  3
#define  LEFT   4

void move(SomeObject *, int direction, int distance);
```

Alternatively, it can be used in typemaps and other directives. For example:

> 另外，它可以在类型映射和其他指令中使用。例如：

```
%aggregate_check(int, check_direction, UP, DOWN, RIGHT, LEFT);

%typemap(check) int direction {
  if (!check_direction($1)) SWIG_exception(SWIG_ValueError, "Bad direction");
}

#define  UP     1
#define  DOWN   2
#define  RIGHT  3
#define  LEFT   4

void move(SomeObject *, int direction, int distance);
```

Regrettably, there is no automatic way to perform similar checks with enums values. Maybe in a future release.

> 遗憾的是，没有自动的方法可以对枚举值执行类似的检查。也许在将来的版本中可以。

## 13.4 注意事项

Contract support was implemented by Songyan (Tiger) Feng and first appeared in SWIG-1.3.20.

> 约定支持由 Songyan（Tiger）Feng 实施，并首次出现在 SWIG-1.3.20 中。
