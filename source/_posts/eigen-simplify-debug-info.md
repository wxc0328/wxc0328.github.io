---
title: Eigen调试信息优化显示
date: 2026-03-03 15:45:00
tags: 
  - Debug
  - Eigen
categories:
  - [C++, Eigen]
---

# 简述
因为Eigen采用了大量模板元编程，所以在调试时，其数据对象的原始堆栈信息非常复杂，在Eigen的官方源码中，实际上提供了优化显示用的配置文件。
# 获取配置文件
```shell
git clone https://gitlab.com/libeigen/eigen.git

cd eigen/debug
tree

# 可见下述内容
.
├── gdb
│   ├── __init__.py
│   └── printers.py
├── lldb
│   └── eigenlldb.py
└── msvc
    ├── eigen.natvis
    └── eigen_autoexp_part.dat
    
# 三者分别对应三者不同调试器的优化方法
# 像msvc，通过IDE来导入natvis即可
```
# 将gdb配置注册到全局
```shell
# 拷贝到用户目录下
mkdir -p "$HOME/.gdbinit.d"
cp ./printers.py "$HOME/.gdbinit.d/printers.py"
cp ./__init__.py "$HOME/.gdbinit.d/__init__.py"

# 创建gdb初始化文件
echo -en "\
python \n\
import sys \n\
sys.path.insert(0, '$HOME/.gdbinit.d') \n\
from printers import register_eigen_printers \n\
register_eigen_printers (None) \n\
end\n" > "$HOME/.gdbinit"
```