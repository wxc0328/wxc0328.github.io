---
title: Pybind11文档笔记
date: 2026-03-01 16:13:00
tags: pybind11
categories:
  - [C++, pybind11]
---

# 简介
pybind11是一个轻量级的仅头文件C++库，用于生成于基于C++代码的Python调用，使用户在Python中可以更为便利的调用由C/C++实现的对象。

## 支持的核心特性
pybind11可以将下列C++核心特性映射到Python：
- 以自定义数据结构（值、引用、指针形式）作为输入和返回值的函数
- 实例化方法和静态方法
- 函数重载
- 实例化属性和静态属性
- 任意的异常类型
- 枚举
- 回调
- Iteratores（迭代器）和ranges（区间）
- 自定义运算符
- 单继承和多继承
- STL数据结构
- 附带引用计数的智能指针（std::shared_ptr共享指针）
- 具有正确引用计数的内部引用
- 虚类和纯虚类的Python拓展
- 支持反向引入Numpy（NumPy 2 要求 pybind11 2.12+）

# 安装
使用Pypi进行安装：
```shell
pip install pybind11
```

全局下的pip安装：
```shell
pip install "pybind11[global]"
```
该方法默认部署的路径会在“/usr/local/include/pybind11”和“/usr/local/share/cmake/pybind11”下

使用conda-forge安装：
```shell
conda install -c conda-forge pybind11
```

使用vcpkg安装：
```shell
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.sh
./vcpkg integrate install
vcpkg install pybind11
```

# 编译环境配置
## 采用CMake工具进行配置：

```cmake
cmake_minimum_required(VERSION 3.15...3.29)
project(example LANGUAGES CXX)

set(PYBIND11_FINDPYTHON ON)
find_package(pybind11 CONFIG REQUIRED)

pybind11_add_module(example example.cpp)
install(TARGETS example DESTINATION .)
```

需要在系统环境下指定好pybind所在的位置，使得find_package可以正常工作，环境变量为：
```shell
pybind11_DIR=$(path of pybind11Config.cmake)
```

如果使用vcpkg进行三方库管理，应该在cmake设置输入参数：
```shell
-DCMAKE_TOOLCHAIN_FILE=$(VCPKG_ROOT)/scripts/buildsystems/vcpkg.cmake
```
VCPKG_ROOT为vcpkg所在路径。
如果使用的是cmake gui工具，则应该在==第一次点击Configure按钮之前==：
```
点击Add Entry按钮
设置Name: CMAKE_TOOLCHAIN_FILE
设置Type: FILEPATH
设置Value: $(VCPKG_ROOT)/scripts/buildsystems/vcpkg.cmake
```

### 配置项
```cmake
//设置C++标准,可以是11、14、17、20
set(CMAKE_CXX_STANDARD 14 CACHE STRING "C++ version selection") 
//设置C++标准支持性检测，REQUIRED必须的
set(CMAKE_CXX_STANDARD_REQUIRED ON)
//禁用编译器扩展，确保代码符合 C++ 标准，可移植性
set(CMAKE_CXX_EXTENSIONS OFF)

//设置对应的python版本
set(PYBIND11_PYTHON_VERSION 3.7)
//设置python执行器位置
set(PYTHON_EXECUTABLE /path/to/python)

```

## 手动构建
```shell
$ c++ -O3 -Wall -shared -std=c++11 -fPIC $(python3 -m pybind11 --includes) example.cpp -o example$(python3-config --extension-suffix)
```

# 功能语法

## 包含头文件
```
#include <pybind11/pybind11.h>

namespace py = pybind11;//简化名称空间
```

## 创建module模块
语法：
```c++
PYBIND11_MODULE(name, variable, ...)
{
//your code;
}
```
创建pybind11模块，该模块直接对应了python调用的模块。
name：模块名
variable：实例名
注意事项：
- 所以的绑定实现代码都是定义在{}中的
- 必须存在一个模块名和生成动态库的前缀名完全相同，否则Python 将无法正确加载模块
- 同名模块不可以在不同位置或文件中多次定义

样例：
```c++
PYBIND11_MODULE(example, m) {
    m.doc() = "pybind11 example module";

    // Add bindings here
    m.def("foo", []() {
        return "Hello, World!";
    });
}
```

所有绑定的内容可以在python中查看
```python
import example
help(example)
```

## 模块简介
设置模块的功能简介
```cpp
m.doc() = "pybind11 example module";
```

## 函数绑定

绑定方法
```cpp
template <typename Func, typename... Extra>
module_ &def(const char *name_, Func &&f, const Extra &...extra)
```
- name_ ：python的函数名
- f ： C++函数对象
- extra ：扩展参数，处理特定的情况

