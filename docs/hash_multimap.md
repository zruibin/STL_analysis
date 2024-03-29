## 前言
`hash_multimap`和`hash_map`的区别就像multimap与map的区别一样，`hash_multimap`的底层机制是基于hash table，它可以存在重复的键值，所以插入函数使用`insert_equal()`，`hash_multimap`和`hash_map`一样，容器的内容不自动排序。由于`hash_multimap`和`hash_map`的源码剖析是一样的，这里就不对`hash_multimap`进行剖析，可以参照[《关联容器之hash_map》](http://blog.csdn.net/chenhanzhun/article/details/39609093)。本文源码出自SGI STL的`<stl_hash_map.h>`文件。

## hash_multimap源码

```cpp
// Forward declaration of equality operator; needed for friend declaration.

//hash_multimap与hash_map的差别就是插入函数，前者的插入函数是采用底层机制hash table的insert_equal()
//后者则采用insert_unique()
//其他的功能都和hash_map类似
//hash_multimap允许key重复
//这里就不再进行注释了，可以参考hash_map的解析
template <class _Key, class _Tp,
          class _HashFcn  __STL_DEPENDENT_DEFAULT_TMPL(hash<_Key>),
          class _EqualKey __STL_DEPENDENT_DEFAULT_TMPL(equal_to<_Key>),
          class _Alloc =  __STL_DEFAULT_ALLOCATOR(_Tp) >
class hash_multimap;

template <class _Key, class _Tp, class _HF, class _EqKey, class _Alloc>
inline bool 
operator==(const hash_multimap<_Key,_Tp,_HF,_EqKey,_Alloc>& __hm1,
           const hash_multimap<_Key,_Tp,_HF,_EqKey,_Alloc>& __hm2);

template <class _Key, class _Tp, class _HashFcn, class _EqualKey, 
          class _Alloc>
class hash_multimap
{
  // requirements:

  __STL_CLASS_REQUIRES(_Key, _Assignable);
  __STL_CLASS_REQUIRES(_Tp, _Assignable);
  __STL_CLASS_UNARY_FUNCTION_CHECK(_HashFcn, size_t, _Key);
  __STL_CLASS_BINARY_FUNCTION_CHECK(_EqualKey, bool, _Key, _Key);

private:
  typedef hashtable<pair<const _Key, _Tp>, _Key, _HashFcn,
                    _Select1st<pair<const _Key, _Tp> >, _EqualKey, _Alloc> 
          _Ht;
  _Ht _M_ht;

public:
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

  hasher hash_funct() const { return _M_ht.hash_funct(); }
  key_equal key_eq() const { return _M_ht.key_eq(); }
  allocator_type get_allocator() const { return _M_ht.get_allocator(); }

public:
  hash_multimap() : _M_ht(100, hasher(), key_equal(), allocator_type()) {}
  explicit hash_multimap(size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type()) {}
  hash_multimap(size_type __n, const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type()) {}
  hash_multimap(size_type __n, const hasher& __hf, const key_equal& __eql,
                const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a) {}

#ifdef __STL_MEMBER_TEMPLATES
  template <class _InputIterator>
  hash_multimap(_InputIterator __f, _InputIterator __l)
    : _M_ht(100, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_equal(__f, __l); }
  template <class _InputIterator>
  hash_multimap(_InputIterator __f, _InputIterator __l, size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_equal(__f, __l); }
  template <class _InputIterator>
  hash_multimap(_InputIterator __f, _InputIterator __l, size_type __n,
                const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type())
    { _M_ht.insert_equal(__f, __l); }
  template <class _InputIterator>
  hash_multimap(_InputIterator __f, _InputIterator __l, size_type __n,
                const hasher& __hf, const key_equal& __eql,
                const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a)
    { _M_ht.insert_equal(__f, __l); }

#else
  hash_multimap(const value_type* __f, const value_type* __l)
    : _M_ht(100, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_equal(__f, __l); }
  hash_multimap(const value_type* __f, const value_type* __l, size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_equal(__f, __l); }
  hash_multimap(const value_type* __f, const value_type* __l, size_type __n,
                const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type())
    { _M_ht.insert_equal(__f, __l); }
  hash_multimap(const value_type* __f, const value_type* __l, size_type __n,
                const hasher& __hf, const key_equal& __eql,
                const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a)
    { _M_ht.insert_equal(__f, __l); }

  hash_multimap(const_iterator __f, const_iterator __l)
    : _M_ht(100, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_equal(__f, __l); }
  hash_multimap(const_iterator __f, const_iterator __l, size_type __n)
    : _M_ht(__n, hasher(), key_equal(), allocator_type())
    { _M_ht.insert_equal(__f, __l); }
  hash_multimap(const_iterator __f, const_iterator __l, size_type __n,
                const hasher& __hf)
    : _M_ht(__n, __hf, key_equal(), allocator_type())
    { _M_ht.insert_equal(__f, __l); }
  hash_multimap(const_iterator __f, const_iterator __l, size_type __n,
                const hasher& __hf, const key_equal& __eql,
                const allocator_type& __a = allocator_type())
    : _M_ht(__n, __hf, __eql, __a)
    { _M_ht.insert_equal(__f, __l); }
#endif /*__STL_MEMBER_TEMPLATES */

public:
  size_type size() const { return _M_ht.size(); }
  size_type max_size() const { return _M_ht.max_size(); }
  bool empty() const { return _M_ht.empty(); }
  void swap(hash_multimap& __hs) { _M_ht.swap(__hs._M_ht); }

#ifdef __STL_MEMBER_TEMPLATES
  template <class _K1, class _T1, class _HF, class _EqK, class _Al>
  friend bool operator== (const hash_multimap<_K1, _T1, _HF, _EqK, _Al>&,
                          const hash_multimap<_K1, _T1, _HF, _EqK, _Al>&);
#else /* __STL_MEMBER_TEMPLATES */
  friend bool __STD_QUALIFIER
  operator== __STL_NULL_TMPL_ARGS (const hash_multimap&,const hash_multimap&);
#endif /* __STL_MEMBER_TEMPLATES */

  iterator begin() { return _M_ht.begin(); }
  iterator end() { return _M_ht.end(); }
  const_iterator begin() const { return _M_ht.begin(); }
  const_iterator end() const { return _M_ht.end(); }

public:
  iterator insert(const value_type& __obj) 
    { return _M_ht.insert_equal(__obj); }
#ifdef __STL_MEMBER_TEMPLATES
  template <class _InputIterator>
  void insert(_InputIterator __f, _InputIterator __l) 
    { _M_ht.insert_equal(__f,__l); }
#else
  void insert(const value_type* __f, const value_type* __l) {
    _M_ht.insert_equal(__f,__l);
  }
  void insert(const_iterator __f, const_iterator __l) 
    { _M_ht.insert_equal(__f, __l); }
#endif /*__STL_MEMBER_TEMPLATES */
  iterator insert_noresize(const value_type& __obj)
    { return _M_ht.insert_equal_noresize(__obj); }    

  iterator find(const key_type& __key) { return _M_ht.find(__key); }
  const_iterator find(const key_type& __key) const 
    { return _M_ht.find(__key); }

  size_type count(const key_type& __key) const { return _M_ht.count(__key); }
  
  pair<iterator, iterator> equal_range(const key_type& __key)
    { return _M_ht.equal_range(__key); }
  pair<const_iterator, const_iterator>
  equal_range(const key_type& __key) const
    { return _M_ht.equal_range(__key); }

  size_type erase(const key_type& __key) {return _M_ht.erase(__key); }
  void erase(iterator __it) { _M_ht.erase(__it); }
  void erase(iterator __f, iterator __l) { _M_ht.erase(__f, __l); }
  void clear() { _M_ht.clear(); }

public:
  void resize(size_type __hint) { _M_ht.resize(__hint); }
  size_type bucket_count() const { return _M_ht.bucket_count(); }
  size_type max_bucket_count() const { return _M_ht.max_bucket_count(); }
  size_type elems_in_bucket(size_type __n) const
    { return _M_ht.elems_in_bucket(__n); }
};

template <class _Key, class _Tp, class _HF, class _EqKey, class _Alloc>
inline bool 
operator==(const hash_multimap<_Key,_Tp,_HF,_EqKey,_Alloc>& __hm1,
           const hash_multimap<_Key,_Tp,_HF,_EqKey,_Alloc>& __hm2)
{
  return __hm1._M_ht == __hm2._M_ht;
}

#ifdef __STL_FUNCTION_TMPL_PARTIAL_ORDER

template <class _Key, class _Tp, class _HF, class _EqKey, class _Alloc>
inline bool 
operator!=(const hash_multimap<_Key,_Tp,_HF,_EqKey,_Alloc>& __hm1,
           const hash_multimap<_Key,_Tp,_HF,_EqKey,_Alloc>& __hm2) {
  return !(__hm1 == __hm2);
}

template <class _Key, class _Tp, class _HashFcn, class _EqlKey, class _Alloc>
inline void 
swap(hash_multimap<_Key,_Tp,_HashFcn,_EqlKey,_Alloc>& __hm1,
     hash_multimap<_Key,_Tp,_HashFcn,_EqlKey,_Alloc>& __hm2)
{
  __hm1.swap(__hm2);
}

#endif /* __STL_FUNCTION_TMPL_PARTIAL_ORDER */

// Specialization of insert_iterator so that it will work for hash_map
// and hash_multimap.

#ifdef __STL_CLASS_PARTIAL_SPECIALIZATION

template <class _Key, class _Tp, class _HashFn,  class _EqKey, class _Alloc>
class insert_iterator<hash_map<_Key, _Tp, _HashFn, _EqKey, _Alloc> > {
protected:
  typedef hash_map<_Key, _Tp, _HashFn, _EqKey, _Alloc> _Container;
  _Container* container;
public:
  typedef _Container          container_type;
  typedef output_iterator_tag iterator_category;
  typedef void                value_type;
  typedef void                difference_type;
  typedef void                pointer;
  typedef void                reference;

  insert_iterator(_Container& __x) : container(&__x) {}
  insert_iterator(_Container& __x, typename _Container::iterator)
    : container(&__x) {}
  insert_iterator<_Container>&
  operator=(const typename _Container::value_type& __value) { 
    container->insert(__value);
    return *this;
  }
  insert_iterator<_Container>& operator*() { return *this; }
  insert_iterator<_Container>& operator++() { return *this; }
  insert_iterator<_Container>& operator++(int) { return *this; }
};

template <class _Key, class _Tp, class _HashFn,  class _EqKey, class _Alloc>
class insert_iterator<hash_multimap<_Key, _Tp, _HashFn, _EqKey, _Alloc> > {
protected:
  typedef hash_multimap<_Key, _Tp, _HashFn, _EqKey, _Alloc> _Container;
  _Container* container;
  typename _Container::iterator iter;
public:
  typedef _Container          container_type;
  typedef output_iterator_tag iterator_category;
  typedef void                value_type;
  typedef void                difference_type;
  typedef void                pointer;
  typedef void                reference;

  insert_iterator(_Container& __x) : container(&__x) {}
  insert_iterator(_Container& __x, typename _Container::iterator)
    : container(&__x) {}
  insert_iterator<_Container>&
  operator=(const typename _Container::value_type& __value) { 
    container->insert(__value);
    return *this;
  }
  insert_iterator<_Container>& operator*() { return *this; }
  insert_iterator<_Container>& operator++() { return *this; }
  insert_iterator<_Container>& operator++(int) { return *this; }
};

#endif /* __STL_CLASS_PARTIAL_SPECIALIZATION */

#if defined(__sgi) && !defined(__GNUC__) && (_MIPS_SIM != _MIPS_SIM_ABI32)
#pragma reset woff 1174
#pragma reset woff 1375
#endif

__STL_END_NAMESPACE

#endif /* __SGI_STL_INTERNAL_HASH_MAP_H */

// Local Variables:
// mode:C++
// End:
```

## 参考资料

* 《STL源码剖析》侯捷