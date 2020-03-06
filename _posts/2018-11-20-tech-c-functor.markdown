---
layout: post
title:  "c++中的function, functor和lambda表达式"
date:   2018-11-19 08:14:12 +0800
categories: 技术 
---

最近在读代码的时候，遇到之前没接触过的语法：类A重载了()运算符。在使用处，将该类的一个默认对象作为函数指针类型的参数传入某函数。在网上找资料研究了一番，才知道这是所谓的functor。现将学习到的东西整理如下。

## function 
函数，和函数指针是C里就有的概念，这里就不赘述。

## functor 
functor中文是仿函数，官方名称是function object，函数对象。
functor是STL六大组件（容器，算法，迭代器，仿函数，adapter，分配器）之一。并不是C++ 11引入的概念。

考虑如下情况，实现对数据的遍历并封装相关的处理函数。

```

#include <vector>
using std::vector;
#include <iostream>
using std::cout;
using std::endl;

class A {
 
 public: 
  A() { 
    content_.push_back(1);
    content_.push_back(2);
  }
  template<class Func>
  void for_each(Func func) {
    for (const auto& val : content_) {  
      func(val);
      
    }
  }

 private:
  vector<int> content_;
};

int print(int i) {
  cout << i << endl;
  return 0;
}

int main() {
  A inst;
  inst.for_each(print);

  return 0;
}


```
如果我们只是对数据结构中的元素做打印类似的简单处理，使用函数就可以实现类似功能。考虑，如果我们要对该数据结构A所有元素做累加。考虑上述实现，我们如果用函数来做的话，可能需要额外的全局变量用于缓存中间的累加结果。
如何比较优雅的实现该功能呢。这时候就可以使用functor。

**Functor就是定义了函数调用运算符的类对象，称作class type functor**。可以看如下示例。


```

...
  template<class Func>
  void for_each(Func& func) {
    for (const auto& val : content_) {
      
      func(val);
      
    }
  }
...

class SumFunctor {

 public:
  SumFunctor() { sum_ = 0; }
 public:
  int operator()(int i) {
    sum_ += i;
    return sum_;
  }

  int sum() { return sum_; };
 private:
  int sum_;
};

int main() {
  A inst;

  SumFunctor sum_functor; 
  inst.for_each(sum_functor);
  cout<< sum_functor.sum() << endl;
  return 0;
}

```

**注意：新例中，A的for_each函数，参数改成了引用。** 函数指针的话，只是值传递就可以，但如果要在for_each调用后，还想使用sum_functor对象的结果，那必须使用引用。

在STL中常用的实现和这里有些区别，一般的，他会将传入的Functor再次返回。可以修改为如下方式
```
...
  template<class Func>
  Func for_each(Func func) {
    for (const auto& val : content_) {      
      func(val);      
    }
    return func;
  }
...

int main() {
  A inst;

  SumFunctor sum_functor = inst.for_each(SumFunctor());
  cout<< sum_functor.sum() << endl;
  return 0;
}

```


## lambda表达式
lambda表达式的官方名称是anonymous function，匿名函数，又称为function literal, lambda abstraction, lambda expression。

根据网上一篇文章描述，编译器实际上将lambda函数转化为一个函数对象，所以，它与使用函数对象是一样高效的。

他的语法规则如下。

[ captures ] <tparams>(可选)(C++20) ( params ) specifiers(可选) exception attr -> ret requires(可选)(C++20) { body }

我们先看比较简单的一个使用形式。

[ captures ]  ( params ) -> ret{ body }

该语法需要解释两点，1 匿名函数的返回值使用 -> ret_type来定义。 2 匿名函数开始的[]指定了需要捕获的对象。

Functor把需要保存的中间变量，或者依赖的外部变量保存在类内。而lambda表达式则是通过capture的方式将这些变量直接捕获到函数体内。

具体引用的使用规则，引用其他文章的说明如下
```
[&] 表示通过引用捕获引用的所有变量，而 [=] 表示通过值捕获它们。 
可以使用默认捕获模式，然后为特定变量显式指定相反的模式。
[&total, factor]  
[factor, &total]  
[&, factor]  
[factor, &]  
[=, &total]  
[&total, =]
```
参见下面的代码

```
...
  template<class Func>
  Func for_each(Func func) {
    for (const auto& val : content_) {      
      func(val);      
    }
    return func;
  }
...

int main() {
  A inst;

  int total = 0;
  inst.for_each( [&total](int i)->int{total += i; return total;} );
  cout<< total << endl;
  return 0;
}

```

这里要说的是， for_each这里必须用非引用的参数传递方式。 否则会编译报错。

## 效率
根据网上文章的说法，functor方式要比function的方式效率要高。


## std::function, std::bind
这里顺带说下stl中function和bind功能。


在需要使用函数指针的地方，我们有的时候保存某个具体对象的非静态成员函数。这个时候C++语言本身是没有提供这种机制的。
这时我们需要使用std::function，声明一个这类函数指针的类型，模版里是一个函数指针类型。

std::function<int(int a, int b)>;

在给该类型的变量赋值的时候使用如下语句

std::bind(&A::a_member_func, this, std::placeholders::_1, std::placeholders::_2);