### 普通函数
样例：
```cpp
int add(int i, int j) {
    return i + j;
}
```
```cpp
m.def("add", &add, "A function that adds two numbers");
```
直接绑定函数，会自动处理输入输出参数，按照变量按照值传递。

### 输入参数绑定键值
```cpp
m.def("add", &add, "A function which adds two numbers",
      py::arg("i"), py::arg("j"));
```
或者
```cpp
using namespace pybind11::literals;
m.def("add", &add, "i"_a, "j"_a);
```
方法py::arg("key")对应了输入参数
导入pybind11::literals名称空间，允许将py::arg()使用字面量表示

### 参数携带默认值
对于简单类型：
```cpp
// regular notation
m.def("add", &add, py::arg("i") = 1, py::arg("j") = 2);
```
或
```cpp
// shorthand
m.def("add", &add, "i"_a=1, "j"_a=2);
```

对于复杂类型
```cpp
py::class_<MyClass>("MyClass")
    .def("myFunction", py::arg("arg") = SomeType(123));
```
这里的SomeType必须先使用py::class_\<SomeType\>完成绑定
会导致help文档中出现：
```
FUNCTIONS
...
|  myFunction(...)
|      Signature : (MyClass, arg : SomeType = <SomeType object at 0x101b7b080>) -> NoneType
...
```
使用argv可以使生成的文档注释更满足人类可读性
```cpp
py::class_<MyClass>("MyClass")
    .def("myFunction", py::arg_v("arg", SomeType(123), "SomeType(123)"));
```

对于空指针需要先预先完成强制转换：
```cpp
py::class_<MyClass>("MyClass")
    .def("myFunction", py::arg("arg") = static_cast<SomeType *>(nullptr));
```

### 返回静态对象的指针
```cpp
/* Function declaration */
Data *get_data() { return _data; /* (pointer to a static data structure) */ }
```
```cpp
m.def("get_data", &get_data, py::return_value_policy::reference);
```
必需要设置return_value_policy::reference策略。
因为pybind默认情况下会采用return_value_policy::automatic策略，这将使得对象在python的生命周期结束后，有python垃圾处理器回收时，由pybind11调用delete释放内存。而_data指向对象为静态（生命周期为全局）释放内存将导致错误崩溃。

### Python对象作为输入参数
以python字典对象为例
C++端接收函数：
```cpp
void print_dict(const py::dict& dict) {
    /* Easily interact with Python types */
    for (auto item : dict)
        std::cout << "key=" << std::string(py::str(item.first)) << ", "
                  << "value=" << std::string(py::str(item.second)) << std::endl;
}
```
绑定
```cpp
m.def("print_dict", &print_dict);
```
可以将python字典对象，以特定的类型传输给c++。
其余查看[[Python C++接口]]。

### 接收 *args 和 **kwargs
```cpp
void generic(py::args args, const py::kwargs& kwargs) {
    /// .. do something with args
    if (kwargs)
        /// .. do something with kwargs
}

/// Binding code
m.def("generic", &generic);
```

### 仅关键字（Keyword-only）输入参数
功能上对应python中以\*作为输入参数的特殊函数。
- \*之后的参数是**仅关键字参数**，必须通过关键字传递。
```python
def f(a, *, b): 
```
参数a可以是位置或者键值，参数b必须是键值
对应：
```cpp
m.def("f", [](int a, int b) { /* ... */ },
      py::arg("a"), py::kw_only(), py::arg("b"));
```

### 仅位置（Positional-only）输入参数
- `/` 之前的参数是**仅位置参数**，必须通过位置传递。
- `/` 和 `*` 之间的参数可以是位置参数或关键字参数。
```python
def f(a, /, b): 
```
对应：
```cpp
m.def("f", [](int a, int b) { /* ... */ },
       py::arg("a"), py::pos_only(), py::arg("b"));
```

### 要求参数不可转变（Non-converting）
为了避免隐式类型转换引入的错误。
```cpp
m.def("floats_only", [](double f) { return 0.5 * f; }, py::arg("f").noconvert());
m.def("floats_preferred", [](double f) { return 0.5 * f; }, py::arg("f"));
```
当floats_preferred输入int值时，会被默认转为浮点继续执行。floats_only输入int值，则会失败触发TypeError。

### 允许/阻止 None 参数
当设置.none(true)时，None参数在C++中会被解释为nullptr空指针
```cpp
m.def("bark", [](Dog *dog) -> std::string {
    if (dog) return "woof!"; 
    else return "(no dog)"; 
}, py::arg("dog").none(true));
m.def("meow", [](Cat *cat) -> std::string {
    return "meow";
}, py::arg("cat").none(false));
```
在python中，调用bark(None)会返回(no dog)，调用meow(None)触发TypeError。

