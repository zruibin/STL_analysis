## 前言

queue是一种“先进先出”的数据结构，可以对两端进行操作，但是只能在队列头部进行移除元素，只能在队列尾部新增元素，可以访问队列尾部和头部的元素，但是不能遍历容器，所以queue不需要设计自己的容器。在SGI STL的源码`<stl_queue.h>`的class queue设计中，它是基于某种容器作为底部结构的，默认容器是deque容器，用户也可以自己指定容器的类型。

## queue容器配接器

源码剖析如下：

```cpp
#ifndef __SGI_STL_INTERNAL_QUEUE_H
#define __SGI_STL_INTERNAL_QUEUE_H

#include <sequence_concepts.h>

__STL_BEGIN_NAMESPACE

// Forward declarations of operators < and ==, needed for friend declaration.
//默认底层容器为deque容器
template <class _Tp, 
          class _Sequence __STL_DEPENDENT_DEFAULT_TMPL(deque<_Tp>) >
class queue;

template <class _Tp, class _Seq>
inline bool operator==(const queue<_Tp, _Seq>&, const queue<_Tp, _Seq>&);

template <class _Tp, class _Seq>
inline bool operator<(const queue<_Tp, _Seq>&, const queue<_Tp, _Seq>&);

template <class _Tp, class _Sequence>
class queue {

  // requirements:

  __STL_CLASS_REQUIRES(_Tp, _Assignable);
  __STL_CLASS_REQUIRES(_Sequence, _FrontInsertionSequence);
  __STL_CLASS_REQUIRES(_Sequence, _BackInsertionSequence);
  typedef typename _Sequence::value_type _Sequence_value_type;
  __STL_CLASS_REQUIRES_SAME_TYPE(_Tp, _Sequence_value_type);

#ifdef __STL_MEMBER_TEMPLATES 
  template <class _Tp1, class _Seq1>
  friend bool operator== (const queue<_Tp1, _Seq1>&,
                          const queue<_Tp1, _Seq1>&);
  template <class _Tp1, class _Seq1>
  friend bool operator< (const queue<_Tp1, _Seq1>&,
                         const queue<_Tp1, _Seq1>&);
#else /* __STL_MEMBER_TEMPLATES */
  friend bool __STD_QUALIFIER
  operator== __STL_NULL_TMPL_ARGS (const queue&, const queue&);
  friend bool __STD_QUALIFIER
  operator<  __STL_NULL_TMPL_ARGS (const queue&, const queue&);
#endif /* __STL_MEMBER_TEMPLATES */

public:
	// queue仅支持对头部和尾部的操作, 所以不定义STL要求的  
  // pointer, iterator, difference_type 
  typedef typename _Sequence::value_type      value_type;
  typedef typename _Sequence::size_type       size_type;
  typedef          _Sequence                  container_type;

  typedef typename _Sequence::reference       reference;
  typedef typename _Sequence::const_reference const_reference;
protected:
  _Sequence c;//底层容器，默认为deque容器，用户可自行指定容器类型
public:
	//下面对queue的维护完全依赖于底层容器的操作 
  queue() : c() {}
  explicit queue(const _Sequence& __c) : c(__c) {}

  //判断容器是否为空
  bool empty() const { return c.empty(); }
  //返回容器中元素的个数
  size_type size() const { return c.size(); }
  //返回队头元素的引用
  reference front() { return c.front(); }
  const_reference front() const { return c.front(); }
  //返回队尾元素的引用
  reference back() { return c.back(); }
  const_reference back() const { return c.back(); }
  //只能在队尾新增元素
  void push(const value_type& __x) { c.push_back(__x); }
  //只能在队头移除元素
  void pop() { c.pop_front(); }
};

//下面是依赖于底层容器的操作运算符  
template <class _Tp, class _Sequence>
bool 
operator==(const queue<_Tp, _Sequence>& __x, const queue<_Tp, _Sequence>& __y)
{
  return __x.c == __y.c;
}

template <class _Tp, class _Sequence>
bool
operator<(const queue<_Tp, _Sequence>& __x, const queue<_Tp, _Sequence>& __y)
{
  return __x.c < __y.c;
}

#ifdef __STL_FUNCTION_TMPL_PARTIAL_ORDER

template <class _Tp, class _Sequence>
bool
operator!=(const queue<_Tp, _Sequence>& __x, const queue<_Tp, _Sequence>& __y)
{
  return !(__x == __y);
}

template <class _Tp, class _Sequence>
bool 
operator>(const queue<_Tp, _Sequence>& __x, const queue<_Tp, _Sequence>& __y)
{
  return __y < __x;
}

template <class _Tp, class _Sequence>
bool 
operator<=(const queue<_Tp, _Sequence>& __x, const queue<_Tp, _Sequence>& __y)
{
  return !(__y < __x);
}

template <class _Tp, class _Sequence>
bool 
operator>=(const queue<_Tp, _Sequence>& __x, const queue<_Tp, _Sequence>& __y)
{
  return !(__x < __y);
}

#endif /* __STL_FUNCTION_TMPL_PARTIAL_ORDER */
```

下面举例说明：

```cpp
// constructing queues
#include <iostream>       // std::cout
#include <deque>          // std::deque
#include <list>           // std::list
#include <queue>          // std::queue

int main ()
{
  std::deque<int> mydeck (3,100);        // deque with 3 elements
  std::list<int> mylist (2,200);         // list with 2 elements

  std::queue<int> first;                 // empty queue
  std::queue<int> second (mydeck);       // queue initialized to copy of deque

  std::queue<int,std::list<int> > third; // empty queue with list as underlying container
  std::queue<int,std::list<int> > fourth (mylist);

  std::cout << "size of first: " << first.size() << '\n';
  std::cout << "size of second: " << second.size() << '\n';
  std::cout << "size of third: " << third.size() << '\n';
  std::cout << "size of fourth: " << fourth.size() << '\n';
  std::cout << "The element at the front of queue second is: "
        << second.front( )  << std::endl;
  second.push(10);
  std::cout << "The element at the back of queue second is: "
        << second.back( ) << std::endl;

  return 0;
}

Output:
size of first: 0
size of second: 3
size of third: 0
size of fourth: 2
The element at the front of queue second is: 100
The element at the back of queue second is: 10
```

## 参考资料

* 《STL源码剖析》侯捷