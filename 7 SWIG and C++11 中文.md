[TOC]

# 7 SWIG 与 C++11

## 7.1 引言

This chapter gives you a brief overview about the SWIG implementation of the C++11 standard. This part of SWIG is still a work in progress.

SWIG supports the new C++ syntax changes with some minor limitations in some areas such as decltype expressions and variadic templates. Wrappers for the new STL types (unordered_ containers, result_of, tuples) are incomplete. The wrappers for the new containers would work much like the C++03 containers and users are welcome to help by adapting the existing container interface files and submitting them as a patch for inclusion in future versions of SWIG.

> 本章为你简要概述了 C++11 标准的SWIG实现。SWIG 的这一部分仍在开发中。
>
> SWIG 支持新的 C++ 语法改动，但在诸如 decltype 表达式和可变参数模板等某些方面存在一些小的限制。STL 新类型（`unordered_`容器、`result_of`、元组）的包装不完整。新容器的包装器将像 C++03 容器一样工作，欢迎用户适配现有容器接口文件并将其作为补丁提交以供将来的 SWIG 版本使用。

## 7.2 核心语言变更

### 7.2.1 右值引用与转移语义

SWIG correctly parses the rvalue reference syntax `&&`, for example the typical usage of it in the move constructor and move assignment operator below:

> SWIG 正确地解析了右值引用语法 `&&`，例如，下面转移构造函数和转移赋值运算符的典型用法：

```c++
class MyClass {
...
  std::vector<int> numbers;
public:
  MyClass(MyClass &&other) : numbers(std::move(other.numbers)) {}
  MyClass & operator=(MyClass &&other) {
    numbers = std::move(other.numbers);
    return *this;
  }
};
```

Rvalue references are designed for C++ temporaries and so are not very useful when used from non-C++ target languages. Generally you would just ignore them via `%ignore` before parsing the class. For example, ignore the move constructor:

> 右值引用是为 C++ 临时对象设计的，因此从非 C++ 目标语言中使用时，它不是很有用。通常，你只需要在解析类之前通过 `%ignore` 忽略它们即可。例如，忽略转移构造函数：

```
%ignore MyClass::MyClass(MyClass &&);
```

The plan is to ignore move constructors by default in a future version of SWIG. Note that both normal assignment operators as well as move assignment operators are ignored by default in most target languages with the following warning:

> 计划在 SWIG 的未来版本中默认忽略转移构造函数。请注意，在大多数目标语言中，默认情况下普通赋值运算符和移动赋值运算符都会被忽略，并显示以下警告：

```
example.i:18: Warning 503: Can't wrap `operator =` unless renamed to a valid identifier.
```

### 7.2.2 通用常量表达式

SWIG parses and identifies the keyword `constexpr`, but cannot fully utilise it. These C++ compile time constants are usable as runtime constants from the target languages. Below shows example usage for assigning a C++ compile time constant from a compile time constant function:

> SWIG 解析并识别关键字 `constexpr`，但无法充分利用它。这些 C++ 编译时常量可用作目标语言的运行时常量。下面显示了从编译时常量函数分配 C++ 编译时常量的示例用法：

```c++
constexpr int XXX() { return 10; }
constexpr int YYY = XXX() + 100;
```

When either of these is used from a target language, a runtime call is made to obtain the underlying constant.

> 当从目标语言中使用这两种方法中的任何一种时，都会进行运行时调用以获取基础常量。

### 7.2.3 外部模板

SWIG correctly parses the keywords `extern template`. However, this template instantiation suppression in a translation unit has no relevance outside of the C++ compiler and so is not used by SWIG. SWIG only uses `%template` for instantiating and wrapping templates.

> SWIG 正确地解析了关键字 `extern template`。但是，转换单元中的模板实例化抑制在 C++ 编译器之外没有任何关联，因此 SWIG 不会使用它。SWIG 仅使用 `%template` 实例化和包装模板。

```
template class std::vector<int>;        // C++03 explicit instantiation in C++
extern template class std::vector<int>; // C++11 explicit instantiation suppression in C++
%template(VectorInt) std::vector<int>;  //SWIGinstantiation
```

### 7.2.4 初始化列表

Initializer lists are very much a C++ compiler construct and are not very accessible from wrappers as they are intended for compile time initialization of classes using the special `std::initializer_list` type. SWIG detects usage of initializer lists and will emit a special informative warning each time one is used:

> 初始化列表是 C++ 编译器的一种构造，并且对于包装器来说不是很容易访问，因为它们打算使用特殊的 `std::initializer_list` 类型进行类的编译时初始化。SWIG 会检测到初始化列表的使用，并且每次使用初始化列表时都会发出特殊的提示性警告：

```
example.i:33: Warning 476: Initialization using std::initializer_list.
```

Initializer lists usually appear in constructors but can appear in any function or method. They often appear in constructors which are overloaded with alternative approaches to initializing a class, such as the std container's push_back method for adding elements to a container. The recommended approach then is to simply ignore the initializer-list constructor, for example:

