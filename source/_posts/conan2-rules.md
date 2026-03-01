---
title: conan2简要使用指南
date: 2026-03-01 21:25:00
tags: conan2
categories:
  - [C++, conan2]
---


# 简介
conan2是一款为C++提供的跨平台包管理器，可以通过配置文件的方式获取并管理不同版本的三方库，极大程度上解决了C++在库管理上的痛点。
编译主要的依赖的依旧是cmake这类的工具，可以理解为该工具自动化了cmake的参数配置，使其可以更容易的获取到指定的三方库。
# 安装conan2
## 通过pip直接安装
这种方法最简单，不过需要系统环境中有python，用conda的虚拟环境也可以
```shell
pip install conan
```
## 通过数据包安装
这种方式安装的二进制执行文件中内嵌了一个python执行器，不需要外部依赖。
从[github项目](https://github.com/conan-io/conan)中获取安装包
```shell
wget https://github.com/conan-io/conan/releases/download/2.26.1/conan-2.26.1-linux-x86_64.tgz
tar -xvf conan-2.26.1-linux-x86_64.tgz
mv conan-2.26.1-linux-x86_64 conan
cd conan

# 将下述指令加入~/.bashrc
export PATH=/XXXX:$PATH
# 或者，直接将其拷贝到系统目录下，这样就可以直接找到了
sudo cp ./conan /usr/local/bin/
```
更多方法见[官方安装文档](https://docs.conan.io/2/installation.html)
## 生成编译配置文件
```shell
# 该指令会生成一个名称为default的文件，其中记录着系统环境下的编译器以及相关信息
conan profile detect --force

# 默认情况下会处于~/.conan2/profiles下
```
详细见下文

# 项目编译执行
执行的前提需要有conan配置文件
## 生成构建目录
生成build构建文件夹，并在文件夹内生成相关的配置文件
```shell
conan install . --output-folder=build --build=missing
```
[详情](https://docs.conan.io/2/reference/commands/build.html)
### 检测编译下的构建
```shell
conan install . --build missing -pr:b=default -pr:h=./profiles/raspberry -of arm_build

# -pr:b= 构建平台的编译配置文件
# -pr:h= 目标运行平台的编译配置文件
```
## 编译
```shell
# 激活编译环境
source conanbuild.sh

# 常见的cmake编译流程
cd build
# 将工具链文件传入给cmake,这里同时指定了编译版本
cmake .. -DCMAKE_TOOLCHAIN_FILE=conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Release
cmake --build .

# 销毁编译环境
source deactivate_conanbuild.sh
```
## 执行
```shell
# 激活执行环境，实质上是将相关路径加入PATH
source conanrun.sh

# 这里执行程序，就可以正常发现依赖
./your_app

# 销毁执行环境
source deactivate_conanrun.sh
```
# 其他常用的指令
详细信息见[官方文档](https://docs.conan.io/2/reference/commands.html)
## 创建工程项目
```
# 该指令会直接初始化一个项目
conan new cmake_lib -d name=hello -d version=1.0

# cmake_lib 指定了项目目录风格，其他如cmake_exe等
# -d 表示后面追加参数
# name= 项目名称
# version= 版本号
```
[详情](https://docs.conan.io/2/reference/commands/new.html)
## 一键构建
```shell
conan build .

# 调用conanfile中定义的构建事件
# 会直接调用cmake完成构建，不需要手动激活环境
```
## 部署conan项目
```shell
conan create .

# 指令将当前项目复制进conan缓存，在缓存中，执行生成、构建、部署进本地缓存（打包），执行引用测试
# 部署后其他项目可以通过conan引用该项目
```
## conan项目编辑模式
```
# 将当前项目以编辑模式部署入conan
conan editable add cur_dir_lib

# 实质上是通过软连接加入cache
# 编辑模式，针对的是该项目处于频繁的修改中，且有其他项目通过conan依赖该项目，因此需要避免频繁的标准部署流程
```
## 执行引用测试
```
conan test test_package hello/1.0

# 会编译运行工程目录下的test_package中的程序，将hello/1.0(当前项目)传入
# 以此来检测当前项目能否被正常使用
```
## 列举库信息
```shell
# 该情况下列举的缓存中的库信息
conan list lib_name 
```
[详情](https://docs.conan.io/2/reference/commands/list.html)
## 查看conan编译配置文件的默认路径

```shell
conan profile path default
```
# 相关文件的详细介绍
## conan配置文件
conan配置文件有两种形式，一种.txt，另一种是.py。就功能性而言更推荐后者。
该文件需要放在工作目录下（一般与CMakeLists.txt同级）
在配置文件中指定的库并不能直接被cmake使用，需要像通常情况一样使用`find_package(XXX)`和`target_link_libraries(${PROJECT_NAME} XXX::XXX)`
### conanfile.txt
```
[requires] # 依赖的三方库
zlib/1.3.1 # 库名/版本号
... # 多行

[tool_requires] # 编译工具依赖
cmake/3.27.9 # 工具名/版本号，像这种情况就是指定特定版本的cmake，如果不指定就会使用系统环境下的

[generators] # 要生成的内容
CMakeDeps # 生成项目所依赖的库的安装位置信息， 为find_package()提供定位信息，一般必带
CMakeToolchain # 生成CMake工具链文件，可向CMake传递构建信息，一般必带
...

[conf] # 配置参数
...

```
### conanfile.py
拥有比txt更丰富的功能
```python
from conan import ConanFile
from conan.tools.cmake import CMakeToolchain, CMake, cmake_layout, CMakeDeps

# 继承ConanFile，以此来定制自己的项目配置
class ProjectRecipe(ConanFile):
    name = "ProjectName" # 项目名称
    version = "1.0" # 版本号
    package_type = "library" # 类型说明，这里表明是库

    # Optional metadata
    license = "<Put the package license here>"
    author = "<Put your name here> <And your email here>"
    url = "<Package recipe repository url here, for issues about the package>"
    description = "<Description of sudoku package here>"
    topics = ("<Put some tag here>", "<here>", "<and here>")

    # Binary configuration
    settings = "os", "compiler", "build_type", "arch" # 需要获取的编译信息，可通过self.settings.XXX访问
    options = {"shared": [True], "fPIC": [True, False], "with_pybind11":[True, False]} # 对外开放的编译选项，可自定义，可通过self.options访问
    default_options = {"shared": True, "fPIC": True, "with_pybind11":True} # 编译选项的默认值

    # Sources are located in the same place as this recipe, copy them to the recipe
    exports_sources = "CMakeLists.txt", "src/*", "include/*" # 需要导出的源码位置
    
    def source(self): # 设置和源码相关的事件
	    pass # 例如从git中拉取项目源码

    def config_options(self): # 设置执行时调整编译选项
        if self.settings.os == "Windows":
            del self.options.fPIC # 这里是对平台为Windows时，移除fPIC对象，表现为从未有过该项，在构建时传入该参数会导致异常

    def configure(self): # 设置配置信息，晚于config_options
        if self.options.shared:
            self.options.rm_safe("fPIC") # 不影响传入，执行后该配置项不会再被引入用于计算版本哈希码

    def requirements(self): # 设置依赖项
        if self.options.with_pybind11:
            self.requires("pybind11/3.0.1")
        self.requires("fmt/8.1.1", transitive_headers=True) # transitive_headers表示其所依赖的头文件传递给当前项目，默认情况下是不传递
        self.requires("libuvc/0.0.7",options={"shared": True}) # 此处的options是libuvc内定义
            
    def build_requirements(self): # 设置依赖工具
	    self.tool_requires("...") # 引入编译工具依赖项
	    self.test_requires("gtest/1.17.0") # 引入测试工具依赖项

    def layout(self): # 设置构建目录的布局
        cmake_layout(self) #这是一个泛用型布局模板
    
    def generate(self): # 设置生成内容
        deps = CMakeDeps(self) 
        deps.generate() # 依赖库位置信息
        tc = CMakeToolchain(self)
        tc.variables["PROJECT_EXPORTS"]=True
        if self.options.with_pybind11:
            tc.variables["WITH_PYBIND11"]=True
        tc.generate() # 工具链文件，可以附带传给cmake指定的宏

    def build(self): # 设置编译构建事件，通过conan build .运行
        cmake = CMake(self)
        cmake.configure()
        cmake.build()
        
    def test(self): # 设置和测试相关的事件
	    if can_run(self): 
		    cmd = os.path.join(self.cpp.build.bindir, "example") 
		    self.run(cmd, env="conanrun")

    def package(self): # 设置打包事件，用于部署项目
        cmake = CMake(self)
        cmake.install() # 调用cmake的install,该目标会是conan缓存
        # 此外也可以完成部分的文件拷贝工作

    def package_info(self): # 设置打包信息，用于告知依赖该库的项目应该去哪里找到相关文件
        self.cpp_info.libs = ["project"] # 库文件的名称，对于了生成的libXXX.xxx
        self.cpp_info.libdirs = ["libdir"] # 库文件路径
        self.cpp_info.includedirs = ["include"] #头文件路径
		self.cpp_info.set_property("cmake_target_name", "project::XXX") # 修改cmake中的目标名

```

## 编译配置文件
```
# 这是一个树莓派交叉编译用的profiles，相比默认多出的对编译工具的设置

[settings]
os=Linux # 指定了操作系统
arch=armv8 # 使用的指令架构这里是arm架构，比较常见的有x86_64
compiler=gcc #使用的编译器
compiler.cppstd=gnu17 #c++标准
compiler.libcxx=libstdc++11 #使用的c++库
compiler.version=12 #编译器的版本
build_type=Release #默认的构建类型
[buildenv]
CC=/opt/arm-gnu-12.2/bin/aarch64-none-linux-gnu-gcc-12.2.1 # gcc编译器路径
CXX=/opt/arm-gnu-12.2/bin/aarch64-none-linux-gnu-g++ # g++编译器路径
LD=/opt/arm-gnu-12.2/bin/aarch64-none-linux-gnu-ld # 连接器路径
AR=/opt/arm-gnu-12.2/bin/aarch64-none-linux-gnu-ar # 静态库创建与管理工具
STRIP=/opt/arm-gnu-12.2/bin/aarch64-none-linux-gnu-strip # 符号表与调试信息裁剪工具
SYSROOT=/home/wxc0328/cross_env/raspberry5-sysroot # 系统根路径
```
# 三方库版本的管理
## 指定版本号
```
# 在通过requires引入时，使用的字符串

# 例子
zlib/[~1.2] # 表示的是一个近似版本，只要版本号是1.2.XXXX形式就满足条件
zlib/[<1.2.12] # 表示取小于特定版本的
cmake/[>3.10] # 表示取大于特定版本的
```
## 版本哈希码
比版本号管理更加严格，如果代码发生改变，即使版本号不变，也会导致构建时版本哈希码发生变化，因此可以锚定一个非常确定的版本。
### 获取哈希码信息
```shell
conan list "zlib/1.2.12#*" -r=conancenter
采用#* ，从远程仓库中，列举特定版本库下的哈希信息
```
### 通过哈希码指定版本
```python
self.requires("zlib/1.2.12#87a7211557b6690ef5bf7fc599dd8349")
```
### 锁定当前的库版本
#### 锁定指令
```shell
# 会创建一个锁定文件conan.lock，其中记录三方库的哈希版本
conan lock create .

#有锁定文件存在时，会拉取指定版本
```
# 线上资源
[共享库管理平台](https://conan.io/center)
[官方文档](https://docs.conan.io/2/index.html)
