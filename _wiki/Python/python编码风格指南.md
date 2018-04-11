---
layout: wiki
title: "Python编码风格指南"
date: 2017-01-06 09:16
updated: 2017-01-08 13:10
keywords: Python, 风格, 规范
categories: Python
---

# 代码布局

## 缩进

每一级缩进使用4个空格。

续行应使用Python隐式续行(即垂直对齐圆括号、方括号和花括号)或悬挂缩进。

采用悬挂缩进时需注意两点：第一行不应有参数；在续行中需要再缩进一级以便清楚表示。

正确示例:

```python
# 垂直对齐开始分界符(左括号)
foo = long_function_name(var_one, var_two,
                         var_three, var_four)

# 悬挂缩进需要多缩进一级
foo = long_function_name(
    var_one, var_two,
    var_three, var_four)

# 续行多缩进一级以同其他代码区别
def long_function_name(
        var_one, var_two, var_three,
        var_four):
    print(var_one)
```

错误示例:

```python
# 采用非垂直对齐时第一行不应有参数
foo = long_function_name(var_one, var_two,
    var_three, var_four)

# 续行没有被区分开，因此需要再缩进一级
def long_function_name(
    var_one, var_two, var_three,
    var_four):
    print(var_one)
```

对于续行来说，4空格的规则可以不遵守:

```python
# 悬挂缩进可以不采用4空格的缩进方法。
foo = long_function_name(
  var_one, var_two,
  var_three, var_four)
```

如果`if`语句太长，需要用多行书写，以下几种方法都是可行的:

```python
# 不采用额外缩进
if (this_is_one_thing and
    that_is_another_thing):
    do_something()

# 增加一行注释，在支持语法高亮编辑器中显示时能有所区分
if (this_is_one_thing and
    that_is_another_thing):
    # Since both conditions are true, we can frobnicate.
    do_something()

# 在条件语句的续行增加一级缩进
if (this_is_one_thing
        and that_is_another_thing):
    do_something()
```

多行结束右圆/方/花括号可以单独一行书写，和上一行的缩进对齐:

```python
my_list = [
    1, 2, 3,
    4, 5, 6,
    ]
result = some_function_that_takes_arguments(
    'a', 'b', 'c',
    'd', 'e', 'f',
    )
```

也可以和多行开始的第一行的第一个字符对齐:

```python
my_list = [
    1, 2, 3,
    4, 5, 6,
]
result = some_function_that_takes_arguments(
    'a', 'b', 'c',
    'd', 'e', 'f',
)
```

## Tab还是空格?

空格是首选的缩进方案。

Tab应该只在现有代码已经使用tab进行缩进的情况下使用，以便和现有代码保持一致。

Python 3不允许tab和空格混合使用。

Python 2的代码若有tab和空格混合使用的情况，应该把tab全部转换为只有空格。

当使用命令行运行Python 2时，使用`-t`选项，会出现非法混用tab和空格的警告。当使用`-tt`选项时，这些警告会变成错误。强烈推荐使用这些选项！

## 行最大长度

限制所有行的长度在79个字符以内。

对于连续大段的文字（比如文档字符串(docstring)或注释），其结构上的限制更少，这些行应该被限制在72个字符长度内。

有时续行只能使用反斜杠才。例如，较长的多个`with`语句不能采用隐式续行，只能接受反斜杠表示换行:

```python
with open('/path/to/some/file/you/want/to/read') as file_1, \
     open('/path/to/some/file/being/written', 'w') as file_2:
    file_2.write(file_1.read())
```

要确保续行的缩进适当。逻辑运算符附近的换行处最好是在运算符**之后**，而不是在其之前:

```python
class Rectangle(Blob):

    def __init__(self, width, height,
                 color='black', emphasis=None, highlight=0):
        if (width == 0 and height == 0 and
                color == 'red' and emphasis == 'strong' or
                highlight > 100):
            raise ValueError("sorry, you lose")
        if width == 0 and height == 0 and (color == 'red' or
                                           emphasis is None):
            raise ValueError("I don't think so -- values are %s, %s" %
                             (width, height))
        Blob.__init__(self, width, height,
                      color, emphasis, highlight)
```

## 二元操作符的换行应该在前还是后?

长久以来推荐的风格都是在二元操作符后换行，但是它影响了可读性:

```python
# No: operators sit far away from their operands
income = (gross_wages +
          taxable_interest +
          (dividends - qualified_dividends) -
          ira_deduction -
          student_loan_interest)
```

应该使用可读性更高的写法:

