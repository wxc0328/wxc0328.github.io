---
title: Eigen编程简述以及关键宏
date: 2026-02-28 12:06:46
tags:
categories:
  - [C++, Eigen]
---

# 基本概念
## 公式类型
在`Eigen`中，存在特殊的`expression`(公式类型)，其返回类型为`Derived`(导出类型)。在实现上具体的说，其实际上是由模板元实现，从而避免了传统编程下通过值传递导致效率损失。
假设一个公式`"D=A*B+C"`，这个公式的右侧`"A*B+C"`在`eigen`中实际上是存储为一个对象(构建了一个表达式树)，在`"="`执行时才发生实际的计算，而`"A*B+C"`的类型和`"D"`的类型实际上不同，其是一个非左值对象。
返回值为Vector的，一般称为Vector表达式
返回值为Matrix的，一般称为Matrix表达式
也称为延迟求值（或者惰性求值）。
也意味着使用类似于`“A=A*B”`这种型式，这会导致计算时发生覆盖（实际上对于这种乘法情况eigen会创建一个临时变量使计算正确，但依旧不推荐）
# 基础类型
```c++
//基类
//稠密矩阵基本类型
MatrixBase

//稠密数组基本类型
ArrayBase

//稠密基本类型，（MatrixBase和ArrayBase的基）
DenseBase

//Eigen框架下矩阵类型(是以上的基类)，包括对角矩阵等（可以稀疏存储的矩阵）
EigenBase
```
# 方法
## `Eigen::Vector3f/Eigen::VectorXf`下方法
```c++
//作为对角矩阵返回
.asDiagonal()
```
## DenseBase 下方法
只能作用于稠密类型
```c++
//获取子块，(起始行下标，起始列下标，获取的行数，获取的列数)
//返回的对象不允许赋予非const的引用
.block(x,y,r,c)
```
## 可变对象引用
```
//允许将矩阵的数据内容以引用的方式传递，但不要求是相同的数据类型
Eigen::Ref<EigenType>(Type)
```
# 辅助功能
## 打印矩阵信息
```c++
//使用cout打印矩阵信息
std::cout<<A<<std::endl;
```
## 自定义下标访问规则
```c++
//创建函数的实现
const typename ArgType::Scalar& operator()(Eigen::Index row, Eigen::Index col) const;
```
## 多行列索引
```c++
A =
 1804289383 -1550966999  1365180540   336465782
 -465790871 -1122281286   304089172 -1868760786
 -189735855 -1364114958    35005211    -2309581
  719885386  2044897763 -1852781081  1101513929

//Matrix([rows],[cols]),返回一个Matrix
A([1 2 1], [3 2 1 0 0 2]) =
-1868760786   304089172 -1122281286  -465790871  -465790871   304089172
   -2309581    35005211 -1364114958  -189735855  -189735855    35005211
-1868760786   304089172 -1122281286  -465790871  -465790871   304089172
```
# 特殊的宏
## 编译相关
```c++
//矩阵相关
EIGEN_DEFAULT_DENSE_INDEX_TYPE //默认的稠密矩阵索引类型
EIGEN_DEFAULT_IO_FORMAT //默认的IO格式
EIGEN_INITIALIZE_MATRICES_BY_ZERO //矩阵构造时，使用0初始化
EIGEN_INITIALIZE_MATRICES_BY_NAN //矩阵构造时，使用Nan初始化
EIGEN_NO_AUTOMATIC_RESIZING //禁止矩阵自动调整大小。设置后，赋值矩阵时两侧形状大小必须一致

//要求编译器支持
EIGEN_MAX_CPP_VER //设置最高支持的C++版本号
EIGEN_HAS_C99_MATH //需要支持C99数学函数
EIGEN_HAS_STD_RESULT_OF //需要支持std::result_of
EIGEN_NO_IO //不需要IO

//断言设置
EIGEN_NO_DEBUG //设置不使用断言
EIGEN_NO_STATIC_ASSERT //设置不使用静态断言，并将其转为动态断言
eigen_assert //设置断言的具体行为（用于调整断言触发后的行为）

//内存对齐
EIGEN_MALLOC_ALREADY_ALIGNED //设置要求malloc分配经过对齐的内存，否则由编译器和系统决定
EIGEN_MAX_ALIGN_BYTES //设置最大对齐字节的大小
EIGEN_MAX_STATIC_ALIGN_BYTES //设置最大对齐字节的大小，仅作用于静态分配的数据内存

//计算性能
EIGEN_DONT_PARALLELIZE //设置关闭并行计算，仅当启用OpenMP时生效
EIGEN_DONT_VECTORIZE //设置关闭矢量化，不生成SIMD指令
EIGEN_UNALIGNED_VECTORIZE //设置允许使用非内存对齐的矢量化指令（默认启用）
EIGEN_FAST_MATH //设置使用快速数学计算函数，会影响精度（默认启用）
EIGEN_UNROLLING_LIMIT //设置循环展开的阈值，低于阈值会在编译器中展开循环（默认110）
EIGEN_STACK_ALLOCATION_LIMIT //设置单次栈内存的申请限制（默认128KB）
EIGEN_NO_CUDA //设置关闭CUDA的支持，对于cu文件中定义的仅在host端使用的代码起效
EIGEN_STRONG_INLINE //设置强烈建议内联，针对高频调用的函数
EIGEN_DEFAULT_L1_CACHE_SIZE //设置默认cpu L1缓存大小
EIGEN_DEFAULT_L2_CACHE_SIZE //设置默认cpu L2缓存大小
EIGEN_DEFAULT_L3_CACHE_SIZE //设置默认cpu L3缓存大小
EIGEN_ALTIVEC_ENABLE_MMA_DYNAMIC_DISPATCH //设置运行时动态检测MMA指令的支持，如果不支持自动改为采用VSX指令
EIGEN_ALTIVEC_DISABLE_MMA //设置禁用MMA指令
EIGEN_ALTIVEC_USE_CUSTOM_PACK //设置是否使用 Eigen 针对 AltiVec 优化的自定义数据打包（packing）例程

//插件宏，为Eigen中的类插入额外的自定义方法
//宏名形式EIGEN_xxx_PLUGIN，值为对于方法定义的文件名（一般是h文件，要加""）
EIGEN_ARRAY_PLUGIN 
EIGEN_ARRAYBASE_PLUGIN 
EIGEN_CWISE_PLUGIN 
EIGEN_DENSEBASE_PLUGIN 
EIGEN_DYNAMICSPARSEMATRIX_PLUGIN
EIGEN_FUNCTORS_PLUGIN 
EIGEN_MAPBASE_PLUGIN 
EIGEN_MATRIX_PLUGIN 
EIGEN_MATRIXBASE_PLUGIN 
EIGEN_PLAINOBJECTBASE_PLUGIN 
EIGEN_QUATERNION_PLUGIN 
EIGEN_QUATERNIONBASE_PLUGIN 
EIGEN_SPARSEMATRIX_PLUGIN 
EIGEN_SPARSEMATRIXBASE_PLUGIN 
EIGEN_SPARSEVECTOR_PLUGIN 
EIGEN_TRANSFORM_PLUGIN 
EIGEN_VECTORWISEOP_PLUGIN 

//开发宏
EIGEN_DEFAULT_TO_ROW_MAJOR //设置为内存行优先，默认情况下是列优先
EIGEN_INTERNAL_DEBUGGING //设置启用内部断言，额外安全检查（更加严格），针对内部算法的调用，影响性能
EIGEN_NO_MALLOC //设置编译时检测禁用动态内存分配
EIGEN_RUNTIME_NO_MALLOC //设置运行时检测禁用动态内存分配
```
## 代码检测
```c++
EIGEN_STATIC_ASSERT_FIXED_SIZE(TYPE) //检测类型是否是固定大小
EIGEN_STATIC_ASSERT_DYNAMIC_SIZE(TYPE) //检测类型是否是动态大小
EIGEN_STATIC_ASSERT_LVALUE(Derived) //检测导出类型是否为左值（可被赋值）
EIGEN_STATIC_ASSERT_ARRAYXPR(Derived) //检测导出类型是否为Array类型
EIGEN_STATIC_ASSERT_SAME_XPR_KIND(Derived1, Derived2) //检测导出类型是否相同

EIGEN_STATIC_ASSERT_VECTOR_ONLY(TYPE) //检测类型为Vector
EIGEN_STATIC_ASSERT_VECTOR_SPECIFIC_SIZE(TYPE, SIZE) //检测类型为Vector且具有确定的大小
EIGEN_STATIC_ASSERT_MATRIX_SPECIFIC_SIZE(TYPE, ROWS, COLS) //检测类型为Matrix且具有确定的行列数

EIGEN_STATIC_ASSERT_SAME_VECTOR_SIZE(TYPE0,TYPE1) //检测两个Vector表达式大小是否相同
EIGEN_STATIC_ASSERT_SAME_MATRIX_SIZE(TYPE0,TYPE1) //检测两个Matrix表达式大小是否相同
EIGEN_STATIC_ASSERT_SIZE_1x1(TYPE) //检测类型是否是一个1x1表达式
```
# 并行执行
`Eigen`的并行执行，需要基于`OpenMP`，在编译时启用。
```c++
//设置使用线程数
OMP_NUM_THREADS=n ./my_program //启动前，外部设置
omp_set_num_threads(n);//由OpenMP设置
Eigen::setNbThreads(n);//由Eigen设置

//获取当前采用的线程数
n = Eigen::nbThreads( );
```
## 多线程执行环境
```
//对于eigen3.3以及c++11之前的版本，需要在创建线程之前初始化Eigen
Eigen::initParallel();
```