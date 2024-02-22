## 前言

由于前文介绍的《散列表hashtable》中，可以知道hash table在查找、删除和插入节点是常数时间，优于RB-Tree红黑树，所以在SGI STL中提供了底层机制基于hash table的`hash_map`容器，`hash_map`和`map`类似，但是不同点是`hash_map`容器中的元素是没有排序的，因为hash table没有提供排序功能。本文源码出自SGI STL的`<stl_hash_map.h>`文件。

## hash_map源码剖析

```cpp
#ifndef __SGI_STL_INTERNAL_HASH_MAP_H
#define __SGI_STL_INTERNAL_HASH_MAP_H

#include <concept_checks.h>

__STL_BEGIN_NAMESPACE

#if defined(__sgi) && !defined(__GNUC__) && (_MIPS_SIM != _MIPS_SIM_ABI32)
#pragma set woff 1174
#pragma set woff 1375
#endif
//hash_map的底层是基于hash table的，hash table没有提供排序,所以hash_map容器里面的内容是没排序的

// Forward declaration of equality operator; needed for friend declaration.

template <class _Key, class _Tp,
          class _HashFcn  __STL_DEPENDENT_DEFAULT_TMPL(hash<_Key>),
          class _EqualKey __STL_DEPENDENT_DEFAULT_TMPL(equal_to<_Key>),
          class _Alloc =  __STL_DEFAULT_ALLOCATOR(_Tp) >
class hash_map;

template <class _Key, class _Tp, class _HashFn, class _EqKey, class _Alloc>
inline bool operator==(const hash_map<_Key, _Tp, _HashFn, _EqKey, _Alloc>&,
                       const hash_map<_Key, _Tp, _HashFn, _EqKey, _Alloc>&);

template <class _Key, class _Tp, class _HashFcn, class _EqualKey,
          class _Alloc>
class hash_map
{
  // requirements:

  __STL_CLASS_REQUIRES(_Key, _Assignable);
  __STL_CLASS_REQUIRES(_Tp, _Assignable);
  __STL_CLASS_UNARY_FUNCTION_CHECK(_HashFcn, size_t, _Key);
  __STL_CLASS_BINARY_FUNCTION_CHECK(_EqualKey, bool, _Key, _Key);

private:
	//_Select1st<>取出键值key,_Select1st<>定义在<stl_function.h>
	/*
	template <class _Arg, class _Result>
	struct unary_function {
	  typedef _Arg argument_type;
	  typedef _Result result_type;
	};
	template <class _Pair>
	struct _Select1st : public unary_function<_Pair, typename _Pair::first_type> {
	  const typename _Pair::first_type& operator()(const _Pair& __x) const {
		return __x.first;
	  }
	};
	*/
  typedef hashtable<pair<const _Key,_Tp>,_Key,_HashFcn,
                    _Select1st<pair<const _Key,_Tp> >,_EqualKey,_Alloc> _Ht;
  _Ht _M_ht;//底层机制以hash table完成

public:
	//以下的内嵌类型均来是hash table
  typedef typename _Ht::key_type key_type;
  typedef _Tp data_type;
  typedef _Tp mapped_type;
  typedef typename _Ht::value_type value_type;
  typedef typename _Ht::hasher hasher;
  typedef typename _Ht::key_equal key_equal;
  
  typedef typename _Ht::size_type size_type;
  typedef typename _Ht::difference_type difference_type;
  typedef typename _Ht::pointer pointer;
  typedef typename _Ht::const_pointer const_pointer;
  typedef typename _Ht::reference reference;
  typedef typename _Ht::const_reference const_reference;

  typedef typename _Ht::iterator iterator;
  typedef typename _Ht::const_iterator const_iterator;

  typedef typename _Ht::allocator_type allocator_type;

  //返回hash相关函数
  hasher hash_funct() const { return _M_ht.hash_funct(); }
  key_equal key_eq() const { return _M_ht.key_eq(); }
  allocator_type get_allocator() const { return _M_ht.get_allocator(); }

public:
//构造函数
//缺省情况使用大小为100,但是实际分配的空间大小为不小于100的最小素数
//只是空的hash_map，不存储元素节点
  hash_map() : _M_ht(100, hasher(), key_equal(), allocator_type()) {}
  //指定大小n的hash_map表
  explicit hash_map(size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type()) {}
  //指定大小为n，且指定hash函数的hash_map
  hash_map(size_type __n, const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type()) {}
  //指定大小为n，且指定hash函数和键值比较函数的hash_map
  hash_map(size_type __n, const hasher& __hf, const key_equal& __eql,
           const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a) {}

#ifdef __STL_MEMBER_TEMPLATES
  //以下hash_map的插入操作使用hash table的insert_unique插入
  //不允许有相同的键值插入

  //用某个范围的元素初始化hash_map对象
  //相当于把某个范围[f,l)插入到空的hash_map
  template <class _InputIterator>
  hash_map(_InputIterator __f, _InputIterator __l)
    : _M_ht(100, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }//调用hash table的插入函数
  template <class _InputIterator>
  hash_map(_InputIterator __f, _InputIterator __l, size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  template <class _InputIterator>
  hash_map(_InputIterator __f, _InputIterator __l, size_type __n,
           const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  template <class _InputIterator>
  hash_map(_InputIterator __f, _InputIterator __l, size_type __n,
           const hasher& __hf, const key_equal& __eql,
           const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a)
    { _M_ht.insert_unique(__f, __l); }

#else
  hash_map(const value_type* __f, const value_type* __l)
    : _M_ht(100, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_map(const value_type* __f, const value_type* __l, size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_map(const value_type* __f, const value_type* __l, size_type __n,
           const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_map(const value_type* __f, const value_type* __l, size_type __n,
           const hasher& __hf, const key_equal& __eql,
           const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a)
    { _M_ht.insert_unique(__f, __l); }

  hash_map(const_iterator __f, const_iterator __l)
    : _M_ht(100, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_map(const_iterator __f, const_iterator __l, size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_map(const_iterator __f, const_iterator __l, size_type __n,
           const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type())
    { _M_ht.insert_unique(__f, __l); }
  hash_map(const_iterator __f, const_iterator __l, size_type __n,
           const hasher& __hf, const key_equal& __eql,
           const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a)
    { _M_ht.insert_unique(__f, __l); }
#endif /*__STL_MEMBER_TEMPLATES */

public:
//以下的函数操作只是调用hash table的成员函数
//返回 hash_map 容器中元素的个数.
  size_type size() const { return _M_ht.size(); }
//返回hash_map容器最大存储元素的个数.
  size_type max_size() const { return _M_ht.max_size(); }
//Returns a bool value indicating whether the hash_map container is empty,
//i.e. whether its size is 0.
  bool empty() const { return _M_ht.empty(); }
  //交换两个存储相同元素类型的hash_map容器内容,但是hash_map容器大小可以不同
  void swap(hash_map& __hs) { _M_ht.swap(__hs._M_ht); }

#ifdef __STL_MEMBER_TEMPLATES
  template <class _K1, class _T1, class _HF, class _EqK, class _Al>
  friend bool operator== (const hash_map<_K1, _T1, _HF, _EqK, _Al>&,
                          const hash_map<_K1, _T1, _HF, _EqK, _Al>&);
#else /* __STL_MEMBER_TEMPLATES */
  friend bool __STD_QUALIFIER
  operator== __STL_NULL_TMPL_ARGS (const hash_map&, const hash_map&);
#endif /* __STL_MEMBER_TEMPLATES */

  iterator begin() { return _M_ht.begin(); }
  iterator end() { return _M_ht.end(); }
  const_iterator begin() const { return _M_ht.begin(); }
  const_iterator end() const { return _M_ht.end(); }

public:
	//插入元素
	/*
	(1)	
	pair<iterator,bool> insert ( const value_type& val );
	(2)	
	template <class InputIterator>
		void insert ( InputIterator first, InputIterator last );
	*/
	//不允许有重复的键值
	//返回pair第二个参数second若为true则插入成功
  pair<iterator,bool> insert(const value_type& __obj)
    { return _M_ht.insert_unique(__obj); }//调用hash table的insert_unique()函数
#ifdef __STL_MEMBER_TEMPLATES
  template <class _InputIterator>
  void insert(_InputIterator __f, _InputIterator __l)
    { _M_ht.insert_unique(__f,__l); }
#else
  void insert(const value_type* __f, const value_type* __l) {
    _M_ht.insert_unique(__f,__l);
  }
  void insert(const_iterator __f, const_iterator __l)
    { _M_ht.insert_unique(__f, __l); }
#endif /*__STL_MEMBER_TEMPLATES */
  pair<iterator,bool> insert_noresize(const value_type& __obj)
    { return _M_ht.insert_unique_noresize(__obj); }    
  //Searches the container for an element with k as key and returns an iterator to it if found, 
  //otherwise it returns an iterator to hash_map::end 
  iterator find(const key_type& __key) { return _M_ht.find(__key); }
  const_iterator find(const key_type& __key) const 
    { return _M_ht.find(__key); }

  //If k matches the key of an element in the container, the function returns a reference to its mapped value.
  _Tp& operator[](const key_type& __key) {
    return _M_ht.find_or_insert(value_type(__key, _Tp())).second;
  }
  //Searches the container for elements whose key is k and returns the number of elements found. 
  //由于不存在重复的键值,所以返回的个数最多为1个
  size_type count(const key_type& __key) const { return _M_ht.count(__key); }
  
 //Returns the bounds of a range that includes all the elements in the container with a key that compares equal to k
  //由于不存在重复的键值,所以返回的元素最多为1个
  pair<iterator, iterator> equal_range(const key_type& __key)
    { return _M_ht.equal_range(__key); }
  pair<const_iterator, const_iterator>
  equal_range(const key_type& __key) const
    { return _M_ht.equal_range(__key); }

  //删除元素
  /*
	by position (1)	
		iterator erase ( const_iterator position );
	by key (2)	
		size_type erase ( const key_type& k );
	range (3)	
		iterator erase ( const_iterator first, const_iterator last );
  */
  //擦除指定键值的元素，并返回擦除的个数
  //因为键值唯一,则该键值的元素最多为1个
  size_type erase(const key_type& __key) {return _M_ht.erase(__key); }
  //擦除指定位置的元素
  void erase(iterator __it) { _M_ht.erase(__it); }
  //擦除指定范围的元素
  void erase(iterator __f, iterator __l) { _M_ht.erase(__f, __l); }
  //清空hash_map容器
  void clear() { _M_ht.clear(); }

  //调整hash_set容器的容量
  void resize(size_type __hint) { _M_ht.resize(__hint); }
  //Returns the number of buckets in the hash_map container.
  size_type bucket_count() const { return _M_ht.bucket_count(); }
  //Returns the maximum number of buckets that the hash_map container can have.
  size_type max_bucket_count() const { return _M_ht.max_bucket_count(); }
  //Returns the number of elements in bucket n
  size_type elems_in_bucket(size_type __n) const
    { return _M_ht.elems_in_bucket(__n); }//返回指定桶子键值key中list链表的元素个数
};

template <class _Key, class _Tp, class _HashFcn, class _EqlKey, class _Alloc>
inline bool 
operator==(const hash_map<_Key,_Tp,_HashFcn,_EqlKey,_Alloc>& __hm1,
           const hash_map<_Key,_Tp,_HashFcn,_EqlKey,_Alloc>& __hm2)
{
  return __hm1._M_ht == __hm2._M_ht;
}

#ifdef __STL_FUNCTION_TMPL_PARTIAL_ORDER

template <class _Key, class _Tp, class _HashFcn, class _EqlKey, class _Alloc>
inline bool 
operator!=(const hash_map<_Key,_Tp,_HashFcn,_EqlKey,_Alloc>& __hm1,
           const hash_map<_Key,_Tp,_HashFcn,_EqlKey,_Alloc>& __hm2) {
  return !(__hm1 == __hm2);
}

//交换两个hash_map容器的内容
template <class _Key, class _Tp, class _HashFcn, class _EqlKey, class _Alloc>
inline void 
swap(hash_map<_Key,_Tp,_HashFcn,_EqlKey,_Alloc>& __hm1,
     hash_map<_Key,_Tp,_HashFcn,_EqlKey,_Alloc>& __hm2)
{
  __hm1.swap(__hm2);
}

#endif /* __STL_FUNCTION_TMPL_PARTIAL_ORDER */
```

## 参考资料

* 《STL源码剖析》侯捷