> 初始化列表通常出现在构造函数中，但也可以出现在任何函数或方法中。它们经常出现在构造函数中，这些构造函数被初始化类的替代方法重载，例如 std 容器用于添加元素的 `push_back` 方法。推荐的方法是简单地忽略初始化列表构造函数，例如：

```
%ignore Container::Container(std::initializer_list<int>);
class Container {
public:
  Container(std::initializer_list<int>); // initializer-list constructor
  Container();
  void push_back(const int &);
  ...
};
```

Alternatively you could modify the class and add another constructor for initialization by some other means, for example by a `std::vector`:

> 或者，你可以修改该类并通过其他方法（例如，通过 `std::vector`）添加另一个用于初始化的构造函数：

```
%include <std_vector.i>
class Container {
public:
  Container(const std::vector<int> &);
  Container(std::initializer_list<int>); // initializer-list constructor
  Container();
  void push_back(const int &);
  ...
};
```

And then call this constructor from your target language, for example, in Python, the following will call the constructor taking the `std::vector`:

> 然后从你的目标语言调用此构造函数，例如在 Python 中，以下将使用 `std::vector` 调用该构造函数：

```python
>>> c = Container([1, 2, 3, 4])
```

If you are unable to modify the class being wrapped, consider ignoring the initializer-list constructor and using %extend to add in an alternative constructor:

> 如果你无法修改被包装的类，请考虑忽略初始化列表构造函数，并使用 `%extend` 添加备用构造函数：

```
%include <std_vector.i>
%extend Container {
  Container(const std::vector<int> &elements) {
    Container *c = new Container();
    for (int element : elements)
      c->push_back(element);
    return c;
  }
}

%ignore Container::Container(std::initializer_list<int>);

class Container {
public:
  Container(std::initializer_list<int>); // initializer-list constructor
  Container();
  void push_back(const int &);
  ...
};
```

The above makes the wrappers look is as if the class had been declared as follows:

> 这使得包装器看起来好像类是如下声明的：

```
%include <std_vector.i>
class Container {
public:
  Container(const std::vector<int> &);
//  Container(std::initializer_list<int>); // initializer-list constructor (ignored)
  Container();
  void push_back(const int &);
  ...
};
```

`std::initializer_list` is simply a container that can only be initialized at compile time. As it is just a C++ type, it is possible to write typemaps for a target language container to map onto`std::initializer_list`. However, this can only be done for a fixed number of elements as initializer lists are not designed to be constructed with a variable number of arguments at runtime. The example below is a very simple approach which ignores any parameters passed in and merely initializes with a fixed list of fixed integer values chosen at compile time:

> `std::initializer_list` 只是一个只能在编译时初始化的容器。由于它只是一种 C++ 类型，因此可以为目标语言容器编写类型映射以映射到 `std::initializer_list` 上。但是，只能对固定数量的元素执行此操作，因为初始化列表并非设计为在运行时使用可变数量的参数构造。下面的示例是一个非常简单的方法，它忽略传入的任何参数，仅使用在编译时选择的固定整数值的固定列表进行初始化：

```
%typemap(in) std::initializer_list<int> {
  $1 = {10, 20, 30, 40, 50};
}
class Container {
public:
  Container(std::initializer_list<int>); // initializer-list constructor
  Container();
  void push_back(const int &);
  ...
};
```

Any attempt at passing in values from the target language will be ignored and be replaced by `{10, 20, 30, 40, 50}`. Needless to say, this approach is very limited, but could be improved upon, but only slightly. A typemap could be written to map a fixed number of elements on to the `std::initializer_list`, but with values decided at runtime. The typemaps would be target language specific.

Note that the default typemap for `std::initializer_list` does nothing but issue the warning and hence any user supplied typemaps will override it and suppress the warning.

> 从目标语言传递值的任何尝试都将被忽略，并由 `{10, 20, 30, 40, 50}` 代替。不用说，这种方法是非常局限的，但是可以改进，不过只能稍微改进。可以编写一个类型映射来将固定数量的元素映射到 `std::initializer_list` 上，但是要在运行时确定其值。类型映射将是特定于目标语言的。
>
> 请注意，`std::initializer_list` 的默认类型映射只会发出警告，而不会执行任何操作，因此任何用户提供的类型映射都将覆盖它并禁止显示警告。

### 7.2.5 统一初始化

The curly brackets `{}` for member initialization are fully supported by SWIG:

> SWIG 完全支持用 `{}` 来实现成员初始化。

```c++
struct BasicStruct {
 int x;
 double y;
};

struct AltStruct {
  AltStruct(int x, double y) : x_{x}, y_{y} {}

  int x_;
  double y_;
};

BasicStruct var1{5, 3.2}; // only fills the struct components
AltStruct var2{2, 4.3};   // calls the constructor
```

Uniform initialization does not affect usage from the target language, for example in Python:

> 统一初始化对目标语言的使用不起作用，例如在 Python 中：

```python
>>> a = AltStruct(10, 142.15)
>>> a.x_
10
>>> a.y_
142.15
```

### 7.2.6 类型推断

SWIG supports `decltype()` with some limitations. Single variables are allowed, however, expressions are not supported yet. For example, the following code will work:

> SWIG 对 `decltype()` 的支持有一些限制。允许使用单个变量，但是尚不支持表达式。例如，以下代码将起作用：

