---
title: C++中的errno关键字
date: 2026-03-01 16:57:00
tags: C++
categories:
  - C++
---

在C和C++编程中，errno是一个全局变量，用于指示函数失败的原因。

例如，在执行文件操作或数学运算时，如果函数无法成功完成其任务，它可能会设置errno来提供错误的具体原因。

# 访问errno的基础方式

在C和C++中，你可以通过直接访问errno变量来获取当前的错误代码。例如：
```c++
#include <stdio.h>
#include <errno.h>
#include <string.h>
 
int main() {
    FILE *fp = fopen("nonexistent_file.txt", "r");
    if (fp == NULL) {
        // 打印错误的详细信息
        printf("Error opening file: %s\n", strerror(errno));
    }
    return 0;
}
```
# 使用errno的正确方式

虽然直接访问errno很方便，但在多线程环境中直接访问全局变量（如errno）可能导致竞态条件（race conditions）。为了避免这种问题，C99标准引入了errno.h中的errno_location()函数，它返回一个指向当前线程的errno变量的指针。这样，你可以通过解引用这个指针来修改或读取errno的值，而不会引起多线程问题。

使用errno_location的示例
```c++
#include <stdio.h>
#include <errno.h>
#include <string.h>
 
int main() {
    FILE *fp = fopen("nonexistent_file.txt", "r");
    if (fp == NULL) {
        // 使用errno_location来安全地访问errno
        int *err = errno_location();
        printf("Error opening file: %s\n", strerror(*err));
    }
    return 0;
}
```
# 注意事项

在使用errno_location()时，确保你的编译器和库支持C99标准或更高版本。

在单线程程序中，直接使用errno通常是足够的，但在多线程程序中，使用errno_location()可以避免多线程访问冲突。

使用strerror()函数可以将errno值转换为人类可读的错误描述字符串。这是获取错误信息的有用方法。

总之，虽然直接使用errno在某些情况下是足够的，但在编写需要处理多线程的程序时，使用errno_location()可以提高代码的安全性和健壮性。