---
title: Eigen结构体
date: 2026-02-28 21:40:00
tags: Eigen
categories:
  - [C++, Eigen]
---

# 矩阵 Matrix
该类型用于存储一个稠密矩阵
```c++
//基本型式
Matrix<typename Scalar, int Rows, int Cols> //常用的表达形式

//详细
Matrix<typename Scalar, //类型
       int RowsAtCompileTime, //行数，值为Dynamic表动态
       int ColsAtCompileTime, //列数，同上
       int Options = 0, //数据布局，RowMajor行优先，ColMajor列优先，默认情况下是列优先
       int MaxRowsAtCompileTime = RowsAtCompileTime, //针对动态大小，预分配最大上限
       int MaxColsAtCompileTime = ColsAtCompileTime>
```
## 分类
### 静态大小
在编译时确定对应的行列数，内存会优先分配在栈上（过大的会分配到堆上）
### 动态大小
在编译时，对于的行列数设置为Dynamic，由运行时确定，内存会分配在堆上
## 向量 Vector 
行或列维数为一的矩阵
## 简化名称

```c++
//N表示静态大小(可以是2,3,4)，X表示动态大小
typedef Matrix<type, N, N> MatrixNt //N阶方阵
typedef Matrix<type, Dynamic, Dynamic> MatrixXt //方阵
typedef Matrix<type, Dynamic, N> MatrixXNt //具有N列的矩阵
typedef Matrix<type, N, Dynamic> MatrixNXt //具有N行的矩阵

typedef Matrix<type, N, 1> VectorNt //N维列向量
typedef Matrix<type, Dynamic, 1> VectorXt //列向量
typedef Matrix<type, 1, N> RowVectorNt //N维行向量
typedef Matrix<type, 1, Dynamic> RowVectorXt //行向量

//数据类型t
i //int
f //float
d //double
cf //complex<float>
cd //complex<double>

```
## 方法
### 构造
```c++
Matrix3f a; //创建3x3方阵，默认情况下内存不初始化
MatrixXf b; //创建动态方阵，当前大小为0

MatrixXf a(10,15); //创建动态方阵，大小为10x15
VectorXf b(30); //创建动态列向量，大小为30

//创建固定大小的矩阵，并设置初始值
//该形式下，仅支持最高4位的初始值
Vector2d a(5.0, 6.0); 
Vector3d b(5.0, 6.0, 7.0);
Vector4d c(5.0, 6.0, 7.0, 8.0);

//固定大小矩阵，通过数组设置初始值
Matrix<int, 5, 1> b {1, 2, 3, 4, 5};
Matrix<int, 1, 5> c = {1, 2, 3, 4, 5};

//由二维数组设置初始值
//动态矩阵
MatrixXi a {      // construct a 2x2 matrix
      {1, 2},     // first row
      {3, 4}      // second row
};
//静态矩阵
Matrix<double, 2, 3> b {
      {2, 3, 4},
      {5, 6, 7},
};
```
### 系数访问
```c++
Eigen::MatrixXd m(...);
m(row_index, col_index); //访问矩阵中，位于row_index行col_index列下的系数，可以作为左值
m(index); //访问矩阵中特定行(或列)的一组系数，取决于存储格式，行优先时返回一行，列优先时返回一列，默认列优先

Eigen::VectorXd v(...);
v(index); //访问向量中，下标为0的系数，可以作为左值
```
### 流式赋值
```c++
Matrix3f m;
m << 1, 2, 3, 4, 5, 6, 7, 8, 9; //通过流式传输，为矩阵赋值

RowVectorXd vec1(3),vec2(4);
RowVectorXd joined(7);
joined << vec1, vec2; //可以多个进行拼接

MatrixXf matA(2, 2);
matA << 1, 2, 3, 4;
MatrixXf matB(4, 4);
matB << matA, matA / 10, matA / 10, matA;
std::cout << matB << std::endl; //矩阵拼接
//matB:
//  1   2 0.1 0.2
//  3   4 0.3 0.4
//0.1 0.2   1   2
//0.3 0.4   3   4

//如果使用<<赋值是一个中间变量，则需要调用finished()方法显式告知赋值完成
mat = (MatrixXf(2, 2) << 0, 1, 1, 0).finished() * mat
```
### 生成随机矩阵
```c++
MatrixXf::Random(rows, cols); //生成一个随机矩阵
m.setRandom(...);//将矩阵自身置为随机，可以选定需要随机的范围
```
### 生成特殊矩阵
```c++
//需要根据具体的MatrixType或者ArrayType，使用对应的静态方法（参数会略有不同）
MatrixType::Zero(...) //生成全零矩阵
MatrixType::Constant(...) //生成具有同一数值的矩阵
VectorXt::LinSpaced(_size_,low,high) //生成线性差值的向量
MatrixType::Identity() //生成单位矩阵，对于非方阵的情况（以左上角为1，依次向右下填充，其余为0）

//对于调整矩阵自身，有setXXX()方法
```
### 矩阵大小
```c++
Eigen::MatrixXd m(2, 5);
m.rows(); //获取行数
m.cols(); //获取列数
m.size(); //获取系数个数
m.resize(rows,cols); //重置矩阵形状，原系数可能会改变
m.conservativeResize(rows,cols) //重置矩阵形状，原系数不会改变，rows和cols可以设置为NoChange_t(不改变)
//矩阵的赋值操作会自动调整矩阵大小

//对静态矩阵调整大小会导致错误
```
### 调整矩阵形状
```c++
DenseBase::reshaped(NRowsType,NColsType) //返回一个调整形状后的矩阵，调整后系数的分布取决于存储格式是行优先还是列优先

DenseBase::reshaped() //返回一个调整后的列向量
DenseBase::reshaped<Order>() //ColMajor 表示视为内存采用列优先存储的方式读取，RowMajor 表示视为内存采用行优先存储的方式读取

//对自身赋值时，会导致混淆
```
### 矩阵运算
```c++
Eigen::Matrix2d a;
a << 1, 2, 3, 4;
Eigen::MatrixXd b;
b << 2, 3, 1, 4;
double val = 1.0;
```
#### 同型矩阵间加减
```c++
a + b //矩阵间的加法
a - b //矩阵间的减法
a += b //矩阵自加
a -= b //矩阵自减
```
#### 矩阵与系数乘除法

