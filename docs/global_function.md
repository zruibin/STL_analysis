
## 前言

在STL中，内存处理时很关键的，特别是对于容器的实现，内存处理相当的重要。为了实现内存配置跟对象的构造行为分离开来，STL定义的五个基本全局函数，这五个全局函数中，贯穿在里面的基本上是Traits和__type_traits技术，之前看的时候不了解这两技术，因此，现在了解之后才来写这篇文章。有关Traits技术见前文《Traits编程技术》。

STL五个全局函数分别是：construct(),destroy(),uninitialized_copy(),uninitialized_fill(),uninitialized_fill_n();以下分别对这些函数进行讲解。

## 构造和析构基本工具：construct()和destroy()

在STL源码`<stl_construct.h`>中,内存的配置和对象的构造与析构是分离开的，这两个函数实现对象的构造和析构，不进行分配内存空间，construct()函数在已分配内存上构造对象，具体实现看下面的源码及其解析：

```cpp
#ifndef __SGI_STL_INTERNAL_CONSTRUCT_H
#define __SGI_STL_INTERNAL_CONSTRUCT_H

#include <new.h>

__STL_BEGIN_NAMESPACE

// construct and destroy.  These functions are not part of the C++ standard,
// and are provided for backward compatibility with the HP STL.  We also
// provide internal names _Construct and _Destroy that can be used within
// the library, so that standard-conforming pieces don't have to rely on
// non-standard extensions.

// Internal names

template <class _T1, class _T2>
//参数接受一个指针和一个初值
inline void _Construct(_T1* __p, const _T2& __value) {
  new ((void*) __p) _T1(__value);/*这里是placement new;调用构造函数T1::T1(value)
								 *将初值设定到指针所指的空间上
								 */
}

template <class _T1>
//这里只接受指针
inline void _Construct(_T1* __p) {
  new ((void*) __p) _T1();//这里是placement new;调用默认构造函数T1::T1()
}

template <class _Tp>
inline void _Destroy(_Tp* __pointer) {//第一个版本：接受一个指针
  __pointer->~_Tp();//调用对象析构函数，析构指针所指对象
}

template <class _ForwardIterator>
/*第二个版本：接受两个迭代器first和last，自迭代器first开始到迭代器last结束，析构每个元素对象
*为求最大效率，首先以__VALUE_TYPE()萃取出迭代器first的value_type
*再利用__type_traits判断该型别是否trivial destructor*/
inline void _Destroy(_ForwardIterator __first, _ForwardIterator __last) {
  __destroy(__first, __last, __VALUE_TYPE(__first));
}

template <class _ForwardIterator, class _Tp>
inline void 
__destroy(_ForwardIterator __first, _ForwardIterator __last, _Tp*)
{//利用__type_traits判断元素的数值型别是否有trivial destructor
  typedef typename __type_traits<_Tp>::has_trivial_destructor
          _Trivial_destructor;
  __destroy_aux(__first, __last, _Trivial_destructor());
}

template <class _ForwardIterator> 
/*若元素型别是有trivial destructor，就派送到该函数
*/
inline void __destroy_aux(_ForwardIterator, _ForwardIterator, __true_type) {}

template <class _ForwardIterator>
void
__destroy_aux(_ForwardIterator __first, _ForwardIterator __last, __false_type)
{//若元素型别不具有trivial destructor，就派送到该函数
  for ( ; __first != __last; ++__first)
    destroy(&*__first);//这里最终还是调用了指针版的析构函数,析构每个元素对象
}

//以下是第二版析构函数针对内置类型的特化版
inline void _Destroy(char*, char*) {}
inline void _Destroy(int*, int*) {}
inline void _Destroy(long*, long*) {}
inline void _Destroy(float*, float*) {}
inline void _Destroy(double*, double*) {}
#ifdef __STL_HAS_WCHAR_T
inline void _Destroy(wchar_t*, wchar_t*) {}
#endif /* __STL_HAS_WCHAR_T */

// --------------------------------------------------
// Old names from the HP STL.

template <class _T1, class _T2>
inline void construct(_T1* __p, const _T2& __value) {
  _Construct(__p, __value);
}

template <class _T1>
inline void construct(_T1* __p) {
  _Construct(__p);
}

template <class _Tp>//参数为指针版的析构函数
inline void destroy(_Tp* __pointer) {
  _Destroy(__pointer);
}

template <class _ForwardIterator>//参数为迭代器版的析构函数
inline void destroy(_ForwardIterator __first, _ForwardIterator __last) {
  _Destroy(__first, __last);
}

__STL_END_NAMESPACE

#endif /* __SGI_STL_INTERNAL_CONSTRUCT_H */

// Local Variables:
// mode:C++
// End:
```

