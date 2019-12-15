# 6 SWIG 和 C++

[TOC]

This chapter describes SWIG's support for wrapping C++. As a prerequisite, you should first read the chapter [SWIG Basics](http://swig.org/Doc3.0/SWIG.html#SWIG) to see how SWIG wraps ANSI C. Support for C++ builds upon ANSI C wrapping and that material will be useful in understanding this chapter.

> 本章介绍了 SWIG 对包装 C++ 的支持。作为准备条件，你应首先阅读 [SWIG 基础知识](http://swig.org/Doc3.0/SWIG.html#SWIG)一章，了解 SWIG 如何包装 ANSI C。SWIG 对 C++ 的支持建立在对 ANSI C 包装的基础上，而且这部分资料将有助于理解本章。

## 6.1 关于包装 C++

Because of its complexity and the fact that C++ can be difficult to integrate with itself let alone other languages, SWIG only provides support for a subset of C++ features. Fortunately, this is now a rather large subset.

In part, the problem with C++ wrapping is that there is no semantically obvious (or automatic ) way to map many of its advanced features into other languages. As a simple example, consider the problem of wrapping C++ multiple inheritance to a target language with no such support. Similarly, the use of overloaded operators and overloaded functions can be problematic when no such capability exists in a target language.

A more subtle issue with C++ has to do with the way that some C++ programmers think about programming libraries. In the world of SWIG , you are really trying to create binary-level software components for use in other languages. In order for this to work, a "component" has to contain real executable instructions and there has to be some kind of binary linking mechanism for accessing its functionality. In contrast, C++ has increasingly relied upon generic programming and templates for much of its functionality. Although templates are a powerful feature, they are largely orthogonal to the whole notion of binary components and libraries. For example, an STL `vector` does not define any kind of binary object for which SWIG can just create a wrapper. To further complicate matters, these libraries often utilize a lot of behind the scenes magic in which the semantics of seemingly basic operations (e.g., pointer dereferencing, procedure call, etc.) can be changed in dramatic and sometimes non-obvious ways. Although this "magic" may present few problems in a C++-only universe, it greatly complicates the problem of crossing language boundaries and provides many opportunities to shoot yourself in the foot. You will just have to be careful.

> 由于其复杂性以及 C++ 难以将其自身与其他语言集成的事实，SWIG 仅支持 C++ 功能的一个子集。幸运的是，现在这是一个相当大的子集。
>
> 在某种程度上，C++ 包装的问题在于没有语义上明显（或自动）的方式将其许多高级功能映射到其他语言。举个简单的例子，考虑将 C++ 多继承包装到不支持多继承的目标语言。类似地，当目标语言中不支持重载时，使用重载运算符和重载函数可能是有问题的。
>
> C++ 的一个更微妙的问题与某些 C++ 程序员对程序库的思考方式有关。在 SWIG 的世界中，你正在尝试创建用于其他语言的二进制级软件组件。为了使其工作，“组件”必须包含真正的可执行指令，并且必须有某种二进制链接机制来访问其功能。相比之下，C++ 越来越依赖泛型编程和模板来实现其大部分功能。虽然模板是一个强大的功能，但它们在很大程度上与二进制组件和库的整个概念正交。例如，STL `vector` 没有定义任何二进制对象供 SWIG 为其创建包装器。为了使问题进一步复杂化，这些库通常利用许多幕后魔法，其中看似基本操作（例如，指针解引用、过程调用等）的语义可以以戏剧性，且有时非显而易见的方式改变。虽然这种“魔法”可能在 C++ 中只会造成一点问题——但它会使跨语言变得非常复杂，并且可能会使你射到自己的脚（shoot yourself in the foot）。你必须要小心。

## 6.2 方法

To wrap C++, SWIG uses a layered approach to code generation. At the lowest level, SWIG generates a collection of procedural ANSI-C style wrappers. These wrappers take care of basic type conversion, type checking, error handling, and other low-level details of the C++ binding. These wrappers are also sufficient to bind C++ into any target language that supports built-in procedures. In some sense, you might view this layer of wrapping as providing a C library interface to C++. On top of the low-level procedural (flattened) interface, SWIG generates proxy classes that provide a natural object-oriented (OO) interface to the underlying code. The proxy classes are typically written in the target language itself. For instance, in Python, a real Python class is used to provide a wrapper around the underlying C++ object.

It is important to emphasize that SWIG takes a deliberately conservative and non-intrusive approach to C++ wrapping. SWIG does not encapsulate C++ classes inside a special C++ adaptor, it does not rely upon templates, nor does it add in additional C++ inheritance when generating wrappers. The last thing that most C++ programs need is even more compiler magic. Therefore, SWIG tries to maintain a very strict and clean separation between the implementation of your C++ application and the resulting wrapper code. You might say that SWIG has been written to follow the principle of least surprise--it does not play sneaky tricks with the C++ type system, it doesn't mess with your class hierarchies, and it doesn't introduce new semantics. Although this approach might not provide the most seamless integration with C++, it is safe, simple, portable, and debuggable.

Some of this chapter focuses on the low-level procedural interface to C++ that is used as the foundation for all language modules. Keep in mind that the target languages also provide the high-level OO interface via proxy classes. More detailed coverage can be found in the documentation for each target language.

> 为了包装 C++ ，SWIG 使用分层方法来生成代码。在最底层，SWIG 生成一组 ANSI C 程序样式的包装器。这些包装器负责基本类型转换、类型检查、错误处理以及 C++ 绑定的其他低级细节。这些包装器足以将 C++ 绑定到任何支持内置程序的目标语言。从某种意义上说，你可以将这个包装层视为为 C++ 提供 C 库接口。在低级程序（扁平化）接口之上，SWIG 生成代理类，为低级代码提供自然的面向对象（OO）接口。代理类通常用目标语言本身编写。例如，在 Python 中，一个真正的 Python 类用于提供底层 C++ 对象的包装器。
>
> 重要的是要强调 SWIG 采用故意保守且非侵入性的 C++ 包装方法。SWIG 不会将 C++ 类封装在特殊的 C++ 适配器中，它不依赖于模板，也不会在生成包装器时添加额外的 C++ 继承。大多数 C++ 程序需要的最后一件事就是编译魔法。因此，SWIG 尝试在 C++ 应用程序的实现和生成的包装器代码之间保持非常严格和清晰的分离。你可能会说 SWIG 的编写遵循最少惊喜的原则——它不会与 C++ 类型系统一起玩狡猾的技巧，它不会破坏你的类层次结构，并且它不会引入新的语义。虽然这种方法可能无法提供与 C++ 最无缝的集成，但它是安全、简单、可移植和可调试的。
>
> 本章的一些内容侧重于 C++ 的低级程序接口，作为所有语言模块的基础使用。请记住，目标语言还通过代理类提供高级 OO 接口。可以在每种目标语言的文档中找到更详细的介绍。

## 6.3 支持的 C++ 功能

SWIG currently supports most C++ features including the following:

> SWIG 目前支持的 C++ 功能如下：

* Classes
* Constructors and destructors
* Virtual functions
* Public inheritance (including multiple inheritance)
* Static functions
* Function and method overloading
* Operator overloading for many standard operators
* References
* Templates (including specialization and member templates)
* Pointers to members
* Namespaces
* Default parameters
* Smart pointers

The following C++ features are not currently supported:
* Overloaded versions of certain operators (new, delete, etc.)

As a rule of thumb, SWIG should not be used on raw C++ source files, use header files only.

SWIG's C++ support is an ongoing project so some of these limitations may be lifted in future releases. However, we make no promises. Also, submitting a bug report is a very good way to get problems fixed (wink).

> 目前不支持下列 C++ 功能：
>
>* 某些运算符的重载（`new`、`delete` 等）
>
> 根据经验，SWIG 不应该用于原始 C++ 源文件，只能使用头文件。
>
> SWIG 的 C++ 支持是一个持续的项目，因此在未来的版本中可能会解除其中一些限制。但是，我们不做任何承诺。此外，提交错误报告是解决问题的一个非常好的方法（眨眨眼）。

## 6.4 命令行选项与编译

When wrapping C++ code, it is critical that SWIG be called with the `-c++` option. This changes the way a number of critical features such as memory management are handled. It also enables the recognition of C++ keywords. Without the `-c++` flag, SWIG will either issue a warning or a large number of syntax errors if it encounters C++ code in an interface file.

When compiling and linking the resulting wrapper file, it is normal to use the C++ compiler. For example:

> 在包装 C++ 代码时，使用 `-c++` 选项调用 SWIG 至关重要。这改变了处理许多关键功能的方式，如内存管理等。它还可以识别 C++ 关键字。如果没有 `-c++` 标志，如果 SWIG 在接口文件中遇到 C++ 代码，它将发出警告或大量语法错误。
>
> 在编译和链接生成的包装器文件时，通常使用 C++ 编译器。例如：

```
$ swig -c++ -tcl example.i
$ c++ -fPIC -c example_wrap.cxx
$ c++ example_wrap.o $(OBJS) -o example.so
```

Unfortunately, the process varies slightly on each platform. Make sure you refer to the documentation on each target language for further details. The SWIG Wiki also has further details.

**Compatibility Note:** Early versions of SWIG generated just a flattened low-level C style API to C++ classes by default. The  commandline option is recognised by many target languages and will generate just this interface as in earlier versions.

In order to provide a natural mapping from C++ classes to the target language classes, SWIG's target languages mostly wrap C++ classes with special proxy classes. These proxy classes are typically implemented in the target language itself. For example, if you're building a Python module, each C++ class is wrapped by a Python proxy class. Or if you're building a Java module, each C++ class is wrapped by a Java proxy class.

> 不幸的是，每个操作系统的流程略有不同。请务必参阅每种目标语言的文档以获取更多详细信息。SWIG Wiki 还有更多细节。
>
> **注意兼容性**：默认情况下，早期版本的 SWIG 只为 C++ 类生成了扁平的低级 C 风格 API。命令行选项被许多目标语言识别，并且将像早期版本一样生成此接口。
>
> 为了提供从 C++ 类到目标语言类的自然映射，SWIG 的目标语言主要用特殊的代理类包装 C++ 类。这些代理类通常以目标语言本身实现。例如，如果你正在构建 Python 模块，则每个 C++ 类都由 Python 代理类包装。或者，如果你正在构建 Java 模块，则每个 C++ 类都由 Java 代理类包装。

### 6.5.1 代理类的构造

Proxy classes are always constructed as an extra layer of wrapping that uses low-level accessor functions. To illustrate, suppose you had a C++ class like this:

> 代理类始终构造为使用低级访问器函数的额外包装层。为了说明这一点，假设你有一个这样的 C++ 类：

```c++
class Foo {
  public:
    Foo();
    ~Foo();
    int  bar(int x);
    int  x;
};
```

Using C++ as pseudocode, a proxy class looks something like this:

> 使用 C++ 作为伪代码，代理类看起来像这样：

```c++
class FooProxy {
  private:
    Foo    *self;
  public:
    FooProxy() {
      self = new_Foo();
    }
    ~FooProxy() {
      delete_Foo(self);
    }
    int bar(int x) {
      return Foo_bar(self, x);
    }
    int x_get() {
      return Foo_x_get(self);
    }
    void x_set(int x) {
      Foo_x_set(self, x);
    }
};
```

Of course, always keep in mind that the real proxy class is written in the target language. For example, in Python, the proxy might look roughly like this:

> 当然，请记住，真正的代理类是用目标语言编写的。例如，在 Python 中，代理可能看起来大致如下：

```python
class Foo:
    def __init__(self):
        self.this = new_Foo()
    def __del__(self):
        delete_Foo(self.this)
    def bar(self, x):
        return Foo_bar(self.this, x)
    def __getattr__(self, name):
        if name == 'x':
            return Foo_x_get(self.this)
        ...
    def __setattr__(self, name, value):
        if name == 'x':
            Foo_x_set(self.this, value)
        ...
```

Again, it's important to emphasize that the low-level accessor functions are always used by the proxy classes. Whenever possible, proxies try to take advantage of language features that are similar to C++. This might include operator overloading, exception handling, and other features.

> 同样，重点强调代理类总是使用低级访问器函数。只要有可能，代理类就会尝试利用与 C++ 类似的语言功能。这可能包括运算符重载、异常处理和其他功能。

### 6.5.2 代理类中的资源管理

A major issue with proxies concerns the memory management of wrapped objects. Consider the following C++ code:

> 代理类的主要问题涉及对包装对象的内存管理。考虑以下 C++ 代码：

```c++
class Foo {
public:
  Foo();
  ~Foo();
  int bar(int x);
  int x;
};

class Spam {
public:
  Foo *value;
  ...
};
```

Consider some script code that uses these classes:

> 考虑使用这些类的脚本代码：

```python
f = Foo()               # Creates a new Foo
s = Spam()              # Creates a new Spam
s.value = f             # Stores a reference to f inside s
g = s.value             # Returns stored reference
g = 4                   # Reassign g to some other value
del f                   # Destroy f
```

Now, ponder the resulting memory management issues. When objects are created in the script, the objects are wrapped by newly created proxy classes. That is, there is both a new proxy class instance and a new instance of the underlying C++ class. In this example, both `f` and `s` are created in this way. However, the statement `s.value` is rather curious---when executed, a pointer to `f` is stored inside another object. This means that the scripting proxy class *AND* another C++ class share a reference to the same object. To make matters even more interesting, consider the statement `g = s.value`. When executed, this creates a new proxy class `g` that provides a wrapper around the C++ object stored in `s.value`. In general, there is no way to know where this object came from---it could have been created by the script, but it could also have been generated internally. In this particular example, the assignment of `g` results in a second proxy class for `f`. In other words, a reference to `f` is now shared by two proxy classes *and* a C++ class.

Finally, consider what happens when objects are destroyed. In the statement, `g=4`, the variable `g` is reassigned. In many languages, this makes the old value of `g` available for garbage collection. Therefore, this causes one of the proxy classes to be destroyed. Later on, the statement `del f` destroys the other proxy class. Of course, there is still a reference to the original object stored inside another C++ object. What happens to it? Is the object still valid?

To deal with memory management problems, proxy classes provide an API for controlling ownership. In C++ pseudocode, ownership control might look roughly like this:

> 现在，思考由此产生的内存管理问题。在脚本中创建对象时，对象将由新创建的代理类包装。也就是说，既有新的代理类实例又有底层 C++ 类的新实例。在这个例子中，`f` 和 `s` 都是以这种方式创建的。但是，语句 `s.value` 相当奇怪——执行时，指向 `f` 的指针存储在另一个对象中。这意味着脚本代理类*以及*另一个 C++ 类共享对同一对象的引用。为了使事情变得更有趣，请考虑语句 `g = s.value`。执行时，这将创建一个新的代理类对象 `g`，它提供了存储在 `s.value` 中的 C++ 对象的包装器。通常，无法知道此对象的来源——它可能是由脚本创建的，但也可能是在内部生成的。在这个特定的例子中，`g` 的赋值导致了 `f` 的第二个代理类对象。换句话说，对 `f` 的引用现在由两个代理类*以及*一个 C++ 类共享。
>
> 最后，考虑一下对象被销毁时会发生什么。在声明中，`g = 4`，变量 `g` 被重新分配。在许多语言中，这使得 `g` 的旧值可被垃圾收集。因此，这会导致其中一个代理类对象被销毁。后来，语句 `del f` 销毁了另一个代理类。当然，仍然存在对存储在另一个 C++ 对象中的原始对象的引用。怎么回事？对象仍然有效吗？
>
> 为了处理内存管理问题，代理类提供了用于控制所有权的 API。在 C++ 伪代码中，所有权控制可能看起来大致如下：

```c++
class FooProxy {
  public:
    Foo *self;
    int thisown;

    FooProxy() {
      self = new_Foo();
      thisown = 1;       // Newly created object
    }
    ~FooProxy() {
      if (thisown) delete_Foo(self);
    }
    ...
    // Ownership control API
    void disown() {
      thisown = 0;
    }
    void acquire() {
      thisown = 1;
    }
};

class FooPtrProxy: public FooProxy {
public:
  FooPtrProxy(Foo *s) {
    self = s;
    thisown = 0;
  }
};

class SpamProxy {
  ...
  FooProxy *value_get() {
    return FooPtrProxy(Spam_value_get(self));
  }
  void value_set(FooProxy *v) {
    Spam_value_set(self, v->self);
    v->disown();
  }
  ...
};
```

Looking at this code, there are a few central features:

* Each proxy class keeps an extra flag to indicate ownership. C++ objects are only destroyed if the ownership flag is set.
* When new objects are created in the target language, the ownership flag is set.
* When a reference to an internal C++ object is returned, it is wrapped by a proxy class, but the proxy class does not have ownership.
* In certain cases, ownership is adjusted. For instance, when a value is assigned to the member of a class, ownership is lost.
* Manual ownership control is provided by special `disown()` and `acquire()` methods.

Given the tricky nature of C++ memory management, it is impossible for proxy classes to automatically handle every possible memory management problem. However, proxies do provide a mechanism for manual control that can be used (if necessary) to address some of the more tricky memory management problems.

> 查看此代码，有一些核心功能：
>
> * 每个代理类都有一个额外的标志来表示所有权。只有设置了所有权标志，才会销毁 C++ 对象。
> * 以目标语言创建新对象时，将设置所有权标志。
> * 当返回对内部 C++ 对象的引用时，它由代理类包装，但代理类没有所有权。
> * 在某些情况下，所有权会进行调整。例如，将值分配给类的成员时，所有权将丢失。
> * 手动所有权控制由特殊的 `disown()` 和 `acquire()` 方法提供。
>
> 鉴于 C++ 内存管理的棘手性，代理类无法自动处理每个可能的内存管理问题。但是，代理确实提供了一种手动控制机制，可以用（如果需要）来解决一些更棘手的内存管理问题。

### 6.5.3 语言特定的细节

Language specific details on proxy classes are contained in the chapters describing each target language. This chapter has merely introduced the topic in a very general way.

> 代理类的语言特定详细信息包含在描述每种目标语言的章节中。本章仅以非常概括的方式介绍该主题。

## 6.6 简单 C++ 包装

The following code shows a SWIG interface file for a simple C++ class.

> 下面的代码显示了一个简单 C++ 类的 SWIG 接口文件。

```
%module list
%{
#include "list.h"
%}

// Very simple C++ example for linked list

class List {
public:
  List();
  ~List();
  int  search(char *value);
  void insert(char *);
  void remove(char *);
  char *get(int n);
  int  length;
static void print(List *l);
};
```

To generate wrappers for this class, SWIG first reduces the class to a collection of low-level C-style accessor functions which are then used by the proxy classes.

> 要为此类生成包装器，SWIG 首先将类降级为低级 C 风格访问器函数的集合，然后由代理类使用。

### 6.6.1 构造函数和析构函数

C++ constructors and destructors are translated into accessor functions such as the following :

> C++ 构造函数和析构函数被翻译成如下访问器函数：

```c++
List * new_List(void) {
  return new List;
}
void delete_List(List *l) {
  delete l;
}
```

### 6.6.2 默认构造函数、拷贝构造函数和隐式析构函数

Following the C++ rules for implicit constructor and destructors, SWIG will automatically assume there is one even when they are not explicitly declared in the class interface.

In general then:

* If a C++ class does not declare any explicit constructor, SWIG will automatically generate a wrapper for one.
* If a C++ class does not declare an explicit copy constructor, SWIG will automatically generate a wrapper for one if the `%copyctor` is used.
* If a C++ class does not declare an explicit destructor, SWIG will automatically generate a wrapper for one.

And as in C++, a few rules that alters the previous behavior:

* A default constructor is not created if a class already defines a constructor with arguments.
* Default constructors are not generated for classes with pure virtual methods or for classes that inherit from an abstract class, but don't provide definitions for all of the pure methods.
* A default constructor is not created unless all base classes support a default constructor.
* Default constructors and implicit destructors are not created if a class defines them in a `private` or `protected` section.
* Default constructors and implicit destructors are not created if any base class defines a non-public default constructor or destructor.

SWIG should never generate a default constructor, copy constructor or default destructor wrapper for a class in which it is illegal to do so. In some cases, however, it could be necessary (if the complete class declaration is not visible from SWIG , and one of the above rules is violated) or desired (to reduce the size of the final interface) by manually disabling the implicit constructor/destructor generation.

To manually disable these, the `%nodefaultctor` and `%nodefaultdtor` [feature flag](http://swig.org/Doc3.0/Customization.html#Customization_feature_flags) directives can be used. Note that these directives only affects the implicit generation, and they have no effect if the default/copy constructors or destructor are explicitly declared in the class interface.

For example:

> 遵循 C++ 的隐式构造函数和析构函数规则，即使未在类接口中显式声明它们，SWIG 也会自动假设存在一个。
>
> 一般来说：
>
> * 如果 C++ 类没有声明任何显式构造函数，SWIG 将自动生成一个包装器。
> * 如果 C++ 类没有声明显式拷贝构造函数，如果使用 `%copyctor`，SWIG 将自动生成一个包装器。
> * 如果 C++ 类没有声明显式析构函数，SWIG 将自动生成一个包装器。
>
> 就像在 C++ 中一样，有一些改变上述行为的规则：
>
> * 如果类已经定义了带参数的构造函数，则不会创建默认构造函数。
> * 不为具有纯虚方法的类，或从抽象类继承的类生成默认构造函数，但不为所有纯虚方法提供定义。
> * 除非所有基类都支持默认构造函数，否则不会创建默认构造函数。
> * 如果类在 `private` 或 `protected` 部分中定义默认构造函数和隐式析构函数，则不会创建它们。
> * 如果任何基类定义非公有默认构造函数或析构函数，则不会创建默认构造函数和隐式析构函数。
>
> SWIG 永远不应该为一个类非法生成默认构造函数、拷贝构造函数或默认析构函数包装器。但是，在某些情况下，可能有必要（如果从 SWIG 看不到完整的类声明，并且违反了上述规则之一）或者希望（通过手动禁用隐式构造函数来减小最终接口的大小）析构函数生成。
>
> 要手动禁用这些，可以使用 `%nodefaultctor` 和 `%nodefaultdtor` [功能标志](http://swig.org/Doc3.0/Customization.html#Customization_feature_flags)指令。请注意，这些指令仅影响隐式生成，如果在类接口中显式声明了默认、拷贝构造函数或析构函数，则它们不起作用。
>
> 例如：

```
%nodefaultctor Foo;  // Disable the default constructor for class Foo.
class Foo {          // No default constructor is generated, unless one is declared
...
};
class Bar {          // A default constructor is generated, if possible
...
};
```

The directive `%nodefaultctor` can also be applied "globally", as in:

> `%nodefaultctor` 指令可以“全局”使用，例如：

```
%nodefaultctor; // Disable creation of default constructors
class Foo {     // No default constructor is generated, unless one is declared
...
};
class Bar {
public:
  Bar();        // The default constructor is generated, since one is declared
};
%clearnodefaultctor; // Enable the creation of default constructors again
```

The corresponding `%nodefaultdtor` directive can be used to disable the generation of the default or implicit destructor, if needed. Be aware, however, that this could lead to memory leaks in the target language. Hence, it is recommended to use this directive only in well known cases. For example:

> 如果需要，相应的 `%nodefaultdtor` 指令可用于禁用默认或隐式析构函数的生成。但请注意，这可能会导致目标语言中的内存泄漏。因此，建议仅在充分了解的情况下使用此指令。例如：

```
%nodefaultdtor Foo;   // Disable the implicit/default destructor for class Foo.
class Foo {           // No destructor is generated, unless one is declared
...
};
```

**Compatibility Note:** The generation of default constructors/implicit destructors was made the default behavior in SWIG 1.3.7. This may break certain older modules, but the old behavior can be easily restored using `%nodefault` or the `-nodefault` command line option. Furthermore, in order for SWIG to properly generate (or not generate) default constructors, it must be able to gather information from both the `private` and`protected` sections (specifically, it needs to know if a private or protected constructor/destructor is defined). In older versions of SWIG , it was fairly common to simply remove or comment out the private and protected sections of a class due to parser limitations. However, this removal may now cause SWIG to erroneously generate constructors for classes that define a constructor in those sections. Consider restoring those sections in the interface or using `%nodefault` to fix the problem.

**Note:** The `%nodefault` directive/`-nodefault` options described above, which disable both the default constructor and the implicit destructors, could lead to memory leaks, and so it is strongly recommended to not use them.

> **注意兼容性**：默认构造函数、隐式析构函数的生成是 SWIG 1.3.7 中的默认行为。这可能会破坏某些旧模块，但可以使用 `%nodefault` 或 `-nodefault` 命令行选项轻松恢复旧行为。此外，为了使 SWIG 正确生成（或不生成）默认构造函数，它必须能够从 `private` 和 `protected` 部分收集信息（具体来说，它需要知道私有或保护的构造函数、析构函数是否被定义）。在旧版本的 SWIG 中，由于解析器的限制，简单地删除或注释掉类的私有和保护部分是相当常见的。但是，删除行为现在可能导致 SWIG 错误地为在这些部分中定义构造函数的类生成构造函数。考虑在接口中恢复这些部分或使用 `%nodefault` 来解决问题。
>
> **注意**：上面描述和 `%nodefault` 指令和 `-nodefault` 选项会禁用默认构造函数和隐式析构函数，可能导致内存泄漏，因此强烈建议不要使用它们。

### 6.6.3 当不能创建构造函数包装器时

If a class defines a constructor, SWIG normally tries to generate a wrapper for it. However, SWIG will not generate a constructor wrapper if it thinks that it will result in illegal wrapper code. There are really two cases where this might show up.

First, SWIG won't generate wrappers for protected or private constructors. For example:

> 如果一个类定义了一个构造函数，SWIG 通常会尝试为它生成一个包装器。但是，如果 SWIG 认为它将导致非法的包装器代码，它将不会生成构造函数包装器。有两种可能会出现这种情况。
>
> 首先，SWIG 不会为保护或私有构造函数生成包装器。例如：

```c++
class Foo {
protected:
  Foo();         // Not wrapped.
public:
  ...
};
```

Next, SWIG won't generate wrappers for a class if it appears to be abstract--that is, it has undefined pure virtual methods. Here are some examples:

> 接下来，如果某个类似乎是抽象的，它将不会为类生成包装器——也就是说，它具有未定义的纯虚方法。这里有些例子：

```c++
class Bar {
public:
  Bar();               // Not wrapped.  Bar is abstract.
  virtual void spam(void) = 0;
};

class Grok : public Bar {
public:
  Grok();            // Not wrapped. No implementation of abstract spam().
};
```

Some users are surprised (or confused) to find missing constructor wrappers in their interfaces. In almost all cases, this is caused when classes are determined to be abstract. To see if this is the case, run SWIG with all of its warnings turned on:

> 一些用户为在他们的接口中找不到构造函数包装器感到惊讶（或困惑）。几乎在所有情况下，这都是在确定类是抽象类时引起的。要查看是否是这种情况，请运行 SWIG 并打开其所有警告：

```
% swig -Wall -python module.i
```

In this mode, SWIG will issue a warning for all abstract classes. It is possible to force a class to be non-abstract using this:

> 在此模式下，SWIG 将为所有抽象类发出警告。可以使用以下方法强制类为非抽象类：

```
%feature("notabstract") Foo;

class Foo : public Bar {
public:
  Foo();    // Generated no matter what---not abstract.
  ...
};
```

More information about `%feature` can be found in the [Customization features](http://swig.org/Doc3.0/Customization.html#Customization) chapter.

> 更多关于 `%feature` 的信息可以在[自定义功能](http://swig.org/Doc3.0/Customization.html#Customization)章节找到。

### 6.6.4 拷贝构造函数

If a class defines more than one constructor, its behavior depends on the capabilities of the target language. If overloading is supported, the copy constructor is accessible using the normal constructor function. For example, if you have this:

> 如果一个类定义了多个构造函数，则其行为取决于目标语言的功能。如果支持重载，则可以使用常规构造函数访问拷贝构造函数。例如，如果你有这个：

```c++
class List {
public:
    List();
    List(const List &);      // Copy constructor
    ...
};
```

then the copy constructor can be used as follows:

> 拷贝构造函数可以如下使用：

```python
x = List()               # Create a list
y = List(x)              # Copy list x
```

If the target language does not support overloading, then the copy constructor is available through a special function like this:

> 如果目标语言不支持重载，则可以通过如下特殊函数使用拷贝构造函数：

```c++
List *copy_List(List *f) {
    return new List(*f);
}
```

**Note:** For a class `X`, SWIG only treats a constructor as a copy constructor if it can be applied to an object of type `X` or `X *`. If more than one copy constructor is defined, only the first definition that appears is used as the copy constructor--other definitions will result in a name-clash. Constructors such as `X(const X &)`, `X(X &)`, and `X(X *)` are handled as copy constructors in SWIG .

**Note:** SWIG does *not* generate a copy constructor wrapper unless one is explicitly declared in the class. This differs from the treatment of default constructors and destructors. However, copy constructor wrappers can be generated if using the `copyctor` [feature flag](http://swig.org/Doc3.0/Customization.html#Customization_feature_flags). For example:

> **注意**：对于类 `X`，如果应用于类型为 `X` 或 `X *` 的对象，则 SWIG 仅将构造函数视为拷贝构造函数。如果定义了多个拷贝构造函数，则只有出现的第一个定义用作拷贝构造函数——其他定义将导致名称冲突。诸如 `X(const X &)`，`X(X &)` 和 `X(X *)` 之类的构造函数在 SWIG 中都作为拷贝构造函数处理。
>
> **注意**： SWIG 确实*不*生成拷贝构造函数包装器，除非在类中显式声明了一个。这与默认构造函数和析构函数的处理不同。但是，如果使用 `copyctor` [功能标志](http://swig.org/Doc3.0/Customization.html#Customization_feature_flags)，则可以生成拷贝构造函数包装器。例如：

```
%copyctor List;

class List {
public:
    List();
};
```

Will generate a copy constructor wrapper for `List`.

**Compatibility note:** Special support for copy constructors was not added until SWIG-1.3.12. In previous versions, copy constructors could be wrapped, but they had to be renamed. For example:

> 将为 `List` 生成一个拷贝构造函数包装器。
>
> **注意兼容性**：直到 SWIG-1.3.12 才添加对拷贝构造函数的特殊支持。在以前的版本中，可以包装拷贝构造函数，但必须重命名它们。例如：

```
class Foo {
public:
  Foo();
  %name(CopyFoo) Foo(const Foo &);
  ...
};
```

For backwards compatibility, SWIG does not perform any special copy-constructor handling if the constructor has been manually renamed. For instance, in the above example, the name of the constructor is set to `new_CopyFoo()`. This is the same as in older versions.

> 为了向后兼容，如果已手动重命名构造函数，SWIG 不会执行任何特殊的拷贝构造函数处理。例如，在上面的例子中，构造函数的名称被设置为 `new_CopyFoo()`。这与旧版本相同。

### 6.6.5 成员函数

All member functions are roughly translated into accessor functions like this :

> 所有成员函数大致翻译成这样的访问器函数：

```c++
int List_search(List *obj, char *value) {
  return obj->search(value);
}
```

This translation is the same even if the member function has been declared as `virtual`.

It should be noted that SWIG does not *actually* create a C accessor function in the code it generates. Instead, member access such as`obj->search(value)` is directly inlined into the generated wrapper functions. However, the name and calling convention of the low-level procedural wrappers match the accessor function prototype described above.

> 即使成员函数已声明为 `virtual`，此转换也是相同的。
>
> 应该注意的是，SWIG *实际上*并没有在它生成的代码中创建一个 C 访问器函数。相反，成员访问（如 `obj->search(value)`）直接内联到生成的包装器函数中。但是，低级程序包装器的名称和调用约定与上述的访问器函数原型匹配。

### 6.6.6 静态成员

Static member functions are called directly without making any special transformations. For example, the static member function`print(List *l)` directly invokes `List::print(List *l)` in the generated wrapper code.

> 直接调用静态成员函数而不进行任何特殊转换。例如，静态成员函数 `print(List *l)` 直接在生成的包装器代码中调用 `List::print(List *l)`。

### 6.6.7 成员数据

Member data is handled in exactly the same manner as for C structures. A pair of accessor functions are effectively created. For example :

> 成员数据的处理方式与 C 结构体完全相同。有效地创建了一对存取器函数。例如：

```c++
int List_length_get(List *obj) {
  return obj->length;
}
int List_length_set(List *obj, int value) {
  obj->length = value;
  return value;
}
```

A read-only member can be created using the `%immutable` and `%mutable` [feature flag](http://swig.org/Doc3.0/Customization.html#Customization_feature_flags) directive. For example, we probably wouldn't want the user to change the length of a list so we could do the following to make the value available, but read-only.

> 可以使用 `%immutable` 和 `%mutable` [功能标志](http://swig.org/Doc3.0/Customization.html#Customization_feature_flags)指令创建只读成员。例如，我们可能不希望用户更改列表的长度，因此我们可以执行以下操作以使值可用，但是只读。

```
class List {
public:
...
%immutable;
  int length;
%mutable;
...
};
```

Alternatively, you can specify an immutable member in advance like this:

> 或者，你可以提前指定不可变成员，如下所示：

```
%immutable List::length;
...
class List {
  ...
  int length;         // Immutable by above directive
  ...
};
```

Similarly, all data attributes declared as `const` are wrapped as read-only members.

By default, SWIG uses the const reference typemaps for members that are primitive types. There are some subtle issues when wrapping data members that are not primitive types, such as classes. For instance, if you had another class like this,

> 类似地，声明为 `const` 的所有数据属性都包装为只读成员。
>
> 默认情况下，SWIG 将常引用类型映射用于基本类型的成员。在包装非基本类型的数据成员（例如类）时，存在一些微妙的问题。例如，如果你有另一个这样的类，

```c++
class Foo {
public:
    List items;
    ...
```

then the low-level accessor to the `items` member actually uses pointers. For example:

> 然后 `items` 成员的低级访问器实际上使用指针。例如：

```c++
List *Foo_items_get(Foo *self) {
    return &self->items;
}
void Foo_items_set(Foo *self, List *value) {
    self->items = *value;
}
```

More information about this can be found in the SWIG Basics chapter, [Structure data members](http://swig.org/Doc3.0/SWIG.html#SWIG_structure_data_members) section.

The wrapper code to generate the accessors for classes comes from the pointer typemaps. This can be somewhat unnatural for some types. For example, a user would expect the STL std::string class member variables to be wrapped as a string in the target language, rather than a pointer to this class. The const reference typemaps offer this type of marshalling, so there is a feature to tell SWIG to use the const reference typemaps rather than the pointer typemaps. It is the naturalvar feature and can be used to effectively change the way accessors are generated to the following:

> 有关这方面的更多信息，请参阅 SWIG 基础知识章节[结构体数据成员](http://swig.org/Doc3.0/SWIG.html#SWIG_structure_data_members)部分。
>
> 生成类访问器的包装器代码来自指针类型映射。对于某些类型，这可能有点不自然。例如，用户希望将 STL `std::string` 类成员变量包装为目标语言中的字符串，而不是指向此类的指针。常引用类型映射提供了这种类型的编组，因此有一个功能可以告诉 SWIG 使用常引用类型映射而不是指针类型映射。它是 `naturalvar` 功能，可用于有效地将访问器的生成方式，例如：

```c++
const List &Foo_items_get(Foo *self) {
    return self->items;
}
void Foo_items_set(Foo *self, const List &value) {
    self->items = value;
}
```

The `%naturalvar` directive is a macro for, and hence equivalent to, `%feature("naturalvar")`. It can be used as follows:

> `%naturalvar` 指令是一个宏，因此相当于 `%feature("naturalvar")`。它可以如下使用：

```
// All List variables will use const List& typemaps
%naturalvar List;

// Only Foo::myList will use const List& typemaps
%naturalvar Foo::myList;
struct Foo {
  List myList;
};

// All non-primitive types will use const reference typemaps
%naturalvar;
```

The observant reader will notice that `%naturalvar` works like any other [feature flag](http://swig.org/Doc3.0/Customization.html#Customization_feature_flags) directive but with some extra flexibility. The first of the example usages above shows `%naturalvar` attaching to the `myList`'s variable type, that is the `List` class. The second usage shows `%naturalvar` attaching to the variable name. Hence the naturalvar feature can be used on either the variable's name or type. Note that using the naturalvar feature on a variable's name overrides any naturalvar feature attached to the variable's type.

It is generally a good idea to use this feature globally as the reference typemaps have extra NULL checking compared to the pointer typemaps. A pointer can be NULL, whereas a reference cannot, so the extra checking ensures that the target language user does not pass in a value that translates to a NULL pointer and thereby preventing any potential NULL pointer dereferences. The `%naturalvar` feature will apply to global variables in addition to member variables in some language modules, eg C# and Java.

The naturalvar behavior can also be turned on as a global setting via the `-naturalvar` commandline option or the module mode option, `%module(naturalvar=1)`. However, any use of `%feature("naturalvar")` will override the global setting.

**Compatibility note:** The `%naturalvar` feature was introduced in SWIG-1.3.28, prior to which it was necessary to manually apply the const reference typemaps, eg `%apply const std::string & { std::string * }`, but this example would also apply the typemaps to methods taking a `std::string` pointer.

**Compatibility note:** Read-only access used to be controlled by a pair of directives `%readonly` and `%readwrite`. Although these directives still work, they generate a warning message. Simply change the directives to `%immutable;` and `%mutable;` to silence the warning. Don't forget the extra semicolon!

**Compatibility note:** Prior to SWIG-1.3.12, all members of unknown type were wrapped into accessor functions using pointers. For example, if you had a structure like this

> 细心的读者会注意到 `%naturalvar` 与任何其他[功能标志](http://swig.org/Doc3.0/Customization.html#Customization_feature_flags)指令一样，但具有一些额外的灵活性。上面的第一个示例用法显示 `%naturalvar` 附加到 `myList` 的变量类型，即 `List` 类。第二种用法显示 `%naturalvar` 附加到变量名称。因此，`naturalvar` 功能可用于变量的名称或类型。请注意，在变量名称上使用 `naturalvar` 功能会覆盖附加到变量类型的任何 `naturalvar` 功能。
>
> 全局使用此功能通常是个好主意，因为与指针类型映射相比，引用类型映射具有额外的 NULL 检查。指针可以为 NULL，而引用则不能，因此额外检查可确保目标语言用户不会传入转换为 NULL 指针的值，从而防止任何可能的 NULL 指针解引用。除了一些语言模块中的成员变量之外，`%naturalvar` 功能还将应用于全局变量，例如在 C# 和 Java 中。
>
> 自然变换行为也可以通过 `-naturalvar` 命令行选项或模块模式选项 `%module(naturalvar = 1)` 作为全局设置打开。但是，任何使用 `%feature("naturalvar")` 都将覆盖全局设置。
>
> **注意兼容性**：SWIG-1.3.28 中引入了 `%naturalvar` 功能，在此之前必须手动应用常引用类型映射，例如 `%apply const std::string&{std::string *}`，但是这个例子也将类型映射应用于带有 `std::string` 指针的方法。
>
> **注意兼容性**：只读访问过去由一对指令 `%readonly` 和 `%readwrite` 控制。尽管这些指令仍然有效，但它们会生成警告消息。只需将指令更改为 `%immutable;` 和 `%mutable;` 即可使警告静音。**不要忘记额外的分号！**
>
> **注意兼容性**：在 SWIG-1.3.12之前，所有未知类型的成员都使用指针包装到访问器函数中。例如，如果你有这样的结构体

```c++
struct Foo {
  size_t  len;
};
```

and nothing was known about `size_t`, then accessors would be written to work with `size_t *`. Starting in SWIG-1.3.12, this behavior has been modified. Specifically, pointers will *only* be used if SWIG knows that a datatype corresponds to a structure or class. Therefore, the above code would be wrapped into accessors involving `size_t`. This change is subtle, but it smooths over a few problems related to structure wrapping and some of SWIG's customization features.

> 并且对 `size_t` 一无所知，然后将使用 `size_t *` 来编写访问器。从 SWIG-1.3.12 开始，此行为已被修改。具体来说，如果 SWIG 知道数据类型对应于结构体或类，则仅使用指针。因此，上面的代码将被包装到涉及 `size_t` 的访问器中。这种变化是微妙的，但它平滑了有关结构体包装和 SWIG 自定义功能的一些问题。

## 6.7 默认参数

SWIG will wrap all types of functions that have default arguments. For example member functions:

> SWIG 将包装具有默认参数的所有类型的函数。例如成员函数：

```c++
class Foo {
public:
    void bar(int x, int y = 3, int z = 4);
};
```

SWIG handles default arguments by generating an extra overloaded method for each defaulted argument. SWIG is effectively handling methods with default arguments as if it was wrapping the equivalent overloaded methods. Thus for the example above, it is as if we had instead given the following to SWIG :

> SWIG 通过为每个默认参数生成额外的重载方法来处理默认参数。SWIG 有效地处理正在使用默认参数的方法，就好像它包装了等效的重载方法一样。因此，对于上面的示例，就好像我们已经向 SWIG 提供了以下内容：

```c++
class Foo {
public:
    void bar(int x, int y, int z);
    void bar(int x, int y);
    void bar(int x);
};
```

The wrappers produced are exactly the same as if the above code was instead fed into SWIG . Details of this are covered later in the [Wrapping Overloaded Functions and Methods](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_overloaded_methods) section. This approach allows SWIG to wrap all possible default arguments, but can be verbose. For example if a method has ten default arguments, then eleven wrapper methods are generated.

Please see the [Features and default arguments](http://swig.org/Doc3.0/Customization.html#Customization_features_default_args) section for more information on using `%feature` with functions with default arguments. The [Ambiguity resolution and renaming](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_ambiguity_resolution_renaming) section also deals with using `%rename` and `%ignore` on methods with default arguments. If you are writing your own typemaps for types used in methods with default arguments, you may also need to write a `typecheck` typemap. See the [Typemaps and overloading](http://swig.org/Doc3.0/Typemaps.html#Typemaps_overloading) section for details or otherwise use the `compactdefaultargs` feature flag as mentioned below.

**Compatibility note:** Versions of SWIG prior to SWIG-1.3.23 wrapped default arguments slightly differently. Instead a single wrapper method was generated and the default values were copied into the C++ wrappers so that the method being wrapped was then called with all the arguments specified. If the size of the wrappers are a concern then this approach to wrapping methods with default arguments can be re-activated by using the `compactdefaultargs`[feature flag](http://swig.org/Doc3.0/Customization.html#Customization_feature_flags).

> 生成的包装器与上面的代码通过 SWIG 得到的完全相同。稍后将在[包装重载函数与方法](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_overloaded_methods)部分中介绍其详细信息。这种方法允许 SWIG 包装所有可能的默认参数，但可能是冗长的。例如，如果方法有十个默认参数，则会生成十一个包装器方法。
>
> 有关将 `%feature` 与具有默认参数的函数一起使用的更多信息，请参阅[功能和默认参数](http://swig.org/Doc3.0/Customization.html#Customization_features_default_args)部分。[消歧义和重命名](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_ambiguity_resolution_renaming)部分还讨论了对具有默认参数的方法使用 `%rename` 和 `%ignore`。如果你正在为具有默认参数的方法中使用的类型编写自己的类型映射，则可能还需要编写 `typecheck` 类型映射。有关详细信息，请参阅[类型映射与重载](http://swig.org/Doc3.0/Typemaps.html#Typemaps_overloading)部分，否则请使用下面提到和 `compactdefaultargs` 功能标志。
>
> **注意兼容性**： SWIG-1.3.23 与之前的 SWIG 版本包含的默认参数略有不同，而是生成了一个包装器方法，并将默认值复制到 C++ 包装器中，以便随后使用指定的所有参数调用包装的方法。如果包装器的大小是一个问题，那么使用 `compactdefaultargs` [功能标志](http://swig.org/Doc3.0/Customization.html#)可以重新激活这种使用默认参数包装方法的方法。

```
%feature("compactdefaultargs") Foo::bar;
class Foo {
public:
    void bar(int x, int y = 3, int z = 4);
};
```

This is great for reducing the size of the wrappers, but the caveat is it does not work for the statically typed languages, such as C# and Java, which don't have optional arguments in the language, Another restriction of this feature is that it cannot handle default arguments that are not public. The following example illustrates this:

> 这对于减小包装器的大小非常有用，但需要注意的是它不适用于静态类型语言，例如 C# 和 Java，它们在语言中没有可选参数。此功能的另一个限制是它无法处理非公有的默认参数。以下示例说明这一点：

```c++
class Foo {
private:
  static const int spam;
public:
  void bar(int x, int y = spam);   // Won't work with %feature("compactdefaultargs") -
                                   // private default value
};
```

This produces uncompilable wrapper code because default values in C++ are evaluated in the same scope as the member function whereas SWIG evaluates them in the scope of a wrapper function (meaning that the values have to be public).

The `compactdefaultargs` feature is automatically turned on when wrapping [C code with default arguments](http://swig.org/Doc3.0/ SWIG .html# SWIG _default_args). Some target languages will also automatically turn on this feature if the keyword arguments feature (kwargs) is specified for either C or C++ functions, and the target language supports kwargs, the `compactdefaultargs` feature is also automatically turned on. Keyword arguments are a language feature of some scripting languages, for example Ruby and Python. SWIG is unable to support kwargs when wrapping overloaded methods, so the default approach cannot be used.

> 这会产生无法编译的包装器代码，因为 C++ 中的默认值是在与成员函数相同的范围内计算的，而 SWIG 在包装函数的范围内赋值它们（意味着值必须是公有的）。
>
> 在包装[带有默认参数的 C 代码](http://swig.org/Doc3.0/SWIG.html#SWIG_default_args)时，会自动打开 `compactdefaultargs` 功能。如果为 C 或 C++ 函数指定了关键字参数功能（kwargs），并且目标语言支持 kwargs，则某些目标语言也将自动打开此功能，`compactdefaultargs` 功能也会自动打开。关键字参数是某些脚本语言的语言特性，例如 Ruby 和 Python。在包装重载方法时，SWIG 无法支持 kwargs，因此无法使用默认方法。

## 6.8 保护

SWIG wraps class members that are public following the C++ conventions, i.e., by explicit public declaration or by the use of the `using` directive. In general, anything specified in a private or protected section will be ignored, although the internal code generator sometimes looks at the contents of the private and protected sections so that it can properly generate code for default constructors and destructors. Directors could also modify the way non-public virtual protected members are treated.

By default, members of a class definition are assumed to be private until you explicitly give a ``public:`' declaration (This is the same convention used by C++).

> SWIG 根据 C++ 约定包装公有的类成员，即通过显式公有声明或使用 `using` 指令。通常，私有或受保护部分中指定的任何内容都将被忽略，尽管内部代码生成器有时会查看私有部分和保护部分的内容，以便它可以正确地为默认构造函数和析构函数生成代码。导向器（director）还可以修改非公有的虚保护成员的处理方式。
>
> 默认情况下，在你明确给出 `public` 声明之前，假定类定义的成员是私有的（这与 C++ 使用的约定相同）。

## 6.9 枚举与常量

Enumerations and constants are handled differently by the different language modules and are described in detail in the appropriate language chapter. However, many languages map enums and constants in a class definition into constants with the classname as a prefix. For example :

> 枚举和常量由不同的语言模块以不同方式处理，并在相应的语言章节中详细描述。但是，许多语言将类定义中的枚举和常量映射到以类名作为前缀的常量。例如 ：

```c++
class Swig {
public:
  enum {ALE, LAGER, PORTER, STOUT};
};
```

Generates the following set of constants in the target scripting language :

> 在目标脚本语言中生成以下常量集：

```c++
Swig_ALE = Swig::ALE
Swig_LAGER = Swig::LAGER
Swig_PORTER = Swig::PORTER
Swig_STOUT = Swig::STOUT
```

Members declared as `const` are wrapped as read-only members and do not create constants.

> 声明为 `const` 的成员被包装为只读成员，不会创建常量。

## 6.10 友元

Friend declarations are recognised by SWIG . For example, if you have this code:

> SWIG 识别友元声明。例如，如果你有如下代码：

```c++
class Foo {
public:
  ...
  friend void blah(Foo *f);
  ...
};
```

then the `friend` declaration does result in a wrapper code equivalent to one generated for the following declaration

> 那么 `friend` 声明确实会产生一个等效于以下声明生成的包装代码

```c++
class Foo {
public:
    ...
};

void blah(Foo *f);
```

A friend declaration, as in C++, is understood to be in the same scope where the class is declared, hence, you can have

> 在 C++ 中，友元声明被理解为与声明类的作用域相同，因此，你有

```
%ignore bar::blah(Foo *f);

namespace bar {

  class Foo {
  public:
    ...
    friend void blah(Foo *f);
    ...
  };
}
```

and a wrapper for the method 'blah' will not be generated.

> 并且 `blah` 方法的包装器将不会生成。

## 6.11 引用与指针

C++ references are supported, but SWIG transforms them back into pointers. For example, a declaration like this:

> 支持 C++ 引用，但 SWIG 将其转换回指针。例如，这样的声明：

```c++
class Foo {
public:
  double bar(double &a);
}
```

has a low-level accessor

> 有一个低级访问器函数

```c++
double Foo_bar(Foo *obj, double *a) {
  obj->bar(*a);
}
```

As a special case, most language modules pass `const` references to primitive datatypes (`int`, `short`, `float`, etc.) by value instead of pointers. For example, if you have a function like this,

> 作为一种特殊情况，大多数语言模块都通过值而不是指针传递对原始数据类型（`int`、`short`、`float` 等）的常引用。例如，如果你具有这样的函数，

```c++
void foo(const int &x);
```

it is called from a script as follows:

> 在脚本中这样调用：

```python
foo(3)              # Notice pass by value
```

Functions that return a reference are remapped to return a pointer instead. For example:

> 返回引用的函数将重新映射以返回指针。例如：

```c++
class Bar {
public:
  Foo &spam();
};
```

Generates an accessor like this:

> 创建如下访问器函数：

```c++
Foo *Bar_spam(Bar *obj) {
  Foo &result = obj->spam();
  return &result;
}
```

However, functions that return `const` references to primitive datatypes (`int`, `short`, etc.) normally return the result as a value rather than a pointer. For example, a function like this,

> 但是，返回原始数据类型（`int`、`short` 等）常引用的函数通常将结果作为值而不是指针返回。例如，像这样的函数

```c++
const int &bar();
```

will return integers such as 37 or 42 in the target scripting language rather than a pointer to an integer.

Don't return references to objects allocated as local variables on the stack. SWIG doesn't make a copy of the objects so this will probably cause your program to crash.

**Note:** The special treatment for references to primitive datatypes is necessary to provide more seamless integration with more advanced C++ wrapping applications---especially related to templates and the STL. This was first added in SWIG-1.3.12.

> 将在目标脚本语言中返回诸如 37 或 42 之类的整数，而不是指向整数的指针。
>
> 不要返回对在堆栈上分配为局部变量的对象的引用。SWIG 不会复制对象，因此可能会导致程序崩溃。
>
> **注意**：必须提供对原始数据类型的引用的特殊处理，以提供与更高级的 C++ 包装应用程序（尤其是与模板和 STL 有关）的无缝集成。这是在 SWIG-1.3.12 中首次添加的。

## 6.12 传值与返回值

Occasionally, a C++ program will pass and return class objects by value. For example, a function like this might appear:

> 有时，C++ 程序会传值和返回类对象。例如，可能会出现如下函数：

```c++
Vector cross_product(Vector a, Vector b);
```

If no information is supplied about `Vector`, SWIG creates a wrapper function similar to the following:

> 如果未提供有关 `Vector` 的信息，SWIG 将创建类似于以下内容的包装器函数：

```c
Vector *wrap_cross_product(Vector *a, Vector *b) {
  Vector x = *a;
  Vector y = *b;
  Vector r = cross_product(x, y);
  return new Vector(r);
}
```

In order for the wrapper code to compile, `Vector` must define a copy constructor and a default constructor.

If `Vector` is defined as a class in the interface, but it does not support a default constructor, SWIG changes the wrapper code by encapsulating the arguments inside a special C++ template wrapper class, through a process called the "Fulton Transform". This produces a wrapper that looks like this:

> 为了编译包装代码，`Vector` 必须定义一个拷贝构造函数和一个默认构造函数。
>
> 如果 `Vector` 在接口中定义为类，但不支持默认构造函数，则 SWIG 通过称为“Fulton Transform”的过程将参数封装在特殊的 C++ 模板包装器类中来更改包装器代码。这将产生一个如下所示的包装器：

```c++
Vector cross_product(Vector *a, Vector *b) {
  SWIGValueWrapper<Vector> x = *a;
  SWIGValueWrapper<Vector> y = *b;
  SWIGValueWrapper<Vector> r = cross_product(x, y);
  return new Vector(r);
}
```

This transformation is a little sneaky, but it provides support for pass-by-value even when a class does not provide a default constructor and it makes it possible to properly support a number of SWIG's customization options. The definition of `SwigValueWrapper` can be found by reading the SWIG wrapper code. This class is really nothing more than a thin wrapper around a pointer.

Although SWIG usually detects the classes to which the Fulton Transform should be applied, in some situations it's necessary to override it. That's done with `%feature("valuewrapper")` to ensure it is used and `%feature("novaluewrapper")` to ensure it is not used:

> 这种转换有点偷偷摸摸，但是即使类没有提供默认的构造函数，它也提供了对传值的支持，并且可以正确支持许多 SWIG 的自定义选项。通过阅读 SWIG 包装器代码，可以找到 `SwigValueWrapper` 的定义。此类实际上仅是指针周围的轻度包装器。
>
> 尽管 SWIG 通常会检测应该应用“Fulton Transform”变换的类，但在某些情况下有必要重写它。这可以通过 `%feature("valuewrapper")` 开启使用，以及 `%feature("novaluewrapper")` 关闭使用：

```
%feature("novaluewrapper") A;
class A;

%feature("valuewrapper") B;
struct B {
    B();
    // ....
};
```

It is well worth considering turning this feature on for classes that do have a default constructor. It will remove a redundant constructor call at the point of the variable declaration in the wrapper, so will generate notably better performance for large objects or for classes with expensive construction. Alternatively consider returning a reference or a pointer.

**Note:** this transformation has no effect on typemaps or any other part of SWIG ---it should be transparent except that you may see this code when reading the SWIG output file.

**Note:** This template transformation is new in SWIG-1.3.11 and may be refined in future SWIG releases. In practice, it is only absolutely necessary to do this for classes that don't define a default constructor.

**Note:** The use of this template only occurs when objects are passed or returned by value. It is not used for C++ pointers or references.

> 考虑为确实具有默认构造函数的类启用此功能。它将在包装器中变量声明的位置删除多余的构造函数调用，因此对于大型对象或构造成本高的类将产生明显更好的性能。或者，考虑返回引用或指针。
>
> **注意**：此转换对类型映射或 SWIG 的任何其他部分没有影响——它应该是透明的，除了在读取 SWIG 输出文件时可能会看到此代码。
>
> **注意**：此模板转换是 SWIG-1.3.11 中的新增功能，可能会在以后的 SWIG 版本中进行完善。实际上，对于没有定义默认构造函数的类，这样做绝对是必要的。
>
> **注意**：仅当对象通过传值或返回值时才使用此模板。它不用于 C++ 指针或引用。

## 6.13 继承

SWIG supports C++ inheritance of classes and allows both single and multiple inheritance, as limited or allowed by the target language. The SWIG type-checker knows about the relationship between base and derived classes and allows pointers to any object of a derived class to be used in functions of a base class. The type-checker properly casts pointer values and is safe to use with multiple inheritance.

SWIG treats private or protected inheritance as close to the C++ spirit, and target language capabilities, as possible. In most cases, this means that SWIG will parse the non-public inheritance declarations, but that will have no effect in the generated code, besides the implicit policies derived for constructors and destructors.

The following example shows how SWIG handles inheritance. For clarity, the full C++ code has been omitted.

> SWIG 支持 C++ 类的继承，并允许单继承和多继承，这取决于目标语言的限制。SWIG 类型检查器了解基类和派生类之间的关系，并允许在接受基类的函数中使用指向任何派生类对象的指针。类型检查器可以正确地转换指针值，并且可以安全地用于多继承。
>
> SWIG 尽可能将私有或保护继承与 C++ 精神和目标语言功能联系起来。在大多数情况下，这意味着 SWIG 将解析非公有继承声明，但是除了为构造函数和析构函数派生的隐式策略之外，这对生成的代码无效。
>
> 下面的示例显示 SWIG 如何处理继承。为了清楚起见，完整的 C++ 代码已被省略。

```
// shapes.i
%module shapes
%{
#include "shapes.h"
%}

class Shape {
public:
  double x, y;
  virtual double area() = 0;
  virtual double perimeter() = 0;
  void    set_location(double x, double y);
};
class Circle : public Shape {
public:
  Circle(double radius);
  ~Circle();
  double area();
  double perimeter();
};
class Square : public Shape {
public:
  Square(double size);
  ~Square();
  double area();
  double perimeter();
}
```

When wrapped into Python, we can perform the following operations (shown using the low level Python accessors):

> 当包装到 Python 中时，我们可以执行以下操作（使用低级 Python 访问器）：

```python
$ python
>>> import shapes
>>> circle = shapes.new_Circle(7)
>>> square = shapes.new_Square(10)
>>> print shapes.Circle_area(circle)
153.93804004599999757
>>> print shapes.Shape_area(circle)
153.93804004599999757
>>> print shapes.Shape_area(square)
100.00000000000000000
>>> shapes.Shape_set_location(square, 2, -3)
>>> print shapes.Shape_perimeter(square)
40.00000000000000000
>>>
```

In this example, Circle and Square objects have been created. Member functions can be invoked on each object by making calls to `Circle_area`, `Square_area`, and so on. However, the same results can be accomplished by simply using the `Shape_area`function on either object.

One important point concerning inheritance is that the low-level accessor functions are only generated for classes in which they are actually declared. For instance, in the above example, the method`set_location()` is only accessible as `Shape_set_location()` and not as `Circle_set_location()` or `Square_set_location()`. Of course, the `Shape_set_location()` function will accept any kind of object derived from Shape. Similarly, accessor functions for the attributes `x` and `y` are generated as `Shape_x_get()`, `Shape_x_set()`, `Shape_y_get()`, and `Shape_y_set()`. Functions such as `Circle_x_get()` are not available--instead you should use`Shape_x_get()`.

Note that there is a one to one correlation between the low-level accessor functions and the proxy methods and therefore there is also a one to one correlation between the C++ class methods and the generated proxy class methods.

**Note:** For the best results, SWIG requires all base classes to be defined in an interface. Otherwise, you may get a warning message like this:

> 在此示例中，已经创建了 `Circle` 和 `Square` 对象。可以通过调用 `Circle_area`，`Square_area` 等在每个对象上调用成员函数。但是，只需在任一对象上使用 `Shape_area` 函数，即可获得相同的结果。
>
> 关于继承的重要一点是，低级访问器函数仅针对实际声明了它们的类生成。例如，在以上示例中，只能以 `Shape_set_location()` 而不是 `Circle_set_location()` 或 `Square_set_location()` 访问 `set_location()` 方法。当然，`Shape_set_location()` 函数将接受任何从 `Shape` 派生的对象。类似地，属性 `x` 和 `y` 的访问器函数生成为 `Shape_x_get()`、`Shape_x_set()`、`Shape_y_get()` 和 `Shape_y_set()`。诸如 `Circle_x_get()` 之类的函数不可用——相反，你应使用 `Shape_x_get()`。
>
> 请注意，低级访问器函数与代理方法之间存在一对一的关联，因此 C++ 类方法与生成的代理类方法之间也具有一一对应的关系。
>
> **注意**：为了获得最佳结果，SWIG 要求在接口中定义所有基类。否则，你可能会收到如下警告消息：

```
example.i:18: Warning 401: Nothing known about base class 'Foo'. Ignored.
```

If any base class is undefined, SWIG still generates correct type relationships. For instance, a function accepting a `Foo *` will accept any object derived from `Foo` regardless of whether or not SWIG actually wrapped the `Foo` class. If you really don't want to generate wrappers for the base class, but you want to silence the warning, you might consider using the `%import` directive to include the file that defines `Foo`. `%import` simply gathers type information, but doesn't generate wrappers. Alternatively, you could just define `Foo` as an empty class in the SWIG interface or use[warning suppression](http://swig.org/Doc3.0/Warnings.html#Warnings_suppression).

**Note:** `typedef`-names *can* be used as base classes. For example:

> 如果未定义任何基类，则 SWIG 仍会生成正确的类型关系。例如，一个接受 `Foo *` 的函数将接受任何从 `Foo` 派生的对象，而不管 SWIG 是否实际包装了 `Foo` 类。如果你确实不想为基类生成包装器，但是想要使警告静音，则可以考虑使用 `%import` 指令包含定义 `Foo` 的文件。`%import` 仅收集类型信息，但不生成包装器。另外，你可以在 SWIG 接口中将 `Foo` 定义为空类，或使用[警告抑制](http://swig.org/Doc3.0/Warnings.html#Warnings_suppression)。
>
> **注意**： `typedef-names` 可以用作基类。例如：

```c++
class Foo {
...
};

typedef Foo FooObj;
class Bar : public FooObj {     // Ok.  Base class is Foo
...
};
```

Similarly, `typedef` allows unnamed structures to be used as base classes. For example:

> 类似的，`typedef` 允许未命名的结构体作为基类使用。例如：

```c++
typedef struct {
  ...
} Foo;

class Bar : public Foo {    // Ok.
...
};
```

**Compatibility Note:** Starting in version 1.3.7, SWIG only generates low-level accessor wrappers for the declarations that are actually defined in each class. This differs from SWIG 1.1 which used to inherit all of the declarations defined in base classes and regenerate specialized accessor functions such as`Circle_x_get()`, `Square_x_get()`, `Circle_set_location()`, and `Square_set_location()`. This behavior resulted in huge amounts of replicated code for large class hierarchies and made it awkward to build applications spread across multiple modules (since accessor functions are duplicated in every single module). It is also unnecessary to have such wrappers when advanced features like proxy classes are used.

**Note:** Further optimizations are enabled when using the `-fvirtual` option, which avoids the regenerating of wrapper functions for virtual members that are already defined in a base class.

> **注意兼容性**：从版本 1.3.7 开始，SWIG 仅为每个类中实际定义的声明生成低级访问器函数的包装器。这与 SWIG 1.1 不同，SWIG 1.1 用来继承基类中定义的所有声明，并重新生成专用的访问器函数，例如 `Circle_x_get()`、`Square_x_get()`、`Circle_set_location()` 和 `Square_set_location()`。此行为导致为大型类层次结构的大量重复代码，并使构建分布在多个模块中的应用程序变得笨拙（因为访问器功能在每个模块中都有重复）。当使用诸如代理类之类的高级功能时，也不必具有此类包装器。
>
> **注意**：当使用 `-fvirtual` 选项时，会启用进一步的优化，这避免了为已经在基类中定义的虚成员重新生成包装函数。

## 6.14 关于多继承、指针与类型检查的讨论

When a target scripting language refers to a C++ object, it normally uses a tagged pointer object that contains both the value of the pointer and a type string. For example, in Tcl, a C++ pointer might be encoded as a string like this:

> 当目标脚本语言引用 C++ 对象时，它通常使用带有标记的指针对象，该对象既包含指针的值，又包含类型字符串。例如，在 Tcl 中，C++ 指针可能被编码为这样的字符串：

```tcl
_808fea88_p_Circle
```

A somewhat common question is whether or not the type-tag could be safely removed from the pointer. For instance, to get better performance, could you strip all type tags and just use simple integers instead?

In general, the answer to this question is no. In the wrappers, all pointers are converted into a common data representation in the target language. Typically this is the equivalent of casting a pointer to `void *`. This means that any C++ type information associated with the pointer is lost in the conversion.

The problem with losing type information is that it is needed to properly support many advanced C++ features--especially multiple inheritance. For example, suppose you had code like this:

> 一个常见的问题是是否可以安全地从指针中删除类型标记。例如，为了获得更好的性能，是否可以剥离所有类型标记，而仅使用简单的整数？
>
> 一般来说，这个问题的答案是否定的。在包装器中，所有指针都转换为目标语言中的通用数据表示形式。通常，这等效于将指针转换为 `void *`。这意味着与指针关联的所有 C++ 类型信息都将在转换中丢失。
>
> 丢失类型信息是个问题，由于需要正确支持许多高级 C++ 功能，尤其是多继承。例如，假设你有如下代码：

```c++
class A {
public:
  int x;
};

class B {
public:
  int y;
};

class C : public A, public B {
};

int A_function(A *a) {
  return a->x;
}

int B_function(B *b) {
  return b->y;
}
```

Now, consider the following code that uses `void *`.

> 现在，考虑使用 `void *` 的代码：

```c++
C *c = new C();
void *p = (void *) c;
...
int x = A_function((A *) p);
int y = B_function((B *) p);
```

In this code, both `A_function()` and `B_function()` may legally accept an object of type `C *` (via inheritance). However, one of the functions will always return the wrong result when used as shown. The reason for this is that even though `p` points to an object of type `C`, the casting operation doesn't work like you would expect. Internally, this has to do with the data representation of `C`. With multiple inheritance, the data from each base class is stacked together. For example:

> 在这段代码中，`A_function()` 和 `B_function()` 都可以合法地接受类型为 `C *` 的对象（通过继承）。但是，如图所示使用其中一个函数时，总是会返回错误的结果。原因是即使 `p` 指向类型为 `C` 的对象，强制转换操作也不像你期望的那样工作。在内部，这与 `C` 的数据表示有关。通过多继承，来自每个基类的数据将堆叠在一起。例如：

```
             ------------    <--- (C *), (A *)
            |     A      |
            |------------|   <--- (B *)
            |     B      |
             ------------
```

Because of this stacking, a pointer of type `C *` may change value when it is converted to a `A *` or `B *`. However, this adjustment does *not* occur if you are converting from a `void *`.

The use of type tags marks all pointers with the real type of the underlying object. This extra information is then used by SWIG generated wrappers to correctly cast pointer values under inheritance (avoiding the above problem).

Some of the language modules are able to solve the problem by storing multiple instances of the pointer, for example, `A *`, in the A proxy class as well as `C *` in the C proxy class. The correct cast can then be made by choosing the correct `void *` pointer to use and is guaranteed to work as the cast to a void pointer and back to the same type does not lose any type information:

> 由于这种堆叠，类型为 `C *` 的指针在转换为 `A *` 或 `B *` 时可能会改变值。但是，如果从 `void *` 转换，则不会*进行*这种调整。
>
> 类型标记的使用将所有指针标记为底层对象的真实类型。然后，SWIG 生成的包装器将使用这些额外的信息来在继承下正确地转换指针值（避免上述问题）。
>
> 一些语言模块可以通过在 `A` 代理类中存储指针的多个实例（例如，`A *`）以及在 `C` 代理类中存储 `C`，来解决该问题。然后可以通过选择要使用的正确 `void *` 指针来进行正确的转换，并保证可以正常工作，因为转换为 `void` 指针并返回相同类型的转换不会丢失任何类型信息：

```c++
C *c = new C();
void *p = (void *) c;
void *pA = (void *) c;
void *pB = (void *) c;
...
int x = A_function((A *) pA);
int y = B_function((B *) pB);
```

In practice, the pointer is held as an integral number in the target language proxy class.

> 实际上，指针在目标语言代理类中被保存为整数。

## 6.15 包装重载函数和方法

In many language modules, SWIG provides partial support for overloaded functions, methods, and constructors. For example, if you supply SWIG with overloaded functions like this:

> 在许多语言模块中，SWIG 为重载的函数、方法和构造函数提供部分支持。例如，如果为 SWIG 提供像这样的重载函数：

```c++
void foo(int x) {
  printf("x is %d\n", x);
}
void foo(char *x) {
  printf("x is '%s'\n", x);
}
```

The function is used in a completely natural way. For example:

> 函数可以用完全自然地方式使用。例如：

```python
>>> foo(3)
x is 3
>>> foo("hello")
x is 'hello'
>>>
```

Overloading works in a similar manner for methods and constructors. For example if you have this code,

> 方法和构造函数重载的工作方式相似。例如，如果你有如下代码，

```c++
class Foo {
public:
  Foo();
  Foo(const Foo &);   // Copy constructor
  void bar(int x);
  void bar(char *s, int y);
};
```

it might be used like this

> 看起来会像这样

```python
>>> f = Foo()          # Create a Foo
>>> f.bar(3)
>>> g = Foo(f)         # Copy Foo
>>> f.bar("hello", 2)
```

### 6.15.1 调度函数生成

The implementation of overloaded functions and methods is somewhat complicated due to the dynamic nature of scripting languages. Unlike C++, which binds overloaded methods at compile time, SWIG must determine the proper function as a runtime check for scripting language targets. This check is further complicated by the typeless nature of certain scripting languages. For instance, in Tcl, all types are simply strings. Therefore, if you have two overloaded functions like this,

> 由于脚本语言的动态特性，重载函数和方法的实现有些复杂。与 C++ 会在编译时绑定重载的方法不同，对于目标脚本语言，SWIG 必须以运行时检查确定适当的函数。某些脚本语言是无类型的，会使此检查更加复杂。例如，在 Tcl 中，所有类型都只是字符串。因此，如果你有两个这样的重载函数，

```c++
void foo(char *x);
void foo(int x);
```

the order in which the arguments are checked plays a rather critical role.

For statically typed languages, SWIG uses the language's method overloading mechanism. To implement overloading for the scripting languages, SWIG generates a dispatch function that checks the number of passed arguments and their types. To create this function, SWIG first examines all of the overloaded methods and ranks them according to the following rules:

1. **Number of required arguments.** Methods are sorted by increasing number of required arguments.
2. **Argument type precedence.** All C++ datatypes are assigned a numeric type precedence value (which is determined by the language module).

> 检查参数的顺序起着至关重要的作用。
>
> 对于静态类型的语言，SWIG 使用该语言的方法重载机制。为了实现脚本语言的重载，SWIG 生成一个调度函数，该函数检查传递来的参数的数量及其类型。为了创建此函数，SWIG 首先检查所有重载方法，然后根据以下规则对它们进行排序：
>
> 1. **所需参数的数量**。方法按所需参数数量进行升序排序。
> 2. **参数类型优先级**。所有 C++ 数据类型均分配有数字类型的优先级值（由语言模块确定）。

```
Type              Precedence
----------------  ----------
TYPE *            0     (High)
void *            20
Integers          40
Floating point    60
char              80
Strings           100   (Low)
```

Using these precedence values, overloaded methods with the same number of required arguments are sorted in increased order of precedence values.

This may sound very confusing, but an example will help. Consider the following collection of overloaded methods:

> 使用这些优先级值，具有相同数量必需参数的重载方法将按优先级值进行升序排序。
>
> 这听起来可能很令人困惑，但是一个示例会有所帮助。请考虑以下重载方法的集合：

```c++
void foo(double);
void foo(int);
void foo(Bar *);
void foo();
void foo(int x, int y, int z, int w);
void foo(int x, int y, int z = 3);
void foo(double x, double y);
void foo(double x, Bar *z);
```

The first rule simply ranks the functions by required argument count. This would produce the following list:

> 第一条规则只是根据所需的参数个数对函数进行排序。这将产生以下列表：

```
rank
-----
[0]   foo()
[1]   foo(double);
[2]   foo(int);
[3]   foo(Bar *);
[4]   foo(int x, int y, int z = 3);
[5]   foo(double x, double y)
[6]   foo(double x, Bar *z)
[7]   foo(int x, int y, int z, int w);
```

The second rule, simply refines the ranking by looking at argument type precedence values.

> 第二条规则只是通过查看参数类型优先级值来简化排序。

```
rank
-----
[0]   foo()
[1]   foo(Bar *);
[2]   foo(int);
[3]   foo(double);
[4]   foo(int x, int y, int z = 3);
[5]   foo(double x, Bar *z)
[6]   foo(double x, double y)
[7]   foo(int x, int y, int z, int w);
```

Finally, to generate the dispatch function, the arguments passed to an overloaded method are simply checked in the same order as they appear in this ranking.

If you're still confused, don't worry about it--- SWIG is probably doing the right thing.

> 最后，要生成调度函数，只需按照该排序中的顺序检查传递给重载方法的参数。
>
> 如果你仍然感到困惑，请不要担心——SWIG 不会出错。

### 6.15.2 重载中的歧义

Regrettably, SWIG is not able to support every possible use of valid C++ overloading. Consider the following example:

> 遗憾的是，SWIG 无法支持所有有效 C++ 重载的可能使用情形。考虑以下示例：

```c++
void foo(int x);
void foo(long x);
```

In C++, this is perfectly legal. However, in a scripting language, there is generally only one kind of integer object. Therefore, which one of these functions do you pick? Clearly, there is no way to truly make a distinction just by looking at the value of the integer itself (`int` and `long` may even be the same precision). Therefore, when SWIG encounters this situation, it may generate a warning message like this for scripting languages:

> 在 C++ 中，这是完全合法的。但是，在脚本语言中，通常只有一种整数对象。因此，你选择的是哪一个函数？显然，仅通过查看整数本身的值无法真正做出区分（`int` 和 `long` 甚至可能具有相同的精度）。因此，当 SWIG 遇到这种情况时，它可能会针对脚本语言生成如下警告消息：

```
example.i:4: Warning 509: Overloaded method foo(long) effectively ignored,
example.i:3: Warning 509: as it is shadowed by foo(int).
```

or for statically typed languages like Java:

> 或者，对于 Java 这样的静态类型语言：

```
example.i:4: Warning 516: Overloaded method foo(long) ignored,
example.i:3: Warning 516: using foo(int) instead.
at example.i:3 used.
```

This means that the second overloaded function will be inaccessible from a scripting interface or the method won't be wrapped at all. This is done as SWIG does not know how to disambiguate it from an earlier method.

Ambiguity problems are known to arise in the following situations:

* Integer conversions. Datatypes such as `int`, `long`, and `short`cannot be disambiguated in some languages. Shown above.
* Floating point conversion. `float` and `double` can not be disambiguated in some languages.
* Pointers and references. For example, `Foo *` and `Foo &`.
* Pointers and arrays. For example, `Foo *` and `Foo [4]`.
* Pointers and instances. For example, `Foo` and `Foo *`. Note: SWIG converts all instances to pointers.
* Qualifiers. For example, `const Foo *` and `Foo *`.
* Default vs. non default arguments. For example, `foo(int a, int b)` and `foo(int a, int b = 3)`.

When an ambiguity arises, methods are checked in the same order as they appear in the interface file. Therefore, earlier methods will shadow methods that appear later.

When wrapping an overloaded function, there is a chance that you will get a warning message like this:

> 这意味着第二个重载函数将无法从脚本接口访问，或者该方法将不会被包装。之所以这样做是因为 SWIG 不知道如何将其与先出现的方法进行区分。
>
> 已知在以下情况下会出现歧义问题：
>
> * 整数转换。在某些语言中，诸如 `int`、`long` 和 `short` 之类的数据类型无法消除歧义。如上所示。
> * 浮点转换。在某些语言中，`float` 和 `double` 无法消除歧义。
> * 指针和引用。例如，`Foo *` 和 `Foo&`。
> * 指针和数组。例如，`Foo *` 和 `Foo [4]`。
> * 指针和实例。例如，`Foo` 和 `Foo *`。注意： SWIG 将所有实例转换为指针。
> * 限定词。例如，`const Foo *` 和 `Foo *`。
> * 默认与非默认参数。例如，`foo(int a, int b)` 和 `foo(int a, int b = 3)`。
>
> 当出现歧义时，将按照接口文件中出现的顺序检查方法。因此，先出现的方法将覆盖后出现的方法。
>
> 当包装一个重载的函数时，你可能会收到如下警告消息：

```
example.i:3: Warning 467: Overloaded foo(int) not supported (incomplete type checking rule -
no precedence level in typecheck typemap for 'int').
```

This error means that the target language module supports overloading, but for some reason there is no type-checking rule that can be used to generate a working dispatch function. The resulting behavior is then undefined. You should report this as a bug to the [ SWIG bug tracking database](http://www.swig.org/bugs.html) if this is due to one of the typemaps supplied with SWIG .

If you get an error message such as the following,

> 该错误意味着目标语言模块支持重载，但是由于某种原因，没有类型检查规则可用于生成有效的调度函数。最终的行为是不确定的。如果这是由 SWIG 附带的类型映射带来的，则应将此错误报告给 [SWIG 错误跟踪数据库](http://www.swig.org/bugs.html)。
>
> 如果你收到以下错误消息，

```
foo.i:6. Overloaded declaration ignored.  Spam::foo(double )
foo.i:5. Previous declaration is Spam::foo(int )
foo.i:7. Overloaded declaration ignored.  Spam::foo(Bar *, Spam *, int )
foo.i:5. Previous declaration is Spam::foo(int )
```

it means that the target language module has not yet implemented support for overloaded functions and methods. The only way to fix the problem is to read the next section.

> 这意味着目标语言模块尚未实现对重载函数和方法的支持。解决该问题的唯一方法是阅读下一节。

### 6.15.3 消歧义与重命名

If an ambiguity in overload resolution occurs or if a module doesn't allow overloading, there are a few strategies for dealing with the problem. First, you can tell SWIG to ignore one of the methods. This is easy---simply use the `%ignore` directive. For example:

> 如果在重载上出现歧义，或者模块不允许重载，则有一些策略可以解决该问题。首先，你可以告诉 SWIG 忽略其中一种方法。这很容易——只需使用 `%ignore` 指令即可。例如：

```
%ignore foo(long);

void foo(int);
void foo(long);       // Ignored.  Oh well.
```

The other alternative is to rename one of the methods. This can be done using `%rename`. For example:

> 另一种选择是重命名其中一种方法。这可以使用 `%rename` 完成。例如：

```
%rename("foo_short") foo(short);
%rename(foo_long) foo(long);

void foo(int);
void foo(short);      // Accessed as foo_short()
void foo(long);       // Accessed as foo_long()
```

Note that the quotes around the new name are optional, however, should the new name be a C/C++ keyword they would be essential in order to avoid a parsing error. The `%ignore` and `%rename`directives are both rather powerful in their ability to match declarations. When used in their simple form, they apply to both global functions and methods. For example:

> 请注意，新名称周围的引号是可选的，但是，为避免解析错误，新名称不能是 C/C++ 关键字。`%ignore` 和 `%rename` 指令在匹配声明方面都非常强大。当以简单形式使用时，它们适用于全局函数和方法。例如：

```
/* Forward renaming declarations */
%rename(foo_i) foo(int);
%rename(foo_d) foo(double);
...
void foo(int);           // Becomes 'foo_i'
void foo(char *c);       // Stays 'foo' (not renamed)

class Spam {
public:
  void foo(int);      // Becomes 'foo_i'
  void foo(double);   // Becomes 'foo_d'
  ...
};
```

If you only want the renaming to apply to a certain scope, the C++ scope resolution operator (`::`) can be used. For example:

> 如果只想将重命名应用于某个范围，则可以使用 C++ 范围解析运算符（`::`）。例如：

```
%rename(foo_i) ::foo(int);      // Only rename foo(int) in the global scope.
                                // (will not rename class members)

%rename(foo_i) Spam::foo(int);  // Only rename foo(int) in class Spam
```

When a renaming operator is applied to a class as in `Spam::foo(int)`, it is applied to that class and all derived classes. This can be used to apply a consistent renaming across an entire class hierarchy with only a few declarations. For example:

> 当将重命名运算符应用于 `Spam::foo(int)` 中的类时，它将应用于该类和所有派生类。这样一来，仅使用几个声明就可以在整个类层次结构中应用一致的重命名。例如：

```
%rename(foo_i) Spam::foo(int);
%rename(foo_d) Spam::foo(double);

class Spam {
public:
  virtual void foo(int);      // Renamed to foo_i
  virtual void foo(double);   // Renamed to foo_d
  ...
};

class Bar : public Spam {
public:
  virtual void foo(int);      // Renamed to foo_i
  virtual void foo(double);   // Renamed to foo_d
  ...
};

class Grok : public Bar {
public:
  virtual void foo(int);      // Renamed to foo_i
  virtual void foo(double);   // Renamed to foo_d
  ...
};
```

It is also possible to include `%rename` specifications in the class definition itself. For example:

> 也可以在类定义本身中包含 `%rename` 规范。例如：

```
class Spam {
  %rename(foo_i) foo(int);
  %rename(foo_d) foo(double);
public:
  virtual void foo(int);      // Renamed to foo_i
  virtual void foo(double);   // Renamed to foo_d
  ...
};

class Bar : public Spam {
public:
  virtual void foo(int);      // Renamed to foo_i
  virtual void foo(double);   // Renamed to foo_d
...
};
```

In this case, the `%rename` directives still get applied across the entire inheritance hierarchy, but it's no longer necessary to explicitly specify the class prefix `Spam::`.

A special form of `%rename` can be used to apply a renaming just to class members (of all classes):

> 在这种情况下，`%rename` 指令仍将应用于整个继承层次结构，但是不再需要显式指定类前缀 `Spam::`。
>
> `%rename` 的特殊形式可用于将重命名仅应用于类成员（所有类的）：

```
%rename(foo_i) *::foo(int);   // Only rename foo(int) if it appears in a class.
```

Note: the `*::` syntax is non-standard C++, but the '*' is meant to be a wildcard that matches any class name (we couldn't think of a better alternative so if you have a better idea, send email to the [swig-devel mailing list](http://www.swig.org/mail.html).

Although this discussion has primarily focused on `%rename` all of the same rules also apply to `%ignore`. For example:

> **注意**：`*::` 语法是非标准的 C++ ，但是 `*` 是一个与任何类名匹配的通配符（我们想不出更好的替代方法，因此，如果你有更好的主意，请发送通过电子邮件发送到 [swig-devel 邮件列表](http://www.swig.org/mail.html)。
>
> 尽管此讨论主要集中在 `%rename` 上，所有相同的规则也适用于 `%ignore`。例如：

```
%ignore foo(double);          // Ignore all foo(double)
%ignore Spam::foo;            // Ignore foo in class Spam
%ignore Spam::foo(double);    // Ignore foo(double) in class Spam
%ignore *::foo(double);       // Ignore foo(double) in all classes
```

When applied to a base class, `%ignore` forces all definitions in derived classes to disappear. For example, `%ignore Spam::foo(double)` will eliminate `foo(double)` in `Spam` and all classes derived from `Spam`.

**Notes on %rename and %ignore:**

* Since, the `%rename` declaration is used to declare a renaming in advance, it can be placed at the start of an interface file. This makes it possible to apply a consistent name resolution without having to modify header files. For example:

> 当应用于基类时，`%ignore` 会强制派生类中的所有定义消失。例如，`%ignore Spam::foo(double)` 将消除 `Spam` 和所有 `Spam` 派生类中的 `foo(double)` 。
>
> **有关 `%rename` 和 `%ignore` 的说明：**
>
> * 由于 `%rename` 声明用于预先声明重命名，因此可以将其放在接口文件的开头。这样就可以应用一致的名称解析，而不必修改头文件。例如：

```
%module foo

/* Rename these overloaded functions */
%rename(foo_i) foo(int);
%rename(foo_d) foo(double);

%include "header.h"
```

* The scope qualifier (`::`) can also be used on simple names. For example:

> * 范围限定符（`::`）也可以用于简单名称。例如：

```
%rename(bar) ::foo;       // Rename foo to bar in global scope only
%rename(bar) Spam::foo;   // Rename foo to bar in class Spam only
%rename(bar) *::foo;      // Rename foo in classes only
```

* Name matching tries to find the most specific match that is defined. A qualified name such as `Spam::foo` always has higher precedence than an unqualified name `foo`. `Spam::foo`has higher precedence than `*::foo` and `*::foo` has higher precedence than `foo`. A parameterized name has higher precedence than an unparameterized name within the same scope level. However, an unparameterized name with a scope qualifier has higher precedence than a parameterized name in global scope (e.g., a renaming of `Spam::foo` takes precedence over a renaming of `foo(int)`).
* The order in which `%rename` directives are defined does not matter as long as they appear before the declarations to be renamed. Thus, there is no difference between saying:

> * 名称匹配尝试查找已定义的最具体匹配。诸如 `Spam::foo` 之类的限定名称总是比非限定名称 `foo` 具有更高的优先级。`Spam::foo` 的优先级高于 `*::foo`，而 `*::foo` 的优先级高于 `foo`。在相同作用域级别中，参数化名称的优先级高于未参数化名称的优先级。但是，具有范围限定符的未参数化名称的优先级高于全局范围内的参数化名称的优先级（例如，`Spam::foo` 的重命名优先于 `foo(int)` 的重命名）。
> * 定义 `%rename` 指令的顺序无所谓，只要它们出现在要重命名的声明之前即可。因此，说下面两个没有区别：

```
%rename(bar) foo;
%rename(foo_i) Spam::foo(int);
%rename(Foo) Spam::foo;
```

and this

> 与

```
%rename(Foo) Spam::foo;
%rename(bar) foo;
%rename(foo_i) Spam::foo(int);
```

(the declarations are not stored in a linked list and order has no importance). Of course, a repeated `%rename` directive will change the setting for a previous `%rename` directive if exactly the same name, scope, and parameters are supplied.

* For multiple inheritance where renaming rules are defined for multiple base classes, the first renaming rule found on a depth-first traversal of the class hierarchy is used.

* The name matching rules strictly follow member qualification rules. For example, if you have a class like this:

> （声明不存储在链接列表中，顺序不重要）。当然，如果提供的名称、作用域和参数完全相同，重复的 `%rename` 指令将更改先前的 `%rename` 指令的设置。
>
> * 对于多继承，如果多个基类定义了重命名规则，使用类层次结构的深度优先遍历中找到的第一个重命名规则。
> * 名称匹配规则严格遵循成员限定规则。例如，如果你有一个像这样的类：

```c++
class Spam {
public:
  ...
  void bar() const;
  ...
};
```

the declaration

> 声明

```
%rename(name) Spam::bar();
```

will not apply as there is no unqualified member `bar()`. The following will apply as the qualifier matches correctly:

> 将不适用，因为没有不合法的成员 `bar()`。当限定词正确匹配时，将适用以下条件：

```
%rename(name) Spam::bar() const;
```

An often overlooked C++ feature is that classes can define two different overloaded members that differ only in their qualifiers, like this:

> 一个经常被忽视的 C++ 功能是，类可以定义两个不同的重载成员函数，仅仅是它们的限定符不同，如下所示：

```c++
class Spam {
public:
...
void bar();         // Unqualified member
void bar() const;   // Qualified member
...
};
```

%rename can then be used to target each of the overloaded methods individually. For example we can give them separate names in the target language:

> 然后，可以使用 `%rename` 分别针对每个重载方法。例如，我们可以在目标语言中给它们单独命名：

```
%rename(name1) Spam::bar();
%rename(name2) Spam::bar() const;
```

Similarly, if you merely wanted to ignore one of the declarations, use `%ignore` with the full qualification. For example, the following directive would tell SWIG to ignore the `const` version of `bar()` above:

> 同样，如果你只想忽略其中一个声明，请使用带有完整限定符的 `%ignore`。例如，以下指令将告诉 SWIG 忽略上述 `bar()` 的 `const` 版本：

```
%ignore Spam::bar() const;   // Ignore bar() const, but leave other bar() alone
```

* Currently no resolution is performed in order to match function parameters. This means function parameter types must match exactly. For example, namespace qualifiers and typedefs will not work. The following usage of typedefs demonstrates this:

> * 当前不执行区分以匹配函数参数。这意味着函数参数类型必须完全匹配。例如，命名空间限定符和 `typedef` 将不起作用。`typedef` 的以下用法说明了这一点：

```
typedef int Integer;

%rename(foo_i) foo(int);

class Spam {
public:
  void foo(Integer);  // Stays 'foo' (not renamed)
};
class Ham {
public:
  void foo(int);      // Renamed to foo_i
};
```

* The name matching rules also use default arguments for finer control when wrapping methods that have default arguments. Recall that methods with default arguments are wrapped as if the equivalent overloaded methods had been parsed ([Default arguments](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args) section). Let's consider the following example class:

> * 当包装具有默认参数的方法时，名称匹配规则还使用默认参数来进行更好的控制。回想一下，带有默认参数的方法被包装起来，就好像解析了等效的重载方法一样（[默认参数](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_default_args)部分）。让我们考虑以下示例类：

```c++
class Spam {
public:
  ...
  void bar(int i=-1, double d=0.0);
  ...
};
```

The following `%rename` will match exactly and apply to all the target language overloaded methods because the declaration with the default arguments exactly matches the wrapped method:

> 以下 `%rename` 将完全匹配并适用于所有目标语言重载方法，因为带有默认参数的声明与包装的方法完全匹配：

```
%rename(newbar) Spam::bar(int i=-1, double d=0.0);
```

The C++ method can then be called from the target language with the new name no matter how many arguments are specified, for example: `newbar(2, 2.0)`, `newbar(2)` or `newbar()`. However, if the `%rename` does not contain the default arguments, it will only apply to the single equivalent target language overloaded method. So if instead we have:

> 然后，无论指定多少个参数，都可以使用新名称从目标语言中调用 C++ 方法，例如：`newbar(2, 2.0)`、`newbar(2)` 或 `newbar()`。但是，如果 `%rename` 不包含默认参数，它将仅适用于单个等效的目标语言重载方法。因此，如果相反，我们有：

```
%rename(newbar) Spam::bar(int i, double d);
```

The C++ method must then be called from the target language with the new name `newbar(2, 2.0)` when both arguments are supplied or with the original name as `bar(2)` (one argument) or `bar()` (no arguments). In fact it is possible to use `%rename` on the equivalent overloaded methods, to rename all the equivalent overloaded methods:

> 如果同时提供了两个参数，则必须使用新名称 `newbar(2, 2.0)` 从目标语言中调用 C++ 方法，或者将原始名称称为 `bar(2)`（一个参数）或 `bar()`（无参数）。实际上，可以在等效的重载方法上使用 `%rename` 来重命名所有等效的重载方法：

```
%rename(bar_2args)   Spam::bar(int i, double d);
%rename(bar_1arg)    Spam::bar(int i);
%rename(bar_default) Spam::bar();
```

Similarly, the extra overloaded methods can be selectively ignored using `%ignore`.

**Compatibility note:** The `%rename` directive introduced the default argument matching rules in SWIG-1.3.23 at the same time as the changes to wrapping methods with default arguments was introduced.

> 类似地，额外的重载方法可以使用 `%ignore` 有选择地忽略。
>
> **注意兼容性**：`%rename` 指令在 SWIG-1.3.23 中引入了默认参数匹配规则，同时改变了对带有默认参数的方法的包装方式。

### 6.15.4 对重载的评论

Support for overloaded methods was first added in SWIG-1.3.14. The implementation is somewhat unusual when compared to similar tools. For instance, the order in which declarations appear is largely irrelevant in SWIG . Furthermore, SWIG does not rely upon trial execution or exception handling to figure out which method to invoke.

Internally, the overloading mechanism is completely configurable by the target language module. Therefore, the degree of overloading support may vary from language to language. As a general rule, statically typed languages like Java are able to provide more support than dynamically typed languages like Perl, Python, Ruby, and Tcl.

> SWIG-1.3.14 首先添加了对重载方法的支持。与类似工具相比，该实现有些不同寻常。例如，声明出现的顺序在 SWIG 中基本上无关紧要。此外，SWIG 并不依靠试验执行或异常处理来确定要调用的方法。
>
> 在内部，重载机制可以由目标语言模块完全配置。因此，重载支持的程度可能因语言而异。通常，与 Perl、Python、Ruby 和 Tcl 等动态类型的语言相比，像 Java 这样的静态类型语言能够提供更多的支持。

## 6.16 包装重载运算符

C++ overloaded operator declarations can be wrapped. For example, consider a class like this:

> 可以包装 C++ 重载运算符声明。例如，考虑这样的一个类：

```c++
class Complex {
private:
  double rpart, ipart;
public:
  Complex(double r = 0, double i = 0) : rpart(r), ipart(i) { }
  Complex(const Complex &c) : rpart(c.rpart), ipart(c.ipart) { }
  Complex &operator=(const Complex &c) {
    rpart = c.rpart;
    ipart = c.ipart;
    return *this;
  }
  Complex operator+(const Complex &c) const {
    return Complex(rpart+c.rpart, ipart+c.ipart);
  }
  Complex operator-(const Complex &c) const {
    return Complex(rpart-c.rpart, ipart-c.ipart);
  }
  Complex operator*(const Complex &c) const {
    return Complex(rpart*c.rpart - ipart*c.ipart,
                   rpart*c.ipart + c.rpart*ipart);
  }
  Complex operator-() const {
    return Complex(-rpart, -ipart);
  }
  double re() const { return rpart; }
  double im() const { return ipart; }
};
```

When operator declarations appear, they are handled in *exactly* the same manner as regular methods. However, the names of these methods are set to strings like "`operator +`" or "`operator -`". The problem with these names is that they are illegal identifiers in most scripting languages. For instance, you can't just create a method called "`operator +`" in Python--there won't be any way to call it.

Some language modules already know how to automatically handle certain operators (mapping them into operators in the target language). However, the underlying implementation of this is really managed in a very general way using the `%rename` directive. For example, in Python a declaration similar to this is used:

> 出现运算符声明时，它们的处理方式与常规方法*完全*相同。但是，这些方法的名称被这样的设置为字符串，例如 `operator +` 或 `operator -`。这些名称的问题在于，它们在大多数脚本语言中是非法标识符。例如，你不能在 Python 中创建一个称为 `operator +` 的方法——不会有任何方法可以调用它。
>
> 一些语言模块已经知道如何自动处理某些运算符（将它们映射为目标语言的运算符）。但是，实际上使用 `%rename` 指令以一种非常通用的方式来管理它的底层实现。例如，在 Python 中，使用类似于以下的声明：

```
%rename(__add__) Complex::operator+;
```

This binds the + operator to a method called `__add__` (which is conveniently the same name used to implement the Python + operator). Internally, the generated wrapper code for a wrapped operator will look something like this pseudocode:

> 它将 `+` 运算符绑定到名为 `__add__` 的方法（方便地与用于实现 Python `+` 运算符的名称相同）。在内部，为包装的运算符生成的包装器代码将类似于以下伪代码：

```c++
_wrap_Complex___add__(args) {
  ... get args ...
  obj->operator+(args);
  ...
}
```

When used in the target language, it may now be possible to use the overloaded operator normally. For example:

> 当以目标语言使用时，现在可以正常使用重载运算符了。例如：

```python
>>> a = Complex(3, 4)
>>> b = Complex(5, 2)
>>> c = a + b           # Invokes __add__ method
```

It is important to realize that there is nothing magical happening here. The `%rename` directive really only picks a valid method name. If you wrote this:

> 重要的是要意识到这里没有神奇的事情发生。`%rename` 指令实际上只选择一个有效的方法名称。如果你编写下列代码：

```
%rename(add) operator+;
```

The resulting scripting interface might work like this:

> 最终的脚本接口会像这样：

```python
a = Complex(3, 4)
b = Complex(5, 2)
c = a.add(b)      # Call a.operator+(b)
```

All of the techniques described to deal with overloaded functions also apply to operators. For example:

> 之前描述的处理重载函数的所有技术也适用于运算符。例如：

```
%ignore Complex::operator=;             // Ignore = in class Complex
%ignore *::operator=;                   // Ignore = in all classes
%ignore operator=;                      // Ignore = everywhere.

%rename(__sub__) Complex::operator-;
%rename(__neg__) Complex::operator-();  // Unary -
```

The last part of this example illustrates how multiple definitions of the `operator-` method might be handled.

Handling operators in this manner is mostly straightforward. However, there are a few subtle issues to keep in mind:

* In C++, it is fairly common to define different versions of the operators to account for different types. For example, a class might also include a friend function like this:

> 该示例的最后一部分说明了如何处理 `operator -` 方法的多个定义。
>
> 以这种方式处理运算符通常很简单。但是，请记住一些细微的问题：
>
> * 在 C++ 中，定义不同版本的运算符以对应不同类型是相当普遍的。例如，一个类可能还包含一个如下所示的友元函数：

```c++
class Complex {
public:
  friend Complex operator+(Complex &, double);
};
Complex operator+(Complex &, double);
```

SWIG simply ignores all `friend` declarations. Furthermore, it doesn't know how to associate the associated `operator+` with the class (because it's not a member of the class).

It's still possible to make a wrapper for this operator, but you'll have to handle it like a normal function. For example:

> SWIG 简单地忽略所有 `friend` 声明。此外，它不知道如何将 `operator +` 与该类相关联（因为它不是该类的成员）。
>
> 仍然可以为该运算符创建包装器，但是你必须像处理普通函数一样处理它。例如：

```
%rename(add_complex_double) operator+(Complex &, double);
```

* Certain operators are ignored by default. For instance, `new`and `delete` operators are ignored as well as conversion and index operators. A warning such as the one below is shown:

> * 默认情况下会忽略某些运算符。例如，`new` 和 `delete` 运算符以及转换和索引运算符都将被忽略。显示如下警告：

```
example.i:12: Warning 503: Can't wrap 'operator []' unless renamed to a valid identifier.
```

* The index operator, `operator[]`, is particularly difficult to overload due to differences in C++ implementations. Specifically, the get and set operators in other languages typically are separated into two methods such that additional logic can be packed into the operations; C# uses `this[type key] { get { ... } set { ... }}`, Python uses`__getitem__` and `__setitem__`, etc. In C++ if the return type of `operator[]` is a reference and the method is const, it is often indicative of the *setter*, and and the *getter* is usually a const function return an object by value. In the absence of any hard and fast rules and the fact that there may be multiple index operators, it is up to the user to choose the getter and setter to use by using %rename as shown earlier.

* The semantics of certain C++ operators may not match those in the target language.

> * 由于 C++ 实现的差异，索引运算符 `operator []` 特别难以重载。具体来说，其他语言中的 get 和 set 运算符通常分为两种方法，以便可以将其他逻辑打包到这些运算中；C# 使用 `this[type key] { get { ... } set { ... }}`，Python 使用 `__getitem__` 和 `__setitem__`，等等。在 C++ 中，如果 `operator []` 的返回类型是引用并且方法是常量的，它通常表示 *setter*，而 *getter* 通常是一个常量函数，通过值返回对象。在没有任何严格的规则且有多个索引运算符的情况下，用户可以通过使用 `%rename` 来选择要使用的 getter 和 setter，如先前所示。
> * 某些 C++ 运算符的语义可能与目标语言中的语义不匹配。

## 6.17 对类的扩展

New methods can be added to a class using the `%extend` directive. This directive is primarily used in conjunction with proxy classes to add additional functionality to an existing class. For example :

> 可以使用 `%extend` 指令将新方法添加到类中。该指令主要与代理类结合使用，以向现有类添加其他功能。例如：

```
%module vector
%{
#include "vector.h"
%}

class Vector {
public:
  double x, y, z;
  Vector();
  ~Vector();
  ... bunch of C++ methods ...
  %extend {
    char *__str__() {
      static char temp[256];
      sprintf(temp, "[ %g, %g, %g ]", $self->x, $self->y, $self->z);
      return &temp[0];
    }
  }
};
```

This code adds a `__str__` method to our class for producing a string representation of the object. In Python, such a method would allow us to print the value of an object using the `print`command.

> 这段代码向我们的类中添加了一个 `__str__` 方法，用于生成对象的字符串表示形式。在 Python 中，这种方法将允许我们使用 `print` 命令来打印对象的值。

```python
>>>
>>> v = Vector();
>>> v.x = 3
>>> v.y = 4
>>> v.z = 0
>>> print(v)
[ 3.0, 4.0, 0.0 ]
>>>
```

The C++ 'this' pointer is often needed to access member variables, methods etc. The `$self` special variable should be used wherever you could use 'this'. The example above demonstrates this for accessing member variables. Note that the members dereferenced by `$self` must be public members as the code is ultimately generated into a global function and so will not have any access to non-public members. The implicit 'this' pointer that is present in C++ methods is not present in `%extend` methods. In order to access anything in the extended class or its base class, an explicit 'this' is required. The following example shows how one could access base class members:

> 经常需要 C++ 的 `this` 指针来访问成员变量、方法等。在任何可以使用 `this` 的地方都应使用 `$self` 特殊变量。上面的示例演示了如何访问成员变量。请注意，由于最终将代码生成到全局函数中，因此被 `$self` 解引用的成员必须是公有成员，因此将无法访问非公有成员。C++ 方法中存在的隐式 `this` 指针在 `%extend` 方法中不存在。为了访问扩展类或其基类中的任何内容，需要显式的 `this`。以下示例显示了如何访问基类成员：

```
struct Base {
  virtual void method(int v) {
    ...
  }
  int value;
};
struct Derived : Base {
};
%extend Derived {
  virtual void method(int v) {
    $self->Base::method(v); // akin to this->Base::method(v);
    $self->value = v;       // akin to this->value = v;
    ...
  }
}
```

The following special variables are expanded if used within a %extend block: `$name`, `$symname`, `$overname`, `$decl`, `$fulldecl`, `$parentclassname` and `$parentclasssymname`. The [Special variables](http://swig.org/Doc3.0/Customization.html#Customization_exception_special_variables) section provides more information each of these special variables.

The `%extend` directive follows all of the same conventions as its use with C structures. Please refer to the [Adding member functions to C structures](http://swig.org/Doc3.0/ SWIG .html# SWIG _adding_member_functions) section for further details.

**Compatibility note:** The `%extend` directive is a new name for the `%addmethods` directive in SWIG 1.1. Since `%addmethods` could be used to extend a structure with more than just methods, a more suitable directive name has been chosen.

> 如果在 `%extend` 块中使用以下特殊变量，则会对其进行扩展：`$name`、`$symname`、`$overname`、`$decl`、`$fulldecl`、`$parentclassname` 和 `$parentclasssymname`。[特殊变量](http://swig.org/Doc3.0/Customization.html#Customization_exception_special_variables)部分提供了每个这些特殊变量的更多信息。
>
> `%extend` 指令遵循与 C 结构体一起使用时一样的约定。有关更多详细信息，请参阅[向 C 结构体添加成员函数](http://swig.org/Doc3.0/SWIG.html#SWIG_adding_member_functions)。
>
> **注意兼容性**：`%extend` 指令是 SWIG 1.1 中 `%addmethods` 指令的新名称。由于 `%addmethods` 可以用于扩展结构体而不仅仅是方法，因此选择了一个更合适的指令名称。

## 6.18 模板

Template type names may appear anywhere a type is expected in an interface file. For example:

> 模板类型名称可能会出现在接口文件中任何需要该类型的位置。例如：

```c++
void foo(vector<int> *a, int n);
void bar(list<int, 100> *x);
```

There are some restrictions on the use of non-type arguments. Simple literals are supported, and so are some constant expressions. However, use of '<' and '>' within a constant expressions currently is not supported by SWIG ('<=' and '>=' are though). For example:

> 使用非类型参数有一些限制。支持简单文字，某些常量表达式也受支持。但是，SWIG 当前不支持在常量表达式中使用 `<` 和 `>`（但是使用 `<=` 和 `> =`）。例如：

```c++
void bar(list<int, 100> *x);                // OK
void bar(list<int, 2*50> *x);               // OK
void bar(list<int, (2>1 ? 100 : 50)> *x)    // Not supported
```

The type system is smart enough to figure out clever games you might try to play with `typedef`. For instance, consider this code:

> 类型系统足够聪明，可以找出你可以尝试使用 `typedef` 玩的把戏。例如，考虑以下代码：

```c++
typedef int Integer;
void foo(vector<int> *x, vector<Integer> *y);
```

In this case, `vector<Integer>` is exactly the same type as `vector<int>`. The wrapper for `foo()` will accept either variant.

Starting with SWIG-1.3.7, simple C++ template declarations can also be wrapped. SWIG-1.3.12 greatly expands upon the earlier implementation. Before discussing this any further, there are a few things you need to know about template wrapping. First, a bare C++ template does not define any sort of runnable object-code for which SWIG can normally create a wrapper. Therefore, in order to wrap a template, you need to give SWIG information about a particular template instantiation (e.g., `vector<int>`,`array<double>`, etc.). Second, an instantiation name such as `vector<int>` is generally not a valid identifier name in most target languages. Thus, you will need to give the template instantiation a more suitable name such as `intvector` when creating a wrapper.

To illustrate, consider the following template definition:

> 在这种情况下，`vector<Integer>` 与 `vector<int>` 的类型完全相同。`foo()` 的包装器将接受任何一个变体。
>
> 从 SWIG-1.3.7 开始，还可以包装简单的 C++ 模板声明。SWIG-1.3.12 在较早的实现上有了很大的扩展。在进一步讨论之前，你需要了解一些有关模板包装的知识。首先，裸露的 C++ 模板没有定义任何类型的可运行目标代码以供 SWIG 为其创建包装器。因此，为了包装模板，你需要为 SWIG 提供有关特定模板实例化的信息（例如，`vector<int>`、`array<double>` 等）。其次，在大多数目标语言中，诸如 `vector<int>` 之类的实例化名称通常不是有效的标识符名称。因此，在创建包装器时，需要为模板实例化指定一个更合适的名称，例如 `intvector`。
>
> 为了说明，请考虑以下模板定义：

```c++
template<class T> class List {
private:
    T *data;
    int nitems;
    int maxitems;
public:
    List(int max) {
      data = new T [max];
      nitems = 0;
      maxitems = max;
    }
    ~List() {
      delete [] data;
    };
    void append(T obj) {
      if (nitems < maxitems) {
        data[nitems++] = obj;
      }
    }
    int length() {
      return nitems;
    }
    T get(int n) {
      return data[n];
    }
};
```

By itself, this template declaration is useless-- SWIG simply ignores it because it doesn't know how to generate any code until unless a definition of `T` is provided.

One way to create wrappers for a specific template instantiation is to simply provide an expanded version of the class directly like this:

> 就其本身而言，此模板声明是无用的——SWIG 只会忽略它，因为除非提供 `T` 的定义，否则它不知道如何生成代码。
>
> 为特定模板实例创建包装器的一种方法是，直接像下面这样直接提供类的扩展版本：

```
%rename(intList) List<int>;       // Rename to a suitable identifier
class List<int> {
private:
    int *data;
    int nitems;
    int maxitems;
public:
    List(int max);
    ~List();
    void append(int obj);
    int length();
    int get(int n);
};
```

The `%rename` directive is needed to give the template class an appropriate identifier name in the target language (most languages would not recognize C++ template syntax as a valid class name). The rest of the code is the same as what would appear in a normal class definition.

Since manual expansion of templates gets old in a hurry, the `%template` directive can be used to create instantiations of a template class. Semantically, `%template` is simply a shortcut---it expands template code in exactly the same way as shown above. Here are some examples:

> 需要使用 `%rename` 指令，在目标语言为模板类提供适当的标识符名称（大多数语言无法将 C++ 模板语法识别为有效的类名称）。其余代码与普通类定义中显示的代码相同。
>
> 由于模板的手动扩展已经很老旧了，因此可以使用 `%template` 指令来创建模板类的实例化。从语义上讲，`%template` 只是一种快捷方式——它以与上面所示完全相同的方式扩展模板代码。这里有些例子：

```
/* Instantiate a few different versions of the template */
%template(intList) List<int>;
%template(doubleList) List<double>;
```

The argument to `%template()` is the name of the instantiation in the target language. The name you choose should not conflict with any other declarations in the interface file with one exception---it is okay for the template name to match that of a typedef declaration. For example:

> `%template()` 的参数是目标语言中实例化的名称。你选择的名称不应与接口文件中的任何其他声明相冲突，只有一个例外——模板名称可以与 `typedef` 声明的名称匹配。例如：

```
%template(intList) List<int>;
...
typedef List<int> intList;    // OK
```

 SWIG can also generate wrappers for function templates using a similar technique. For example:

> SWIG 还可以使用类似的技术为函数模板生成包装器。例如：

```
// Function template
template<class T> T max(T a, T b) { return a > b ? a : b; }

// Make some different versions of this function
%template(maxint) max<int>;
%template(maxdouble) max<double>;
```

In this case, `maxint` and `maxdouble` become unique names for specific instantiations of the function.

The number of arguments supplied to `%template` should match that in the original template definition. Template default arguments are supported. For example:

> 在这种情况下，对于函数的特定实例，`maxint` 和 `maxdouble` 成为唯一的名称。
>
> 提供给 `%template` 的参数数量应与原始模板定义中的参数数量匹配。支持模板默认参数。例如：

```
template vector<typename T, int max=100> class vector {
...
};

%template(intvec) vector<int>;           // OK
%template(vec1000) vector<int, 1000>;     // OK
```

The `%template` directive should not be used to wrap the same template instantiation more than once in the same scope. This will generate an error. For example:

> `%template` 指令不应在同一作用域中多次包装相同的模板实例。这将产生一个错误。例如：

```
%template(intList) List<int>;
%template(Listint) List<int>;    // Error.   Template already wrapped.
```

This error is caused because the template expansion results in two identical classes with the same name. This generates a symbol table conflict. Besides, it probably more efficient to only wrap a specific instantiation only once in order to reduce the potential for code bloat.

Since the type system knows how to handle `typedef`, it is generally not necessary to instantiate different versions of a template for typenames that are equivalent. For instance, consider this code:

> 导致此错误的原因是模板扩展导致两个相同名称的类。这会产生符号表冲突。此外，为减少代码膨胀的可能性，仅将特定实例包装一次可能更有效。
>
> 由于类型系统知道如何处理 `typedef`，因此通常不必为等效的类型名实例化不同版本的模板。例如，考虑以下代码：

```
%template(intList) vector<int>;
typedef int Integer;
...
void foo(vector<Integer> *x);
```

In this case, `vector<Integer>` is exactly the same type as`vector<int>`. Any use of `Vector<Integer>` is mapped back to the instantiation of `vector<int>` created earlier. Therefore, it is not necessary to instantiate a new class for the type `Integer` (doing so is redundant and will simply result in code bloat).

When a template is instantiated using `%template`, information about that class is saved by SWIG and used elsewhere in the program. For example, if you wrote code like this,

> 在这种情况下，`vector<Integer>` 与 `vector<int>` 的类型完全相同。对 `Vector<Integer>` 的任何使用都将映射回先前创建的 `vector<int>` 的实例化。因此，不必为类型 `Integer` 实例化新类（这样做是多余的，只会导致代码膨胀）。
>
> 使用 `%template` 实例化模板时，有关该类的信息将由 SWIG 保存并在程序的其他位置使用。例如，如果你编写了这样的代码，

```
...
%template(intList) List<int>;
...
class UltraList : public List<int> {
  ...
};
```

then SWIG knows that `List<int>` was already wrapped as a class called `intList` and arranges to handle the inheritance correctly. If, on the other hand, nothing is known about `List<int>`, you will get a warning message similar to this:

> 然后 SWIG 知道 `List<int>` 已经被包装为名为 `intList` 的类，并安排正确处理继承。另一方面，如果对 `List<int>` 一无所知，则会收到类似以下的警告消息：

```
example.h:42: Warning 401. Nothing known about class 'List<int >'. Ignored.
example.h:42: Warning 401. Maybe you forgot to instantiate 'List<int >' using %template.
```

If a template class inherits from another template class, you need to make sure that base classes are instantiated before derived classes. For example:

> 如果模板类是从另一个模板类继承的，则需要确保在派生类之前实例化基类。例如：

```
template<class T> class Foo {
...
};

template<class T> class Bar : public Foo<T> {
...
};

// Instantiate base classes first
%template(intFoo) Foo<int>;
%template(doubleFoo) Foo<double>;

// Now instantiate derived classes
%template(intBar) Bar<int>;
%template(doubleBar) Bar<double>;
```

The order is important since SWIG uses the instantiation names to properly set up the inheritance hierarchy in the resulting wrapper code (and base classes need to be wrapped before derived classes). Don't worry--if you get the order wrong, SWIG should generate a warning message.

Occasionally, you may need to tell SWIG about base classes that are defined by templates, but which aren't supposed to be wrapped. Since SWIG is not able to automatically instantiate templates for this purpose, you must do it manually. To do this, simply use the empty template instantiation, that is, `%template` with no name. For example:

> 该顺序很重要，因为 SWIG 在生成的包装器代码中使用实例化名称设置了继承层次结构（并且需要在派生类之前包装基类）。不用担心——如果你收到错误的顺序，SWIG 应该会生成一条警告消息。
>
> 有时，你可能需要告诉 SWIG 有关模板定义的基类的信息，但这些基类不应包装。由于 SWIG 不能为此自动实例化模板，因此你必须手动进行。为此，只需使用空模板实例化，即不带名称和 `%template`。例如：

```
// Instantiate traits<double, double>, but don't wrap it.
%template() traits<double, double>;
```

If you have to instantiate a lot of different classes for many different types, you might consider writing a SWIG macro. For example:

> 如果必须为许多不同的类型实例化许多不同的类，则可以考虑编写 SWIG 宏。例如：

```
%define TEMPLATE_WRAP(prefix, T...)
%template(prefix ## Foo) Foo<T >;
%template(prefix ## Bar) Bar<T >;
...
%enddef

TEMPLATE_WRAP(int, int)
TEMPLATE_WRAP(double, double)
TEMPLATE_WRAP(String, char *)
TEMPLATE_WRAP(PairStringInt, std::pair<string, int>)
...
```

Note the use of a vararg macro for the type T. If this wasn't used, the comma in the templated type in the last example would not be possible.

The SWIG template mechanism *does* support specialization. For instance, if you define a class like this,

> 请注意，类型 `T` 使用了 `vararg` 宏。如果不使用该宏，则上一个示例中的模板化类型的逗号将不可能。
>
> SWIG 模板机制*支持*特化。例如，如果你定义这样的类，

```c++
template<> class List<int> {
private:
    int *data;
    int nitems;
    int maxitems;
public:
    List(int max);
    ~List();
    void append(int obj);
    int length();
    int get(int n);
};
```

then SWIG will use this code whenever the user expands `List<int>`. In practice, this may have very little effect on the underlying wrapper code since specialization is often used to provide slightly modified method bodies (which are ignored by SWIG ). However, special SWIG directives such as `%typemap`, `%extend`, and so forth can be attached to a specialization to provide customization for specific types.

Partial template specialization is partially supported by SWIG . For example, this code defines a template that is applied when the template argument is a pointer.

> 然后 SWIG 将在用户每次扩展 `List<int>` 时使用此代码。在实践中，这对底层包装器代码的影响可能很小，因为特化通常用于提供经过稍微修改的方法主体（SWIG 会忽略它们）。但是，特殊的 SWIG 指令（如 `%typemap`、`%extend` 等）可以附加到特化中，以为特定类型提供自定义。
>
> SWIG 部分支持部分模板特化。例如，此代码定义了一个模板，当模板参数为指针时将应用该模板。

```c++
template<class T> class List<T*> {
private:
    T *data;
    int nitems;
    int maxitems;
public:
    List(int max);
    ~List();
    void append(int obj);
    int length();
    T get(int n);
};
```

SWIG supports both template explicit specialization and partial specialization. Consider:

> SWIG 同时支持模板显式特化和部分特化。考虑下面的情况

```c++
template<class T1, class T2> class Foo { };                     // (1) primary template
template<>                   class Foo<double *, int *> { };    // (2) explicit specialization
template<class T1, class T2> class Foo<T1, T2 *> { };           // (3) partial specialization
```

SWIG is able to properly match explicit instantiations:

> SWIG 可以正确匹配显式特化：

```c++
Foo<double *, int *>     // explicit specialization matching (2)
```

SWIG implements template argument deduction so that the following partial specialization examples work just like they would with a C++ compiler:

> SWIG 实现了模板参数推导，因此以下部分特化示例的工作方式与使用 C++ 编译器一样：

```c++
Foo<int *, int *>        // partial specialization matching (3)
Foo<int *, const int *>  // partial specialization matching (3)
Foo<int *, int **>       // partial specialization matching (3)
```

Member function templates are supported. The underlying principle is the same as for normal templates-- SWIG can't create a wrapper unless you provide more information about types. For example, a class with a member template might look like this:

> 支持成员函数模板。基本原理与普通模板相同——除非你提供有关类型的更多信息，否则 SWIG 无法创建包装器。例如，带有成员模板的类可能如下所示：

```c++
class Foo {
public:
  template<class T> void bar(T x, T y) { ... };
  ...
};
```

To expand the template, simply use `%template` inside the class.

> 为了扩展模板，在类中使用 `%template`。

```
class Foo {
public:
  template<class T> void bar(T x, T y) { ... };
  ...
  %template(barint)    bar<int>;
  %template(bardouble) bar<double>;
};
```

Or, if you want to leave the original class definition alone, just do this:

> 或者，如果你想初始类定义独立出来，这样做：

```
class Foo {
public:
  template<class T> void bar(T x, T y) { ... };
  ...
};
...
%extend Foo {
  %template(barint)    bar<int>;
  %template(bardouble) bar<double>;
};
```

or simply

> 或者简单点

```
class Foo {
public:
  template<class T> void bar(T x, T y) { ... };
  ...
};
...

%template(bari) Foo::bar<int>;
%template(bard) Foo::bar<double>;
```

In this case, the `%extend` directive is not needed, and `%template`does exactly the same job, i.e., it adds two new methods to the Foo class.

Note: because of the way that templates are handled, the `%template` directive must always appear *after* the definition of the template to be expanded.

Now, if your target language supports overloading, you can even try

> 在这种情况下，不需要 `%extend` 指令，并且 `%template` 可以完成完全相同的工作，即它向 `Foo` 类添加了两个新方法。
>
> 注意：由于处理模板的方式，`%template` 指令必须始终在要扩展的模板定义之后出现。
>
> 现在，如果你的目标语言支持重载，你甚至可以尝试

```
%template(bar) Foo::bar<int>;
%template(bar) Foo::bar<double>;
```

and since the two new wrapped methods have the same name 'bar', they will be overloaded, and when called, the correct method will be dispatched depending on the argument type.

When used with members, the `%template` directive may be placed in another template class. Here is a slightly perverse example:

> 并且由于这两个新包装的方法具有相同的名称 `bar`，因此它们将被重载，并且在调用时，将根据参数类型调度正确的方法。
>
> 与成员一起使用时，可以将 `%template` 指令放置在另一个模板类中。这是一个有点反常的例子：

```
// A template
template<class T> class Foo {
public:
  // A member template
  template<class S> T bar(S x, S y) { ... };
  ...
};

// Expand a few member templates
%extend Foo {
  %template(bari) bar<int>;
  %template(bard) bar<double>;
}

// Create some wrappers for the template
%template(Fooi) Foo<int>;
%template(Food) Foo<double>;
```

Miraculously, you will find that each expansion of `Foo` has member functions `bari()` and `bard()` added.

A common use of member templates is to define constructors for copies and conversions. For example:

> 奇迹，你会发现 `Foo` 的每个扩展都添加了成员函数 `bari()` 和 `bard()`。
>
> 成员模板的常见用法是为拷贝和转换定义构造函数。例如：

```c++
template<class T1, class T2> struct pair {
  T1 first;
  T2 second;
  pair() : first(T1()), second(T2()) { }
  pair(const T1 &x, const T2 &y) : first(x), second(y) { }
  template<class U1, class U2> pair(
      const pair<U1, U2> &x) : first(x.first), second(x.second) { }
};
```

This declaration is perfectly acceptable to SWIG , but the constructor template will be ignored unless you explicitly expand it. To do that, you could expand a few versions of the constructor in the template class itself. For example:

> SWIG 完全可以接受此声明，但是除非明确扩展它，否则构造函数模板将被忽略。为此，你可以在模板类本身中扩展构造函数的几个版本。例如：

```
%extend pair {
  %template(pair) pair<T1, T2>;        // Generate default copy constructor
};
```

When using `%extend` in this manner, notice how you can still use the template parameters in the original template definition.

Alternatively, you could expand the constructor template in selected instantiations. For example:

> 当以这种方式使用 `%extend` 时，请注意如何仍然可以在原始模板定义中使用模板参数。
>
> 另外，你可以在选定的实例中扩展构造函数模板。例如：

```
// Instantiate a few versions
%template(pairii) pair<int, int>;
%template(pairdd) pair<double, double>;

// Create a default constructor only
%extend pair<int, int> {
  %template(paird) pair<int, int>;         // Default constructor
};

// Create default and conversion constructors
%extend pair<double, double> {
  %template(paird) pair<double, dobule>;   // Default constructor
  %template(pairc) pair<int, int>;         // Conversion constructor
};
```

And if your target language supports overloading, then you can try instead:

> 而且，如果你的目标语言支持重载，你还可以试试：

```
// Create default and conversion constructors
%extend pair<double, double> {
  %template(pair) pair<double, dobule>;   // Default constructor
  %template(pair) pair<int, int>;         // Conversion constructor
};
```

In this case, the default and conversion constructors have the same name. Hence, SWIG will overload them and define an unique visible constructor, that will dispatch the proper call depending on the argument type.

If all of this isn't quite enough and you really want to make someone's head explode, SWIG directives such as `%rename`, `%extend`, and `%typemap` can be included directly in template definitions. For example:

> 在这种情况下，默认构造函数和转换构造函数具有相同的名称。因此，SWIG 将重载它们并定义一个唯一的可见构造函数，该构造函数将根据参数类型调度适当的调用。
>
> 如果所有这些还不够，并且你真的想让某人的脑袋爆炸，那么可以在模板定义中直接包含诸如 `%rename`、`%extend` 和 `%typemap` 之类的 SWIG 指令。例如：

```
// File : list.h
template<class T> class List {
  ...
public:
  %rename(__getitem__) get(int);
  List(int max);
  ~List();
  ...
  T get(int index);
  %extend {
    char *__str__() {
      /* Make a string representation */
      ...
    }
  }
};
```

In this example, the extra SWIG directives are propagated to *every*template instantiation.

It is also possible to separate these declarations from the template class. For example:

> 在此示例中，额外的 SWIG 指令传播到*每个*模板实例化。
>
> 也可以将这些声明与模板类分开。例如：

```
%rename(__getitem__) List::get;
%extend List {
  char *__str__() {
    /* Make a string representation */
    ...
  }
  /* Make a copy */
  T *__copy__() {
    return new List<T>(*$self);
  }
};

...
template<class T> class List {
    ...
    public:
    List() { }
    T get(int index);
    ...
};
```

When `%extend` is decoupled from the class definition, it is legal to use the same template parameters as provided in the class definition. These are replaced when the template is expanded. In addition, the `%extend` directive can be used to add additional methods to a specific instantiation. For example:

> 当 `%extend` 与类定义脱钩时，使用与类定义中提供的相同的模板参数是合法的。扩展模板时将替换它们。另外，`%extend` 指令可用于向特定实例添加其他方法。例如：

```
%template(intList) List<int>;

%extend List<int> {
    void blah() {
        printf("Hey, I'm an List<int>!\n");
    }
};
```

SWIG even supports overloaded templated functions. As usual the `%template` directive is used to wrap templated functions. For example:

> SWIG 甚至支持重载函数模板。和往常一样，`%template` 指令用于包装函数模板。例如：

```
template<class T> void foo(T x) { };
template<class T> void foo(T x, T y) { };

%template(foo) foo<int>;
```

This will generate two overloaded wrapper methods, the first will take a single integer as an argument and the second will take two integer arguments.

It is even possible to extend a class via `%extend` with template methods, for example:

> 这将生成两个重载的包装器方法，第一个将使用单个整数作为参数，第二个将使用两个整数参数。
>
> 甚至可以通过 `%extend` 模板方法扩展一个类，例如：

```
%include <std_string.i>

%inline %{
class ExtendMe {
public:
  template <typename T>
  T do_stuff_impl(int a, T b, double d) {
    return b;
  }
};
%}

%extend ExtendMe {
  template<typename T>
  T do_overloaded_stuff(T b) {
    return $self->do_stuff_impl(0, b, 4.0);
  }
}
%template(do_overloaded_stuff) ExtendMe::do_overloaded_stuff<std::string>;
%template(do_overloaded_stuff) ExtendMe::do_overloaded_stuff<double>;
```

The wrapped `ExtendMe` class will then have two (overloaded) methods called `do_overloaded_stuff`.

**Compatibility Note**: Extending a class with template methods was added in version 3.0.12

Needless to say, SWIG's template support provides plenty of opportunities to break the universe. That said, an important final point is that **SWIG does not perform extensive error checking of templates!** Specifically, SWIG does not perform type checking nor does it check to see if the actual contents of the template declaration make any sense. Since the C++ compiler checks this when it compiles the resulting wrapper file, there is no practical reason for SWIG to duplicate this functionality.

> 包装好的 `ExtendMe` 类将有两个（重载）方法，称为 `do_overloaded_stuff`。
>
> **注意兼容性**：在版本 3.0.12 中添加了使用模板方法扩展类。
>
> 不用说，SWIG 的模板支持为打通宇宙提供了很多机会。就是说，最后的重点是 **SWIG 不会执行模板的大量错误检查**！具体来说，SWIG 不会执行类型检查，也不会检查模板声明的实际内容是否有意义。由于 C++ 编译器在编译结果包装器文件时会对此进行检查，因此 SWIG 没有实际理由来复制此功能。

```
template <class T> class OuterTemplateClass {};

// The nested class OuterClass::InnerClass inherits from the template class
// OuterTemplateClass<OuterClass::InnerStruct> and thus the template needs
// to be expanded with %template before the OuterClass declaration.
%template(OuterTemplateClass_OuterClass__InnerStruct)
    OuterTemplateClass<OuterClass::InnerStruct>


// Don't forget to use %feature("flatnested") for OuterClass::InnerStruct and
// OuterClass::InnerClass if the target language doesn't support nested classes.
class OuterClass {
    public:
        // Forward declarations:
        struct InnerStruct;
        class InnerClass;
};

struct OuterClass::InnerStruct {};

// Expanding the template at this point with %template is too late as the
// OuterClass::InnerClass declaration is processed inside OuterClass.

class OuterClass::InnerClass : public OuterTemplateClass<InnerStruct> {};
```

**Compatibility Note**: The first implementation of template support relied heavily on macro expansion in the preprocessor. Templates have been more tightly integrated into the parser and type system in SWIG-1.3.12 and the preprocessor is no longer used. Code that relied on preprocessing features in template expansion will no longer work. However, SWIG still allows the # operator to be used to generate a string from a template argument.

**Compatibility Note**: In earlier versions of SWIG , the `%template`directive introduced a new class name. This name could then be used with other directives. For example:

> **注意兼容性**：模板支持的第一个实现在很大程度上依赖于预处理器中的宏扩展。模板已在 SWIG-1.3.12 中更紧密地集成到解析器和类型系统中，并且不再使用预处理器。依靠模板扩展中的预处理功能的代码将不再起作用。但是，SWIG 仍然允许使用 `#` 运算符从模板参数生成字符串。
>
> **注意兼容性**：在 SWIG 的早期版本中，`%template` 指令引入了新的类名。然后，该名称可以与其他指令一起使用。例如：

```
%template(vectori) vector<int>;
%extend vectori {
    void somemethod() { }
};
```

This behavior is no longer supported. Instead, you should use the original template name as the class name. For example:

> 不再支持这种行为。但是，你要使用原始模板名作为类名。例如：

```
%template(vectori) vector<int>;
%extend vector<int> {
    void somemethod() { }
};
```

Similar changes apply to typemaps and other customization features.

> 类似的更改适用于类型映射和其他自定义功能。

## 6.19 命名空间

Support for C++ namespaces is comprehensive, but by default simple, however, some target languages can turn on more advanced namespace support via the [nspace feature](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_nspace), described later. Code within unnamed namespaces is ignored as there is no external access to symbols declared within the unnamed namespace. Before detailing the default implementation for named namespaces, it is worth noting that the semantics of C++ namespaces is extremely non-trivial--especially with regard to the C++ type system and class machinery. At a most basic level, namespaces are sometimes used to encapsulate common functionality. For example:

> 对 C++ 命名空间的支持是全面的，默认情况下却很简单，但是某些目标语言可以通过 [`nspace` 功能](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_nspace)打开更高级的命名空间支持，稍后会描述。未命名的命名空间中的代码将被忽略，因为无法从外部访问未命名的命名空间中声明的符号。在详细说明命名空间的默认实现之前，值得注意的是 C++ 命名空间的语义非常重要——特别是对于 C++ 类型系统和类机制而言。在最基本的级别上，命名空间有时用于封装通用功能。例如：

```c++
namespace math {
  double sin(double);
  double cos(double);

  class Complex {
    double im, re;
  public:
    ...
  };
  ...
};
```

Members of the namespace are accessed in C++ by prepending the namespace prefix to names. For example:

> 在 C++ 中，通过将命名空间前缀放在名称之前，可以访问命名空间的成员。例如：

```c++
double x = math::sin(1.0);
double magnitude(math::Complex *c);
math::Complex c;
...
```

At this level, namespaces are relatively easy to manage. However, things start to get very ugly when you throw in the other ways a namespace can be used. For example, selective symbols can be exported from a namespace with `using`.

> 在此级别上，命名空间相对易于管理。但是，当你以其他方式使用命名空间时，事情变得非常丑陋。例如，可以使用 `using` 从命名空间中导出选择的符号。

```c++
using math::Complex;
double magnitude(Complex *c);       // Namespace prefix stripped
```

Similarly, the contents of an entire namespace can be made available like this:

> 类似地，整个命名空间的内容可以这样提供：

```c++
using namespace math;
double x = sin(1.0);
double magnitude(Complex *c);
```

Alternatively, a namespace can be aliased:

> 或者，命名空间可以这样表示：

```c++
namespace M = math;
double x = M::sin(1.0);
double magnitude(M::Complex *c);
```

Using combinations of these features, it is possible to write head-exploding code like this:

> 使用这些功能的组合，可以编写如下烧脑的代码：

```c++
namespace A {
  class Foo {
  };
}

namespace B {
  namespace C {
    using namespace A;
  }
  typedef C::Foo FooClass;
}

namespace BIGB = B;

namespace D {
  using BIGB::FooClass;
  class Bar : public FooClass {
  }
};

class Spam : public D::Bar {
};

void evil(A::Foo *a, B::FooClass *b, B::C::Foo *c, BIGB::FooClass *d,
          BIGB::C::Foo *e, D::FooClass *f);
```

Given the possibility for such perversion, it's hard to imagine how every C++ programmer might want such code wrapped into the target language. Clearly this code defines three different classes. However, one of those classes is accessible under at least six different names!

SWIG fully supports C++ namespaces in its internal type system and class handling code. If you feed SWIG the above code, it will be parsed correctly, it will generate compilable wrapper code, and it will produce a working scripting language module. However, the default wrapping behavior is to flatten namespaces in the target language. This means that the contents of all namespaces are merged together in the resulting scripting language module. For example, if you have code like this,

> 考虑到这种变态的可能性，很难想象 C++ 程序员可能会如何将这样的代码包装到目标语言中。显然，此代码定义了三个不同的类。但是，可以至少使用六个不同的名称来访问其中一个类！
>
> SWIG 在其内部类型系统和类处理代码中完全支持 C++ 命名空间。如果将上面的代码提供给 SWIG ，它将被正确地解析，它将生成可编译的包装器代码，并且将产生一个有效的脚本语言模块。但是，默认的包装行为是将目标语言中的命名空间展平。这意味着所有命名空间的内容在结果脚本语言模块中合并在一起。例如，如果你有这样的代码，

```
%module foo
namespace foo {
  void bar(int);
  void spam();
}

namespace bar {
  void blah();
}
```

then SWIG simply creates three wrapper functions `bar()`, `spam()`, and `blah()` in the target language. SWIG does not prepend the names with a namespace prefix nor are the functions packaged in any kind of nested scope.

There is some rationale for taking this approach. Since C++ namespaces are often used to define modules in C++, there is a natural correlation between the likely contents of a SWIG module and the contents of a namespace. For instance, it would not be unreasonable to assume that a programmer might make a separate extension module for each C++ namespace. In this case, it would be redundant to prepend everything with an additional namespace prefix when the module itself already serves as a namespace in the target language. Or put another way, if you want SWIG to keep namespaces separate, simply wrap each namespace with its own SWIG interface.

Because namespaces are flattened, it is possible for symbols defined in different namespaces to generate a name conflict in the target language. For example:

> 然后 SWIG 只需用目标语言创建三个包装函数 `bar()`、`spam()` 和 `blah()`。SWIG 不会在名称前添加命名空间前缀，函数也不会打包在任何嵌套作用域中。
>
> 采用这种方法有一些理由。由于 C++ 命名空间通常用于在 C++ 中定义模块，因此可能 SWIG 模块的内容与命名空间的内容之间存在自然的关联。例如，假设程序员可以为每个 C++ 命名空间创建一个单独的扩展模块，这并非没有道理。在这种情况下，当模块本身已经用作目标语言中的命名空间时，在所有内容前面都添加一个额外的命名空间前缀将是多余的。或换一种说法，如果你希望 SWIG 将命名空间分隔开，只需将每个命名空间都用其自己的 SWIG 接口包装即可。
>
> 由于命名空间被展平，因此在不同命名空间中定义的符号可能会在目标语言中产生名称冲突。例如：

```c++
namespace A {
  void foo(int);
}
namespace B {
  void foo(double);
}
```

When this conflict occurs, you will get an error message that resembles this:

> 发生此冲突时，你将收到类似于以下内容的错误消息：

```
example.i:26. Error. 'foo' is multiply defined in the generated target language module.
example.i:23. Previous declaration of 'foo'
```

To resolve this error, simply use `%rename` to disambiguate the declarations. For example:

> 要解决此错误，只需使用 `%rename` 来消除声明的歧义。例如：

```
%rename(B_foo) B::foo;
...
namespace A {
  void foo(int);
}
namespace B {
  void foo(double);     // Gets renamed to B_foo
}
```

Similarly, `%ignore` can be used to ignore declarations.

`using` declarations do not have any effect on the generated wrapper code. They are ignored by SWIG language modules and they do not result in any code. However, these declarations *are*used by the internal type system to track type-names. Therefore, if you have code like this:

> 同样，`%ignore` 可用于忽略声明。
>
> `using` 声明对生成的包装器代码没有任何影响。SWIG 语言模块将忽略它们，并且不会产生任何代码。但是，这些声明由内部类型系统用于跟踪类型名称。因此，如果你有这样的代码：

```c++
namespace A {
  typedef int Integer;
}
using namespace A;
void foo(Integer x);
```

SWIG knows that `Integer` is the same as `A::Integer` which is the same as `int`.

Namespaces may be combined with templates. If necessary, the`%template` directive can be used to expand a template defined in a different namespace. For example:

> SWIG 知道 `Integer` 与 `A::Integer` 相同，后者与 `int` 相同。
>
> 命名空间可以与模板结合使用。如有必要，可以使用 `%template` 指令扩展在不同命名空间中定义的模板。例如：

```
namespace foo {
    template<typename T> T max(T a, T b) { return a > b ? a : b; }
}

using foo::max;

%template(maxint)   max<int>;           // Okay.
%template(maxfloat) foo::max<float>;    // Okay (qualified name).

namespace bar {
    using namespace foo;
    %template(maxdouble)  max<double>;    // Okay.
}
```

The combination of namespaces and other SWIG directives may introduce subtle scope-related problems. The key thing to keep in mind is that all SWIG generated wrappers are produced in the *global* namespace. Symbols from other namespaces are always accessed using fully qualified names---names are never imported into the global space unless the interface happens to do so with a `using` declaration. In almost all cases, SWIG adjusts typenames and symbols to be fully qualified. However, this is not done in code fragments such as function bodies, typemaps, exception handlers, and so forth. For example, consider the following:

> 命名空间和其他 SWIG 指令的组合可能会引起与作用域相关的微妙问题。关键要记住，所有 SWIG 生成的包装器都是在*全局*命名空间中生成的。总是使用完全限定的名称来访问来自其他命名空间的符号——除非接口碰巧使用 `using` 声明，否则名称永远不会导入全局空间。在几乎所有情况下，SWIG 都会将类型名和符号调整为完全合法的。但是，在代码片段（例如函数体、类型映射、异常处理程序等）中并未做到这一点。例如，考虑以下内容：

```
namespace foo {
  typedef int Integer;
  class bar {
    public:
      ...
  };
}

%extend foo::bar {
  Integer add(Integer x, Integer y) {
    Integer r = x + y;        // Error. Integer not defined in this scope
    return r;
  }
};
```

In this case, SWIG correctly resolves the added method parameters and return type to `foo::Integer`. However, since function bodies aren't parsed and such code is emitted in the global namespace, this code produces a compiler error about `Integer`. To fix the problem, make sure you use fully qualified names. For example:

> 在这种情况下，SWIG 会正确解析添加的方法参数，并将类型返回为 `foo::Integer`。但是，由于未解析函数体，并且此类代码在全局命名空间中发出，因此此代码会产生有关 `Integer` 的编译器错误。要解决此问题，请确保使用完全限定的名称。例如：

```
%extend foo::bar {
  Integer add(Integer x, Integer y) {
    foo::Integer r = x + y;        // Ok.
    return r;
  }
};
```

**Note:** SWIG does *not* propagate `using` declarations to the resulting wrapper code. If these declarations appear in an interface, they should *also* appear in any header files that might have been included in a `%{ ... %}` section. In other words, don't insert extra`using` declarations into a SWIG interface unless they also appear in the underlying C++ code.

**Note:** Code inclusion directives such as `%{ ... %}` or `%inline %{ ... %}` should not be placed inside a namespace declaration. The code emitted by these directives will not be enclosed in a namespace and you may get very strange results. If you need to use namespaces with these directives, consider the following:

> **注意**：SWIG 不会将 `using` 声明传播到包装器代码。如果这些声明出现在接口中，它们也应该*还*出现在任何可能包含 `%{...%}` 部分的头文件中。换句话说，除非在基础 C++ 代码中也出现了多余的 `using` 声明，否则不要在 SWIG 接口中插入它们。
>
> **注意**：不应将代码包含指令，例如 `%{...%}` 或 `%inline %{...%}` 放在命名空间声明中。这些指令发出的代码不会包含在命名空间中，你可能会得到非常奇怪的结果。如果需要通过这些指令使用命名空间，请考虑以下事项：

```
// Good version
%inline %{
namespace foo {
  void bar(int) { ... }
  ...
}
%}

// Bad version.  Emitted code not placed in namespace.
namespace foo {
%inline %{
  void bar(int) { ... }   /* I'm bad */
  ...
  %}
}
```

**Note:** When the `%extend` directive is used inside a namespace, the namespace name is included in the generated functions. For example, if you have code like this,

> **注意**：在命名空间中使用 `%extend` 指令时，命名空间名称包含在生成的函数中。例如，如果你有这样的代码，

```
namespace foo {
  class bar {
    public:
      %extend {
        int blah(int x);
      };
  };
}
```

the added method `blah()` is mapped to a function `int foo_bar_blah(foo::bar *self, int x)`. This function resides in the global namespace.

**Note:** Although namespaces are flattened in the target language, the SWIG generated wrapper code observes the same namespace conventions as used in the input file. Thus, if there are no symbol conflicts in the input, there will be no conflicts in the generated code.

**Note:** In the same way that no resolution is performed on parameters, a conversion operator name must match exactly to how it is defined. Do not change the qualification of the operator. For example, suppose you had an interface like this:

> 添加的方法 `blah()` 映射到函数 `int foo_bar_blah(foo::bar *self, int x)`。该函数位于全局命名空间中。
>
> **注意**：尽管命名空间使用目标语言进行了展平，但是 SWIG 生成的包装器代码遵守与输入文件中相同的命名空间约定。因此，如果输入中没有符号冲突，则生成的代码中将没有冲突。
>
> **注意**：与不对参数执行任何解析一样，转换运算符名称必须与定义方式完全匹配。请勿更改运算符的限定词。例如，假设你有一个像这样的接口：

```c++
namespace foo {
  class bar;
  class spam {
    public:
    ...
    operator bar();      // Conversion of spam -> bar
    ...
  };
}
```

The following is how the feature is expected to be written for a successful match:

> 以下是成功匹配期望的功能编写方式：

```
%rename(tofoo) foo::spam::operator bar();
```

The following does not work as no namespace resolution is performed in the matching of conversion operator names:

> 由于在转换运算符名称的匹配中未执行命名空间解析，因此以下操作不起作用：

```
%rename(tofoo) foo::spam::operator foo::bar();
```

Note, however, that if the operator is defined using a qualifier in its name, then the feature must use it too...

> 但是请注意，如果使用名称中的限定符定义了运算符，则该功能也必须使用它。

```
%rename(tofoo) foo::spam::operator bar();      // will not match
%rename(tofoo) foo::spam::operator foo::bar(); // will match
namespace foo {
  class bar;
  class spam {
    public:
    ...
    operator foo::bar();
    ...
  };
}
```

**Compatibility Note:** Versions of SWIG prior to 1.3.32 were inconsistent in this approach. A fully qualified name was usually required, but would not work in some situations.

**Note:** The flattening of namespaces is only intended to serve as a basic namespace implementation. None of the target language modules are currently programmed with any namespace awareness. In the future, language modules may or may not provide more advanced namespace support.

> **注意兼容性**：1.3.32 之前的 SWIG 版本在此方法中不一致。通常需要一个完全限定的名称，但在某些情况下不起作用。
>
> **注意**：命名空间的展平仅旨在用作基本的命名空间实现。当前，目标语言模块都没有任何使用命名空间进行编程的意识。将来，语言模块可能会或可能不会提供更高级的命名空间支持。

### 6.19.1 针对命名空间的 `nspace` 功能

Some target languages provide support for the `nspace` [feature](http://swig.org/Doc3.0/Customization.html#Customization_features). The feature can be applied to any class, struct, union or enum declared within a named namespace. The feature wraps the type within the target language specific concept of a namespace, for example, a Java package or C# namespace. Please see the language specific sections to see if the target language you are interested in supports the nspace feature.

The feature is demonstrated below for C# using the following example:

> 某些目标语言为 `nspace` [功能](http://swig.org/Doc3.0/Customization.html#Customization_features)提供支持。该功能可以应用于命名空间中声明的任何类、结构体、共用体或枚举。该功能将类型包装在目标语言中对应命名空间的特定概念内，例如 Java 包或 C# 命名空间。请参阅特定于语言的部分，以查看你感兴趣的目标语言是否支持 `nspace` 功能。
>
> 下面使用以下示例针对 C# 演示了该功能：

```
%feature("nspace") MyWorld::Material::Color;
%nspace MyWorld::Wrapping::Color; // %nspace is a macro for %feature("nspace")

namespace MyWorld {
  namespace Material {
    class Color {
    ...
    };
  }
  namespace Wrapping {
    class Color {
    ...
    };
  }
}
```

Without the `nspace` feature directives above or `%rename`, you would get the following warning resulting in just one of the `Color` classes being available for use from the target language:

> 如果没有上面和 `nspace` 功能指令或 `%rename`，你将得到以下警告，导致只有一种 `Color` 类可以从目标语言中使用：

```
example.i:9: Error: 'Color' is multiply defined in the generated target language module.
example.i:5: Error: Previous declaration of 'Color'
```

With the `nspace` feature the two `Color` classes are wrapped into the equivalent C# namespaces. A fully qualified constructor call of each these two types in C# is then:

> 通过 `nspace` 功能，两个 `Color` 类被包装到等效的 C# 命名空间中。然后，在 C# 中对这两种类型的完全限定构造函数调用如下：

```csharp
MyWorld.Material.Color materialColor = new MyWorld.Material.Color();
MyWorld.Wrapping.Color wrappingColor = new MyWorld.Wrapping.Color();
```

Note that the `nspace` feature does not apply to variables and functions simply declared in a namespace. For example, the following symbols cannot co-exist in the target language without renaming. This may change in a future version.

> 注意，`nspace` 功能不适用于仅在命名空间中声明的变量和函数。例如，以下符号不能在不重命名的情况下在目标语言中共存。这可能会在将来的版本中更改。

```csharp
namespace MyWorld {
  namespace Material {
    int quantity;
    void dispatch();
  }
  namespace Wrapping {
    int quantity;
    void dispatch();
  }
}
```

**Compatibility Note:** The nspace feature was first introduced in SWIG-2.0.0.

> **注意兼容性**：`nspace` 功能在 SWIG-2.0.0 中首次引入。

## 6.20 在命名空间中重命名模板类型

As has been mentioned, when `%rename` includes parameters, the parameter types must match exactly (no typedef or namespace resolution is performed). SWIG treats templated types slightly differently and has an additional matching rule so unlike non-templated types, an exact match is not always required. If the fully qualified templated type is specified, it will have a higher precedence over the generic template type. In the example below, the generic template type is used to rename to `bbb` and the fully qualified type is used to rename to `ccc`.

> 如前所述，当 `%rename` 包含参数时，参数类型必须完全匹配（不执行 `typedef` 或命名空间解析）。SWIG 对模板化类型的处理略有不同，并且具有其他匹配规则，因此与非模板化类型不同，不一定总是需要完全匹配。如果指定了完全限定的模板化类型，则其优先级高于通用模板类型。在下面的示例中，通用模板类型重命名为 `bbb`，完全限定类型重命名为 `ccc`。

```
%rename(bbb) Space::ABC::aaa(T t);                  // will match but with lower precedence than ccc
%rename(ccc) Space::ABC<Space::XYZ>::aaa(Space::XYZ t);// will match but with higher precedence than bbb

namespace Space {
  class XYZ {};
  template<typename T> struct ABC {
    void aaa(T t) {}
  };
}
%template(ABCXYZ) Space::ABC<Space::XYZ>;
```

It should now be apparent that there are many ways to achieve a renaming with %rename. This is demonstrated by the following two examples, which are effectively the same as the above example. Below shows how %rename can be placed inside a namespace.

> 现在应该很明显，有很多方法可以使用 `%rename` 进行重命名。以下两个示例可以证明这一点，这些示例实际上与上述示例相同。下面显示了如何将 `%rename` 放在命名空间中。

```
namespace Space {
  %rename(bbb) ABC::aaa(T t);                     // will match but with lower precedence than ccc
  %rename(ccc) ABC<Space::XYZ>::aaa(Space::XYZ t);// will match but with higher precedence than bbb
  %rename(ddd) ABC<Space::XYZ>::aaa(XYZ t);       // will not match
}

namespace Space {
  class XYZ {};
  template<typename T> struct ABC {
    void aaa(T t) {}
  };
}
%template(ABCXYZ) Space::ABC<Space::XYZ>;
```

Note that `ddd` does not match as there is no namespace resolution for parameter types and the fully qualified type must be specified for template type expansion. The following example shows how `%rename` can be placed within `%extend`.

> 请注意，由于参数类型没有命名空间解析，并且必须为模板类型扩展指定完全限定的类型，因此 `ddd` 不匹配。以下示例显示如何将 `%rename` 放在 `%extend` 中。

```
namespace Space {
  %extend ABC {
    %rename(bbb) aaa(T t);         // will match but with lower precedence than ccc
  }
  %extend ABC<Space::XYZ> {
    %rename(ccc) aaa(Space::XYZ t);// will match but with higher precedence than bbb
    %rename(ddd) aaa(XYZ t);       // will not match
  }
}

namespace Space {
  class XYZ {};
  template<typename T> struct ABC {
    void aaa(T t) {}
  };
}
%template(ABCXYZ) Space::ABC<Space::XYZ>;
```

## 6.21 异常规范

When C++ programs utilize exceptions, exceptional behavior is sometimes specified as part of a function or method declaration. For example:

> 当 C++ 程序利用异常时，有时会将异常行为指定为函数或方法声明的一部分。例如：

```c++
class Error { };

class Foo {
public:
    ...
    void blah() throw(Error);
    ...
};
```

If an exception specification is used, SWIG automatically generates wrapper code for catching the indicated exception and, when possible, rethrowing it into the target language, or converting it into an error in the target language otherwise. For example, in Python, you can write code like this:

> 如果使用了异常规范，SWIG 会自动生成包装器代码，以捕获指示的异常，并在可能的情况下，将其在目标语言中重新抛出，否则将其转换为目标语言中的错误。例如，在 Python 中，你可以编写如下代码：

```python
f = Foo()
try:
    f.blah()
except Error, e:
    # e is a wrapped instance of "Error"
```

Details of how to tailor code for handling the caught C++ exception and converting it into the target language's exception/error handling mechanism is outlined in the ["throws" typemap](http://swig.org/Doc3.0/Typemaps.html#throws_typemap) section.

Since exception specifications are sometimes only used sparingly, this alone may not be enough to properly handle C++ exceptions. To do that, a different set of special SWIG directives are used. Consult the "[Exception handling with %exception](http://swig.org/Doc3.0/Customization.html#Customization_exception)" section for details. The next section details a way of simulating an exception specification or replacing an existing one.

> [抛出类型映射](http://swig.org/Doc3.0/Typemaps.html)中概述了有关如何自定义代码以处理捕获的 C++ 异常，并将其转换为目标语言的异常/错误处理机制的详细信息。
>
> 由于有时仅很少使用异常规范，因此仅凭其本身可能不足以正确处理 C++ 异常。为此，使用了一组不同的特殊 SWIG 指令。有关详细信息，请查阅[使用 `%exception` 处理异常](http://swig.org/Doc3.0/Customization.html#Customization_exception)部分。下一节将详细介绍一种模拟异常规范或替换现有规范的方法。

## 6.22 用 `%catches` 处理异常

Exceptions are automatically handled for methods with an exception specification. Similar handling can be achieved for methods without exception specifications through the `%catches` feature. It is also possible to replace any declared exception specification using the `%catches` feature. In fact, `%catches` uses the same ["throws" typemaps](http://swig.org/Doc3.0/Typemaps.html#throws_typemap) that SWIG uses for exception specifications in handling exceptions. The `%catches` feature must contain a list of possible types that can be thrown. For each type that is in the list, SWIG will generate a catch handler, in the same way that it would for types declared in the exception specification. Note that the list can also include the catch all specification "...". For example,

> 带有异常规范的方法会自动处理异常。通过 `%catches` 功能，可以为没有异常规范的方法实现类似的处理。也可以使用 `%catches` 功能替换任何声明的异常规范。实际上，`%catches` 使用的[抛出类型映射](http://swig.org/Doc3.0/Typemaps.html#throws_typemap)与 SWIG 处理异常的异常规范相同。`%catches` 功能必须包含可能抛出的类型列表。对于列表中的每种类型，SWIG 都将生成捕获处理程序，并与异常规范中声明的类型一致。请注意，该列表还可以包含捕获所有规范 `...`。例如，

```
struct EBase { virtual ~EBase(); };
struct Error1 : EBase { };
struct Error2 : EBase { };
struct Error3 : EBase { };
struct Error4 : EBase { };

%catches(Error1, Error2, ...) Foo::bar();
%catches(EBase) Foo::blah();

class Foo {
public:
    ...
    void bar();
    void blah() throw(Error1, Error2, Error3, Error4);
    ...
};
```

For the `Foo::bar()` method, which can throw anything, SWIG will generate catch handlers for `Error1`, `Error2` as well as a catch all handler (...). Each catch handler will convert the caught exception and convert it into a target language error/exception. The catch all handler will convert the caught exception into an unknown error/exception.

Without the `%catches` feature being attached to `Foo::blah()`, SWIG will generate catch handlers for all of the types in the exception specification, that is, `Error1, Error2, Error3, Error4`. However, with the `%catches` feature above, just a single catch handler for the base class, `EBase` will be generated to convert the C++ exception into a target language error/exception.

> 对于可以抛出任何内容的 `Foo::bar()` 方法，SWIG 将为 `Error1`、`Error2` 以及所有（`...`）生成捕获处理程序。每个 `catch` 处理程序都将转换捕获的异常，并将其转换为目标语言错误/异常。捕获所有会将捕获的异常转换为未知的错误/异常。
>
> 没有在 `Foo::blah()` 上附加 `%catches` 功能，SWIG 将为异常规范中的所有类型（即 `Error1`、`Error2`、`Error3`、`Error4`）生成捕获处理程序。但是，使用上面的 `%catches` 功能，仅生成基类 `EBase` 的单个捕获处理程序即可将 C++ 异常转换为目标语言错误/异常。

## 6.23 成员指针

Starting with SWIG-1.3.7, there is limited parsing support for pointers to C++ class members. For example:

> 从 SWIG-1.3.7 开始，可以有限地支持对指向 C++ 类成员的指针的解析。例如：

```
double do_op(Object *o, double (Object::*callback)(double, double));
extern double (Object::*fooptr)(double, double);
%constant double (Object::*FOO)(double, double) = &Object::foo;
```

Although these kinds of pointers can be parsed and represented by the SWIG type system, few language modules know how to handle them due to implementation differences from standard C pointers. Readers are *strongly* advised to consult an advanced text such as the "The Annotated C++ Manual" for specific details.

When pointers to members are supported, the pointer value might appear as a special string like this:

> 尽管可以通过 SWIG 类型系统解析和表示这些类型的指针，但是由于与标准 C 指针的实现有所不同，很少有语言模块知道如何处理它们。强烈建议读者阅读高级教程（例如《The Annotated C++ Manual》）以了解详细信息。
>
> 当支持指向成员的指针时，指针值可能显示为特殊字符串，如下所示：

```python
>>> print example.FOO
_ff0d54a800000000_m_Object__f_double_double__double
>>>
```

In this case, the hexadecimal digits represent the entire value of the pointer which is usually the contents of a small C++ structure on most machines.

SWIG's type-checking mechanism is also more limited when working with member pointers. Normally SWIG tries to keep track of inheritance when checking types. However, no such support is currently provided for member pointers.

> 在这种情况下，十六进制数字表示指针的整个值，该值通常是大多数计算机上小型 C++ 结构体的内容。
>
> 在使用成员指针时，SWIG 的类型检查机制也受到更多限制。通常，SWIG 在检查类型时会尝试跟踪继承。但是，当前没有为成员指针提供这种支持。

## 6.24 智能指针与 `operator->()`

In some C++ programs, objects are often encapsulated by smart-pointers or proxy classes. This is sometimes done to implement automatic memory management (reference counting) or persistence. Typically a smart-pointer is defined by a template class where the `->` operator has been overloaded. This class is then wrapped around some other class. For example:

> 在某些 C++ 程序中，对象通常由智能指针或代理类封装。有时这样做是为了实现自动内存管理（引用计数）或持久性。通常，智能指针是由模板类定义的，在该模板类中已重载了 `->` 运算符。然后，将此类包装在其他一些类别上。例如：

```
// Smart-pointer class
template<class T> class SmartPtr {
    T *pointee;
public:
    SmartPtr(T *p) : pointee(p) { ... }
    T *operator->() {
        return pointee;
    }
    ...
};

// Ordinary class
class Foo_Impl {
public:
    int x;
    virtual void bar();
    ...
};

// Smart-pointer wrapper
typedef SmartPtr<Foo_Impl> Foo;

// Create smart pointer Foo
Foo make_Foo() {
    return SmartPtr<Foo_Impl>(new Foo_Impl());
}

// Do something with smart pointer Foo
void do_something(Foo f) {
    printf("x = %d\n", f->x);
    f->bar();
}

// Call the wrapped smart pointer proxy class in the target language 'Foo'
%template(Foo) SmartPtr<Foo_Impl>;
```

A key feature of this approach is that by defining `operator->` the methods and attributes of the object wrapped by a smart pointer are transparently accessible. For example, expressions such as these (from the previous example),

> 这种方法的关键特征是通过定义 `operator->`，可以透明地访问由智能指针所包装对象的方法和属性。例如，这些表达式（来自上一个示例）

```c++
f->x
f->bar()
```

are transparently mapped to the following

> 透明地映射到

```c++
(f.operator->())->x;
(f.operator->())->bar();
```

When generating wrappers, SWIG tries to emulate this functionality to the extent that it is possible. To do this, whenever`operator->()` is encountered in a class, SWIG looks at its returned type and uses it to generate wrappers for accessing attributes of the underlying object. For example, wrapping the above code produces wrappers like this:

> 在生成包装器时，SWIG 尝试尽可能的模拟此功能。为此，每当在类中遇到 `operator->()` 时，SWIG 都会查看其返回的类型，并使用它生成用于访问底层对象属性的包装器。例如，包装上面的代码将产生如下包装：

```c++
int Foo_x_get(Foo *f) {
  return (*f)->x;
}
void Foo_x_set(Foo *f, int value) {
  (*f)->x = value;
}
void Foo_bar(Foo *f) {
  (*f)->bar();
}
```

These wrappers take a smart-pointer instance as an argument, but dereference it in a way to gain access to the object returned by`operator->()`. You should carefully compare these wrappers to those in the first part of this chapter (they are slightly different).

The end result is that access looks very similar to C++. For example, you could do this in Python:

> 这些包装器将智能指针实例作为参数，但是以某种方式解引用它以获得对 `operator->()` 返回对象的访问。你应该将这些包装器与本章第一部分中的包装器进行仔细比较（它们略有不同）。
>
> 最终结果是访问看起来与 C++ 非常相似。例如，你可以在 Python 中执行此操作：

```python
>>> f = make_Foo()
>>> print f.x
0
>>> f.bar()
>>>
```

When generating wrappers through a smart-pointer, SWIG tries to generate wrappers for all methods and attributes that might be accessible through `operator->()`. This includes any methods that might be accessible through inheritance. However, there are a number of restrictions:

* Member variables and methods are wrapped through a smart pointer. Enumerations, constructors, and destructors are not wrapped.
* If the smart-pointer class and the underlying object both define a method or variable of the same name, then the smart-pointer version has precedence. For example, if you have this code

> 通过智能指针生成包装器时，SWIG 会尝试为可能通过 `operator->()` 访问的所有方法和属性生成包装器。这包括可以通过继承访问的任何方法。但是，有许多限制：
>
> * 成员变量和方法通过智能指针包装。枚举、构造函数和析构函数不会被包装。
> * 如果智能指针类和底层对象都定义了相同名称的方法或变量，则智能指针版本具有优先级别。例如，如果你有此代码

```c++
class Foo {
public:
  int x;
};

class Bar {
public:
  int x;
  Foo *operator->();
};
```

then the wrapper for `Bar::x` accesses the `x` defined in `Bar`, and not the `x` defined in `Foo`.

If your intent is to only expose the smart-pointer class in the interface, it is not necessary to wrap both the smart-pointer class and the class for the underlying object. However, you must still tell SWIG about both classes if you want the technique described in this section to work. To only generate wrappers for the smart-pointer class, you can use the %ignore directive. For example:

> 那么 `Bar::x` 的包装器访问 `Bar` 定义的 `x`，而不访问 `Foo` 定义的 `x`。
>
> 如果你的目的只是在接口中公开智能指针类，则不必同时包装智能指针类和底层对象的类。但是，如果你希望本节中介绍的技术起作用，则仍必须将两个类都告诉 SWIG。要仅为智能指针类生成包装器，可以使用 `%ignore` 指令。例如：

```
%ignore Foo;
class Foo {       // Ignored
};

class Bar {
public:
  Foo *operator->();
  ...
};
```

Alternatively, you can import the definition of `Foo` from a separate file using `%import`.

**Note:** When a class defines `operator->()`, the operator itself is wrapped as a method `__deref__()`. For example:

> 另外，你可以使用 `%import` 从一个单独的文件中导入 `Foo` 的定义。
>
> **注意**：当一个类定义 `operator->()` 时，运算符本身被包装为方法 `__deref __()`。例如：

```python
f = Foo()               # Smart-pointer
p = f.__deref__()       # Raw pointer from operator->
```

**Note:** To disable the smart-pointer behavior, use `%ignore` to ignore `operator->()`. For example:

> **注意**：为了取消智能指针的行为，使用 `%ignore` 忽略 `operator->()`。例如：

```
%ignore Bar::operator->;
```

**Note:** Smart pointer support was first added in SWIG-1.3.14.

> **注意**：对智能指针的支持在 SWIG-1.3.14 中首次添加。

## 6.25 C++ 引用计数对象——`ref` / `unref` 功能

Another similar idiom in C++ is the use of reference counted objects. Consider for example:

> C++ 中的另一个惯用法是引用计数对象的使用。考虑例如：

```c++
class RCObj  {
  // implement the ref counting mechanism
  int add_ref();
  int del_ref();
  int ref_count();

public:
  virtual ~RCObj() = 0;

  int ref() const {
    return add_ref();
  }

  int unref() const {
    if (ref_count() == 0 || del_ref() == 0 ) {
      delete this;
      return 0;
    }
    return ref_count();
  }
};


class A : RCObj {
public:
  A();
  int foo();
};


class B {
  A *_a;

public:
  B(A *a) : _a(a) {
    a->ref();
  }

  ~B() {
    a->unref();
  }
};

int main() {
  A *a  = new A();       // (count: 0)
  a->ref();           // 'a' ref here (count: 1)

  B *b1 = new B(a);   // 'a' ref here (count: 2)
  if (1 + 1 == 2) {
    B *b2 = new B(a); // 'a' ref here (count: 3)
    delete b2;        // 'a' unref, but not deleted (count: 2)
  }

  delete b1;          // 'a' unref, but not deleted (count: 1)
  a->unref();         // 'a' unref and deleted (count: 0)
}
```

In the example above, the 'A' class instance 'a' is a reference counted object, which can't be deleted arbitrarily since it is shared between the objects 'b1' and 'b2'. 'A' is derived from a *Reference Counted Object* 'RCObj', which implements the ref/unref idiom.

To tell SWIG that 'RCObj' and all its derived classes are reference counted objects, use the "ref" and "unref" [features](http://swig.org/Doc3.0/Customization.html#Customization_features). These are also available as `%refobject` and `%unrefobject`, respectively. For example:

> 在上面的示例中，`A` 类实例 `a` 是一个引用计数的对象，由于它在对象 `b1` 和 `b2` 之间共享，因此不能任意删除。`A` 源自“引用计数对象”（RCObj），该引用实现了 `ref` / `unref` 惯用法。
>
> 要告诉 SWIG RCObj 及其所有派生类都是引用计数对象，请使用 `ref` 和 `unref` [功能](http://swig.org/Doc3.0/Customization.html#Customization_features)。这些也可以分别以 `%refobject` 和 `%unrefobject` 的形式获得。例如：

```
%module example
...

%feature("ref")   RCObj "$this->ref();"
%feature("unref") RCObj "$this->unref();"

%include "rcobj.h"
%include "A.h"
...
```

where the code passed to the "ref" and "unref" features will be executed as needed whenever a new object is passed to python, or when python tries to release the proxy object instance, respectively.

On the python side, the use of a reference counted object is no different to any other regular instance:

> 其中，每当将新对象传递给 python，或 python 尝试释放代理对象实例时，传递给 `ref` 和 `unref` 功能的代码将根据需要执行。
>
> 在 python 方面，引用计数对象的使用与任何其他常规实例没有什么不同：

```python
def create_A():
    a = A()         # SWIG ref 'a' - new object is passed to python (count: 1)
    b1 = B(a)       # C++ ref 'a (count: 2)
    if 1 + 1 == 2:
        b2 = B(a)   # C++ ref 'a' (count: 3)
    return a        # 'b1' and 'b2' are released and deleted, C++ unref 'a' twice (count: 1)

a = create_A()      # (count: 1)
exit                # 'a' is released, SWIG unref 'a' called in the destructor wrapper (count: 0)
```

Note that the user doesn't explicitly need to call 'a->ref()' nor 'a->unref()' (and neither 'delete a'). Instead, SWIG takes cares of executing the "ref" and "unref" calls as needed. If the user doesn't specify the "ref/unref" feature for a type, SWIG will produce code equivalent to defining these features:

> 请注意，用户不需要显式调用 `a->ref()` 或 `a->unref()`（也无需 `delete a`）。相反，SWIG 会根据需要执行 `ref` 和 `unref` 调用。如果用户未为类型指定 `ref` / `unref` 功能，SWIG 将产生与定义这些功能等效的代码：

```
%feature("ref")   ""
%feature("unref") "delete $this;"
```

In other words, SWIG will not do anything special when a new object is passed to python, and it will always 'delete' the underlying object when python releases the proxy instance.

The [%newobject feature](http://swig.org/Doc3.0/Customization.html#Customization_ownership) is designed to indicate to the target language that it should take ownership of the returned object. When used in conjunction with a type that has the "ref" feature associated with it, it additionally emits the code in the "ref" feature into the C++ wrapper. Consider wrapping the following factory function in addition to the above:

> 换句话说，将新对象传递给 python 时，SWIG 不会做任何特殊的事情，而当 python 释放代理实例时，它将始终“删除”底层对象。
>
> [`%newobject` 功能](http://swig.org/Doc3.0/Customization.html#Customization_ownership)旨在向目标语言指示它应该对返回的对象拥有所有权。与具有 `ref` 功能关联的类型一起使用时，它还会将 `ref` 功能中的代码发送到 C++ 包装器中。除了上述内容外，还考虑包装以下工厂函数：

```
%newobject AFactory;
A *AFactory() {
  return new A();
}
```

The `AFactory` function now acts much like a call to the `A`constructor with respect to memory handling:

> 现在，关于内存处理，`AFactory` 函数的行为很像对 `A` 构造函数的调用：

```python
a = AFactory()    # SWIG ref 'a' due to %newobject (count: 1)
exit              # 'a' is released, SWIG unref 'a' called in the destructor wrapper (count: 0)
```

## 6.26 `using` 声明与继承

`using` declarations are sometimes used to adjust access to members of base classes. For example:

> 有时使用 `using` 声明来调整对基类成员的访问。例如：

```c++
class Foo {
public:
    int blah(int x);
};

class Bar {
public:
    double blah(double x);
};

class FooBar : public Foo, public Bar {
public:
    using Foo::blah;
    using Bar::blah;
    char *blah(const char *x);
};
```

In this example, the `using` declarations make different versions of the overloaded `blah()` method accessible from the derived class. For example:

> 在这个例子中，`using` 声明使派生类可以访问重载的 `blah()` 方法的不同版本。例如：

```c++
FooBar *f;
f->blah(3);         // Ok. Invokes Foo::blah(int)
f->blah(3.5);       // Ok. Invokes Bar::blah(double)
f->blah("hello");   // Ok. Invokes FooBar::blah(const char *);
```

SWIG emulates the same functionality when creating wrappers. For example, if you wrap this code in Python, the module works just like you would expect:

> SWIG 在创建包装器时会模拟相同的功能。例如，如果将此代码包装在 Python 中，则该模块的工作方式与你期望的一样：

```python
>>> import example
>>> f = example.FooBar()
>>> f.blah(3)
>>> f.blah(3.5)
>>> f.blah("hello")
```

`using` declarations can also be used to change access when applicable. For example:

> 可以的话，也可以使用 `using` 声明来更改访问权限。例如：

```c++
class Foo {
protected:
    int x;
    int blah(int x);
};

class Bar : public Foo {
public:
    using Foo::x;       // Make x public
    using Foo::blah;    // Make blah public
};
```

This also works in SWIG ---the exposed declarations will be wrapped normally.

When `using` declarations are used as shown in these examples, declarations from the base classes are copied into the derived class and wrapped normally. When copied, the declarations retain any properties that might have been attached using `%rename`, `%ignore`, or `%feature`. Thus, if a method is ignored in a base class, it will also be ignored by a `using` declaration.

Because a `using` declaration does not provide fine-grained control over the declarations that get imported, it may be difficult to manage such declarations in applications that make heavy use of SWIG customization features. If you can't get `using` to work correctly, you can always change the interface to the following:

> 这在 SWIG 中也适用——暴露的声明将被正常包装。
>
> 如这些示例所示，当使用 `using` 声明时，基类中的声明将被复制到派生类中并正常包装。复制后，声明保留 `%rename`、`%ignore` 或 `%feature` 附加的任何属性。因此，如果一个方法在基类中被忽略，它也会被 `using` 声明所忽略。
>
> 由于 `using` 声明不能对导入的声明提供细粒度的控制，因此在大量使用 SWIG 自定义功能的应用程序中，可能难以管理此类声明。如果无法使 `using` 正常工作，则可以始终将接口更改为以下内容：

```c++
class FooBar : public Foo, public Bar {
public:
#ifndef SWIG
    using Foo::blah;
    using Bar::blah;
#else
    int blah(int x);         // explicitly tell SWIG about other declarations
    double blah(double x);
#endif

    char *blah(const char *x);
};
```

**Notes:**

* If a derived class redefines a method defined in a base class, then a `using` declaration won't cause a conflict. For example:

> **注意**：
>
> * 如果派生类重新定义了基类中定义的方法，则 `using` 声明不会引起冲突。例如：

```c++
class Foo {
public:
  int blah(int );
  double blah(double);
};

class Bar : public Foo {
public:
  using Foo::blah;    // Only imports blah(double);
  int blah(int);
};
```

* Resolving ambiguity in overloading may prevent declarations from being imported by `using`. For example:

> * 解决重载中的歧义可能会阻止通过 `using` 导入的声明。例如：

```
%rename(blah_long) Foo::blah(long);
class Foo {
public:
  int blah(int);
  long blah(long);  // Renamed to blah_long
};

class Bar : public Foo {
public:
  using Foo::blah;     // Only imports blah(int)
  double blah(double x);
};
```

## 6.27 嵌套类

If the target language supports the nested classes concept (like Java), the nested C++ classes are wrapped as nested target language proxy classes. (In case of Java - "static" nested classes.) Only public nested classes are wrapped. Otherwise there is little difference between nested and normal classes.

If the target language doesn't support nested classes directly, or the support is not implemented in the language module (like for python currently), then the visible nested classes are moved to the same name space as the containing class (nesting hierarchy is "flattened"). The same behaviour may be turned on for C# and Java by the %feature ("flatnested"); If there is a class with the same name in the outer namespace the inner class (or the global one) may be renamed or ignored:

> 如果目标语言支持嵌套类概念（例如 Java），则将嵌套的 C++ 类包装为嵌套的目标语言代理类。（对于 Java 的“静态”嵌套类。）仅包装公有嵌套类。否则，嵌套类和普通类之间几乎没有区别。
>
> 如果目标语言不直接支持嵌套类，或者未在语言模块中实现此支持（例如当前针对的是 python），则可见的嵌套类将移至与包含类相同的命名空间（嵌套层次结构被“展平”）。可以通过 `%feature("flatnested")` 为 C# 和 Java 打开相同的行为。如果外部命名空间中存在一个具有相同名称的类，则可以重命名或忽略内部类（或全局类）：

```
%rename (Bar_Foo) Bar::Foo;
class Foo {};
class Bar {
  public:
  class Foo {};
};
```

If a nested class, within an outer class, has to be used as a template parameter within the outer class, then the template will have to be instantiated with `%template` before the beginning of the outer class. An example can be found in the [Templates](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_template_nested_class_example) section.

**Compatibility Note:** Prior to SWIG-3.0.0, there was limited nested class support. Nested classes were treated as opaque pointers. However, there was a workaround for nested class support in these older versions requiring the user to replicate the nested class in the global scope, adding in a typedef for the nested class in the global scope and using the "nestedworkaround" feature on the nested class. This resulted in approximately the same behaviour as the "flatnested" feature. With proper nested class support now available in SWIG-3.0.0, this feature has been deprecated and no longer works requiring code changes. If you see the following warning:

> 如果外部类中的嵌套类必须用作外部类中的模板参数，则必须在外部类开始之前使用 `%template` 实例化该模板。可以在[模板](http://swig.org/Doc3.0/SWIGPlus.html#SWIGPlus_template_nested_class_example)部分中找到示例。
>
> **注意兼容性**：在 SWIG-3.0.0 之前，对嵌套类的支持有限。嵌套类被视为不透明指针。但是，在这些较旧的版本中，存在一种支持嵌套类的解决方法，要求用户在全局范围内复制嵌套类，为全局范围内的嵌套类添加 `typedef`，并在嵌套类上使用 `nestedworkaround` 功能 。这导致与 `flatnested` 功能大致相同的行为。现在借助 SWIG-3.0.0 中提供的嵌套类支持，该功能已被弃用，并且不再需要更改代码而起作用。如果看到以下警告：

```
example.i:8: Warning 126: The nestedworkaround feature is deprecated
```

consider using the "flatnested" feature discussed above which generates a non-nested proxy class, like the "nestedworkaround" feature did. Alternatively, use the default nested class code generation, which may generate an equivalent to a nested proxy class in the target language, depending on the target language support.

SWIG-1.3.40 and earlier versions did not have the `nestedworkaround` feature and the generated code resulting from parsing nested classes did not always compile. Nested class warnings could also not be suppressed using %warnfilter.

> 考虑使用上面讨论的 `flatnested` 功能来生成非嵌套的代理类，就像 `nestedworkaround` 功能一样。或者，使用默认的嵌套类代码生成，这可能会生成与目标语言中的嵌套代理类等效的代码，具体取决于目标语言的支持。
>
> SWIG-1.3.40 和更早版本不具有 `nestedworkaround` 功能，并且由于解析嵌套类而生成的代码并不总是可以编译。使用 `%warnfilter` 也不能禁止嵌套类警告。

## 6.28 关于 `const` 正确性的争论

A common issue when working with C++ programs is dealing with all possible ways in which the `const` qualifier (or lack thereof) will break your program, all programs linked against your program, and all programs linked against those programs.

Although SWIG knows how to correctly deal with `const` in its internal type system and it knows how to generate wrappers that are free of const-related warnings, SWIG does not make any attempt to preserve const-correctness in the target language. Thus, it is possible to pass `const` qualified objects to non-const methods and functions. For example, consider the following code in C++:

> 使用 C++ 程序时的一个常见问题是处理 `const` 限定符（或缺少 `const` 限定符），因为程序、与程序链接的所有程序，以及与这些程序链接的所有程序可能因此被破坏。
>
> 尽管 SWIG 知道如何在其内部类型系统中正确处理 `const`，并且知道如何生成没有 `const` 相关警告的包装器，但 SWIG 并未尝试保留目标语言中的 `const` 正确性。因此，可以将合法的常量对象传递给非常量方法和函数。例如，请考虑以下 C++ 代码：

```c++
const Object * foo();
void bar(Object *);

...
// C++ code
void blah() {
  bar(foo());         // Error: bar discards const
};
```

Now, consider the behavior when wrapped into a Python module:

> 现在，考虑将行为包装进 Python 模块：

```python
>>> bar(foo())         # Okay
>>>
```

Although this is clearly a violation of the C++ type-system, fixing the problem doesn't seem to be worth the added implementation complexity that would be required to support it in the SWIG run-time type system. There are no plans to change this in future releases (although we'll never rule anything out entirely).

The bottom line is that this particular issue does not appear to be a problem for most SWIG projects. Of course, you might want to consider using another tool if maintaining constness is the most important part of your project.

> 尽管这显然违反了 C++ 类型系统，但为了解决该问题而在 SWIG 运行时类型系统中支持该实现所增加的复杂性看起来似乎不值得。没有计划在将来的版本中对此进行更改（尽管我们永远不会完全排除任何问题）。
>
> 最重要的是，对于大多数 SWIG 项目而言，这个特定问题似乎都不是问题。当然，如果保持常量性是项目中最重要的部分，则可能要考虑使用其他工具。

## 6.29 到哪里获得更多信息

If you're wrapping serious C++ code, you might want to pick up a copy of "The Annotated C++ Reference Manual" by Ellis and Stroustrup. This is the reference document we use to guide a lot of SWIG's C++ support.

> 如果要包装严谨的 C++ 代码，则可能需要阅读 Ellis 和 Stroustrup 撰写的《The Annotated C++ Reference Manual》。这是我们用来指导 SWIG 对 C++ 诸多支持的参考文档。