### 重载的调用顺序
对应不同参数类型的同名函数，pybind11倾向于优先调用不需要隐式转换的方法，而后是按照重载定义的顺序现在更早定义的函数方法进行调用。

### 绑定模板函数
C++模板：
```cpp
template <typename T>
void set(T t);
```
绑定时必须显式的将模板实例化，以完成绑定，如：
```cpp
m.def("set", &set<int>);
m.def("set", &set<std::string>);
```

## 类结构体绑定

绑定方法
```cpp
template <typename... Extra>
class_(handle scope, const char *name, const Extra &...extra)
```
scope：所属的模块实例
name：对象名称
Extra：拓展参数

class_实例对应python中的类对象，对象的成员函数可以对class_实例调用.def()进行添加（与函数绑定方法相同）
样例：
```cpp
struct Pet {
    Pet(const std::string &name) : name(name) { }
    void setName(const std::string &name_) { name = name_; }
    const std::string &getName() const { return name; }

    std::string name;
};
```
对应的绑定：
```cpp
py::class_<Pet>(m, "Pet")
        .def(py::init<const std::string &>())
        .def("setName", &Pet::setName)
        .def("getName", &Pet::getName);
```
py::init<...>会指向构造函数

### 绑定lambda函数
将lambda函数绑定到python特殊方法__repr__用于对象的字符串表示
```cpp
py::class_<Pet>(m, "Pet")
    .def(py::init<const std::string &>())
    .def("setName", &Pet::setName)
    .def("getName", &Pet::getName)
    .def("__repr__",
        [](const Pet &a) {
            return "<example.Pet named '" + a.name + "'>";
        }
    );
```

### 实例与静态域（fields）
可以将C++类的成员变量，映射到python类中的实例属性和静态（类）属性。
```cpp
py::class_<Pet>(m, "Pet")
    .def(py::init<const std::string &>())
    .def_readwrite("name", &Pet::name)
```
class_::def_readwrite()表示对象可读写，对应的存在class_::def_readonly()表示对象只读。

class_::def_property()方法可以将函数绑定为field，同理存在class_::def_property_readonly()方法
```cpp
py::class_<Pet>(m, "Pet")
    .def(py::init<const std::string &>())
    .def_property("name", &Pet::getName, &Pet::setName)
```

### 动态属性（成员变量）
python类有一个特性，可以在执行过程中动态的添加实例属性（成员变量）。
但是由pybind11创建的python类对象不能直接这种特性，需要在创建时定义额外的参数。
```cpp
py::class_<Pet>(m, "Pet", py::dynamic_attr())
    .def(py::init<>())
    .def_readwrite("name", &Pet::name);
```
需要在class_后追加设置py::dynamic_attr()

### 继承和向下转型
支持C++中使用继承的结构体，例如：
```cpp
struct Pet {
    Pet(const std::string &name) : name(name) { }
    std::string name;
};

struct Dog : Pet {
    Dog(const std::string &name) : Pet(name) { }
    std::string bark() const { return "woof!"; }
};
```
对应
使用模板的方法表示继承关系
```cpp
py::class_<Pet>(m, "Pet")
   .def(py::init<const std::string &>())
   .def_readwrite("name", &Pet::name);

// Method 1: template parameter:
py::class_<Dog, Pet /* <- specify C++ parent type */>(m, "Dog")
    .def(py::init<const std::string &>())
    .def("bark", &Dog::bark);
```
或
使用类对象的方法表示继承关系
```cpp
py::class_<Pet> pet(m, "Pet");
pet.def(py::init<const std::string &>())
   .def_readwrite("name", &Pet::name);

// Method 2: pass parent class_ object:
py::class_<Dog>(m, "Dog", pet /* <- specify Python parent type */)
    .def(py::init<const std::string &>())
    .def("bark", &Dog::bark);
```

对于通过基类传递给python的实例化对象，只有当基类具有虚函数时，才可以被识别由python完成自动的向下转型。
```cpp
struct PolymorphicPet {
    virtual ~PolymorphicPet() = default;
};

struct PolymorphicDog : PolymorphicPet {
    std::string bark() const { return "woof!"; }
};

// Same binding code
py::class_<PolymorphicPet>(m, "PolymorphicPet");
py::class_<PolymorphicDog, PolymorphicPet>(m, "PolymorphicDog")
    .def(py::init<>())
    .def("bark", &PolymorphicDog::bark);

// Again, return a base pointer to a derived instance
m.def("pet_store2", []() { return std::unique_ptr<PolymorphicPet>(new PolymorphicDog); });
```
```python
>>> p = example.pet_store2()
>>> type(p)
PolymorphicDog  # automatically downcast
>>> p.bark()
'woof!'
```

