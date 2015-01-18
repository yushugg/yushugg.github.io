---
layout: post
title: "effective cpp learning part 1"
date: 2014-04-12
categories: corcpp
tags: corcpp
---

### Item 01: View C++ as a federation of languages

1. C
2. Object-Oriented C++
3. Template C++
4. STL

### Item 02: Prefer consts, enums, and inlines to #defines

1. 对于单纯常量，最好以const对象或者enums替换#define
2. 对于形似函数的宏，最好使用inline函数替换#define

在class的头文件里，声明一个class，class的变量都是声明，没有定义，定义写在实现文件中（cpp文件）  
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
取一个const的地址是合法的，但是取一个enum的地址不合法，取define的地址也不合法  
可以用enum代替static const int

{% highlight c++ linenos %}
class GamePlayer {
private:
    enum { NumTurns = 5 };
    int scores[NumTurns];
};
{% endhighlight %}

### Item 03: Use const whenever possible

\*左边的是所指物是const，\*右边是指针本身const

{% highlight c++ linenos %}
const char* const p;
char const* const p;
{% endhighlight %}

#### 对于迭代器：

1. 迭代器类似T\*，所以迭代器为const，表明指针是const，即T\* const，指针不变，但是指针所指的东西可以改变
2. 如果要所指向的东西不变，需用const\_iterator
3. 对函数返回值保持const，可以防止赋值操作的失误：

{% highlight c++ linenos %}
const Rational operator* (const Rational& lhs, const Rational& rhs);
{% endhighlight %}

可以防止 if (a\*b = c)... 防止结果写反，立即被赋值

#### 成员函数const：

1. 保证改不改变类的成员内容，不可以更改对象内任何non-static成员变量，但是如果成员内容添加了mutable，则这些可以被更改
2. 使得const对象可以调用函数，const对象只能调用const**函数**，如参数写的const Rational& xx，在该函数中，xx就可以调用const**函数**了

const对象可以读取成员**变量**，但是不能改变它们，所以读取是没问题的  
为了不写两份代码（一份是const成员，一份不是），可以调用const\_cast\<char&\>将const char&转换为char&，消除const；如，在重载运算符[]的操作中，调用static\_cast\<const TextBlock&\>(\*this)[position]将非const的参数改为const类型的参数。<>里面的是转成的结果利用const版本写出非const版本，先写const，再复用产生非const，反之不好，因为const成员函数绝不改动对象成员

### Item 04: Make sure that objects are initialized before they're used

永远在使用对象之前先将它初始化  
确保每一个构造函数都将对象的每一个成员初始化

#### 成员初值列

构造函数中，使用member initialization list替换赋值动作，效率更高，基于赋值的构造函数首先调用default构造函数为成员设初值，然后立刻再对它们赋予新值，而使用成员初值列的做法，初值列中针对各个成员变量而设的实参被拿去作为各个成员变量的构造函数的实参，少了调用default构造这个过程，只有copy构造，两者都有copy构造这一过程  
内置类型，如int，double等无差别，但为了一致性，最好也写成如此  
有些情况，内置类型也一定要使用初值列，如果成员变量是const或references，它们不能被赋值，一定要初值    
所以最好将所有的成员都列出来，避免遗漏，总是使用成员初值列  
甚至当想要default构造一个成员变量，可以使用成员初值列，指定无物作为初始化实参即可，无参数构造函数

{% highlight c++ linenos %}
ABEntry::ABEntry()
    :theName(),
    theAddress(),
    thePhones(),
    numTimesConsulted(0)
{ }
{% endhighlight %}

当class有多个构造函数，每个函数都有初值列时，则会产生重复，可以合理地在初值列中遗漏那些“赋值表现像初始化一样好”的成员变量，改用它们的赋值操作，并将那些赋值操作移往某个函数（通常是private），供所有的构造函数调用；但成员初值列往往更加可取  
成员变量初始化次序是按照声明的次序，即使成员初值列以不同的次序出现，所以最好按照和声明的顺序一致初始值列表  

#### 不同编译单元内定义的non-local static对象

C++对“定义于不同编译单元内的non-local static对象”的初始化次序并无明确定义，多个编译单元内的non-local static对象经由“模板隐式具现化”形成  
消除方法是，将每个non-local static对象搬到自己的专属函数内，返回一个reference指向它所含的对象，然后用户调用这些函数，而不直接涉及这些对象，换句话说，non-local static被local static对象替换了  
local static只会在该函数被调用期间或者首次遇上该对象定义式时被初始化

{% highlight c++ linenos %}
FileSystem& tfs()
{
    static FileSystem fs;
    return fs;
}
// 调用
std::size_t disks = tfs().numDisks();
{% endhighlight %}