```c++
-a //矩阵系数乘以-1
a * val;
val * a; // 矩阵系数乘以val
a / val; // 矩阵系数除以val
a *= val; //矩阵自乘以系数val
a /= val; //矩阵自除以系数val
```
#### 矩阵间乘法
```c++
Eigen::Matrix2d mat(...);
Eigen::Vector2d v(...);

//矩阵间乘法，需要数学上可相乘（左矩阵列数等于右矩阵行数）
mat * mat
mat * v 

//特殊行为
m=m*m //不会发生混淆，因为乘法会被自动添加临时变量
	tmp = m*m;
	m = tmp;
//因此不会发生混淆
//如果左值和右侧公式不存在混淆，添加临时变量行为会导致性能下降
c.noalias() += a * b; //使用noalias()可以减少内存的拷贝
```
#### 点乘和叉乘
```c++
Eigen::Vector3d v(1, 2, 3);
Eigen::Vector3d w(0, 1, 2);

v.dot(w) //v和w,进行点乘，得到数值，等效于
	v.adjoint() * w

v.cross(w) //进行叉乘
```
#### 矩阵的分组
##### 
#### 求矩阵的特殊值
##### 矩阵分部分
```c++
mat.colwise() //获取由矩阵列组成的一组数据，在其上的算法单独应用于每一列
mat..rowwise() //获取由矩阵行组成的一组数据，在其上的算法单独应用于每一行

//例子
mat.colwise().maxCoeff() //对每一列求最大值，得到由每列最大值构成的行向量
mat.rowwise().maxCoeff() //得到由每行最大值构成的列向量
```

```c++
Eigen::MatrixXf mat(2, 4);
Eigen::VectorXf v(2);
 
mat << 1, 2, 6, 9, 3, 1, 7, 2;
v << 0, 1;
mat.colwise() += v
```
实现了矩阵加法（第二行系数加一）
$$
\begin{bmatrix} 1 & 2 & 6 & 9 \\ 3 & 1 & 7 & 2 \end{bmatrix}
+ \begin{bmatrix} 0 & 0 & 0 & 0 \\ 1 & 1 & 1 & 1 \end{bmatrix}
= \begin{bmatrix} 1 & 2 & 6 & 9 \\ 4 & 2 & 8 & 3 \end{bmatrix}
$$
##### 常见的特殊值
```c++
mat.sum() //矩阵系数的和
mat.prod() //矩阵系数的乘积
mat.mean() //矩阵系数的平均值
mat.minCoeff(...) //矩阵的最小系数，可以传入参数获取对应下标
mat.maxCoeff(...) //矩阵的最大系数，同上
mat.trace() //矩阵的迹（对角元素和）
a.diagonal() //获取矩阵所有的对角元素
```
##### 求范数
Lp范数的公式
$$\|\mathbf{x}\|_p = \left( \sum_{i=1}^{n} |x_i|^p \right)^{1/p}$$
```c++
m.norm() //L2范数（欧几里得范数）
m.squaredNorm() //L2范数的平方
m.lpNorm<1>() //L1范数
m.lpNorm<Eigen::Infinity>() //无穷范数,退化为取绝对值最大
```
##### 求布尔值
```c++
//对矩阵系数进行条件判断，如(a > 0)对一个系数判断其是否大于0
(a > 0).all() //都为true时，返回true
(a > 0).any() //有任意一个true时，返回true
(a > 0).count() //为true的系数个数
```
#### 获取特殊矩阵