```python
# Yes: easy to match operators with operands
income = (gross_wages
          + taxable_interest
          + (dividends - qualified_dividends)
          - ira_deduction
          - student_loan_interest)
```

在Python中两种写法都是可以的，对于新的代码，推荐使用后者。

## 空行

使用2个空行来将最上层的函数(function)与类定义分开。

使用1个空行来分隔类中的方法(method)定义。

可以多使用一些空行（尽量少用这种方式）来将一些相关的方法进行分组。在一批相关的单行代码(比如一组dummy实现)之间的空行可以省略。

在函数中有节制地使用空行来将代码分块，来表明它们是不同的逻辑代码块。

## 源文件编码

核心的Python发行版中的代码总是应该使用UTF-8(Python 2中使用ASCII码)。

使用ASCII（Python 2）或者UTF-8（Python 3）的文件不应该添加编码声明。

## Imports

Imports应该分行写:

```python
Yes: import os
     import sys

No:  import sys, os
```

这样写也是可以的:

```python
from subprocess import Popen, PIPE
```

Imports应该写在代码文件的开头，位于模块(module)注释和文档字符串(docstring)之后，模块全局变量(globals)和常量(constants)声明之前。

Imports应该按照如下顺序进行分组:

1. 标准库的导入
2. 相关的第三方库的导入
3. 本地应用/库指定的import语句

不同分组的imports应该用1个空行将它们分开。

推荐使用绝对(absolute)的imports，因为这样通常更易读，在import系统没有正确配置（比如中的路径以`sys.path`结束）的情况下，也会有更好的表现（或者至少会给出错误信息）:

```python
import mypkg.sibling
from mypkg import sibling
from mypkg.sibling import example
```

相对于绝对import，显式的相对import也是一种可以接受的选择，特别是当处理复杂的包结构，而使用绝对import会显得特别繁琐和啰嗦的时候:

```python
from . import sibling
from .sibling import example
```

标准库代码应当一直使用绝对imports，避免复杂的包布局。

隐式的相对imports应该**永不**使用，并且Python 3中已经被去掉了。

当从某个包含类的模块中导入类时，通常可以这样写：

```python
from myclass import MyClass
from foo.bar.yourclass import YourClass
```

如果和本地命名的拼写产生了冲突，应当直接import模块:

```python
import myclass
import foo.bar.yourclass
```

然后使用`myclass.MyClass` 和 `foo.bar.yourclass.YourClass`

避免使用通配符imports(`from <module> import *`)，因为会造成在当前命名空间出现的命名含义不清晰，给读者和许多自动化工具造成困扰。

## 模块级的双下划线变量名

`__all__`, `__author__`, `__version__`等前后带双下划线的变量名，应该放在模块的文档字符串(docstring)之后，并在`import`语句之前(`from __future__`imports除外)。

Python规定future-imports必须出现在模块其他代码之前。

例子:

```python
"""This is the example module.

This module does stuff.
"""

from __future__ import barry_as_FLUFL

__all__ = ['a', 'b', 'c']
__version__ = '0.1'
__author__ = 'Cardinal Biggles'

import os
import sys
```

# 字符串引号

Python中单引号和双引号是一样的，PEP规范并不对此进行要求，只要选择一种方式并一直用下去就可以了。

