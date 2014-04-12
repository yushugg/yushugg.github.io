---
layout: post
title: "effective cpp learning part 1"
date: 2014-04-12
categories: [update, corcpp]
tags: corcpp
---

###Item 01: View C++ as a federation of languages
<br />
###Item 02: Prefer consts, enums, and inlines to #defines
####尽量使用const，enum, inline而不是define
* 对于单纯常量，最好以`const`对象或者`enums`替换`#define`
* 对于形似函数的宏，最好使用`inline`函数替换`#define`

在`class`的头文件里，声明一个`class`，`class`的`变量`都是声明，没有定义，定义写在实现文件中（cpp文件）

**只有static const integral data members can be initialized，才可以被赋值，但是仍然不是定义，是声明，定义在实现文件中**

在类里面：
{% highlight c++ linenos %}
class GamePlayer {
private:
    static const int NumTurns = 5;
    int scores[NumTurns];
};
{% endhighlight %}
如果编译器不允许，需要在cpp文件中加入：
{% highlight c++ linenos %}
const in GamePlayer::NumTurns;
{% endhighlight %}
如果编译器不允许，可以将类声明里面不赋值，而在cpp文件里加入定义

取一个`const`的地址是合法的，但是取一个`enum`的地址不合法，取`define`的地址也不合法

可以用`enum`代替`static const int`
{% highlight c++ linenos %}
class GamePlayer {
private:
    enum { NumTurns = 5 };
    int scores[NumTurns];
};
{% endhighlight %}

###Item 03: Use const whenever possible
\*左边的是所指物是`const`，\*右边是指针本身`const`
{% highlight c++ linenos %}
const char* const p;
char const* const p;
{% endhighlight %}

####对于迭代器：
1. 迭代器类似`T*`，所以迭代器为`const`，表明指针是`const`，即`T* const`，指针不变，但是指针所指的东西可以改变
2. 如果要所指向的东西不变，需用`const_iterator`
3. 对函数返回值保持`const`，可以防止赋值操作的失误：
{% highlight c++ linenos %}
const Rational operator* (const Rational& lhs, const Rational& rhs);
{% endhighlight %}
可以防止 `if (a*b = c)...` 防止结果写反，立即被赋值

####成员函数const：
1. 保证改不改变类的成员内容，不可以更改对象内任何`non-static`成员变量，但是如果成员内容添加了`mutable`，则这些可以被更改
2. 使得`const`对象可以调用函数，`const`对象只能调用`const`函数，如参数写的`const Rational& xx`，在该函数中，`xx`就可以调用`const`函数了

`const`对象可以读取成员变量，但是不能改变它们，所以读取是没问题的

为了不写两份代码（一份是`const`成员，一份不是），可以调用`const_cast<char&>`将`const char&`转换为`char&`，消除`const`；调用`static_cast<const TextBlock&>(*this)[position]`将非`const`的参数改为`const`类型的参数。<>里面的是转成的结果利用`const`版本写出非`const`版本，先写`const`，再复用产生非`const`，反之不好，因为`const`成员函数绝不改动对象成员