| 特殊矩阵         | 符号             |                   |              |
| ------------ | -------------- | ----------------- | ------------ |
| 转置矩阵         | $A^T$          | Transpose         | .transpose() |
| 共轭矩阵         | $\overline{A}$ | Conjugate         | .conjugate() |
| 共轭转置（埃尔米特伴随） | $A^H$          | Hermitian adjoint | .adjoint()   |
| 古典伴随矩阵       | $A^*$          | Adjugate matrix   | .adjugate()  |

```c++
a.transpose() //获取矩阵的转置矩阵
a.transposeInPlace() //对矩阵自身进行转置，要求矩阵是动态的

a.conjugate() //获取矩阵的共轭矩阵

a.adjoint() //获取矩阵的共轭转置（埃尔米特伴随）
a.transposeInPlace() //对矩阵自身进行共轭转置

a.adjugate() //获取矩阵的古典伴随矩阵
```
#### 提前求值
```c++
//因为eigen的惰性求值机制，在处理如a = a.transpose()会导致混淆
//使用eval()可以使公式提前计算，得到一个临时变量，避免数值错误
a.transpose().eval() 
```
# 数组 Array
## 和矩阵 Matrix的区别
数组相对于矩阵，更关注的是其上每一系数的计算。
在表现形式上，主要区别是数组间的加减乘是按位（一一对应）进行的。
## 结构

| 类型                            | 简化名称     |          |
| ----------------------------- | -------- | -------- |
| Array<float,Dynamic,1>        | ArrayXf  |          |
| Array<float,3,1>              | Array3f  | ArrayNf  |
| Array<double,Dynamic,Dynamic> | ArrayXXd |          |
| Array<double,3,3>             | Array33d | ArrayNNd |

## 方法
### 和数组之间的转换
```c++
MatrixXf m(...);
m.array() //矩阵获取其数值形式

Eigen::ArrayXf a(...);
a.matrix() //数组获取其矩阵形式
```
#### 数组间加减乘
```c++
Eigen::ArrayXXf a(3, 3);
Eigen::ArrayXXf b(3, 3);
float val;

a + b //数组间系数按位相加
a - val //数组每一个系数减去val
a * b //系数按位相乘

```
### 数值计算
```c++
a.abs() //系数求绝对值
a.sqrt() //系数求平方根
a.min(other_array) //系数间比较取最小值
```
# 块 Block （切片对象）
## 功能
用于在矩阵或数组中，获取一个局部的数据块。
## 序列类型 Seq
```c++
//用于生成一段序列号，可以基于序列号获取切片
seq(firstIdx,lastIdx) //生成从firstIdx开始到lastIdx结束的序列，步长为1
seq(firstIdx,lastIdx,incr) //生成从firstIdx开始到lastIdx结束的序列，指定步长
seqN(firstIdx,size) //由firstIdx向后size个序列，步长1
seqN(firstIdx,size,incr) ////由firstIdx向后size个序列，指定步长
```
## 赋值
在赋值时，和Matrix一样支持使用`<<`进行流式赋值。
## 获取 Block
### 基本方法
```c++
//矩阵和数组都可以

//指定左上角起始位置获取指定大小Block
m.block(i,j,p,q); //获取动态大小Block
m.block<p,q>(i,j); //获取静态大小Block

//获取行列Block
m.row(i); //获取指定行
matrix.col(j); //获取指定列
```
### 基于序列获取

