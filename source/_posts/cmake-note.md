---
title: CMake笔记
date: 2026-03-01 16:34:00
tags: CMake
categories:
  - [C++, CMake]
---

# 脚本基础功能
## 函数输入参数基本语法
```
< > 表示设置量
< >... 表示一组设置量
[ ] 表示可选的内容
```
## 功能函数
### 设置CMake最低版本限制（必须）
```cmake
cmake_minimum_required(VERSION <min>[...<policy_max>])
# 执行后，设置变量CMAKE_MINIMUM_REQUIRED_VERSION = <min>
```
`min`：要求的最低版本。如果当前使用的`cmake`版本低于最低版本，报错。
`policy_max`（可选）：策略版本。`cmake 3.12`版本以上有效。设置当前`cmake`的行为策略应于策略版本相同，该值必须大于最低版本，否则报错。前面的`...`不能去除。
### 定义工程项目（必须）
```cmake
project(<PROJECT-NAME> [<language-name>...])
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [COMPAT_VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
#执行后
#设置变量PROJECT_SOURCE_DIR，项目源码目录的绝对路径
#设置变量PROJECT_BINARY_DIR，项目二进制目录的绝对路径
#设置变量PROJECT_IS_TOP_LEVEL，（版本3.21起）表示当前项目是否是最高层级（既不是被包含的子项目）
#可选部分
#VERSION -> PROJECT_VERSION,PROJECT_VERSION_MAJOR,PROJECT_VERSION_MINOR,
#           PROJECT_VERSION_PATCH,PROJECT_VERSION_TWEAK,
#           CMAKE_PROJECT_VERSION（版本3.12起）（由最高层级project()设置）
#COMPAT_VERSION -> PROJECT_COMPAT_VERSION
#DESCRIPTION -> PROJECT_DESCRIPTION
#HOMEPAGE_URL -> PROJECT_HOMEPAGE_URL
#
#注，上述中的PROJECT_...形式的变量，具有等效的<PROJECT-NAME>_...变量
```
`PROJECT-NAME`：工程项目名。
`VERSION`（可选）：版本号。
`COMPAT_VERSION`（可选，版本4.1起）：兼容版本号。
`DESCRIPTION`（可选，版本3.9起）：项目内容描述。
`HOMEPAGE_URL`（可选，版本3.12起）：项目规范主页的`URL`。
`LANGUAGES`（可选）：选择构建项目所需的编程语言。不设置情况下默认是`C`和`CXX`。
	可设置值：`C`，`CXX`（`C++`），`CSharp`（`C#`）（版本3.8以上），`CUDA`（版本3.8以上），`OBJC`（`Objective-C`）（版本3.16以上），`OBJCXX`（`Objective-C++`）（版本3.16以上），`Fortran`， `HIP`（版本3.21以上）， `ISPC`（版本3.18以上）， `Swift`（版本3.15以上），`ASM`， `ASM_NASM`， `ASM_MARMASM`（版本3.26以上）， `ASM_MASM`， `ASM-ATT`。

### 添加可执行目标
#### 定义需要构建的可执行目标
```cmake
add_executable(<name> <options>... <sources>...)
```
`name`：可执行目标的名称。
`options`：可选项。
	`WIN32_EXECUTABLE`。构建为`Windows`下的`GUI`可执行文件，以`WinMain()`替代`main()`作为程序入口。
	`MACOSX_BUNDLE`。构建为`macOS`或`iOS`下的程序包。
	`EXCLUDE_FROM_ALL`。是否从项目的所有生成目标中排除，主要影响了`install(TARGETS)`。默认情况是`false`。
`sources`：包含的源码路径

#### 定义由外部导入可执行目标
```cmake
add_executable(<name> IMPORTED [GLOBAL])
#
#主要功能是让 CMake 知道系统中已经存在一个外部的可执行文件，并可以在构建过程中引用它
#通常需要配合 set_target_properties() 来指定可执行文件的路径
```
`name`：可执行目标的名称。
`GLOBAL`（可选）：
	默认情况下，`IMPORTED` 目标的作用域仅限于当前目录及其子目录。如果加上 `GLOBAL`，则该目标在整个`CMake`项目中可见，可以在任何地方使用。

#### 定义可执行目标的别名
```
add_executable(<name> ALIAS <target>)
```
`name`：可执行目标的别名
`target`：可执行目标名。该名称不能是别名。

### 设置变量值
#### 设置普通变量
```cmake
set(<variable> <value>... [PARENT_SCOPE])
```
`variable`：变量名
`value`：变量值。如果不设置变量值，等效于`unset(<variable>)`。
`PARENT_SCOPE`（可选）：使该变量，作用于父（上一级的）作用域。

#### 设置缓存变量
```cmake
set(<variable> <value>... CACHE <type> <docstring> [FORCE])
#缓存变量会持久化存储在 CMakeCache.txt 文件中，
#并在多次配置（configure）过程中保留其值。
#缓存变量会显示在 CMake GUI 或 ccmake 等工具中
```
`variable`：变量名
`value`：变量值
`type`：变量类型
	`BOOL`：布尔型`ON/OFF`
	`FILEPATH`：文件路径
	`PATH`：文件目录路径
	`STRING`：文本字符串
	`INTERNAL`：文本字符串。不会显示在`CMake GUI`中。隐式的包含`FORCE`。
