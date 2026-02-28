---
title: Eigen安装
date: 2026-02-28 12:01:46
tags:
categories:
  - C++
  - Eigen
---

# 简介
eigen是一个用于求解线性代数的库，纯头文件实现。
这里的安装，主要指的是构建测试代码所需要的依赖项（包括使用其他后端工具求解）
如果仅使用eigen本体，可以都不需要。
# 获取源码
```shell
git clone https://gitlab.com/libeigen/eigen.git
```
# 安装依赖项
## Boost
```shell
//更新apt信息
sudo apt update
//安装boost编译依赖
sudo apt install build-essential python3 libbz2-dev libz-dev libicu-dev
//安装boost开发库
sudo apt install libboost-all-dev
```
## Cuda
```shell
//安装cuda-13.1
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/13.1.1/local_installers/cuda-repo-wsl-ubuntu-13-1-local_13.1.1-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-13-1-local_13.1.1-1_amd64.deb
sudo cp /var/cuda-repo-wsl-ubuntu-13-1-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-toolkit-13-1

//设置环境变量
//在.bashrc下增加
export PATH=/usr/local/cuda-13.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-13.1/lib64:$LD_LIBRARY_PATH
```
## 相关求解器
```
# 安装直接稀疏求解器相关库
# 包含 CHOLMOD, UMFPACK, KLU, SPQR
sudo apt install libsuitesparse-dev
# SuperLU
sudo apt install libsuperlu-dev

# 安装 METIS 图划分库
sudo apt install libmetis-dev

# Adolc 自动微分库
sudo apt install libadolc-dev

# MPFR 用于多精度浮点计算的C库
sudo apt install libmpfr-dev

# FFTW 离散傅里叶变换
sudo apt install libfftw3-dev

# 基于 MPFR 的 C++ 封装库，更符合习惯的、面向对象的方式
sudo apt install libmpfrc++-dev
```
# 查看编译器支持的指令集
```
# 查看CPU信息
cat /proc/cpuinfo | grep flags | head -1
# 查看GCC支持的标志
gcc -march=native -Q --help=target | grep enabled
```
# 编译参数
```
//设置C++标准17
CMAKE_CXX_STANDARD 17

//设置显卡对于的计算能力
EIGEN_CUDA_COMPUTE_ARCH 89
```
# 编译测试集
```shell
make buildtests -j6

//生成编译信息，供clangd使用
compiledb -n make buildtests
//compiledb 安装
pip install compiledb
```