| 方法                | 代码                                        | 等价方法                               |
| ----------------- | ----------------------------------------- | ---------------------------------- |
| 左下角由i行开始的n列       | `A(seq(i,last), seqN(0,n))`               | `A.bottomLeftCorner(A.rows()-i,n)` |
| 由i,j开始的m行n列       | `A(seqN(i,m), seqN(j,n))`                 | `A.block(i,j,m,n)`                 |
| 由i0,j0开始，由i1,j1结束 | A(`seq(i0,i1), seq(j0,j1))`               | `A.block(i0,j0,i1-i0+1,j1-j0+1)`   |
| 偶数列               | `A(all, seq(0,last,2))`                   |                                    |
| 前n个奇数行            | `A(seqN(1,n,2), all)`                     |                                    |
| 倒数第二列             | `A(all, last-1)`                          | `A.col(A.cols()-2)`                |
| 中间行               | `A(last/2, all)`                          | `A.row((A.rows()-1)/2)`            |
| 向量由i位起始后的所有元素     | `v(seq(i,last))`                          | `v.tail(v.size()-i)`               |
| 向量后n个元素           | `v(seq(last+1-n,last))`<br>`v(lastN(n)) ` | `v.tail(n) `                       |
| 右下角的m行n列矩阵        | `A(lastN(m), lastN(n))`                   |                                    |
| 后n列（经过3个获取一列）     | `A(all, lastN(n,3))`                      |                                    |
上述的`all`，在代码中使用`Eigen::placeholders::all`
序列对象可以使用`c/c++`风格`{...},std::vector<int>, std::array<int,N>`替换，也可以是`ArrayXi`
可以使用`Eigen::fix<val>`,使编译时确定需要的序列号。`Eigen::last`表示从矩阵末尾开始，表达形式为`v(seq(last-fix<7>, last-fix<2>))`
#### 基于序列获取逆序
```c++
A(all, seq(20, 10, fix<-2>)) //由20起向10，间隔一个取一个
A(seqN(last, n, fix<-1>), all) //所有行逆序
A(lastN(n).reverse(), all) //所有行逆序
```
### 特殊方法

| 方法功能    | 动态大小                             | 静态大小                               |
| ------- | -------------------------------- | ---------------------------------- |
| 左上角p行q列 | `matrix.topLeftCorner(p,q);`     | `matrix.topLeftCorner<p,q>();`     |
| 左下角p行q列 | `matrix.bottomLeftCorner(p,q);`  | `matrix.bottomLeftCorner<p,q>();`  |
| 右上角p行q列 | `matrix.topRightCorner(p,q);`    | `matrix.topRightCorner<p,q>();`    |
| 右下角p行q列 | `matrix.bottomRightCorner(p,q);` | `matrix.bottomRightCorner<p,q>();` |
| 前q行     | `matrix.topRows(q);`             | `matrix.topRows<q>();`             |
| 后q行     | `matrix.bottomRows(q);`          | `matrix.bottomRows<q>();`          |
| 左p列     | `matrix.leftCols(p);`            | `matrix.leftCols<p>();`            |
| 右p列     | `matrix.rightCols(q);`           | `matrix.rightCols<q>();`           |
| 由i开始的q列 | `matrix.middleCols(i,q);`        | `matrix.middleCols<q>(i);`         |
| 由i开始的q行 | `matrix.middleRows(i,q);`        | `matrix.middleRows<q>(i);`         |
### 仅向量支持的Block操作

| 方法      | 动态大小                   | 静态大小                    |
| ------- | ---------------------- | ----------------------- |
| 前n个     | `vector.head(n);`      | `vector.head<n>();`     |
| 后n个     | `vector.tail(n);`      | `vector.tail<n>();`     |
| 由i开始的n个 | `vector.segment(i,n);` | `vector.segment<n>(i);` |
# 内存映射 Map
## 功能
用于将C/C++数组对象，以指针的形式，转换为一个可供Eigen使用的对象。
可以理解为一个包装器。
不负责管理内存。
```c++
//简
Map<Matrix<typename Scalar, int RowsAtCompileTime, int ColsAtCompileTime> >

Map<typename MatrixType, //实际上是一个Matrix的模板
    int MapOptions, //表明数据的内存是否经过对齐，默认是非对齐的，Eigen::AlignmentType
    typename StrideType> //Eigen::Stride<OuterStride,InnerStride>类，OuterStride同行下一数据在内存上的偏移量，InnerStride同列下一数据在内存上的偏移量

//例
Map<MatrixXf> mf(pf,rows,columns); //可写，pf类型为float *
Map<const Vector4i> mi(pi); //只读，pi类型为int *
```
可以像Matrix一样去使用该对象。