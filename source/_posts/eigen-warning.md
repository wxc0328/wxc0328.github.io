---
title: Eigen中的注意事项
date: 2026-03-01 15:10:00
tags: Eigen
categories:
  - [C++, Eigen]
---

# 结构体包含eigen对象
对于使用c++17及以上标准，不需要以下设置
```c++
class Foo
{
  ...
  Eigen::Vector4d v; //准确的说是具有静态大小的Eigen类型
  ...
public:
  EIGEN_MAKE_ALIGNED_OPERATOR_NEW //需要添加这个宏，在new时会返回一个对齐过的内存指针
  //或者（两者取一）
  EIGEN_MAKE_ALIGNED_OPERATOR_NEW_IF(NeedsToAlign) //当不需要对齐时，可以避免eigen注入额外的代码
};
```
## 原因
Eigen::Vector4d包含1个double，默认情况下占用为256bits（`4*8`字节）。例如使用AVX时，需要内存基于256-bit对齐，这意味着需要**起始地址**是 32 的倍数，如果不调整内存的对齐方式，在使用AVX指令时会导致段错误。
## STL容器包含eigen对象
同样，对于使用c++17及以上标准，不需要设置
```c++
//使用eigen提供的分配器

std::map<int, Eigen::Vector4d, std::less<int>, 
         Eigen::aligned_allocator<std::pair<const int, Eigen::Vector4d> > >
         
#include<Eigen/StdVector>
/* ... */
std::vector<Eigen::Vector4f,Eigen::aligned_allocator<Eigen::Vector4f> >
```