```c++
int i;
decltype(i) j;
```

However, using an expression inside the decltype results in syntax error:

> 但是，在 `decltype` 中使用表达式将产生语法错误：

```c++
int i; int j;
decltype(i+j) k;  // syntax error
```

### 7.2.7 基于范围的 `for` 循环

This feature is part of the implementation block only. SWIG ignores it.

> 这一功能只是实现障碍的一部分。SWIG 忽略了它。

### 7.2.8 Lambda 函数和表达式

SWIG correctly parses most of the Lambda functions syntax. For example:

> SWIG 能正确解析绝大部分 Lambda 函数语法。例如：

```c++
auto val = [] { return something; };
auto sum = [](int x, int y) { return x+y; };
auto sum = [](int x, int y) -> int { return x+y; };
```

The lambda functions are removed from the wrappers for now, because of the lack of support for closures (scope of the lambda functions) in the target languages.

Lambda functions used to create variables can also be parsed, but due to limited support of `auto` when the type is deduced from the expression, the variables are simply ignored.

> 由于缺少对目标语言中闭包（lambda 函数范围）的支持，因此暂时将 lambda 函数从包装器中删除了。
>
> 也可以解析用于创建变量的 Lambda 函数，但是由于从表达式推导出类型时对 `auto` 的支持有限，因此变量将被忽略。

```c++
auto six = [](int x, int y) { return x+y; }(4, 2);
```

Better support should be available in a later release.

> 后续版本将会提供更好的支持。

### 7.2.9 替代函数语法（Alternate function syntax）

SWIG fully supports the new definition of functions. For example:

> SWIG 完全支持这种新的函数定义。例如：

```c++
struct SomeStruct {
  int FuncName(int x, int y);
};
```

can now be written as in C++11:

> 在 C++11 中可以写成：

```c++
struct SomeStruct {
  auto FuncName(int x, int y) -> int;
};

auto SomeStruct::FuncName(int x, int y) -> int {
  return x + y;
}
```

The usage in the target languages remains the same, for example in Python:

> 在目标语言中的用法是相同的，例如在 Python 中：

```python
>>> a = SomeStruct()
>>> a.FuncName(10, 5)
15
```

SWIG will also deal with type inference for the return type, as per the limitations described earlier. For example:

> 根据前面所述的限制，SWIG 还将进行返回类型的类型推断。例如：

```c++
auto square(float a, float b) -> decltype(a);
```

### 7.2.10 对象构造改进

There are three parts to object construction improvement. The first improvement is constructor delegation such as the following:

> 对象构造改进分为三个部分。第一部分改进是构造函数委托，例如：

```c++
class A {
public:
  int a;
  int b;
  int c;

  A() : A(10) {}
  A(int aa) : A(aa, 20) {}
  A(int aa, int bb) : A(aa, bb, 30) {}
  A(int aa, int bb, int cc) { a=aa; b=bb; c=cc; }
};
```

where peer constructors can be called. SWIG handles this without any issue.

The second improvement is constructor inheritance via a `using` declaration. This is parsed correctly, but the additional constructors are not currently added to the derived proxy class in the target language. An example is shown below:

> 可以调用对等构造函数的地方。SWIG 可以毫无问题地进行处理。
>
> 第二部分改进是通过使用 `using` 声明的构造函数继承。可以正确地对此进行分析，但是当前未将其他构造函数添加到目标语言中的派生代理类。一个例子如下所示：

```c++
class BaseClass {
public:
  BaseClass(int iValue);
};

class DerivedClass: public BaseClass {
  public:
  using BaseClass::BaseClass; // Adds DerivedClass(int) constructor
};
```

The final part is member initialization at the site of the declaration. This kind of initialization is handled by SWIG.

> 最后一部分是声明位置的成员初始化。这种初始化由 SWIG 处理。

```c++
class SomeClass {
public:
  SomeClass() {}
  explicit SomeClass(int new_value) : value(new_value) {}

  int value = 5;
};
```

### 7.2.11 显式 `overrides` 与 `final`

The special identifiers `final` and `override` can be used on methods and destructors, such as in the following example:

> 特殊标识符 `final` 和 `override` 可用于方法和析构函数，例如以下示例：

```c++
struct BaseStruct {
  virtual void ab() const = 0;
  virtual void cd();
  virtual void ef();
  virtual ~BaseStruct();
};

struct DerivedStruct : BaseStruct {
  virtual void ab() const override;
  virtual void cd() final;
  virtual void ef() final override;
  virtual ~DerivedStruct() override;
};
```

### 7.2.12 空指针常量

The `nullptr` constant is mostly unimportant in wrappers. In the few places it has an effect, it is treated like `NULL`.

> 在包装器中，`nullptr` 常量几乎不重要。少数有它会起作用的地方，它被视为 `NULL`。

### 7.2.13 强类型枚举

SWIG supports strongly typed enumerations and parses the new `enum class` syntax and forward declarator for the enums, such as:

> SWIG 支持强类型枚举，并为枚举解析新的 `enum class` 语法和前向声明符，例如：

```c++
enum class MyEnum : unsigned int;
```

