## 前言

在STL编程中，容器和算法是独立设计的，即数据结构和算法是独立设计的，连接容器和算法的桥梁就是迭代器了，迭代器使其独立设计成为可能。Traits编程技术是STL中最重要的编程技术，Traits可以获取一个类型的相关信息。在学习《STL源码剖析》时，看到关于这方面的知识，在这期间查找了一些资料，下面是我对该技术的理解。

## Traits编程技术

Traits可以获取一个类型的相关信息，首先我们看下面的程序：

```cpp
template <class T, class type>
void function(T t, type u) {
    type temp; 
    // ... The rest work of function…
}
```

在上面的程序中，我们怎么样才能获得已声明变量temp的类型type呢？在模板中，模板参数推导机制可以解决这个问题，在下面程序中编译器直到变量temp的类型为int：

```cpp
template <class T, class type>
void function(T iter, type u) 
{
    type temp; // 通过模板参数推导获得temp变量的类型
    ....
}
template <class T>
void func(T iter) 
{
    function(iter, *iter); 
}

int main()
{
	int i = 12;
	func(&i)
}
```

上面介绍的是获得局部变量的类型，这个可以通过模板参数推导机制完成；如果我们要获得函数返回值的类型时，该怎么处理呢？这个问题针对不同类型有不同的方法可以解决，类型型别：用户自定义类型（如结构体，类）和内置类型（如整型，原生指针等）。

若我们要知道用户自定义类型的函数返回值类型，我们可以使用内嵌型别技术就可以知道返回值的类型；看下面程序：

```cpp
templates <class T>
struct Iterator 
{
typedef T value_type;//内嵌型别声明
...
};

template <class Iterator>
typename Iterator::value_type //返回值类型
GetValue(Iterator iter) 
{
	return *iter;
}

int main()
{
	...
	Iterator<int> ite(new int(9));
	std::cout<<GetValue(ite)<<std::endl;
	return 0;
}
```

在用户自定义类型中我们可以通过内嵌型别获得返回值的类型，若不是用户自定义的类型，而是内置类型时，例如是原生指针，这时候该怎么处理呢？因为原生指针不能内嵌型别声明，所以内嵌型别在这里不适用。于是Traits技术就出现了。以下利用Traits技术也可以获取用户自定义类型的返回值类型。

```cpp
template <class Iterator>
struct iterator_traits {
  typedef typename Iterator::value_type    value_type;
  ...
};

template <class Iterator>
typename iterator_traits<Iterator>::value_type //返回值类型
GetValue(Iterator iter) 
{
	return *iter;
}
```

以上这种方法对原生指针不可行，所以`iterator_traits`针对原生指针的一个版本就应运而生。下面是针对`Tp`和`const Tp`的版本，也称为`iterator_traits`版本的偏特化。`iterator_traits`的偏特化版本解决了原生指针的问题。现在不管迭代器是自定义类模板, 还是原生指针`(Tp, const Tp)`,`struct iterator_traits`都能萃取出正确的`value type`类型。

```cpp
template <class Tp>
struct iterator_traits<Tp*> {
  typedef Tp       value_type;
  ...
};

template <class Tp>
struct iterator_traits<const Tp*> {
  typedef Tp       value_type;
  ...
};
```

我们之所以要萃取（Traits）迭代器相关的类型，就是要把迭代器相关的类型用于声明局部变量、用作函数的返回值等一系列行为。对于原生指针和point-to-const类型的指针，采用模板偏特化技术对其进行特殊处理。要使用Traits功能，则必须自行以内嵌型别定义的方式定义出相应型别。我们上面讲解的只是迭代器其中一种类型value type，在迭代器中还有其他类型：

```cpp
template <class _Iterator>
struct iterator_traits {
  typedef typename _Iterator::iterator_category iterator_category; //迭代器类型
  typedef typename _Iterator::value_type        value_type;		//迭代器所指对象的类型
  typedef typename _Iterator::difference_type   difference_type;//迭代器之间距离
  typedef typename _Iterator::pointer           pointer;		//迭代器所指之物
  typedef typename _Iterator::reference         reference;      //迭代器引用之物
};
```

## 参考文献

* [萃取traits编程技术的介绍和应用](http://www.searchtb.com/2014/03/%E8%90%83%E5%8F%96traits%E7%BC%96%E7%A8%8B%E6%8A%80%E6%9C%AF%E7%9A%84%E4%BB%8B%E7%BB%8D%E5%92%8C%E5%BA%94%E7%94%A8.html)
* [C++的类型萃取技术](http://www.cppblog.com/nacci/archive/2005/11/03/911.aspx)
* [Traits 编程技法+模板偏特化+template参数推导+内嵌型别编程技巧](http://www.cnblogs.com/kanego/archive/2012/08/15/2639761.html)
* [An Introduction to "Iterator Traits"](http://www.codeproject.com/Articles/36530/An-Introduction-to-Iterator-Traits)
