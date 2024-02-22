## 前言

这个文件提供了操作符的重载，但是没有提供`operator==`和`operator<`的重载，用户要使用这些操作符时，必须自己重载`operator==`和`operator<`操作符，因为该文件的重载操作符都是基于这两种操作符。本文件出自SGI STL中的`<stl_relops.h>`。

## relops源码

```cpp
// 这里提供比较运算符的重载, 任何全局的比较运算符如果需要重载  
// 只需要自己提供operator <和==即可 

#ifndef __SGI_STL_INTERNAL_RELOPS
#define __SGI_STL_INTERNAL_RELOPS

__STL_BEGIN_RELOPS_NAMESPACE

template <class _Tp>
inline bool operator!=(const _Tp& __x, const _Tp& __y) {
  return !(__x == __y);
}

template <class _Tp>
inline bool operator>(const _Tp& __x, const _Tp& __y) {
  return __y < __x;
}

template <class _Tp>
inline bool operator<=(const _Tp& __x, const _Tp& __y) {
  return !(__y < __x);
}

template <class _Tp>
inline bool operator>=(const _Tp& __x, const _Tp& __y) {
  return !(__x < __y);
}

__STL_END_RELOPS_NAMESPACE

#endif /* __SGI_STL_INTERNAL_RELOPS */

// Local Variables:
// mode:C++
// End:
```