### 重载函数

需要用户通过强制转换指针显式的告知编译器应该使用哪一个函数
```cpp
py::class_<Pet>(m, "Pet")
   .def("set", static_cast<void (Pet::*)(int)>(&Pet::set), "Set the pet's age")
   .def("set", static_cast<void (Pet::*)(const std::string &)>(&Pet::set), "Set the pet's name");
```

对于兼容C++14的编译器，可以使用py::overload_cast
```cpp
py::class_<Pet>(m, "Pet")
    .def("set", py::overload_cast<int>(&Pet::set), "Set the pet's age")
    .def("set", py::overload_cast<const std::string &>(&Pet::set), "Set the pet's name");
```

如果函数存在const关键字，需要使用py::const_用以区分
```cpp
struct Widget {
    int foo(int x, float y);
    int foo(int x, float y) const;
};

py::class_<Widget>(m, "Widget")
   .def("foo_mutable", py::overload_cast<int, float>(&Widget::foo))
   .def("foo_const",   py::overload_cast<int, float>(&Widget::foo, py::const_));
```

### 内嵌的枚举和类定义
样例：
```cpp
struct Pet {
    enum Kind {
        Dog = 0,
        Cat
    };

    struct Attributes {
        float age = 0;
    };

    Pet(const std::string &name, Kind type) : name(name), type(type) { }

    std::string name;
    Kind type;
    Attributes attr;
};
```
对应
```cpp
py::class_<Pet> pet(m, "Pet");

pet.def(py::init<const std::string &, Pet::Kind>())
    .def_readwrite("name", &Pet::name)
    .def_readwrite("type", &Pet::type)
    .def_readwrite("attr", &Pet::attr);

py::enum_<Pet::Kind>(pet, "Kind")
    .value("Dog", Pet::Kind::Dog)
    .value("Cat", Pet::Kind::Cat)
    .export_values();

py::class_<Pet::Attributes>(pet, "Attributes")
    .def(py::init<>())
    .def_readwrite("age", &Pet::Attributes::age);
```
直接基于class_对象以同样方式定义即可。

enum_::export_values()表示将枚举导出到所属类的作用域下（该行为对应C++中的弱枚举），如果是强枚举类型（使用enum class声明），则不应该使用该扩展。

对enum_使用py::arithmetic()，可以允许枚举的值进行位运算（与、或、非、异或等）
```cpp
py::enum_<Pet::Kind>(pet, "Kind", py::arithmetic())
   ...
```

注：来自pybind11包装的枚举值不要使用is进行比较，应该采用**\==**

### 纯虚类在Python中的使用

纯虚类在C++中属于一种特别的类，其不能被实例化。所以该类也无法被直接绑定到python中，为解决这个问题需要构建一个跳板类由其继承纯虚类。
样例：
```cpp
class Animal {
public:
    virtual ~Animal() { }
    virtual std::string go(int n_times) = 0;
};

class Dog : public Animal {
public:
    std::string go(int n_times) override {
        std::string result;
        for (int i=0; i<n_times; ++i)
            result += "woof! ";
        return result;
    }
};
```
跳板类：
```cpp
class PyAnimal : public Animal {
public:
    /* Inherit the constructors */
    using Animal::Animal;

    /* Trampoline (need one for each virtual function) */
    std::string go(int n_times) override {
        PYBIND11_OVERRIDE_PURE(
            std::string, /* Return type */
            Animal,      /* Parent class */
            go,          /* Name of function in C++ (must match Python name) */
            n_times      /* Argument(s) */
        );
    }
};
```
绑定实现：
```cpp
PYBIND11_MODULE(example, m) {
    py::class_<Animal, PyAnimal /* <--- trampoline*/>(m, "Animal")
        .def(py::init<>())
        .def("go", &Animal::go);

    py::class_<Dog, Animal>(m, "Dog")
        .def(py::init<>());

    m.def("call_go", &call_go);
}
```
class_模板的第二个参数会采用模板元自动检测是第一个参数子类或基类。
需要注意的是，将PyAnimal作为Animal暴露给python时，成员函数的绑定需要使用Animal名称空间下的，而不是PyAnimal。

### 由Python继承类
使用Python去继承由C++暴露的类是可行的，但是必须在构造初始化的时候显式的调用__init__函数，如：
```cpp
class Dachshund(Dog):
    def __init__(self, name):
        Dog.__init__(self)  # Without this, a TypeError is raised.
        self.name = name

    def bark(self):
        return "yap!"
```
对于super()的调用仅当类为线性继承时有效，不要在多继承时使用。