对于三个引号的字符串，[PEP 257](https://www.python.org/dev/peps/pep-0257)中约定文档字符串(docstring)总是使用双引号。

# 表达式和语句中的空格

## 一些痛点

在下列情形中避免使用过多的空格:

- 圆括号、方括号和花括号之后

```python
Yes: spam(ham[1], {eggs: 2})
No:  spam( ham[ 1 ], { eggs: 2 } )
```

- 逗号、分号或冒号之前

```python
Yes: if x == 4: print x, y; x, y = y, x
No:  if x == 4 : print x , y ; x , y = y , x
```

- 在分片操作时，冒号和二元运算符是一样的，应该在其左右两边保留相同数量的空格（就像对待优先级最低的运算符一样）。在扩展的分片操作中，所有冒号的左右两边空格数都应该相等。不过也有例外，当切片操作中的参数被省略时，应该也忽略空格

```python
Yes:
ham[1:9], ham[1:9:3], ham[:9:3], ham[1::3], ham[1:9:]
ham[lower:upper], ham[lower:upper:], ham[lower::step]
ham[lower+offset : upper+offset]
ham[: upper_fn(x) : step_fn(x)], ham[:: step_fn(x)]
ham[lower + offset : upper + offset]

No:
ham[lower + offset:upper + offset]
ham[1: 9], ham[1 :9], ham[1:9 :3]
ham[lower : : upper]
ham[ : upper]
```

- 在调用函数时参数列表的括号之前

```python
Yes: spam(1)
No:  spam (1)
```

- 在索引和切片操作的左括号之前

```python
Yes: dct['key'] = lst[index]
No:  dct ['key'] = lst [index]
```

- 赋值(或其他)运算符周围使用多个空格来和其他语句对齐

```python
Yes:
x = 1
y = 2
long_variable = 3

No:
x             = 1
y             = 2
long_variable = 3
```

## 其他建议

- 避免在末尾出现空格
- 在二元运算符的两边都使用一个空格：赋值运算符(`=`)，增量赋值运算符(`+=`, `-=` etc.)，比较运算符(`==`, `<`, `>`, `!=`, `<>`, `<=`, `>=`, `in`, `not in`, `is`, `is not`)，布尔运算符(`and`, `or`, `not`)
- 如果使用了优先级不同的运算符，则在优先级较低的操作符周围增加空白

```python
Yes:
i = i + 1
submitted += 1
x = x*2 - 1
hypot2 = x*x + y*y
c = (a+b) * (a-b)

No:
i=i+1
submitted +=1
x = x * 2 - 1
hypot2 = x * x + y * y
c = (a + b) * (a - b)
```

- 使用`=`符号来表示关键字参数或默认参数值时，不要在其周围使用空格

```python
Yes:
def complex(real, imag=0.0):
    return magic(r=real, i=imag)

No:
def complex(real, imag = 0.0):
    return magic(r = real, i = imag)
```

- 在带注释的函数定义中需要在`=`符号周围加上空格。此外, 在`:`后使用一个空格，在`->`其两侧各使用一个空格

```python
Yes:
def munge(input: AnyStr): ...
def munge() -> AnyStr: ...
def munge(sep: AnyStr = None): ...
def munge(input: AnyStr, sep: AnyStr = None, limit=1000): ...
    
No:
def munge(input:AnyStr): ...
def munge()->PosInt: ...
def munge(input: AnyStr=None): ...
def munge(input: AnyStr, limit = 1000): ...
```

- 复合语句（即将多行语句写在一行）一般是不鼓励使用的

```python
Yes:
if foo == 'blah':
    do_blah_thing()
do_one()
do_two()
do_three()

Rather not:
if foo == 'blah': do_blah_thing()
do_one(); do_two(); do_three()
```

- 有时也可以将短小的if/for/while中的语句写在一行，但对于有多个分句的语句永远不要这样做，同时也要避免将多行都写在一起

```python
Rather not:
if foo == 'blah': do_blah_thing()
for x in lst: total += x
while t < 10: t = delay()
    
Definitely not:
if foo == 'blah': do_blah_thing()
else: do_non_blah_thing()

try: something()
finally: cleanup()

do_one(); do_two(); do_three(long, argument,
                             list, like, this)

if foo == 'blah': one(); two(); three()
```

# 注释

当代码有改动时，一定要优先更改注释使其保持最新。

注释应该是完整的多个句子。

如果注释很短，结束的句号可以被忽略。块注释通常由一段或几段完整的句子组成，每个句子都应该以句号结束。

你应该在句尾的句号后再加上2个空格。

请使用英文来写注释。

## 块注释

块注释一般写在对应代码之前，并且和对应代码有同样的缩进级别。块注释的每一行都应该以`#`和一个空格开头（除非该文本是在注释内缩进对齐的）。

块注释中的段落应该用只含有单个`#`的一行隔开。

## 行内注释

尽量少用行内注释。

行内注释是和代码语句写在一行内的注释。行内注释应该至少和代码语句之间有两个空格的间隔，并且以`#`和一个空格开始。

## 文档字符串

所有的公共模块，函数，类和方法都应该有文档字符串。对于非公共方法，文档字符串不是必要的，但你应该留有注释说明该方法的功能，该注释应当出现在`def`的下一行。

[PEP 257](https://www.python.org/dev/peps/pep-0257/)描述了好的文档字符应该遵循的规则。其中最重要的是，多行文档字符串以单行`"""`结尾，不能有其他字符，例如:

```python
      """Return a foobang
      
      Optional plotz says to frobnicate the bizbaz first.
      """
```

对于仅有一行的文档字符串，结尾处的`"""`应该也写在这一行:

```python
def kos_root():
    """Return the pathname of the KOS root directory."""
    global _kos_root
    if _kos_root: return _kos_root
    ...
```

## 版本注记

如果你必须在源代码中包含Subversion, CVS或RCS crud，请这样做:

```python
__version__ = "$Revision$"
# $Source$
```

以上几行的内容应当在模块的文档字符串之后，在其他代码之前，并且在其开始和结束都使用一个空行隔开。

# 命名约定

Python标准库的命名约定有一些混乱，因此我们永远都无法保持一致。但如今仍然存在一些推荐的命名标准。新的模块和包（包括第三方框架）应该采用这些标准，但若是已经存在的包有另一套风格的话，还是应当与原有的风格保持内部一致。

## 重写原则

对于用户可见的公共部分API，其命名应当表达出功能用途而不是其具体的实现细节。

## 描述性：命名风格

存在很多不同的命名风格，最好能够独立地从命名对象的用途认出采用了哪种命名风格。

以下是常用于区分的命名风格:

- `b` (单个小写字母)
- `B` (单个大写字母)
- `lowercase` (小写)
- `lower_case_with_underscores` (带下划线的小写)
- `UPPERCASE` (大写)
- `UPPER_CASE_WITH_UNDERSCORES` (带下划线的大写)
- `CapitalizedWords` (也叫CapWords或CamelCase，驼峰命名风格)

- `mixedCase` (和CapitalizedWords不同的是它以小写开头)
- `Capitalized_Words_With_Underscores` (丑陋的风格)

注意：当CapWords里包含缩写时，将缩写部分的字母都大写。`HTTPServerError`比`HttpServerError`要好。

此外，要区别以下划线开始或结尾的特殊形式（可以和其它的规则结合起来）:

- `_single_leading_underscore`: 以单个下划线开头是"内部使用"的弱标志。`from M import *`不会import下划线开头的对象。
- `single_trailing_underscore_`: 以单下划线结尾用来避免和Python的关键词冲突，例如`Tkinter.Toplevel(master, class_='ClassName')`
- `__double_leading_underscore`: 以双下划线开头的风格命名类属性表示触发命名修饰(在FooBar类中，`__boo`会被修饰成`_FooBar__boo`)
- `__double_leading_and_trailing_underscore__`: 以双下划线开头和结尾的命名风格表示生存在用户控制的命名空间里的“魔法”对象或属性。如: `__init__`, `__import__`或`__file__`。请依照文档说明使用这些命名，永远不要自己发明。

## 规范性：命名约定

### 需要避免的命名

不要使用字符`l`（小写的字母el），`O`（大写的字母oh），或者`I`（大写的字母eye）来作为单个字符的变量名。

在一些字体中，这些字符和数字1和0无法区别开来。当想使用`l`时，使用`L`代替。

### 包和模块命名

模块命名应短小，且为全小写。若下划线能提高可读性，也可以在模块名中使用。Python包命名也应该短小，且为全小写，但不应使用下划线。

模块名是对应到文件名的，一些文件系统会区分大小写并且会将长的文件名截断。因此模块名应该尽量短小，这个问题在Unix系统上是不存在的，但把代码移植到较旧的Mac，Windows版本或DOS系统上时，可能会出现问题。

当使用C或C++写的扩展模块有相应的Python模块提供更高级的接口时（e.g. 更加面向对象），C/C++模块名以下划线开头（例如`_sociket`）。

### 类命名

类命名应该使用单词字母大写（CapWords）的命名约定。

当接口已有文档说明且主要是被用作调用时，也可以使用函数的命名约定。

注意对于内建命名(builtin names)有一个特殊的约定：大部分内建名都是一个单词（或者两个一起使用的单词），单词首字母大写(CapWords)的约定只对异常命名和内建常量使用。

### 类型变量命名

类型变量(type variable)通常使用CapWords风格的短小命名: `T`, `AnyStr`, `Num`。建议在定义协变量(covariant)或逆变量(contravariant)时加上后缀`_co`或`_contra`:

```python
from typing import TypeVar

VT_co = TypeVar('VT_co', covariant=True)
KT_contra = TypeVar('KT_contra', contravariant=True)
```

### 异常命名

由于异常实际上也是类，因此类命名约定也适用与异常。不同的是，如果异常实际上是抛出错误的话，异常名前应该加上`Error`后缀。

### 全局变量命名

（在此之前，我们先假定这些变量都仅在同一个模块内使用。）这些约定同样也适用于函数命名。

对于引用方式设计为`from M import *`的模块，应该使用`__all__`机制来避免import全局变量，或者采用下划线前缀的旧约定来命名全局变量，从而表明这些变量是“模块非公开的”。

### 函数命名

函数命名应该都是小写，必要时使用下划线来提高可读性。

只有当已有代码风格已经是混合大小写时（比如threading.py），为了保留向后兼容性才使用混合大小写。

### 函数和方法参数

实例方法的第一参数永远都是`self`。

类方法的第一个参数永远都是`cls`。

在函数参数名和保留关键字冲突时，相对于使用缩写或拼写简化，使用以下划线结尾的命名一般更好。比如，`class_`比`clss`更好。（或许使用同义词避免这样的冲突是更好的方式。）

### 方法命名和实例变量

使用函数命名的规则：小写单词，必要时使用下划线分开以提高可读性。

仅对于非公开方法和变量命名在开头使用一个下划线。

避免和子类的命名冲突，使用两个下划线开头来触发Python的命名修饰机制。

Python类名的命名修饰规则：如果类Foo有一个属性叫`__a`，不能使用`Foo.__a`的方式访问该变量。（有用户可能仍然坚持使用`Foo._Foo__a`的方法访问。）一般来说，两个下划线开头的命名方法只应该用来避免设计为子类的属性中的命名冲突。

### 常量

常量通常是在模块级别定义的，使用全部大写并用下划线将单词分开。例如: `MAX_OVERFLOW` 和 `TOTAL`。

### 继承的设计

记得永远区别类的方法和实例变量（属性）应该是公开的还是非公开的。如果有疑虑的话，请选择非公开的；因为之后将非公开属性变为公开属性要容易些。

公开属性是那些和你希望和你定义的类无关的用户来使用的，并且确保不会出现向后不兼容的问题。非公开属性是那些不希望被第三方使用的部分，你可以不用保证非公开属性不会变化或被移除。

在这里没有使用`私有（private）`这个词，因为在Python里没有什么属性是真正私有的（这样设计省略了大量不必要的工作）。

另一类属性属于子类API的一部分（在其他语言中经常被称为”protected”）。一些类是为继承设计的，要么扩展要么修改类行为的部分。当设计这样的类时，需要谨慎明确地决定哪些属性是公开的，哪些属于子类API，哪些真的只会被你的基类调用。

请记住以上几点，下面是Python风格的指南:

- 公开属性不应该有开头下划线。

- 如果公开属性的名字和保留关键字有冲突，在你的属性名尾部加上一个下划线。这比采用缩写和简写更好。（然而，和这条规则冲突的是，`cls`对任何变量和参数来说都是一个更好地拼写，因为大家都知道这表示class，特别是在类方法的第一个参数里。）

- 对于简单的公共数据属性，最好仅公开属性名字，不要公开复杂的调用或设值方法。记住，Python提供了一条简单的途径来实现未来增强。这种情况下，使用properties将功能实现隐藏在简单数据属性访问语法之后。

  注意 1：Properties仅仅对新式类有用。

  注意 2：尽量保证功能行为没有副作用，尽管缓存这种副作用看上去并没有什么大问题。

  注意 3: 对计算量大的运算避免试用properties；属性的注解会让调用者相信访问的运算量是相对较小的。

- 如果你的类是子类的话，你有一些属性并不想让子类访问，考虑将他们命名为两个下划线开头并且结尾处没有下划线。这样会触发Python命名修饰算法，类名会被修饰添加到属性名中。这样可以避免属性命名冲突，以免子类在不经意间包含相同的命名。

  注意 1：注意命名修饰仅仅是简单地将类名加入到修饰名中，所以如果子类有相同的类名和属性名，你可能仍然会遇到命名冲突问题。

  注意 2：命名修饰可以有特定用途，比如在调试时，`__getattr__()`比较不方便。然而命名修饰算法的可以很好地记录，并且容意手动执行。

  注意3：不是所有人都喜欢命名修饰。尝试权衡避免意外的命名冲突和被高级调用者使用的潜在可能性。

## 公开和内部接口

任何向后兼容性保证仅对公开接口适用。相应地，用户能够清楚分辨公开接口和内部接口是很重要的。

文档化的接口被认为是公开的，除非文档中明确申明了它们是临时的或者内部接口，不保证向后兼容性。所有文档中未提到的接口应该被认为是内部的。

为了更好审视公开接口和内部接口，模块应该在`__all__`属性中明确申明公开API是哪些。将`__all__`设为空list表示该模块中没有公开API。

即使正确设置了`__all__`属性，内部接口（包，模块，类，函数，属性或其他命名）也应该以一个下划线开头。

如果接口的任意一个命名空间（包，模块或类）是内部的，那么该接口也应该是内部的。

导入的命名应该永远被认为是实现细节。其他模块不应当依赖这些非直接访问的导入命名，除非它们在文档中明确地被写为模块的API，例如那些从子模块暴露出来的功能`os.path`或者包的`__init__`模块。
