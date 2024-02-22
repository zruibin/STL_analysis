## 前言

由于前文介绍的《散列表hashtable》中，可以知道hash table在查找、删除和插入节点是常数时间，优于RB-Tree红黑树，所以在SGI STL中提供了底层机制基于hash table的`hash_set`容器，`hash_set`和set类似，但是不同点是`hash_set`容器中的元素是没有排序的，因为hash table没有提供排序功能。本文源码出自SGI STL的`<stl_hash_set.h>`文件。

## hash_set容器源码剖析

```cpp
#ifndef __SGI_STL_INTERNAL_HASH_SET_H
#define __SGI_STL_INTERNAL_HASH_SET_H

#include <concept_checks.h>

__STL_BEGIN_NAMESPACE

#if defined(__sgi) && !defined(__GNUC__) && (_MIPS_SIM != _MIPS_SIM_ABI32)
#pragma set woff 1174
#pragma set woff 1375
#endif
//hash_set的底层是基于hash table的，hash table没有提供排序,所以hash_set容器里面的内容是没排序的
//hash_set和set一样，键值和实值相同，并且键值是唯一的

// Forward declaration of equality operator; needed for friend declaration.

//这里提供默认的参数,其中哈希函数在<stl_hash_fun.h>定义
//前面的博文也对哈希函数进行讲解了
//http://blog.csdn.net/chenhanzhun/article/details/39584025
//用户可自行制定
template <class _Value,
          class _HashFcn  __STL_DEPENDENT_DEFAULT_TMPL(hash<_Value>),
          class _EqualKey __STL_DEPENDENT_DEFAULT_TMPL(equal_to<_Value>),
          class _Alloc =  __STL_DEFAULT_ALLOCATOR(_Value) >
class hash_set;

template <class _Value, class _HashFcn, class _EqualKey, class _Alloc>
inline bool 
operator==(const hash_set<_Value,_HashFcn,_EqualKey,_Alloc>& __hs1,
           const hash_set<_Value,_HashFcn,_EqualKey,_Alloc>& __hs2);

template <class _Value, class _HashFcn, class _EqualKey, class _Alloc>
class hash_set
{
  // requirements:

  __STL_CLASS_REQUIRES(_Value, _Assignable);
  __STL_CLASS_UNARY_FUNCTION_CHECK(_HashFcn, size_t, _Value);
  __STL_CLASS_BINARY_FUNCTION_CHECK(_EqualKey, bool, _Value, _Value);

private:
	//_Identity获取value值，在hash_set中也是键值，_Identity<>定义在<stl_function.h>
	/*
	template <class _Arg, class _Result>
	struct unary_function {
	  typedef _Arg argument_type;
	  typedef _Result result_type;
	};
	template <class _Tp>
	struct _Identity : public unary_function<_Tp,_Tp> {
			const _Tp& operator()(const _Tp& __x) const { return __x; }
	};
	*/
  typedef hashtable<_Value, _Value, _HashFcn, _Identity<_Value>, 
                    _EqualKey, _Alloc> _Ht;
  _Ht _M_ht;//hash_set的底层机制是hash table

public:
	//以下的内嵌类型均来时hash table
  typedef typename _Ht::key_type key_type;
  typedef typename _Ht::value_type value_type;
  typedef typename _Ht::hasher hasher;
  typedef typename _Ht::key_equal key_equal;

  typedef typename _Ht::size_type size_type;
  typedef typename _Ht::difference_type difference_type;

  // 注意: 不能修改hash table内部的元素,reference, pointer, iterator都为const
  typedef typename _Ht::const_pointer pointer;
  typedef typename _Ht::const_pointer const_pointer;
  typedef typename _Ht::const_reference reference;
  typedef typename _Ht::const_reference const_reference;

  typedef typename _Ht::const_iterator iterator;
  typedef typename _Ht::const_iterator const_iterator;

  typedef typename _Ht::allocator_type allocator_type;

  //返回hash函数
  hasher hash_funct() const { return _M_ht.hash_funct(); }
  key_equal key_eq() const { return _M_ht.key_eq(); }
  allocator_type get_allocator() const { return _M_ht.get_allocator(); }

public:
	//构造函数
	//缺省情况使用大小为100,但是实际分配的空间大小为不小于100的最小素数
	//只是空的hash_set，不存储元素节点
  hash_set()
    : _M_ht(100, hasher(), key_equal(), allocator_type()) {}
  //指定大小n的hash_set表
  explicit hash_set(size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type()) {}
  //指定大小为n，且指定hash函数的hash_set
  hash_set(size_type __n, const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type()) {}
  //指定大小为n，且指定hash函数和键值比较函数的hash_set
  hash_set(size_type __n, const hasher& __hf, const key_equal& __eql,
           const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a) {}

#ifdef __STL_MEMBER_TEMPLATES
  //以下hash_set的插入操作使用hash table的insert_unique插入
  //不允许有相同的键值插入

  //用某个范围的元素初始化hash_set对象
  //相当于把某个范围[f,l)插入到空的hash_set
  template <class _InputIterator>
  hash_set(_InputIterator __f, _InputIterator __l)
    : _M_ht(100, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }//调用hash table的插入函数
  template <class _InputIterator>
  hash_set(_InputIterator __f, _InputIterator __l, size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  template <class _InputIterator>
  hash_set(_InputIterator __f, _InputIterator __l, size_type __n,
           const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  template <class _InputIterator>
  hash_set(_InputIterator __f, _InputIterator __l, size_type __n,
           const hasher& __hf, const key_equal& __eql,
           const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a)
    { _M_ht.insert_unique(__f, __l); }
#else

  hash_set(const value_type* __f, const value_type* __l)
    : _M_ht(100, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_set(const value_type* __f, const value_type* __l, size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_set(const value_type* __f, const value_type* __l, size_type __n,
           const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_set(const value_type* __f, const value_type* __l, size_type __n,
           const hasher& __hf, const key_equal& __eql,
           const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a)
    { _M_ht.insert_unique(__f, __l); }

  hash_set(const_iterator __f, const_iterator __l)
    : _M_ht(100, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_set(const_iterator __f, const_iterator __l, size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_set(const_iterator __f, const_iterator __l, size_type __n,
           const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_set(const_iterator __f, const_iterator __l, size_type __n,
           const hasher& __hf, const key_equal& __eql,
           const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a)
    { _M_ht.insert_unique(__f, __l); }
#endif /*__STL_MEMBER_TEMPLATES */

public:
	//以下的函数操作只是调用hash table的成员函数
  size_type size() const { return _M_ht.size(); }
  size_type max_size() const { return _M_ht.max_size(); }
  bool empty() const { return _M_ht.empty(); }
  void swap(hash_set& __hs) { _M_ht.swap(__hs._M_ht); }

#ifdef __STL_MEMBER_TEMPLATES
  template <class _Val, class _HF, class _EqK, class _Al>  
  friend bool operator== (const hash_set<_Val, _HF, _EqK, _Al>&,
                          const hash_set<_Val, _HF, _EqK, _Al>&);
#else /* __STL_MEMBER_TEMPLATES */
  friend bool __STD_QUALIFIER
  operator== __STL_NULL_TMPL_ARGS (const hash_set&, const hash_set&);
#endif /* __STL_MEMBER_TEMPLATES */

  iterator begin() const { return _M_ht.begin(); }
  iterator end() const { return _M_ht.end(); }

public:
	/*
	插入元素
	(1)
		pair<iterator,bool> insert ( const value_type& val );
	(2)		
		template <class InputIterator>
		void insert ( InputIterator first, InputIterator last );
	*/
	//不允许有重复的键值
	//返回pair第二个参数second若为true则插入成功
  pair<iterator, bool> insert(const value_type& __obj)
    {
	//调用hash table的insert_unique()函数
      pair<typename _Ht::iterator, bool> __p = _M_ht.insert_unique(__obj);
      return pair<iterator,bool>(__p.first, __p.second);
    }
#ifdef __STL_MEMBER_TEMPLATES
  template <class _InputIterator>
  void insert(_InputIterator __f, _InputIterator __l) 
    { _M_ht.insert_unique(__f,__l); }
#else
  void insert(const value_type* __f, const value_type* __l) {
    _M_ht.insert_unique(__f,__l);
  }
  void insert(const_iterator __f, const_iterator __l) 
    {_M_ht.insert_unique(__f, __l); }
#endif /*__STL_MEMBER_TEMPLATES */
  pair<iterator, bool> insert_noresize(const value_type& __obj)
  {
    pair<typename _Ht::iterator, bool> __p = 
      _M_ht.insert_unique_noresize(__obj);
    return pair<iterator, bool>(__p.first, __p.second);
  }

  //查找键值对应的元素，并且返回该元素的节点位置
  iterator find(const key_type& __key) const { return _M_ht.find(__key); }
  //返回键值为key的节点元素个数
  size_type count(const key_type& __key) const { return _M_ht.count(__key); }
  
  //Returns the bounds of a range that includes all the elements that compare equal to k. 
  //In hash_set containers, where keys are unique, the range will include one element at most.
  pair<iterator, iterator> equal_range(const key_type& __key) const
    { return _M_ht.equal_range(__key); }

  //擦除指定键值的元素，并返回擦除的个数
  //因为键值唯一,则该键值的元素最多为1个
  size_type erase(const key_type& __key) {return _M_ht.erase(__key); }
  //擦除指定位置的元素
  void erase(iterator __it) { _M_ht.erase(__it); }
  //擦除指定范围的元素
  void erase(iterator __f, iterator __l) { _M_ht.erase(__f, __l); }
  //清除hash_set容器
  void clear() { _M_ht.clear(); }

public:
	//调整hash_set容器的容量
  void resize(size_type __hint) { _M_ht.resize(__hint); }
  //Returns the number of buckets in the hash_set container.
  size_type bucket_count() const { return _M_ht.bucket_count(); }
  size_type max_bucket_count() const { return _M_ht.max_bucket_count(); }
  //Returns the number of elements in bucket n
  size_type elems_in_bucket(size_type __n) const
    { return _M_ht.elems_in_bucket(__n); }//返回指定桶子键值key中list链表的元素个数
};

template <class _Value, class _HashFcn, class _EqualKey, class _Alloc>
inline bool 
operator==(const hash_set<_Value,_HashFcn,_EqualKey,_Alloc>& __hs1,
           const hash_set<_Value,_HashFcn,_EqualKey,_Alloc>& __hs2)
{
  return __hs1._M_ht == __hs2._M_ht;
}

#ifdef __STL_FUNCTION_TMPL_PARTIAL_ORDER

template <class _Value, class _HashFcn, class _EqualKey, class _Alloc>
inline bool 
operator!=(const hash_set<_Value,_HashFcn,_EqualKey,_Alloc>& __hs1,
           const hash_set<_Value,_HashFcn,_EqualKey,_Alloc>& __hs2) {
  return !(__hs1 == __hs2);
}

//交换两个hash_set容器的内容
template <class _Val, class _HashFcn, class _EqualKey, class _Alloc>
inline void 
swap(hash_set<_Val,_HashFcn,_EqualKey,_Alloc>& __hs1,
     hash_set<_Val,_HashFcn,_EqualKey,_Alloc>& __hs2)
{
  __hs1.swap(__hs2);
}

#endif /* __STL_FUNCTION_TMPL_PARTIAL_ORDER */
```

## 参考资料

* 《STL源码剖析》侯捷