对于想要由python继承的包含虚函数的类都需要定义跳板类来确保行为正确：
```cpp
class Animal {
public:
    virtual std::string go(int n_times) = 0;
    virtual std::string name() { return "unknown"; }
};
class Dog : public Animal {
public:
    std::string go(int n_times) override {
        std::string result;
        for (int i=0; i<n_times; ++i)
            result += bark() + " ";
        return result;
    }
    virtual std::string bark() { return "woof!"; }
};
```
```cpp
class PyAnimal : public Animal {
public:
    using Animal::Animal; // Inherit constructors
    std::string go(int n_times) override { PYBIND11_OVERRIDE_PURE(std::string, Animal, go, n_times); }
    std::string name() override { PYBIND11_OVERRIDE(std::string, Animal, name, ); }
};
class PyDog : public Dog {
public:
    using Dog::Dog; // Inherit constructors
    std::string go(int n_times) override { PYBIND11_OVERRIDE(std::string, Dog, go, n_times); }
    std::string name() override { PYBIND11_OVERRIDE(std::string, Dog, name, ); }
    std::string bark() override { PYBIND11_OVERRIDE(std::string, Dog, bark, ); }
};
```
如果Dog类需要被继承，则Dog类也应该创建对应的跳板类（且需要覆盖所有的虚方法）。
PYBIND11_OVERRIDE_PURE对应纯虚函数
PYBIND11_OVERRIDE对应虚函数

### 自定义构造函数
在C++中有部分类实现模式上不允许直接调用构造，而是要通过特定的函数调用来创建实例化对象（工厂模式），如：
```cpp
class Example {
private:
    Example(int); // private constructor
public:
    // Factory function:
    static Example create(int a) { return Example(a); }
};

py::class_<Example>(m, "Example")
    .def(py::init(&Example::create));
```
可以指定函数作为实例化方法

### 大括号参数
对于C++的结构体而言，很多情况下不会使用构造函数，而是采用{}作为初始化方法。
对于这种情况，利用采用py::init进行适配，而无需改变原代码。
样例：
```cpp
struct Aggregate {
    int a;
    std::string b;
};

py::class_<Aggregate>(m, "Aggregate")
    .def(py::init<int, const std::string &>());
```

### 非公有析构
如果一个对象的析构是非公有的，这种情况一般会提供专用的释放函数。
当采用unique_ptr传递所有权时应该避免其对析构函数的使用，设置py::nodelete。
这种情况下一定要记得主动在C++端释放对象防止内存泄漏。
样例：
```cpp
/* ... definition ... */

class MyClass {
private:
    ~MyClass() { }
};

/* ... binding code ... */

py::class_<MyClass, std::unique_ptr<MyClass, py::nodelete>>(m, "MyClass")
    .def(py::init<>())
```

### 析构函数中调用Python
在C++端的析构函数中调用Python函数，有可能会抛出error_already_set错误。这种情况下应该捕获该异常。
样例：
```cpp
class MyClass {
public:
    ~MyClass() {
        try {
            py::print("Even printing is dangerous in a destructor");
            py::exec("raise ValueError('This is an unraisable exception')");
        } catch (py::error_already_set &e) {
            // error_context should be information about where/why the occurred,
            // e.g. use __func__ to get the name of the current function
            e.discard_as_unraisable(__func__);
        }
    }
};
```

### 隐式转换
在C++在如果Class B拥有以Class A作为唯一参数的构造，那么就存在由A到B的隐式转换关系。
样例：
```cpp
py::class_<A>(m, "A")
    /// ... members ...

py::class_<B>(m, "B")
    .def(py::init<A>())
    /// ... members ...

m.def("func",
    [](const B &) { /* .... */ }
);
```
如果是C++函数，func可以隐式的使用A作为输入参数，但是这一特性无法直接映射到Python中，如果需要，添加：
```cpp
py::implicitly_convertible<A, B>();
```
是Python可以知道这种隐式转换关系。

### 附加静态函数
使用lambda函数捕获默认的self参数，封装调用目标静态函数。
```cpp
py::class_<Foo>(m, "Foo")
    .def_property_readonly_static("foo", [](py::object /* self */) { return Foo(); });
```

### 运算符重载
C++端类需要有对应的operator定义。
样例：
```cpp
#include <pybind11/operators.h>

PYBIND11_MODULE(example, m) {
    py::class_<Vector2>(m, "Vector2")
        .def(py::init<float, float>())
        .def(py::self + py::self)
        .def(py::self += py::self)
        .def(py::self *= float())
        .def(float() * py::self)
        .def(py::self * float())
        .def(-py::self)
        .def("__repr__", &Vector2::toString);
}
```
其中
```cpp
.def(py::self * float())
```
是以下形式的一种速记符
```cpp
.def("__mul__", [](const Vector2 &a, float b) {
    return a * b;
}, py::is_operator())
```
使用速记符必须包含pybind11/operators.h

