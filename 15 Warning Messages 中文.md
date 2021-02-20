[TOC]

# 15 警告消息

## 15.1 引言

During compilation, SWIG may generate a variety of warning messages. For example:

> 在编译期间，SWIG 可能会生成各种警告消息。例如：

```
example.i:16: Warning 501: Overloaded declaration ignored.  bar(double)
example.i:15: Warning 501: Previous declaration is bar(int)
```

Typically, warning messages indicate non-fatal problems with the input where the generated wrapper code will probably compile, but it may not work like you expect.

> 通常，警告消息指示输入可能发生的非致命问题，生成的包装程序代码可能会通过编译，但可能无法按预期工作。

## 15.2 警告消息抑制

All warning messages have a numeric code that is shown in the warning message itself. To suppress the printing of a warning message, a number of techniques can be used. First, you can run SWIG with the `-w`command line option. For example:

> 所有警告消息都有一个数字代码，显示在警告消息本身中。可以使用多种技术禁止打印警告消息。首先，你可以使用 `-w` 命令行选项运行 SWIG。例如：

```
% swig -python -w501 example.i
% swig -python -w501,505,401 example.i
```

Alternatively, warnings can be suppressed by inserting a special preprocessor pragma into the input file:

> 或者，可以通过在输入文件中插入特殊的预处理器编译指令来抑制警告：

```
%module example
#pragma SWIG nowarn=501
#pragma SWIG nowarn=501,505,401
```

Finally, code-generation warnings can be disabled on a declaration by declaration basis using the `%warnfilter` directive. For example:

> 最后，可以使用 `%warnfilter` 指令在逐个声明的基础上禁用代码生成警告。例如：

```
%module example
%warnfilter(501) foo;
...
int foo(int);
int foo(double);              // Silently ignored.
```