Strongly typed enums are often used to avoid name clashes such as the following:

> 强类型枚举通常用来避免名称冲突，例如下面：

```c++
struct Color {
  enum class RainbowColors : unsigned int {
    Red, Orange, Yellow, Green, Blue, Indigo, Violet
  };

  enum class WarmColors {
    Yellow, Orange, Red
  };

  // Note normal enum
  enum PrimeColors {
    Red=100, Green, Blue
  };
};
```

There are various ways that the target languages handle enums, so it is not possible to precisely state how they are handled in this section. However, generally, most scripting languages mangle in the strongly typed enumeration's class name, but do not use any additional mangling for normal enumerations. For example, in Python, the following code

> 目标语言使用多种方式处理枚举，因此在本节中无法精确说明它们的处理方式。但是，通常，大多数脚本语言都以强类型枚举的类名进行修饰，但对于普通枚举不使用任何其他修饰。例如，在 Python 中，以下代码

```python
print Color.RainbowColors_Red, Color.WarmColors_Red, Color.Red
```

results in

> 的结果是

```
0 2 100
```

The strongly typed languages often wrap normal enums into an enum class and so treat normal enums and strongly typed enums the same. The equivalent in Java is:

> 强类型语言通常将普通枚举包装到枚举类中，因此普通枚举和强类型枚举处理方式相同。Java 中的等效项是：

```java
System.out.println(
    Color.RainbowColors.Red.SWIGValue() + " " +
    Color.WarmColors.Red.SWIGValue() + " " +
    Color.PrimeColors.Red.SWIGValue());
```

### 7.2.14 双尖括号（`>>`）

SWIG correctly parses the symbols `>>` as closing the template block, if found inside it at the top level, or as the right shift operator `>>` otherwise.

> SWIG 正确地将符号 `>>` 解析为模板块的封闭（如果在顶层的块中发现），或者正确的解析为右移运算符 `>>`。

```c++
std::vector<std::vector<int>> myIntTable;
```

### 7.2.15 显式转换运算符

SWIG correctly parses the keyword `explicit` for operators in addition to constructors now. For example:

> 现在，SWIG 除了为构造函数之外，还为运算符正确解析了关键字 `explicit`。例如：

```c++
class U {
public:
  int u;
};

class V {
public:
  int v;
};

class TestClass {
public:
  //implicit converting constructor
  TestClass(U const &val) { t=val.u; }

  // explicit constructor
  explicit TestClass(V const &val) { t=val.v; }

  int t;
};

struct Testable {
  // explicit conversion operator
  explicit operator bool() const {
    return false;
  }
};
```

The effect of explicit constructors and operators has little relevance for the proxy classes as target languages don't have the same concepts of implicit conversions as C++. Conversion operators either with or without `explicit` need renaming to a valid identifier name in order to make them available as a normal proxy method.

> 显式构造函数和运算符对代理类的影响不大，因为目标语言没有与 C++ 相同的隐式转换概念。带有或不带有 `explicit` 的转换运算符都需要重命名为有效的标识符名称，以使其可以用作常规代理方法。

### 7.2.16 类型别名与别名模板

A type alias is a statement of the form:

> 类型别名是这种形式的语句：

```c++
using PFD = void (*)(double); // New introduced syntax
```

which is equivalent to the old style typedef:

> 这等价于旧式的 `typedef`：

```c++
typedef void (*PFD)(double);  // The old style
```

The following is an example of an alias template:

> 下面的例子是一个别名模板：

```c++
template< typename T1, typename T2, int N >
class SomeType {
public:
  T1 a;
  T2 b;
};

template< typename T2 >
using TypedefName = SomeType<char*, T2, 5>;
```

SWIG supports both type aliasing and alias templates. However, in order to use an alias template, two `%template` directives must be used:

> SWIG 支持类型别名和别名模板。但是，为了使用别名模板，必须使用两个 `%template` 指令：

```
%template(SomeTypeBool) SomeType<char*, bool, 5>;
%template() TypedefName<bool>;
```

