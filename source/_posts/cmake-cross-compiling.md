---
title: Linux下基于CMake交叉编译C++程序
date: 2026-03-01 16:39:00
tags: CMake
categories:
  - [C++, CMake]
---

# 概述
记录了一种交叉编译配置方法，用于编译生成ARM环境下执行的c++程序。

---
## 获取交叉编译器
可以从[https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)网站中获取到交叉编译器工具链。
按照需求下载需要的程序安装包，例如：
执行机器：树莓派5
执行环境：aarch64-linux-gnu  |  gcc version 12.2.0 (Debian 12.2.0-14)
编译环境：Ubuntu 20.04 LTS x86_64 
那么对应的安装包就是[arm-gnu-toolchain-12.2.rel1-x86_64-aarch64-none-linux-gnu.tar.xz](https://developer.arm.com/-/media/Files/downloads/gnu/12.2.rel1/binrel/arm-gnu-toolchain-12.2.rel1-x86_64-aarch64-none-linux-gnu.tar.xz?rev=6750d007ffbf4134b30ea58ea5bf5223&hash=6C7D2A7C9BD409C42077F203DF120385AEEBB3F5)

```bash
#在执行环境下执行，可以知道gcc对应的版本
gcc -v
```
解压并安装交叉编译工具

```bash
//解压数据包到当前文件
tar -xvf ./arm-gnu-toolchain-12.2.rel1-x86_64-aarch64-none-linux-gnu.tar.xz

//重命名文件
mv ./arm-gnu-toolchain-12.2.rel1-x86_64-aarch64-none-linux-gnu ./arm-gnu-12.2

//将文件移动的opt路径下
sudo mv ./arm-gnu-12.2 /opt/arm-gnu-12.2
```
注：“arm-gnu-12.2”是自定义的文件名，可以是任意命名

# cmake交叉编译配置文件
创建 arm-gnu-12.2.cmake
```cmake
# 设置系统信息
SET(CMAKE_SYSTEM_NAME Linux)
#设置处理器信息
SET(CMAKE_SYSTEM_PROCESSOR aarch64)

# 指定交叉编译器
SET(CMAKE_C_COMPILER <Full path to the arm gcc compiler>)
SET(CMAKE_CXX_COMPILER <Full path to the arm g++ compiler>)

# 编译环境的根目录，用于设置find_package的查找路径
SET(CMAKE_FIND_ROOT_PATH  <Full path to the arm compiler environment>)

# 在编译环境目录中搜索程序（从不），因为执行文件在当前环境中无法运行
SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
# 设置在编译环境目录中搜索库和头文件（Only)
SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

#设置默认的安装环境（必须加" "）
SET(CMAKE_INSTALL_PREFIX "<Full path to the arm compiler environment>")
```
注：<...>需要替换为本机的路径，< Full path to the arm compiler environment > 独立建一个文件夹就行和编译器路径无关。

#  执行交叉编译

```bash
mkdir arm_build
cd arm_build

#添加工具链
cmake .. -DCMAKE_TOOLCHAIN_FILE=<Path>/arm-gnu-12.2.cmake
make

#如果需要安装的话，默认会安装到交叉编译环境中
make install
```
生成的执行文件直接拷贝到目标环境下并赋予执行权限即可执行。如果存在依赖的动态库，则动态库也需要部署到执行环境中。

# Boost库交叉编译
Boost库很常用因为不是使用cmake编译工具，使用配置方式会不一样。顺带记录一下。

```bash
#进入boost解压后的目录
cd boost_dir

#生成b2。--with-libraries生成所有库 --prefix设置安装路径
./bootstrap.sh --with-toolset=gcc --with-libraries=all --prefix=<Full path to the arm compiler environment>
```
打开同级目录下的project-config.jam文件（文件名不带.数字后缀的，带后缀的是历史副本）
找到

```bash
if ! gcc in [ feature.values <toolset> ]
{
    using gcc; 
}
```
修改为

```bash
if ! gcc in [ feature.values <toolset> ]
{
    using gcc : arm : /opt/arm-gnu-12.2/bin/aarch64-none-linux-gnu-g++ ; 
}
```
注意，编译器路径和‘ ；’之间必须有一个空格，否则会解析错误。

```bash
#编译
./b2 -j8

#安装
./b2 install
```

# Cmake find_package用法

```bash
find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE]
             [REQUIRED] [[COMPONENTS] [components...]]
             [OPTIONAL_COMPONENTS] [components...]
             [NO_POLICY_SCOPE])
```
参数说明:
**PackageName**: 要查找的包名，通常是库的名称，如 OpenCV、Boost 等。
[version]: 可选参数，指定所需的包版本。例如 2.4.3。
[EXACT]: 可选参数，要求版本必须精确匹配。
[QUIET]: 可选参数，如果找不到包，不输出错误信息。
[MODULE]: 可选参数，优先使用 Find<PackageName>.cmake 模块来查找包。
[REQUIRED]: 可选参数，如果找不到包，CMake 会报错并停止构建。
[COMPONENTS]: 可选参数，指定需要查找的包的组件。例如 Boost 库可能有 system、filesystem 等组件。
[OPTIONAL_COMPONENTS]: 可选参数，指定可选的组件。
[NO_POLICY_SCOPE]: 可选参数，不改变策略范围。

找到包后，CMake 会设置一些变量，通常包括：

< PackageName >_FOUND: 表示是否找到包。
< PackageName >_INCLUDE_DIRS: 包含头文件的路径。
< PackageName >_LIBRARIES: 包含库文件的路径。

可以将这些变量用于 include_directories 和 target_link_libraries 等命令中。

例如：
```cmake
find_package(OpenCV REQUIRED)

if(OpenCV_FOUND)
    include_directories(${OpenCV_INCLUDE_DIRS})
    add_executable(my_program main.cpp)
    target_link_libraries(my_program ${OpenCV_LIBRARIES})
endif()
```
# 参考链接：
交叉编译器下载网站：[https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)
Cmake交叉编译配置：[https://www.cnblogs.com/rickyk/p/3875334.html](https://www.cnblogs.com/rickyk/p/3875334.html)
Boost库编译，ARM交叉编译方法： [https://blog.csdn.net/Eng_ingLi/article/details/135479546](https://blog.csdn.net/Eng_ingLi/article/details/135479546)