### Pickling支持
Python的Pickle模块，可以将Python 对象与二进制数据间进行序列化与反序列化。
pybind11可以提供对pickle和unpickle的定义支持。
样例：
```cpp
class Pickleable {
public:
    Pickleable(const std::string &value) : m_value(value) { }
    const std::string &value() const { return m_value; }

    void setExtra(int extra) { m_extra = extra; }
    int extra() const { return m_extra; }
private:
    std::string m_value;
    int m_extra = 0;
};
```
对应：
```cpp
py::class_<Pickleable>(m, "Pickleable")
    .def(py::init<std::string>())
    .def("value", &Pickleable::value)
    .def("extra", &Pickleable::extra)
    .def("setExtra", &Pickleable::setExtra)
    .def(py::pickle(
        [](const Pickleable &p) { // __getstate__
            /* Return a tuple that fully encodes the state of the object */
            return py::make_tuple(p.value(), p.extra());
        },
        [](py::tuple t) { // __setstate__
            if (t.size() != 2)
                throw std::runtime_error("Invalid state!");

            /* Create a new C++ instance */
            Pickleable p(t[0].cast<std::string>());

            /* Assign any additional state */
            p.setExtra(t[1].cast<int>());

            return p;
        }
    ));
```
python调用：
```python
import pickle

p = Pickleable("test_value")
p.setExtra(15)
data = pickle.dumps(p)
```

### 深拷贝支持
可以定义\__copy\__和\__deepcopy\__方法。
样例：
```cpp
py::class_<Copyable>(m, "Copyable")
    .def("__copy__",  [](const Copyable &self) {
        return Copyable(self);
    })
    .def("__deepcopy__", [](const Copyable &self, py::dict) {
        return Copyable(self);
    }, "memo"_a);//没有拷贝动态属性
```

### 多继承
基于模板的声明：
```cpp
py::class_<MyType, BaseType1, BaseType2, BaseType3>(m, "MyType")
   ...
```
该模式下，基类没有顺序要求，模板会自动检测依赖关系。需要基类是在pybind11中声明过的类型

如果一个类在C++中原本是多继承的，但在绑定时仅希望暴露一个基类，pybind11允许该操作，但需要额外标识multiple_inheritance。
```cpp
py::class_<MyType, BaseType2>(m, "MyType", py::multiple_inheritance());
```

### 绑定局部模块类
pybind11可以将相互使用生成自不同pybind11模块的类。
样例：
```cpp
// In the module1.cpp binding code for module1:
py::class_<Pet>(m, "Pet")
    .def(py::init<std::string>())
    .def_readonly("name", &Pet::name);
```
```cpp
// In the module2.cpp binding code for module2:
m.def("create_pet", [](std::string name) { return new Pet(name); });
```
```python
>>> from module1 import Pet
>>> from module2 import create_pet
>>> pet1 = Pet("Kitty")
>>> pet2 = create_pet("Doggy")
>>> pet2.name()
'Doggy'
```
这种情况下，module1定义了Pet类，module2定义了Pet类的一个工厂函数。
需要注意的是module2不可以重复去定义Pet类（pybind11下的）。

如果需要允许这种重复定义，可以将Pet定义为module_local：
```
// dogs.cpp
py::class<pets::Pet>(m, "Pet", py::module_local())
    .def("name", &pets::Pet::name);

// Binding for local extension class:
py::class<Dog, pets::Pet>(m, "Dog")
    .def(py::init<std::string>());
```
```
// cats.cpp
py::class<pets::Pet>(m, "Pet", py::module_local())
    .def("get_name", &pets::Pet::name);

// Binding for local extending class:
py::class<Cat, pets::Pet>(m, "Cat")
    .def(py::init<std::string>());
```
这会导致模块不能将Pet传递给python，对于Python而言两个模块导入的Pet是完全不同的实例。不过这个认知区别不影响C++端，C++依旧会将其转向内部类型。
```cpp
m.def("pet_name", [](const pets::Pet &pet) { return pet.name(); });
```
```python
>>> import cats, dogs, frogs  # No error because of the added py::module_local()
>>> mycat, mydog = cats.Cat("Fluffy"), dogs.Dog("Rover")
>>> (cats.pet_name(mycat), dogs.pet_name(mydog))
('Fluffy', 'Rover')
>>> (cats.pet_name(mydog), dogs.pet_name(mycat), frogs.pet_name(mycat))
('Rover', 'Fluffy', 'Fluffy')
```
C++端函数的调用，多态性依旧是可以正常使用的。