`docstring`：描述该变量的一段文本字符。
`FORCE`（可选）：
	默认情况下（无`FORCE`），如果缓存变量已存在，`set(... CACHE)` 不会修改其值。设置`FORCE`，会强制覆盖。

#### 设置环境变量
```cmake
set(ENV{<variable>} [<value>])
#临时修改环境变量，离开cmake后不影响系统环境
#后续调用$ENV{<variable>}会返回设置的值
```
`variable`：环境变量名
`value`（可选）：环境变量值。不设置时，会将存在的环境变量清空。

### 创建配置文件
```cmake
configure_file(<input> <output>
               [NO_SOURCE_PERMISSIONS | USE_SOURCE_PERMISSIONS |
                FILE_PERMISSIONS <permissions>...]
               [COPYONLY] [ESCAPE_QUOTES] [@ONLY]
               [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF]])
#对输入文件进行内容替换后输出
```
`input`：输入的文件路径
`output`：生成的文件路径
`NO_SOURCE_PERMISSIONS`（可选）：不设置输出文件的访问权限与输入文件相同，采用默认的标准值`644 (-rw-r--r--)`。
`USE_SOURCE_PERMISSIONS`（可选）：设置输出文件的访问权限与输入文件相同。
`FILE_PERMISSIONS`（可选）：使用指定的权限`<permissions>`，赋予输出文件。
`COPYONLY`（可选）：仅拷贝。不对内容中的变量引用进行替换直接拷贝。不可以与`NEWLINE_STYLE`一起使用。
`ESCAPE_QUOTES`（可选）：用反斜杠转义引号（`C-style`）
`@ONLY`（可选）：仅对`@VAR@`形式的生效。针对`${VAR}`形式在脚本中有歧义的场景。
`NEWLINE_STYLE`（可选）：行尾换行符的风格。不可与`COPYONLY`一起使用。
	对`UNIX`或`LF`，采用`\n`换行。
	对`DOS`或`WIN32`或`CRLF`，采用`\r\n`换行。

#### 替换规则
对于样式`@VAR@`、 `${VAR}`、 `$CACHE{VAR}`和表示环境变量的`$ENV{VAR}`，会直接采用变量的值进行替换，如果变量未定义使用空的字符串替换。
```cpp
//将变量名设置为宏
#cmakedefine VAR ...
//替换为
#define VAR ...  /* VAR有定义的情况(或为ON) */
//或
/* #undef VAR */ /* VAR无定义的情况(或为OFF) */
```
```cpp
//将变量名设置为01宏，基于是否定义(或ON/OFF)
#cmakedefine01 VAR
//替换为
#define VAR 0
//或
#define VAR 1
```

### 为目标添加包含的头文件路径
```cmake
target_include_directories(<target> [SYSTEM] [AFTER|BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```
`target`：目标名。必须是由`add_executable()`或者`add_library()`创建，且不能是别名。
`SYSTEM`：告知编译器将路径下的头文件视为系统环境下的头文件，以此来关闭和这些头文件习惯的编译警告。（通常是针对三方库的头文件）
`AFTER|BEFORE`：将这些路径添加到已有路径组的前部或尾部。如果有多个同名文件的情况，靠前的路径会被优先查找使用。
`INTERFACE`：表示该路径下的头文件，仅由依赖者所需。（`C`风格的接口，或者纯头文件定义实现）
`PUBLIC`：表示该路径下的头文件，由自身以及依赖者所需。
`PRIVATE`：表示该路径下的头文件，仅由自身所需。

#### 特化目标导出头文件时的行为
样例：
```cmake
target_include_directories(mylib PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/mylib>
  $<INSTALL_INTERFACE:include/mylib>  # <prefix>/include/mylib
)
# 如果本身不依赖这些头文件，也可以是INTERFACE
```
生成器表达式：
`$<BUILD_INTERFACE:...>`：作用于`export()`，调试时目标可以更便捷的头文件依赖传递。
`$<INSTALL_INTERFACE:...>`：作用于`install()`，针对更正式的安装后的头文件依赖传递。

## 特殊的功能性变量
### C++标准
```cmake
CMAKE_CXX_STANDARD
```
`value`：对应的`C++`标准版本号
	`98`（`C++98`），`11`（`C++11`），`14`（`C++14`），`17`（`C++17`）（版本3.8以上），`20`（`C++20`）（版本3.12以上），`23`（`C++23`）（版本3.20以上），`26`（`C++26`）（版本3.25以上）

### 强制要求C++标准
```cmake
CMAKE_CXX_STANDARD_REQUIRED
```
`value`：布尔型`ON/OFF`或`True/False`

### 项目版本号
```cmake
#完整的项目版本号
PROJECT_VERSION, <PROJECT-NAME>_VERSION 

#项目版本号的第一位数
PROJECT_VERSION_MAJOR, <PROJECT-NAME>_VERSION_MAJOR

#项目版本号的第二位数
PROJECT_VERSION_MINOR, <PROJECT-NAME>_VERSION_MINOR

#项目版本号的第三位数
PROJECT_VERSION_PATCH, <PROJECT-NAME>_VERSION_PATCH

#项目版本号的第四位数
PROJECT_VERSION_TWEAK, <PROJECT-NAME>_VERSION_TWEAK
```