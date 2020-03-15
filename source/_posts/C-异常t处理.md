---
title: C++异常处理
date: 2018-01-15 13:05:28
tags:
  - Programming
  - C++
---

> 最近在写一个工程，结果因为在异常处理方面没有，最近把代码成功地变成了一堆垃圾，于是乎选择重新做人，拿出《C++ Primer》重学一下C++的异常处理

在看异常处理的时候，我主要想知道的就只有三个方面：

1. 异常产生后是如何被传递与匹配的
2. 处理异常会对原来的flow of control造成什么样的影响
3. 异常没有被处理会怎么样

**由于我看的是英文版的《C++ Primer》，一些名词我不知道该怎么翻译成中文名词，因此可能有些名词可能会直接用英文表达，郑重声明，这绝不是装逼**

<!-- more -->

### 异常的产生，传递与匹配

#### 产生

在C++中，想要抛出异常直接用`throw expression`即可，在抛出异常后，用catch进行捕捉，整体格式如下：

```C++
try {
  //expressions with possible exception behavior
}
catch(exception class/ exception object) {
  //exception handlers
}
catch() {
  
}
```

> tips: 当我们想要捕捉全部的异常时，可以使用catch(...)来进行所有异常的匹配

这样子，一旦try之中抛出了异常，就能用catch进行捕捉处理了。

但上述这种做法在类的构造函数初始化时却行不通，因为在构造函数之中, initializer是先于异常处理被执行的，也就是说，当我们在初始赋值的过程中发生的异常，用形式并没有办法进行检测，因此C++11又定义了一种叫做**function try block**的模式。如下：

```C++
template <typename T>
Blob<T>::Blob(std::initializer_list<T> il) try:
data(std::make_shared<std::vector<T>>(il)) {
  //expressions
} catch(const std::bad_alloc &e) { handle_out_of_memory(e);}
```

#### 传递

传递过程其实很好理解，对于嵌套的try代码块而言，执行过程为从内到外；至于整个程序的调用栈而言，自上到下，等最后到了main函数还是无法匹配时，调用`terminate`函数来结束整个程序。

值得一提的是，在异常处理的过程中，它会逐个释放变量的内存，这里我们可以把throw当作一个函数的return语句，当throw之后，该函数的全部临时变量都会被自动释放，对于类而言，会调用其析构函数进行内存释放，而对于我们通过new产生的动态内存，则需要我们手动释放，读到这我也大概能明白《C++ Primer》为什么这么推崇我们使用容器了，毕竟容器能够在一定程度上实现内存的自动管理。
**但这里书里也强调了，类的析构函数必须保证没有再抛出异常，或者其抛出的异常已经在析构函数内部就能被处理，否则会直接调用`terminate`函数来结束整个程序**

另一个要注意的地方就是如果我们抛出的异常值是一个指针的话，需要满足和函数返回值一样的限制条件，即不能是一个指向内部临时变量的指针。

> rethrow: 我们有时候可能会想要某个异常能够冒泡地被处理，如先被第一层的catch处理后，冒泡到上一层，	  再次进行处理，此时我们就可以在第一层对异常的值进行处理后，通过`throw`语句直接上抛异常。如:
>
> ```C++
> catch(my_error &obj) {
>   obj.status = errCodes:severeErr;
>   throw;
> }
> ```

#### 匹配

异常的匹配主要依据是依赖于异常的类型，因而在**catch**的匹配条件中，在不需要异常对象的情况下，我们完成可以只填写异常的类型，也能实现匹配。但如果我们需要throw的对象的话，就可以将该异常定义为一个对象，如果需要获得抛出对象的修改权的话，可以用指针或者引用来获取。

而另一个要关心的就是匹配的顺序，catch的匹配并不是按照重载那样按照类型的匹配程度来的，而是**按顺序**来进行匹配。因此如果我们返回的异常类型是一个多重继承的类的话，我们应当按照将catch按照继承的顺序反向排序，即让子类在前，父类在后。

同时，在匹配的过程中，除了下述三类的类型转换以外，其它的类型转换都会失效。

1. nonconst -> const
2. conversions from derived type to base type
3. An array -> a pointer to the type of the array; a function -> a pointer to the type of function

### 异常对flow of control的影响

当异常被抛出后，异常产生之后的代码都将不会被运行，而只有当异常被成功匹配并通过运行完相对应的catch的内部代码后，该catch并行的最后的一个catch结构之后的代码才会被执行。

### 没有被处理的异常

如果一个异常一直无法得到匹配的话，就会直接调用`terminate`函数来中止整个程序。

### 其它

当我们能够确认整个函数没有产生异常时，可以将其声明为`noexcept`，如：`void recoup(int) noexcept`。这里做能够缩小编译器编译出来的程序的体积。

> tips: 如果想要声明为`noexcept`，我们必须在函数每个声明和定义都加以声明。在成员函数时，应当处理const之后 ，override, final, =0之前

## 异常使用的一些技巧

> [资料来源](http://culttt.com/2014/04/09/use-exception/)

总而言之，以下三点：

1. 只在真正异常的环境下再使用异常，如网络连接断开，这是异常。用户名输错，这不是异常
2. 异常不是错误，有些程序本身就应当是通过返回true/false/null的
3. 对于外部问题我们可以考虑使用异常处理