### 绑定保护成员函数
无法直接做到这一点。
可以创建新的C++派生类将函数权限提升到publish
```cpp
class A {
protected:
    int foo() const { return 42; }
};

class Publicist : public A { // helper type for exposing protected functions
public:
    using A::foo; // inherited with different access modifier
};

py::class_<A>(m, "A") // bind the primary class
    .def("foo", &Publicist::foo); // expose protected methods via the publicist
```

### 绑定final类
C++中使用final告知编译器类型不可以再被继承。
通过py::is_final可以告知Python该类型不应该被继承（否则抛出TypeError）。
```cpp
class IsFinal final {};

py::class_<IsFinal>(m, "IsFinal", py::is_final());
```

### 绑定模板类
```cpp
struct Cat {};
struct Dog {};

template <typename PetType>
struct Cage {
    Cage(PetType& pet);
    PetType& get();
};
```
需要将模板类实例化，且显式的区分类型：
```cpp
// ok
py::class_<Cage<Cat>>(m, "CatCage")
    .def("get", &Cage<Cat>::get);

// ok
py::class_<Cage<Dog>>(m, "DogCage")
    .def("get", &Cage<Dog>::get);
```

## 返回值策略

|  返回值策略   |  详细   |
| --- | --- |
| return_value_policy::take_ownership | 返回堆对象的引用并移交所有权给python，由python管理生命周期。移交后c++端应该不再管理该对象内存，否则会引发未定义行为（避免类似shared_ptr的使用）。 |
| return_value_policy::copy | 将返回对象创建在python在的拷贝 |
| return_value_policy::move | 使用std::move移交所有权给python(强制性)。|
| return_value_policy::reference | 返回对象的引用。所有权依旧属于c++端管理 |
| return_value_policy::reference_internal | 返回对象成员变量的引用，其生命周期与持有其的对象相同（可以防止在python中父对象被提前回收） |
| return_value_policy::automatic |  返回指针时采用take_ownership，返回右值时采用move， 返回左值时采用copy。类装饰器采用的默认策略 |
| return_value_policy::automatic_reference | 返回指针时采用reference，其余与automatic相同 |

## 调用策略
针对输入参数提供的策略。

### 保持存活（Keep alive）
主要针对类对象是容器的情况，告知python在成员函数被调用后容器类持有了输入对象，使垃圾回收系统可以正确工作。
样例：
```cpp
py::class_<List>(m, "List")
    .def("append", &List::append, py::keep_alive<1, 2>());
```
索引1用于指向隐式的this指针，索引2指向输入参数

### 调用守卫 （Call guard）
可以在调用时先创建一个守卫对象，调用结束时守卫对象释放。
```cpp
m.def("foo", foo, py::call_guard<T>());
```
等效于
```cpp
m.def("foo", [](args...) {
    T scope_guard;
    return foo(args...); // forwarded arguments
});
```

建议与gil_scoped_release结合使用，用于提前释放python的全局解释器锁（GIL）,以获取更好的性能，样例：
```cpp
m.def("call_go", &call_go, py::call_guard<py::gil_scoped_release>());
```
函数call_go执行时间长，应提前释放GIL。

允许py::call_guard<T1, T2, T3...>形式的调用，以从左往右的顺序构造，相反顺序析构。

## 异常处理
pybind11会将std::exception以及其标准环境下的子类，自动的捕获并转换为python异常抛出。

|C++异常类型|转换后的Python异常类型|
|---|---|
|`std::exception`|`RuntimeError`|
|`std::bad_alloc`|`MemoryError`|
|`std::domain_error`|`ValueError`|
|`std::invalid_argument`|`ValueError`|
|`std::length_error`|`ValueError`|
|`std::out_of_range`|`IndexError`|
|`std::range_error`|`ValueError`|
|`std::overflow_error`|`OverflowError`|
|`pybind11::stop_iteration`|`StopIteration` (用于实现自定义的迭代器)|
|`pybind11::index_error`|`IndexError` (表示在 `__getitem__`, `__setitem__`等，发生的越界访问)|
|`pybind11::key_error`|`KeyError` (表示在字典类型 `__getitem__`, `__setitem__`等，发生的越界访问)|
|`pybind11::value_error`|`ValueError` (表示在 `container.remove(...)`中传递错误的值)|
|`pybind11::type_error`|`TypeError`|
|`pybind11::buffer_error`|`BufferError`|
|`pybind11::import_error`|`ImportError`|
|`pybind11::attribute_error`|`AttributeError`|
|Any other exception|`RuntimeError`|