## 未初始化空间的复制和初始化：uninitialized_copy(),uninitialized_fill()和uninitialized_fill_n()

在未初始化空间中，要对对象进行初始化时，会用到这些函数，该源码在文件`<stl_uninitialized.h>`中，具体实现看下面源码及其解析：

```cpp
#ifndef __SGI_STL_INTERNAL_UNINITIALIZED_H
#define __SGI_STL_INTERNAL_UNINITIALIZED_H

__STL_BEGIN_NAMESPACE

// uninitialized_copy

// Valid if copy construction is equivalent to assignment, and if the
//  destructor is trivial.
template <class _InputIter, class _ForwardIter>
/*该函数接受三个迭代器参数：迭代器first是输入的起始地址，
*迭代器last是输入的结束地址，迭代器result是输出的起始地址
*即把数据复制到[result,result+(last-first)]这个范围
*为了提高效率，首先用__VALUE_TYPE()萃取出迭代器result的型别value_type
*再利用__type_traits判断该型别是否为POD型别
*/
inline _ForwardIter
  uninitialized_copy(_InputIter __first, _InputIter __last,
                     _ForwardIter __result)
{
  return __uninitialized_copy(__first, __last, __result,
                              __VALUE_TYPE(__result));
}

template <class _InputIter, class _ForwardIter, class _Tp>
inline _ForwardIter
/*利用__type_traits判断该型别是否为POD型别*/
__uninitialized_copy(_InputIter __first, _InputIter __last,
                     _ForwardIter __result, _Tp*)
{
  typedef typename __type_traits<_Tp>::is_POD_type _Is_POD;
  return __uninitialized_copy_aux(__first, __last, __result, _Is_POD());
}

template <class _InputIter, class _ForwardIter>
_ForwardIter //若不是POD型别，就派送到这里
__uninitialized_copy_aux(_InputIter __first, _InputIter __last,
                         _ForwardIter __result,
                         __false_type)
{
  _ForwardIter __cur = __result;
  __STL_TRY {//这里加入了异常处理机制
    for ( ; __first != __last; ++__first, ++__cur)
      _Construct(&*__cur, *__first);//构造对象，必须是一个一个元素的构造，不能批量
    return __cur;
  }
  __STL_UNWIND(_Destroy(__result, __cur));//析构对象
}

template <class _InputIter, class _ForwardIter>
inline _ForwardIter //若是POD型别，就派送到这里
__uninitialized_copy_aux(_InputIter __first, _InputIter __last,
                         _ForwardIter __result,
                         __true_type)
{
	/*调用STL的算法copy()
	*函数原型：template< class InputIt, class OutputIt >
	* OutputIt copy( InputIt first, InputIt last, OutputIt d_first );
	*/
	return copy(__first, __last, __result);
}
//下面是针对char*，wchar_t* 的uninitialized_copy()特化版本
inline char* uninitialized_copy(const char* __first, const char* __last,
                                char* __result) {
/* void* memmove( void* dest, const void* src, std::size_t count );
* dest指向输出的起始地址
* src指向输入的其实地址
* count要复制的字节数
*/
  memmove(__result, __first, __last - __first);
  return __result + (__last - __first);
}

inline wchar_t* 
uninitialized_copy(const wchar_t* __first, const wchar_t* __last,
                   wchar_t* __result)
{
  memmove(__result, __first, sizeof(wchar_t) * (__last - __first));
  return __result + (__last - __first);
}

// Valid if copy construction is equivalent to assignment, and if the
// destructor is trivial.
template <class _ForwardIter, class _Tp>
/*若是POD型别，则调用此函数
	*/
inline void
__uninitialized_fill_aux(_ForwardIter __first, _ForwardIter __last, 
                         const _Tp& __x, __true_type)
{
/*函数原型：template< class ForwardIt, class T >
  * void fill( ForwardIt first, ForwardIt last, const T& value );  
  */
	fill(__first, __last, __x);
}

template <class _ForwardIter, class _Tp>
/*若不是POD型别，则调用此函数
	*/
void
__uninitialized_fill_aux(_ForwardIter __first, _ForwardIter __last, 
                         const _Tp& __x, __false_type)
{
  _ForwardIter __cur = __first;
  __STL_TRY {
    for ( ; __cur != __last; ++__cur)
      _Construct(&*__cur, __x);
  }
  __STL_UNWIND(_Destroy(__first, __cur));
}

template <class _ForwardIter, class _Tp, class _Tp1>
//用__type_traits技术判断该型别是否
为POD型别
inline void __uninitialized_fill(_ForwardIter __first, 
                                 _ForwardIter __last, const _Tp& __x, _Tp1*)
{
  typedef typename __type_traits<_Tp1>::is_POD_type _Is_POD;
  __uninitialized_fill_aux(__first, __last, __x, _Is_POD());
                   
}

template <class _ForwardIter, class _Tp>
/*该函数接受三个参数：
*迭代器first指向欲初始化的空间起始地址
*迭代器last指向欲初始化的空间结束地址
*x表示初值
*首先利用__VALUE_TYPE()萃取出迭代器first的型别value_type
*然后用__type_traits技术判断该型别是否为POD型别
*/
inline void uninitialized_fill(_ForwardIter __first,
                               _ForwardIter __last, 
                               const _Tp& __x)
{
  __uninitialized_fill(__first, __last, __x, __VALUE_TYPE(__first));
}

// Valid if copy construction is equivalent to assignment, and if the
//  destructor is trivial.
template <class _ForwardIter, class _Size, class _Tp>
inline _ForwardIter
	/*若是POD型别，则调用此函数
	*/
__uninitialized_fill_n_aux(_ForwardIter __first, _Size __n,
                           const _Tp& __x, __true_type)
{
  /*调用STL算法
  *原型：template< class OutputIt, class Size, class T >
  * void fill_n( OutputIt first, Size count, const T& value );
  * template< class OutputIt, class Size, class T >
  * OutputIt fill_n( OutputIt first, Size count, const T& value );
  */
	return fill_n(__first, __n, __x);
}

template <class _ForwardIter, class _Size, class _Tp>
_ForwardIter
	/*若不是POD型别，则调用此函数
	*/
__uninitialized_fill_n_aux(_ForwardIter __first, _Size __n,
                           const _Tp& __x, __false_type)
{
  _ForwardIter __cur = __first;
  __STL_TRY {
    for ( ; __n > 0; --__n, ++__cur)
      _Construct(&*__cur, __x);
    return __cur;
  }
  __STL_UNWIND(_Destroy(__first, __cur));
}

template <class _ForwardIter, class _Size, class _Tp, class _Tp1>
inline _ForwardIter 
	//用__type_traits技术判断该型别是否为POD型别
__uninitialized_fill_n(_ForwardIter __first, _Size __n, const _Tp& __x, _Tp1*)
{
  typedef typename __type_traits<_Tp1>::is_POD_type _Is_POD;
  //_Is_POD()判断value_type是否为POD型别
  return __uninitialized_fill_n_aux(__first, __n, __x, _Is_POD());
}

template <class _ForwardIter, class _Size, class _Tp>
inline _ForwardIter 
/*该函数接受三个参数：
*迭代器first指向欲初始化的空间起始地址
*n表示欲初始化空间大小
*x表示初值
*首先利用__VALUE_TYPE()萃取出迭代器first的型别value_type
*然后用__type_traits技术判断该型别是否为POD型别
*/
uninitialized_fill_n(_ForwardIter __first, _Size __n, const _Tp& __x)
{
  //__VALUE_TYPE(__first)萃取出first的型别value_type
	return __uninitialized_fill_n(__first, __n, __x, __VALUE_TYPE(__first));
}

__STL_END_NAMESPACE

#endif /* __SGI_STL_INTERNAL_UNINITIALIZED_H */

// Local Variables:
// mode:C++
// End:
```

## 总结

在这个五个全局函数中，我们可以看到，基本上是`Traits`和`__type_traits`技术的作用，方便内存处理，并且提高效率。
