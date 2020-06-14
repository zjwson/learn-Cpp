# 深入理解 include guard 和 #pragma once

声明：本文来自转载文章

作者：tristan

转载地址：https://toutiao.io/posts/0d138s/preview



C++中头文件一般使用#include guard来避免重复包含, 而#pragma once也是(所有?)流行编译器所支持的预处理指令, 这里深入解释一下两者的来龙去脉.(注: 编译器这里仅讨论GCC)



## 1. One Definition Rule

include guard 的存在归根结底是为了遵循[One Definition Rule](https://en.wikipedia.org/wiki/One_Definition_Rule), 也就是说在一个编译单元(translation unit)中某个类型(或对象,函数)不能有多个定义.

下面的代码无法编译通过:

```c++
class foo {}; // first definition 
class foo {}; // error!!!
```

通常把class foo的定义放到一个头文件中, 如果一个编译单元中多次include这个头文件, 就会编译失败:

```c++
// File foo.h
class foo { int i; };
// File "a.h"
#include "foo.h"
// File "b.h"
#include "foo.h"
// File "bar.c"
#include "a.h"
#include "b.h"
```

\#include guard就是为了保证 class foo的定义被编译一次:

```c++
// File foo.h
#ifndef FOO_H
#define FOO_H

class foo { int i; };

#endif
```

这样在bar.c中第一次include文件会定义宏FOO_H, 第二次include就避免了再次处理class foo的定义.



## 2. #include guard 及编译器优化

> The hardest problem in programming is what to name your variables and functions.

\#include guard最大的问题也是命名问题, 项目中所有的(包含第三方的)头文件所使用的guard 宏必须没有冲突, 严格来说, 这需要是一个[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier).

实际项目中的命名方式有很多, 只要项目可以保证避免冲突, 用哪种没多大区别:

- FOO_H
- FOO_H_INCLUDED
- PATH_TO_FOO_H (boost使用该方式)
- FOO_H__NsknZfLkajnTFBpHIhKS (随机字符串)

需要注意的是, underscore开头的方式(_FOO_H /__FOO_H)是不推荐使用的, 因为C/C++标准中规定这些特殊名称是保留给编译器的,不应该由用户定义.

如果一个头文件中使用了include guard并且在一个编译单元中被多次include, 编译器需要多次处理该文件, 但在第一次之后,遇到已定义的guard宏时跳过其中的内容. 但是, 每次处理文件仍然会涉及到多个系统调用开销(open, read), 所以编译器会进行优化处理.

gcc支持[include guard optimisation](https://gcc.gnu.org/onlinedocs/cppinternals/Guard-Macros.html), 其原理很简单, 编译器需要识别include guard, 然后避免多次打开同一个文件. 具体来说, 能进行include guard optimisation的头文件需要满足一些条件:

- \#if/#endif块之外没有除空格和注释之外的内容(编译器会strip掉空格和注释)
- \#if/#endif块之内不允许有#else或者#elif指令块



## 3. #pragma once

\#pragma once 是non-standard,但被多数主流编译器支持的预处理指令. 与#include guard相比, 它具有很多优点, 比如更简洁的代码, 避免了guard宏的命名问题, 提升了编译速度(但是前面提到的multiple include优化使编译速度的差异消失)等等.

GCC对#pragma once指令的支持历史比较有趣, Ian Lance Taylor大牛2006年在GCC mailing list中的[post](https://gcc.gnu.org/ml/gcc/2006-08/msg00227.html)介绍了, 故事是这样的:

> gcc是最早支持#pragma once的编译器, 早在1989年RMS就在gcc 1.35 中实现了它. 后来Microsoft 编译器也支持该指令, 所以很多人认为这是一个MS的扩展功能.
>
> 在1991年, RMS在gcc2.0中实现了前面提到的 **include guard optimisation**, 所以他老人家觉得没有必要再继续支持#pragma once, 就对这个指令发出一个obsolete的warning, 但是不会编译失败.
>
> 但是, 程序员们发现#pragma once在这些年里已经被多数主流编译器所支持了,基本上是个de-facto standard, 所以gcc 3.4中又正式支持(undeprecated)#pragma once.
>
> 

## 4. so what?

那么到底用哪一个呢? 如果是已有项目, 当然是follow其规范. 新项目推荐使用#pragma once, **it’s neat!**