这种异常转换不是双向的，无法用C++的标准异常来捕获python的异常，
如果需要捕获请使用`pybind11::error_already_set`
样例：
```cpp
try {
    // open("missing.txt", "r")
    auto file = py::module_::import("io").attr("open")("missing.txt", "r");
    auto text = file.attr("read")();
    file.attr("close")();
} catch (py::error_already_set &e) {
    if (e.matches(PyExc_FileNotFoundError)) {
        py::print("missing.txt not found");
    } else if (e.matches(PyExc_PermissionError)) {
        py::print("missing.txt found but not accessible");
    } else {
        throw;
    }
}
```
这部分代码是经由pybind11在C++中调用了python代码，py::error_already_set会捕获python异常。
需要注意的是，通过：
```
throw py::value_error("The ball");
```
抛出的异常，实际上是C++异常，需要通过py::value_error进行捕获。

### 注册自定义的转换
当提供的默认异常无法清晰表示异常类型的情况下，自定义的C++异常类型也可以转为特定的Python异常类型。
```cpp
py::register_exception<CppExp>(module, "PyExp");
```
CppExp：是自定义的C++异常类型
module：是pybind11创建的模块
PyExp：是对应Python的异常类型（由pybind11创建）

相似于类的情况，异常也可以定义为属于模块内部的，使用：
```cpp
py::register_local_exception<CppExp>(module, "PyExp");
```
如果希望异常可以由其基类捕获，可以使用第三个参数来设置创建Python异常的基类：
```cpp
py::register_exception<CppExp>(module, "PyExp", PyExc_RuntimeError);
py::register_local_exception<CppExp>(module, "PyExp", PyExc_RuntimeError);
```
PyExp会被设置为RuntimeError的子类

还允许注册使用更复杂的对异常类型的转换函数register_exception_translator和register_local_exception_translator，详细见
[[https://pybind11.readthedocs.io/en/stable/advanced/exceptions.html]]

注册局部异常转换和全局异常转换的区别，局部异常具有比全局异常更高的转换优先级。
如果两个模块都定义全局异常转换函数，会将python导入的顺序作为优先级（先导入的优先级更高）

## 智能指针

### std::unique_ptr
可以使用std::unique_ptr作为返回值，由pybind11将数据递交给Python。
这种情况表示将对象的所有权完全交由pybind11管理，当Python中的引用计数归零就会自动释放。
```cpp
std::unique_ptr<Example> create_example() { return std::unique_ptr<Example>(new Example()); }
```
不能使用std::unique_ptr作为函数的输入参数。

### std::shared_ptr
使用std::shared_ptr作为返回值，表示pybind11将会通过引用计数，只有引用计数归零时才会释放。

std::shared_ptr也可以作为pybind11将对象绑定的默认方式
```cpp
py::class_<Example, std::shared_ptr<Example> /* <- holder type */> obj(m, "Example");
```
这种情况下，pybind11会将传递的对象先装入std::shared_ptr再暴露给Python（默认的行为是采用std::unique_ptr）
以上情况下，如果对象已经作为std::shared_ptr绑定，那么不要以其他方式（尤其是原始指针形式），这对导致对同一段内存产生两个std::shared_ptr（最终导致析构两次）
或者，使类继承`std::enable_shared_from_this`:
```cpp
class Child : public std::enable_shared_from_this<Child> { };
```
`std::enable_shared_from_this`通常会在内部为类保存一个弱引用weak_ptr，以允许该类可以通过原始指针传递给其他的std::shared_ptr后，依旧可以正确完成内存释放。
见[[https://www.apiref.com/cpp-zh/cpp/memory/enable_shared_from_this.html]]


## 类型转换
大致上分为3种情况：
- 原生C++类型和Python装饰器
- C++装饰器和Python原生类型
- 原生C++类型和原生Python类型

|Python类型|可转换C++类型|
| --- | --- |
|`str`|`std::string`或`char*`|
|`bytes`|`std::string`或`char*`|
|`chr()`|`char`|
|``|``|

|C++类型|转换的Python类型|
| --- | --- |
|`int8_t`, `uint8_t`|`int`|
|`int16_t`, `uint16_t`|`int`|
|`int32_t`, `uint32_t`|`int`|
|`int64_t`, `uint64_t`|`int`|
|`float`, `double`|`float`|
|`bool`|`bool`|
|`std::string`|`str`|
|`char*`(不能是非空字符结尾)|`str`|
|`py::bytes`|`bytes`|

# 注意事项

- 来自pybind11包装的枚举值不要使用is进行比较，应该采用**\==**