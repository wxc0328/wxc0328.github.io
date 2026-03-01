---
title: C++使用new在已分配的内存上构造对象
date: 2026-03-01 17:21:00
tags: C++
categories:
  - C++
---

# 案例代码
```c++
new (&_u.zclmsg.content->refcnt) zmq::atomic_counter_t ();

//刚好在zmq中看到这个实现方式
```
# 使用格式
```c++
char* ptr = new char[sizeof(T)]; // 分配内存
T* tptr = new(ptr) T;            // 在已分配内存中构造
... // 此时起tptr可以作为一个T*的对象正常使用
tptr->~T();                      // 销毁
delete[] ptr;                    // 解分配内存
```

通过上述方法，在已分配的内存中构造类型T，需要独立调用析构函数来正常销毁对象