The `%warnfilter` directive has the same semantics as other declaration modifiers like `%rename`, `%ignore`and `%feature`, see the [%feature directive](http://swig.org/Doc3.0/Customization.html#Customization_features) section. For example, if you wanted to suppress a warning for a method in a class hierarchy, you could do this:

> `%warnfilter` 指令与其他声明修饰符（如 `%rename`、`%ignore` 和 `%feature`）具有相同的语义，请参见 [`%feature` 指令](http://swig.org/Doc3.0/Customization.html#Customization_features)部分。例如，如果你想取消对类层次结构中方法的警告，则可以执行以下操作：

```
%warnfilter(501) Object::foo;
class Object {
public:
  int foo(int);
  int foo(double);      // Silently ignored
  ...
};

class Derived : public Object {
public:
  int foo(int);
  int foo(double);      // Silently ignored
  ...
};
```

Warnings can be suppressed for an entire class by supplying a class name. For example:

> 通过提供类名称，可以抑制整个类的警告。例如：

```
%warnfilter(501) Object;

class Object {
public:
  ...                      // All 501 warnings ignored in class
};
```

There is no option to suppress all SWIG warning messages. The warning messages are there for a reason---to tell you that something may be *broken* in your interface. Ignore the warning messages at your own peril.

> 没有禁止所有 SWIG 警告消息的选项。出现警告消息是有原因的——告诉你接口中的某些内容可能“损坏”。忽略警告消息，后果自负。

## 15.3 开启警告

Some warning messages are disabled by default and are generated only to provide additional diagnostics. These warnings can be turned on using the `-Wextra` option. For example:

> 默认情况下，某些警告消息是禁用的，并且仅用于提供其他诊断信息而生成。可以使用 `-Wextra` 选项打开这些警告。例如：

```
% swig -Wextra -python example.i
```

To selectively turn on extra warning messages, you can use the directives and options in the previous section--simply add a "+" to all warning numbers. For example:

> 要有选择地打开其他警告消息，可以使用上一节中的指令和选项——只需在所有警告编号上添加一个 `+` 即可。例如：

```
% swig -w+309,+452 example.i
```

or in your interface file use either

> 或者，在你的接口文件中使用

```
#pragma SWIG nowarn=+309,+452
```

or

> 再或

```
%warnfilter(+309,+452) foo;
```

Note: selective enabling of warnings with `%warnfilter` overrides any global settings you might have made using `-w` or `#pragma`.

You can of course also enable all warnings and suppress a select few, for example:

> 注意：使用 `%warnfilter` 选择性启用警告会覆盖你可能使用 `-w` 或 `#pragma` 进行的所有全局设置。
>
> 当然，你也可以启用所有警告，并选择关闭某些警告，例如：

```
% swig -Wextra -w309,452 example.i
```

The warnings on the right take precedence over the warnings on the left, so in the above example `-Wextra`adds numerous warnings including 452, but then `-w309,452` overrides this and so 452 is suppressesed.

If you would like all warnings to appear, regardless of the warning filters used, then use the `-Wall` option. The `-Wall` option also turns on the extra warnings that `-Wextra` adds, however, it is subtely different. When `-Wall` is used, it also disables all other warning filters, that is, any warnings suppressed or added in `%warnfilter`, `#pragma SWIG nowarn` or the `-w` option.

> 右边的警告优先于左边的警告，因此在上面的示例中，`-Wextra` 添加了很多警告，包括 `452`，但随后 `-w309,452` 覆盖了该警告，因此 `452` 被禁止。
>
> 如果你希望所有警告都出现，无论使用什么警告过滤器，请使用 `-Wall` 选项。`-Wall` 选项还打开了 `-Wextra` 添加的额外警告，但是它完全不同。当使用 `-Wall` 时，它也会禁用所有其他警告过滤器，即在 `%warnfilter，#pragma SWIG nowarn` 或 `-w` 选项中禁止或添加的任何警告。

## 15.4 发出警告消息

Warning messages can be issued from an interface file using a number of directives. The `%warn` directive is the most simple:

> 可以使用许多指令从接口文件中发出警告消息。`%warn` 指令是最简单的：

```
%warn "900:This is your last warning!"
```

All warning messages are optionally prefixed by the warning number to use. If you are generating your own warnings, make sure you don't use numbers defined in the table at the end of this section.

The `%ignorewarn` directive is the same as `%ignore` except that it issues a warning message whenever a matching declaration is found. For example:

> 所有警告消息都可以选择以警告编号作为前缀。如果要生成自己的警告，请确保不要使用本节末尾表中定义的数字。
>
> `%ignorewarn` 指令与 `%ignore` 指令相同，不同之处在于，只要找到匹配的声明，它就会发出警告消息。例如：

```
%ignorewarn("362:operator= ignored") operator=;
```

Warning messages can be associated with typemaps using the `warning` attribute of a typemap declaration. For example:

> 可以使用类型映射声明的 `warning` 属性将警告消息与类型映射关联。例如：

```
%typemap(in, warning="901:You are really going to regret this usage of $1_type $1_name") blah * {
  ...
}
```

In this case, the warning message will be printed whenever the typemap is actually used and the [special variables](http://swig.org/Doc3.0/Typemaps.html#Typemaps_special_variables) will be expanded as appropriate, for example:

> 在这种情况下，每当实际使用类型映射表时，就会显示警告消息，并且将适当地扩展[特殊变量](http://swig.org/Doc3.0/Typemaps.html#Typemaps_special_variables)，例如：

```
example.i:23: Warning 901: You are really going to regret this usage of blah * self
example.i:24: Warning 901: You are really going to regret this usage of blah * stuff
```

## 15.5 象征符号

The `swigwarn.swg` file that is installed with SWIG contains symbol constants that could also be used in `%warnfilter` and `#pragma SWIG nowarn`. For example this file contains the following line:

> 随 SWIG 一起安装的 `swigwarn.swg` 文件包含符号常量，这些常量也可以在 `%warnfilter` 和 `#pragma SWIG nowarn` 中使用。例如，此文件包含以下行：

```
%define SWIGWARN_TYPE_UNDEFINED_CLASS 401 %enddef
```

so `SWIGWARN_TYPE_UNDEFINED_CLASS` could be used instead of 401, for example:

> 所以 `SWIGWARN_TYPE_UNDEFINED_CLASS` 可以代替 `401`，例如：

```
#pragma SWIG nowarn=SWIGWARN_TYPE_UNDEFINED_CLASS
```

or

> 或者

```
%warnfilter(SWIGWARN_TYPE_UNDEFINED_CLASS) Foo;
```

## 15.6 说明

The ability to suppress warning messages is really only provided for advanced users and is not recommended in normal use. You are advised to modify your interface to fix the problems highlighted by the warnings wherever possible instead of suppressing warnings.

Certain types of SWIG problems are errors. These usually arise due to parsing errors (bad syntax) or semantic problems for which there is no obvious recovery. There is no mechanism for suppressing error messages.

> 抑制警告消息的功能实际上仅提供给高级用户，在正常使用中不建议使用。建议你尽可能修改接口文件以解决警告突出显示的问题，而不是抑制警告。
>
> 某些类型的 SWIG 问题是错误。这些通常是由于解析错误（语法错误）或语义问题（无法明显恢复）引起的。没有抑制错误消息的机制。

## 15.7 将警告视为错误

Warnings can be handled as errors by using the `-Werror` command line option. This will cause SWIG to exit with a non successful exit code if a warning is encountered.

> 通过使用 `-Werror` 命令行选项，可以将警告视为错误。如果遇到警告，这将导致 SWIG 以非成功的退出代码退出。

## 15.8 消息输出格式

The output format for both warnings and errors can be selected for integration with your favourite IDE/editor. Editors and IDEs can usually parse error messages and if in the appropriate format will easily take you directly to the source of the error. The standard format is used by default except on Windows where the Microsoft format is used by default. These can be overridden using command line options, for example:

> 可以选择警告和错误的输出格式，以与你喜欢的 IDE 或编辑器集成。编辑器和 IDE 通常可以解析错误消息，并且如果采用适当的格式，则可以轻松地直接将你带到错误源。默认使用标准格式，但在 Windows 上默认使用 Microsoft 格式。这些可以使用命令行选项覆盖，例如：

```
$ swig -python -Fstandard example.i
example.i:4: Syntax error in input(1).
$ swig -python -Fmicrosoft example.i
example.i(4) : Syntax error in input(1).
```

## 15.9 警告代码目录

### 15.9.1 弃用的功能 (100-199)

* 101. Deprecated `%extern` directive.
* 102. Deprecated `%val` directive.
* 103. Deprecated `%out` directive.
* 104. Deprecated `%disabledoc` directive.
* 105. Deprecated `%enabledoc` directive.
* 106. Deprecated `%doconly` directive.
* 107. Deprecated `%style` directive.
* 108. Deprecated `%localstyle` directive.
* 109. Deprecated `%title` directive.
* 110. Deprecated `%section` directive.
* 111. Deprecated `%subsection` directive.
* 112. Deprecated `%subsubsection` directive.
* 113. Deprecated `%addmethods` directive.
* 114. Deprecated `%readonly` directive.
* 115. Deprecated `%readwrite` directive.
* 116. Deprecated `%except` directive.
* 117. Deprecated `%new` directive.
* 118. Deprecated `%typemap(except)`.
* 119. Deprecated `%typemap(ignore)`.
* 120. Deprecated command line option (-runtime, -noruntime).
* 121. Deprecated `%name` directive.
* 126. The `nestedworkaround` feature is deprecated.

### 15.9.2 预处理器 (200-299)

* 201. Unable to find `filename`.
* 202. Could not evaluate expression `expr`.
* 203. Both includeall and importall are defined: using includeall.
* 204. CPP `#warning`, `warning`.
* 205. CPP `#error`, `error`.
* 206. Unexpected tokens after `#directive` directive.

### 15.9.3 C/C++ 解析器 (300-399)

* 301. `class` keyword used, but not in C++ mode.
* 302. Identifier `name` redefined (ignored).
* 303. `%extend` defined for an undeclared class `name`.
* 304. Unsupported constant value (ignored).
* 305. Bad constant value (ignored).
* 306. `identifier` is private in this context.
* 307. Can't set default argument value (ignored)
* 308. Namespace alias `name` not allowed here. Assuming `name`
* 309. [private | protected] inheritance ignored.
* 310. Template `name` was already wrapped as `name` (ignored)
* 312. Unnamed nested class not currently supported (ignored).
* 313. Unrecognized extern type `name` (ignored).
* 314. `identifier` is a `lang` keyword.
* 315. Nothing known about `identifier`.
* 316. Repeated %module directive.
* 317. Specialization of non-template `name`.
* 318. Instantiation of template `name` is ambiguous, instantiation `templ` used, instantiation `templ` ignored.
* 319. No access specifier given for base class `name` (ignored).
* 320. Explicit template instantiation ignored.
* 321. `identifier` conflicts with a built-in name.
* 322. Redundant redeclaration of `name`.
* 323. Recursive scope inheritance of `name`.
* 324. Named nested template instantiations not supported. Processing as if no name was given to %template().
* 325. Nested *kind* not currently supported (`name` ignored).
* 326. Deprecated `%extend` name used - the *kind* name `name` should be used instead of the typedef name `name`.
* 350. operator new ignored.
* 351. operator delete ignored.
* 352. operator+ ignored.
* 353. operator- ignored.
* 354. operator* ignored.
* 355. operator/ ignored.
* 356. operator% ignored.
* 357. operator^ ignored.
* 358. operator& ignored.
* 359. operator| ignored.
* 360. operator~ ignored.
* 361. operator! ignored.
* 362. operator= ignored.
* 363. operator< ignored.
* 364. operator> ignored.
* 365. operator+= ignored.
* 366. operator-= ignored.
* 367. operator*= ignored.
* 368. operator/= ignored.
* 369. operator%= ignored.
* 370. operator^= ignored.
* 371. operator&= ignored.
* 372. operator|= ignored.
* 373. operator<< ignored.
* 374. operator>>ignored.
* 375. operator<<= ignored.
* 376. operator>>= ignored.
* 377. operator== ignored.
* 378. operator!= ignored.
* 379. operator<= ignored.
* 380. operator>= ignored.
* 381. operator&& ignored.
* 382. operator|| ignored.
* 383. operator++ ignored.
* 384. operator-- ignored.
* 385. operator, ignored.
* 386. operator-<* ignored.
* 387. operator-< ignored.
* 388. operator() ignored.
* 389. operator[] ignored.
* 390. operator+ ignored (unary).
* 391. operator- ignored (unary).
* 392. operator* ignored (unary).
* 393. operator& ignored (unary).
* 394. operator new[] ignored.
* 395. operator delete[] ignored.

### 15.9.4 类型与类型映射 (400-499)

* 401. Nothing known about class `name`. Ignored.
* 402. Base class `name` is incomplete.
* 403. Class `name` might be abstract.
* 450. Deprecated typemap feature (`$source`/`$target`).
* 451. Setting const char * variable may leak memory.
* 452. Reserved
* 453. Can't apply (pattern). No typemaps are defined.
* 460. Unable to use type `type` as a function argument.
* 461. Unable to use return type `type` in function `name`.
* 462. Unable to set variable of type `type`.
* 463. Unable to read variable of type `type`.
* 464. Unsupported constant value.
* 465. Unable to handle type `type`.
* 466. Unsupported variable type `type`.
* 467. Overloaded `declaration` not supported (incomplete type checking rule - no precedence level in typecheck typemap for '`type`')
* 468. No 'throw' typemap defined for exception type `type`
* 469. No or improper directorin typemap defined for `type`
* 470. Thread/reentrant unsafe wrapping, consider returning by value instead.
* 471. Unable to use return type `type` in director method
* 474. Method `method` usage of the optimal attribute ignored in the out typemap as the following cannot be used to generate optimal code: `code`
* 475. Multiple calls to `method` might be generated due to optimal attribute usage in the out typemap.
* 476. Initialization using `std::initializer_list`.
* 477. No directorthrows typemap defined for `type`

### 15.9.5 代码生成 (500-599)

* 501. Overloaded declaration ignored. `decl`. Previous declaration is `decl`.
* 502. Overloaded constructor ignored. `decl`. Previous declaration is `decl`.
* 503. Can't wrap `identifier` unless renamed to a valid identifier.
* 504. Function `name` must have a return type. Ignored.
* 505. Variable length arguments discarded.
* 506. Can't wrap varargs with keyword arguments enabled.
* 507. Adding native function `name` not supported (ignored).
* 508. Declaration of `name` shadows declaration accessible via operator->(), previous declaration of'`declaration`'.
* 509. Overloaded method `declaration` effectively ignored, as it is shadowed by `declaration`.
* 510. Friend function `name` ignored.
* 511. Can't use keyword arguments with overloaded functions.
* 512. Overloaded method `declaration` ignored, using non-const method `declaration` instead.
* 513. Can't generate wrappers for unnamed struct/class.
* 514.
* 515.
* 516. Overloaded method `declaration` ignored, using `declaration` instead.
* 517.
* 518. Portability warning: File `file1` will be overwritten by `file2` on case insensitive filesystems such as Windows' FAT32 and NTFS unless the class/module name is renamed.
* 519. `%template()` contains no name. Template method ignored: `declaration`
* 520. *Base/Derived* class `classname1` of `classname2` is not similarly marked as a smart pointer.
* 521. Illegal destructor name `name`. Ignored.
* 522. Use of an illegal constructor name `name` in %extend is deprecated, the constructor name should be `name`.
* 523. Use of an illegal destructor name `name` in %extend is deprecated, the destructor name should be `name`.

### 15.9.6 特定语言模块警告 (700-899)

* 801. Wrong name (corrected to `name`). (Ruby).
* 810. No jni typemap defined for `type` (Java).
* 811. No jtype typemap defined for `type` (Java).
* 812. No jstype typemap defined for `type` (Java).
* 813. Warning for `classname`, base `baseclass` ignored. Multiple inheritance is not supported in Java. (Java).
* 814.
* 815. No javafinalize typemap defined for `type` (Java).
* 816. No javabody typemap defined for `type` (Java).
* 817. No javaout typemap defined for `type` (Java).
* 818. No javain typemap defined for `type` (Java).
* 819. No javadirectorin typemap defined for `type` (Java).
* 820. No javadirectorout typemap defined for `type` (Java).
* 821.
* 822. Covariant return types not supported in Java. Proxy method will return `basetype` (Java).
* 823. No javaconstruct typemap defined for `type` (Java).
* 824. Missing JNI descriptor in directorin typemap defined for `type` (Java).
* 825. "directorconnect" attribute missing in `type` "javaconstruct" typemap. (Java).
* 826. The nspace feature is used on `type` without -package. The generated code may not compile as Java does not support types declared in a named package accessing types declared in an unnamed package. (Java).
* 830. No ctype typemap defined for `type` (C#).
* 831. No cstype typemap defined for `type` (C#).
* 832. No cswtype typemap defined for `type` (C#).
* 833. Warning for `classname`, base `baseclass` ignored. Multiple inheritance is not supported in C#. (C#).
* 834.
* 835. No csfinalize typemap defined for `type` (C#).
* 836. No csbody typemap defined for `type` (C#).
* 837. No csout typemap defined for `type` (C#).
* 838. No csin typemap defined for `type` (C#).
* 839.
* 840.
* 841.
* 842. Covariant return types not supported in C#. Proxy method will return `basetype` (C#).
* 843. No csconstruct typemap defined for `type` (C#).
* 844. C# exception may not be thrown - no `$excode` or excode attribute in `typemap` typemap. (C#).
* 845. Unmanaged code contains a call to a SWIG_CSharpSetPendingException method and C# code does not handle pending exceptions via the canthrow attribute. (C#).
* 870. Warning for `classname`: Base `baseclass` ignored. Multiple inheritance is not supported in PHP. (Php).
* 871. Unrecognized pragma `pragma`. (Php).

### 15.9.7 用户自定义 (900-999)

These numbers can be used by your own application.

> 这些数字可被你自己的应用程序使用。

## 15.10 历史

The ability to control warning messages was first added to SWIG-1.3.12.

> SWIG-1.3.12 中首次添加控制警告信息的功能。
