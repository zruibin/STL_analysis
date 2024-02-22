## 前言

在SGI STL中hash表的实现是采用拉链法，其中用到了哈希函数，哈希函数的作用是把元素键值映射到对应的桶子里面，一般哈希值是键值对桶子数取余。在SGI STL提供的哈希函数是有限的，只支持特定的元素类型，若用户需要使用其他类型的哈希函数，则必须自行定义。定义的时候注意一下几点：

1. 使用struct，然后重载operator().
2. 返回是`size_t`
3. 参数是你要hash的key的类型。
4. 函数是const类型的。

例如定义string类型的哈希函数：

```cpp
struct str_hash{
        size_t operator()(const string& str) const
        {
                unsigned long __h = 0;
                for (size_t i = 0 ; i < str.size() ; i ++)
                __h = 5*__h + str[i];
                return size_t(__h);
        }
};
```

## hash函数源码剖析

```cpp
#ifndef __SGI_STL_HASH_FUN_H
#define __SGI_STL_HASH_FUN_H

#include <stddef.h>

__STL_BEGIN_NAMESPACE

//hash function 是计算元素位置的函数
//这些函数可以对hashtable进行取模运算
//这是hashtable所提供的散列函数是取模运算决定的

/*
SGI hashtable以下有限的定义类型：
	struct hash<char*>
	struct hash<const char*>
	struct hash<char> 
	struct hash<unsigned char> 
	struct hash<signed char>
	struct hash<short>
	struct hash<unsigned short> 
	struct hash<int> 
	struct hash<unsigned int>
	struct hash<long> 
	struct hash<unsigned long>
不在这里定义的类型，不能使用，若用户想要使用，则必须自己定义。例如：string，double，float
*/
/*定义自己的哈希函数时要注意以下几点：
	[1]使用struct，然后重载operator().
	[2]返回是size_t
	[3]参数是你要hash的key的类型。
	[4]函数是const类型的。
*/

template <class _Key> struct hash { };

//对const char* 提供字符串转换函数
inline size_t __stl_hash_string(const char* __s)
{
  unsigned long __h = 0; 
  for ( ; *__s; ++__s)
    __h = 5*__h + *__s;
  
  return size_t(__h);
}

__STL_TEMPLATE_NULL struct hash<char*>
{
  size_t operator()(const char* __s) const { return __stl_hash_string(__s); }
};

__STL_TEMPLATE_NULL struct hash<const char*>
{
  size_t operator()(const char* __s) const { return __stl_hash_string(__s); }
};

//下面的hash函数都是直接返回原值
//对于char,unsigned char,signed char,int,unsigned int, 
//short, unsigned short, long,unsigned long都只是返回数值本身
__STL_TEMPLATE_NULL struct hash<char> {
  size_t operator()(char __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<unsigned char> {
  size_t operator()(unsigned char __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<signed char> {
  size_t operator()(unsigned char __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<short> {
  size_t operator()(short __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<unsigned short> {
  size_t operator()(unsigned short __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<int> {
  size_t operator()(int __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<unsigned int> {
  size_t operator()(unsigned int __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<long> {
  size_t operator()(long __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<unsigned long> {
  size_t operator()(unsigned long __x) const { return __x; }
};

__STL_END_NAMESPACE

#endif /* __SGI_STL_HASH_FUN_H */

// Local Variables:
// mode:C++
// End:
```