Firstly, the actual template is instantiated with a name to be used by the target language, as per any template being wrapped. Secondly, the empty template instantiation, `%template()`, is required for the alias template. This second requirement is necessary to add the appropriate instantiated template type into the type system as SWIG does not automatically instantiate templates. See the [Templates](http://SWIG.org/Doc3.0/ SWIG Plus.html# SWIG Plus_nn30) section for more general information on wrapping templates.

> 首先，实际模板被实例化，并赋予一个名字供目标语言使用，与任何对模板的包装一样。其次，别名模板需要空模板实例化 `%template()`。第二个要求是将适当的实例化模板类型添加到类型系统中，这是必需的，因为 SWIG 不会自动实例化模板。有关包装模板的更多常规信息，请参见[模板](http://SWIG.org/Doc3.0/SWIGPlus.html#SWIGPlus_nn30)部分。

### 7.2.17 无限制共用体

SWIGfully supports any type inside a union even if it does not define a trivial constructor. For example, the wrapper for the following code correctly provides access to all members in the union:

> SWIG 完全支持共用体内的任何类型，即使它没有定义琐碎的构造函数。例如，以下代码的包装程序正确地提供了对共用体中所有成员的访问：

```c++
struct point {
  point() {}
  point(int x, int y) : x_(x), y_(y) {}
  int x_, y_;
};

#include <new> // For placement `new` in the constructor below
union P {
  int z;
  double w;
  point p; // Illegal in C++03; legal in C++11.
  // Due to the point member, a constructor definition is required.
  P() {
    new(&p) point();
  }
} p1;
```

### 7.2.18 可变参数模板

SWIG supports the variadic templates syntax (inside the `<>` block, variadic class inheritance and variadic constructor and initializers) with some limitations. The following code is correctly parsed:

> SWIG 有限的支持可变参数模板语法（在 `<>` 块中的可变参数类继承，以及可变参数构造函数和初始化）。以下代码的正确解析：

```c++
template <typename... BaseClasses> class ClassName : public BaseClasses... {
public:
  ClassName (BaseClasses &&... baseClasses) : BaseClasses(baseClasses)... {}
}
```

For now however, the `%template` directive only accepts one parameter substitution for the variable template parameters.

> 但是现在，`%template` 指令只接受一个参数替换可变模板参数。

```
%template(MyVariant1) ClassName<>         // zero argument not supported yet
%template(MyVariant2) ClassName<int>      // ok
%template(MyVariant3) ClassName<int, int> // too many arguments not supported yet
```

Support for the variadic `sizeof()` function is correctly parsed:

> 对可变参数函数 `sizeof()` 的支持被正确解析为：

```c++
const int SIZE = sizeof...(ClassName<int, int>);
```

In the above example `SIZE` is of course wrapped as a constant.

> 在上面的例子中 `SIZE` 被包装为一个常量。

### 7.2.19 新的字符串文字

SWIG supports wide string and Unicode string constants and raw string literals.

> SWIG 支持宽字符串和 Unicode 字符串常量以及原始字符串文字。

```c++
// New string literals
wstring         aa =  L"Wide string";
const char     *bb = u8"UTF-8 string";
const char16_t *cc =  u"UTF-16 string";
const char32_t *dd =  U"UTF-32 string";

// Raw string literals
const char      *xx =        ")I`m an \"ascii\" \\ string.";
const char      *ee =   R"XXX()I`m an "ascii" \ string.)XXX"; // same as xx
wstring          ff =  LR"XXX(I`m a "raw wide" \ string.)XXX";
const char      *gg = u8R"XXX(I`m a "raw UTF-8" \ string.)XXX";
const char16_t  *hh =  uR"XXX(I`m a "raw UTF-16" \ string.)XXX";
const char32_t  *ii =  UR"XXX(I`m a "raw UTF-32" \ string.)XXX";
```

Non-ASCII string support varies quite a bit among the various target languages though.

Note: There is a bug currently where SWIG's preprocessor incorrectly parses an odd number of double quotes inside raw string literals.

> 但是，非 ASCII 字符串支持在各种目标语言中相差很大。
>
> **注意**：当前存在一个错误，其中 SWIG 的预处理程序错误地解析了原始字符串文字中的奇数双引号。

### 7.2.20 用户定义文字

SWIG parses the declaration of user-defined literals, that is, the `operator "" _mysuffix()` function syntax.

Some examples are the raw literal:

> SWIG 可以解析用户定义的文字的声明，即 `operator "" _mysuffix()` 的语法。
>
> 一些示例是原始文字：

```c++
OutputType operator "" _myRawLiteral(const char * value);
```

numeric cooked literals:

> 数值型文字：

```c++
OutputType operator "" _mySuffixIntegral(unsigned long long);
OutputType operator "" _mySuffixFloat(long double);
```

and cooked string literals:

> 字符串型文字：

```c++
OutputType operator "" _mySuffix(const char * string_values, size_t num_chars);
OutputType operator "" _mySuffix(const wchar_t * string_values, size_t num_chars);
OutputType operator "" _mySuffix(const char16_t * string_values, size_t num_chars);
OutputType operator "" _mySuffix(const char32_t * string_values, size_t num_chars);
```

Like other operators that SWIG parses, a warning is given about renaming the operator in order for it to be wrapped:

> 像 SWIG 解析的其他运算符一样，给出了有关重命名该运算符的警告，以便对其进行包装：

```
example.i:27: Warning 503: Can't wrap `operator "" _myRawLiteral` unless renamed to a valid identifier.
```

If %rename is used, then it can be called like any other wrapped method. Currently you need to specify the full declaration including parameters for `%rename`:

> 如果使用 `%rename`，则可以像其他任何包装方法一样调用它。当前，你需要指定完整的声明，包括 `%rename` 的参数：

```
%rename(MyRawLiteral)  operator"" _myRawLiteral(const char * value);
```

Or if you just wish to ignore it altogether:

> 或者，你可以直接忽略掉：

```
%ignore operator "" _myRawLiteral(const char * value);
```

Note that use of user-defined literals such as the following still give a syntax error:

> 请注意，使用用户定义文字（如以下内容）仍会产生语法错误：

```c++
OutputType var1 = "1234"_suffix;
OutputType var2 = 1234_suffix;
OutputType var3 = 3.1416_suffix;
```

### 7.2.21 `thread_local` 存储

SWIG correctly parses the `thread_local` keyword. For example, variables reachable by the current thread can be defined as:

> SWIG 能正确解析 `thread_local` 关键字。例如，当前线程可访问的变量可以定义为：

```c++
struct A {
  static thread_local int val;
};
thread_local int global_val;
```

The use of the `thread_local` storage specifier does not affect the wrapping process; it does not modify the wrapper code compared to when it is not specified. A variable will be thread local if accessed from different threads from the target language in the same way that it will be thread local if accessed from C++ code.

> 使用 `thread_local` 存储说明符不会影响包装过程。与未指定时相比，它不会修改包装器代码。如果从目标语言的不同线程访问变量，则该变量将是线程局部的，就像从 C++ 代码访问该变量时一样。

### 7.2.22 显式默认函数（defaulted function）与删除函数（deleted function）

SWIG handles explicitly defaulted functions, that is, `= default` added to a function declaration. Deleted definitions, which are also called deleted functions, have `= delete` added to the function declaration. For example:

> SWIG 处理显式默认函数，即添加到函数声明中的 `= default`。删除的定义（也称为删除函数）是在函数声明中添加 `= delete`。例如：

```c++
struct NonCopyable {
  NonCopyable & operator=(const NonCopyable &) = delete; /* Removes operator= */
  NonCopyable(const NonCopyable &) = delete;             /* Removes copy constructor */
  NonCopyable() = default;                               /* Explicitly allows the empty constructor */
};
```

Wrappers for deleted functions will not be available in the target language. Wrappers for defaulted functions will of course be available in the target language. Explicitly defaulted functions have no direct effect for SWIG wrapping as the declaration is handled much like any other method declaration parsed by SWIG.

Deleted functions are also designed to prevent implicit conversions when calling the function. For example, the C++ compiler will not compile any code which attempts to use an `int` as the type of the parameter passed to `f` below:

> 目标语言将无法使用删除函数的包装器。当然，目标语言能够使用默认函数的包装器。显式默认函数对 SWIG 包装没有直接影响，因为声明的处理方式与 SWIG 解析的任何其他方法声明非常相似。
>
> 删除函数还旨在防止在调用函数时进行隐式转换。例如，C++ 编译器不会编译任何试图将 `int` 传递给下面 `f` 的代码：

```c++
struct NoInt {
  void f(double i);
  void f(int) = delete;
};
```

This is a C++ compile time check and SWIG does not make any attempt to detect if the target language is using an int instead of a double though, so in this case it is entirely possible to pass an int instead of a double to `f` from Java, Python etc.

> 这是 C++ 编译时检查，但是 SWIG 不会尝试检测目标语言是否使用 `int` 而不是 `double`，因此在这种情况下，完全有可能从 Java，Python 等将 `int` 而不是 `double` 传递给 `f`。

### 7.2.23 `long long int` 类型

SWIG correctly parses and uses the new `long long` type already introduced in C99 some time ago.

> SWIG 正确地解析并使用了早先 C99 中已经引入的 `long long` 类型。

### 7.2.24 静态断言

SWIG correctly parses the new `static_assert` declarations. This is a C++ compile time directive so there isn't anything useful that SWIG can do with it.

> SWIG 正确地解析了新的 `static_assert` 声明。这是一个 C++ 编译时指令，因此 SWIG 不能对其执行任何有用的操作。

```c++
template <typename T>
struct Check {
  static_assert(sizeof(int) <= sizeof(T), "not big enough");
};
```

### 7.2.25 允许在没有显式对象的情况下对类成员使用 `sizeof`

SWIG can parse the new `sizeof()` on types as well as on objects. For example:

> SWIG 可以在类型和对象上解析新的 `sizeof()` 函数。例如：

```c++
struct A {
  int member;
};

const int SIZE = sizeof(A::member); // does not work with C++03. Okay with C++11
```

In Python:

> 在 Python 中：

```python
>>> SIZE
8
```

### 7.2.26 异常规范与 `noexcept`

C++11 added in the noexcept specification to exception specifications to indicate that a function simply may or may not throw an exception, without actually naming any exception. SWIG understands these, although there isn't any useful way that this information can be taken advantage of by target languages, so it is as good as ignored during the wrapping process. Below are some examples of noexcept in function declarations:

> C++11 在 `noexcept` 规范中添加了异常规范，以指示一个函数可能会也可能不会抛出异常，而无需实际命名任何异常。尽管没有任何有用的方法可以使目标语言利用此信息，但 SWIG 能理解这些知识，因此在包装过程中它就像被忽略了一样。以下是函数声明中的 `noexcept` 的一些示例：

```c++
static void noex1() noexcept;
int noex2(int) noexcept(true);
int noex3(int, bool) noexcept(false);
```

### 7.2.27 控制与查询对象对齐

An `alignof` operator is used mostly within C++ to return alignment in number of bytes, but could be used to initialize a variable as shown below. The variable's value will be available for access by the target language as any other variable's compile time initialised value.

> `alignof` 运算符通常在 C++ 中使用，以字节为单位返回对齐方式，但可用于初始化变量，如下所示。该变量的值与其他任何变量在编译时的初始化值一样可供目标语言访问。

```c++
const int align1 = alignof(A::member);
```

The `alignas` specifier for variable alignment is not yet supported. Example usage:

> 尚不支持用于变量对齐的 `alignas` 说明符。示例：

```c++
struct alignas(16) S {
  int num;
};
alignas(double) unsigned char c[sizeof(double)];
```

Use the preprocessor to work around this for now:

> 现在使用预处理器解决此问题：

```c++
#define alignas(T)
```

### 7.2.28 属性

Attributes such as those shown below, are not yet supported and will give a syntax error.

> 尚不支持如下所示的属性，这些属性会产生语法错误。

```c++
int [[attr1]] i [[attr2, attr3]];

[[noreturn, nothrow]] void f [[noreturn]] ();
```

## 7.3 标准库变更

### 7.3.1 线程工具

SWIG does not currently wrap or use any of the new threading classes introduced (thread, mutex, locks, condition variables, task). The main reason is that SWIGtarget languages offer their own threading facilities so there is limited use for them.

> SWIG 当前不包装或使用任何新引入的线程类（线程、互斥锁、锁、条件变量、任务）。主要原因是 SWIG 的目标语言提供了自己的线程工具，因此使用范围有限。

### 7.3.2 元组类型

SWIG does not provide library files for the new tuple types yet. Variadic template support requires further work to provide substantial tuple wrappers.

> SWIG 尚未提供新的元组类型的库文件。可变参数模板支持需要进一步的工作以提供大量的元组包装器。

### 7.3.3 哈希表

The new hash tables in the STL are `unordered_set`, `unordered_multiset`, `unordered_map`, `unordered_multimap`. These are not available inSWIG , but in principle should be easily implemented by adapting the current STL containers.

> STL 中新的哈希表是 `unordered_set`、`unordered_multiset`、`unordered_map` 和 `unordered_multimap`。这些在 SWIG 中不可用，但原则上应通过适应当前的 STL 容器轻松实现。

### 7.3.4 正则表达式

While SWIG could provide wrappers for the new C++11 regular expressions classes, there is little need as the target languages have their own regular expression facilities.

> 尽管 SWIG 可以为新的 C++11 正则表达式类提供包装器，但是几乎没有必要，因为目标语言具有自己的正则表达式工具。

### 7.3.5 通用智能指针

SWIG provides special smart pointer handling for `std::shared_ptr` in the same way it has support for `boost::shared_ptr`. Please see the [shared_ptr smart pointer](http://SWIG.org/Doc3.0/Library.html#Library_std_shared_ptr)library section. There is no special smart pointer handling available for `std::weak_ptr` and `std::unique_ptr` yet.

> SWIG 以支持 `boost::shared_ptr` 的相同方式为 `std::shared_ptr` 提供了特殊的智能指针处理。请参阅 [`shared_ptr` 智能指针](http://SWIG.org/Doc3.0/Library.html#Library_std_shared_ptr)库部分。`std::weak_ptr` 和 `std::unique_ptr` 智能指针尚无特殊处理。

### 7.3.6 扩展的随机数工具

This feature extends and standardizes the standard library only and does not effect the C++ language nor SWIG.

> 此功能仅扩展和标准化了标准库，并且不影响 C++ 与 SWIG 。

### 7.3.7 包装器引用

Wrapper references are similar to normal C++ references but are copy-constructible and copy-assignable. They could conceivably be used in public APIs. There is no special support for `std::reference_wrapper` inSWIGthough. Users would need to write their own typemaps if wrapper references are being used and these would be similar to the plain C++ reference typemaps.

> 包装器引用与普通 C++ 引用相似，但它们可复制构造和可复制分配。可以想象它们可以在公共 API 中使用。但是，SWIG 中没有对 `std::reference_wrapper` 的特殊支持。如果使用包装器引用，则用户将需要编写自己的类型映射，并且它们将与普通的 C++ 引用类型映射相似。

### 7.3.8 函数对象的多态包装器

SWIG supports functor classes in a few languages in a very natural way. However nothing is provided yet for the new `std::function` template. SWIG will parse usage of the template like any other template.

> SWIG 非常自然地支持几种语言的仿函数类。但是，还没有为新的 `std::function` 模板提供任何东西。SWIG 将像解析其他模板一样解析该模板的用法。

```
%rename(__call__) Test::operator(); // Default renaming used for Python

struct Test {
  bool operator()(int x, int y); // function object
};

#include <functional>
std::function<void (int, int)> pF = Test;   // function template wrapper
```

Example of supported usage of the plain functor from Python is shown below. It does not involve `std::function`.

> 下面示例显示了所支持的 Python 仿函数用法。它不涉及 `std::function`。

```python
t = Test()
b = t(1, 2) # invoke C++ function object
```

### 7.3.9 元编程的 `type_traits`

The `type_traits` functions to support C++ metaprogramming is useful at compile time and is aimed specifically at C++ development:

> 支持 C++ 元编程的 `type_traits` 函数在编译时很有用，并且专门针对 C++ 开发：

```c++
#include <type_traits>

// First way of operating.
template< bool B > struct algorithm {
  template< class T1, class T2 > static int do_it(T1 &, T2 &)  { /*...*/ return 1; }
};

// Second way of operating.
template<> struct algorithm<true> {
  template< class T1, class T2 > static int do_it(T1, T2)  { /*...*/ return 2; }
};

// Instantiating `elaborate` will automatically instantiate the
// correct way to operate, depending on the types used.
template< class T1, class T2 > int elaborate(T1 A, T2 B) {
  // Use the second way only if `T1` is an integer and if `T2` is a floating point,
  // otherwise use the first way.
  return algorithm< std::is_integral<T1>::value && std::is_floating_point<T2>::value >::do_it(A, B);
}
```

SWIG correctly parses the template specialization, template types etc. However, metaprogramming and the additional support in the type_traits header is really for compile time and is not much use at runtime for the target languages. For example, as SWIG requires explicit instantiation of templates via `%template`, there isn't much that `std::is_integral<int>` is going to provide by itself. However, template functions using such metaprogramming techniques might be useful to wrap. For example, the following instantiations could be made:

> SWIG 正确地解析了模板特化、模板类型等。但是，元编程和 `type_traits` 头文件中的其他支持实际上是在编译时使用的，在运行时对于目标语言而言使用并不多。例如，由于 SWIG 需要通过 `%template` 来显式实例化模板，因此 `std::is_integral<int>` 本身不会提供太多功能。但是，使用此类元编程技术的模板功能可能对包装有用。例如，可以进行以下实例化：

```
%template(Elaborate) elaborate<int, int>;
%template(Elaborate) elaborate<int, double>;
```

Then the appropriate algorithm can be called for the subset of types given by the above `%template` instantiations from a target language, such as Python:

> 然后，可以针对目标语言（例如 Python）中上述 `%template` 实例化所给出的类型子集调用适当的算法：

```python
>>> Elaborate(0, 0)
1
>>> Elaborate(0, 0.0)
2
```

### 7.3.10 计算函数对象返回类型的统一方法

The new `std::result_of` class introduced in the `<functional>` header provides a generic way to obtain the return type of a function type via `std::result_of::type`. There isn't any library interface file to support this type. With a bit of work, SWIG will deduce the return type of functions when used in `std::result_of` using the approach shown below. The technique basically forward declares the `std::result_of` template class, then partially specializes it for the function types of interest. SWIG will use the partial specialization and hence correctly use the `std::result_of::type` provided in the partial specialization.

> 在 `<functional>` 头文件中引入的 `std::result_of` 类提供了一种通过 `std::result_of::type` 获取函数类型的返回类型的通用方法。没有任何库接口文件支持此类型。经过一点工作，SWIG 将使用以下所示的方法推导使用 `std::result_of` 时函数的返回类型。该技术基本上向前声明了 `std::result_of` 模板类，然后将其偏特化用于感兴趣的函数类型。SWIG 将使用偏特化，因此正确使用了偏特化中提供的 `std::result_of::type`。

```
%inline %{
#include <functional>
typedef double(*fn_ptr)(double);
%}

namespace std {
  // Forward declaration of result_of
  template<typename Func> struct result_of;
  // Add in a partial specialization of result_of
  template<> struct result_of< fn_ptr(double) > {
    typedef double type;
  };
}

%template() std::result_of< fn_ptr(double) >;

%inline %{

double square(double x) {
  return (x * x);
}

template<class Fun, class Arg>
typename std::result_of<Fun(Arg)>::type test_result_impl(Fun fun, Arg arg) {
  return fun(arg);
}
%}

%template(test_result) test_result_impl< fn_ptr, double >;
%constant double (*SQUARE)(double) = square;
```

Note the first use of `%template` whichSWIG requires to instantiate the template. The empty template instantiation suffices as no proxy class is required for `std::result_of<Fun(Arg)>::type` as this type is really just a `double`. The second `%template` instantiates the template function which is being wrapped for use as a callback. The `%constant` can then be used for any callback function as described in [Pointers to functions and callbacks](http://SWIG.org/Doc3.0/SWIG.html#SWIG_nn30).

Example usage from Python should give the not too surprising result:

> 请注意，SWIG 要求实例化模板时首先使用 `%template`。空模板实例化就足够了，因为 `std::result_of<Fun(Arg)>::type` 不需要代理类，因为这种类型实际上只是一个 `double`。第二个 `%template` 实例化模板函数，该函数被包装以用作回调。然后，可以将 `%constant` 用于任何回调函数，如[函数指针与回调](http://SWIG.org/Doc3.0/SWIG.html#SWIG_nn30)中所述。
>
> 来自 Python 的示例用法应该不会太令人惊讶：

```python
>>> test_result(SQUARE, 5.0)
25.0
```

Phew, that is a lot of hard work to get a callback working. You could just go with the more attractive option of just using `double` as the return type in the function declaration instead of `result_of`!

> 唷，要使回调正常工作，有很多艰苦的工作要做。你可以使用更具吸引力的选项，即在函数声明中仅将 `double` 作为返回类型，而不是 `result_of`！
