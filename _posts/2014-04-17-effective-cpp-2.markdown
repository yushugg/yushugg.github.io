---
layout: post
title: "effective cpp learning part 2"
date: 2014-04-17
categories: corcpp
tags: corcpp
---

### Item 05: Know what functions C++ silently writes and calls

如果自己没有声明，编译器会声明一个copy构造函数、一个copy assignment操作符、一个析构函数;如果没有任何构造函数，会声明一个default构造函数。所有这些函数都是public且inline

{% highlight c++ linenos %}
class Empty { };
{% endhighlight %}

当这些函数被需要调用时，才会被编译器创建出来，等价于

{% highlight c++ linenos %}
class Empty {
public:
    Empty() { ... }
    Empty(const Empty& rhs) { ... }
    ~Empty() { ... }

    Empty& operator=(const Empty& rhs) { ... }
};
{% endhighlight %}

当已经自定义了一个构造函数，编译器就不再创建default构造函数  
C++不允许让reference改变指向不同对象，即一个reference永远绑定在了初始化的对象上

### Item 06: Explicitly disallow the use of compiler-generated functions you do not want

可以将copy构造函数或者copy assignment操作符声明为private，进而阻止他人调用;还要将这两者都不进行定义，**只有声明，没有定义**

{% highlight c++ linenos %}
class HomeForSale {
public:
    ...
private:
    ...
    HomeForSale(const HomeForSale&);//只有声明
    HomeForSale& operator=(const HomeForSale&);
};
{% endhighlight %}

如此，如果用户拷贝HomeForSale对象，编译器会阻止;如果不慎在member函数和friend函数内调用，则连接器会阻止  
如果要将连接期的错误移动到编译期，可以设计一个base class，然后将两者放在private里面，然后继承其即可，不一定得以public继承它，而且base class的析构函数不一定是virtual，不包含数据

### Item 07: Declare destructors virtual in polymorphic base classes

给带有多态性的base class一个virtual析构函数，如此删除base指针时，会删除derived class对象，否则只会调用base class的析构函数  
作为base class往往会有virtual函数，而任何只要带有virtual函数都几乎确定应该有一个virtual析构函数;如果class不含virtual函数，通常表示它并不意图被用做一个base class，当不用作base class时，不应该将析构函数声明为virtual，因为虚函数表占用空间，且不具有可移植性  
标准STL容器和string不含任何virtual函数，将STL容器和string作为base class要小心，STL容器和string都不被设计为base class使用  
pure virtual函数，导致abstract class，不能被实例化

### Item 08: Prevent exceptions from leaving destructions

