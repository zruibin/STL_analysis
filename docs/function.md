## 前言

在STL中，函数对象也是比较重要的，有时候可以限定STL算法的行为，例如在前面介绍的[《STL算法剖析》](http://blog.csdn.net/chenhanzhun/article/details/39698523)中，每个算法基本上都提供了两个操作版本，其中就有一个版本允许用户指定函数对象，这样可以根据用户的需要对算法进行操作。函数对象是一种具有函数特质的对象，所以可以作为算法的参数。本文介绍的函数对象比较简单，是基于一元或者二元操作结构的算术类函数对象、关系运算类函数对象、逻辑运算类函数对象。在定义函数对象时，为了使其具有函数行为，则必须重载operator()操作符。本文源码出自SGI STL中的`<stl_function.h>`文件。

## 函数对象源码剖析

```cpp
_STL_BEGIN_NAMESPACE
//一元操作结构定义
template <class _Arg, class _Result>
struct unary_function {
  typedef _Arg argument_type;//参数类型
  typedef _Result result_type;//返回结果类型
};

//二元操作结构定义
template <class _Arg1, class _Arg2, class _Result>
struct binary_function {
  typedef _Arg1 first_argument_type;//参数一类型
  typedef _Arg2 second_argument_type;//参数二类型
  typedef _Result result_type;//返回结果类型
};

//以下是二元操作算术函数对象，继承二元操作binary_function的结构
/*
加法操作plus<T>,减法操作minus<T>,乘法操作multiplies<T>,除法操作divides<T>,
取模运算modulus<T>,
*/
template <class _Tp>
struct plus : public binary_function<_Tp,_Tp,_Tp> {
  _Tp operator()(const _Tp& __x, const _Tp& __y) const { return __x + __y; }
};

template <class _Tp>
struct minus : public binary_function<_Tp,_Tp,_Tp> {
  _Tp operator()(const _Tp& __x, const _Tp& __y) const { return __x - __y; }
};

template <class _Tp>
struct multiplies : public binary_function<_Tp,_Tp,_Tp> {
  _Tp operator()(const _Tp& __x, const _Tp& __y) const { return __x * __y; }
};

template <class _Tp>
struct divides : public binary_function<_Tp,_Tp,_Tp> {
  _Tp operator()(const _Tp& __x, const _Tp& __y) const { return __x / __y; }
};

template <class _Tp>
struct modulus : public binary_function<_Tp,_Tp,_Tp>
{
  _Tp operator()(const _Tp& __x, const _Tp& __y) const { return __x % __y; }
};

//一元操作，继承一元操作unary_function结构
//负值操作negate<T>
template <class _Tp>
struct negate : public unary_function<_Tp,_Tp>
{
  _Tp operator()(const _Tp& __x) const { return -__x; }
};

// identity_element (not part of the C++ standard).
//证同元素：
//以下只提供的两种证同元素
//加法：任何元素加上0结果都为自身
//乘法：任何元素乘以1结果都为自身
template <class _Tp> inline _Tp identity_element(plus<_Tp>) {
  return _Tp(0);
}
template <class _Tp> inline _Tp identity_element(multiplies<_Tp>) {
  return _Tp(1);
}

//以下是二元操作关系函数对象，继承二元操作的结构
/*
返回值的类型是bool型别
equal_to,not_equal_to,greater,less,greater_equal,less_equal,
*/
template <class _Tp>
struct equal_to : public binary_function<_Tp,_Tp,bool>
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x == __y; }
};

template <class _Tp>
struct not_equal_to : public binary_function<_Tp,_Tp,bool>
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x != __y; }
};

template <class _Tp>
struct greater : public binary_function<_Tp,_Tp,bool>
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x > __y; }
};

template <class _Tp>
struct less : public binary_function<_Tp,_Tp,bool>
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x < __y; }
};

template <class _Tp>
struct greater_equal : public binary_function<_Tp,_Tp,bool>
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x >= __y; }
};

template <class _Tp>
struct less_equal : public binary_function<_Tp,_Tp,bool>
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x <= __y; }
};

//以下是二元操作逻辑函数对象，继承二元操作的结构
/*
其中logical_not为一元操作函数
logical_and,logical_or,logical_not
*/
template <class _Tp>
struct logical_and : public binary_function<_Tp,_Tp,bool>
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x && __y; }
};

template <class _Tp>
struct logical_or : public binary_function<_Tp,_Tp,bool>
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x || __y; }
};

template <class _Tp>
struct logical_not : public unary_function<_Tp,bool>
{
  bool operator()(const _Tp& __x) const { return !__x; }
};

// identity is an extensions: it is not part of the standard.
//证同函数
//任何数值通过此函数后，不会有任何修改。所以返回值类型为const引用
template <class _Tp>
struct _Identity : public unary_function<_Tp,_Tp> {
  const _Tp& operator()(const _Tp& __x) const { return __x; }
};

template <class _Tp> struct identity : public _Identity<_Tp> {};

// select1st and select2nd are extensions: they are not part of the standard.
//选择函数
//版本一：选择pair元素的第一个参数，忽略第二个参数
template <class _Pair>
struct _Select1st : public unary_function<_Pair, typename _Pair::first_type> {
  const typename _Pair::first_type& operator()(const _Pair& __x) const {
    return __x.first;
  }
};
//版本二：选择pair元素的第二个参数，忽略第一个参数
template <class _Pair>
struct _Select2nd : public unary_function<_Pair, typename _Pair::second_type>
{
  const typename _Pair::second_type& operator()(const _Pair& __x) const {
    return __x.second;
  }
};

template <class _Pair> struct select1st : public _Select1st<_Pair> {};
template <class _Pair> struct select2nd : public _Select2nd<_Pair> {};

// project1st and project2nd are extensions: they are not part of the standard
//投射函数
//版本一：投射出第一个参数，忽略第二个参数
template <class _Arg1, class _Arg2>
struct _Project1st : public binary_function<_Arg1, _Arg2, _Arg1> {
  _Arg1 operator()(const _Arg1& __x, const _Arg2&) const { return __x; }
};

//版本二：投射出第二个参数，忽略第一个参数
template <class _Arg1, class _Arg2>
struct _Project2nd : public binary_function<_Arg1, _Arg2, _Arg2> {
  _Arg2 operator()(const _Arg1&, const _Arg2& __y) const { return __y; }
};

template <class _Arg1, class _Arg2>
struct project1st : public _Project1st<_Arg1, _Arg2> {};

template <class _Arg1, class _Arg2>
struct project2nd : public _Project2nd<_Arg1, _Arg2> {};
```