---
title: Nav2笔记(一)——安装
date: 2026-03-05 21:18:00
tags: 
  - Nav2
categories:
  - Nav2
---

# 安装Ros2
## 设置系统语言环境
```shell
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8 
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings

# 如果想使用中文环境，修改为
sudo locale-gen zh_CN zh_CN.UTF-8
sudo update-locale LC_ALL=zh_CN.UTF-8 LANG=zh_CN.UTF-8
export LANG=zh_CN.UTF-8
```
## 添加Ros软件源
安装软件源管理工具
```shell
sudo apt install software-properties-common
sudo add-apt-repository universe

# 第一行 添加add-apt-repository命令
# 第二行 将 "universe" 软件源添加到系统的源列表中 universe 表示社区维护
```
获取ros软件源
```shell
sudo apt update && sudo apt install curl -y
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb
```
## 安装ros包
```shell
#更新软件源信息
sudo apt update

# 因为ros在对应系统上更新频繁，其关联依赖可能存在更新
# 更新现有依赖项
sudo apt upgrade

# 安装ROS本体,以及相关调试工具, RViz, demos, tutorials
sudo apt install ros-humble-desktop
# 安装Ros编译工具
sudo apt install ros-dev-tools

# 如果只需要基础的运行环境
sudo apt install ros-humble-ros-base
```
此方法安装的`ros2`位于`/opt`下
## 初始化Ros运行环境
```shell
source /opt/ros/humble/setup.bash
```
## 其他问题
### `wsl`环境，网络配置为`mirrored`，节点无法通信
该环境下，因为网络配置问题，`daemon`服务无法自动启动，影响节点之间的正常通信（表现为无法接收数据）。
需要手动启动`daemon`服务
```shell
ros2 daemon start
```
# 安装Nav2
```shell
source /opt/ros/humble/setup.bash

# 安装相应版本的nav2核心功能包
sudo apt install ros-humble-navigation2

# 安装相应版本nav2的启动示例
sudo apt install ros-humble-nav2-bringup

# 安装demo用的gazebo机器人，包括gazebo环境
sudo apt install ros-humble-turtlebot3-gazebo
```
# 测试案例
```shell
source /opt/ros/humble/setup.bash
export TURTLEBOT3_MODEL=waffle
export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:/opt/ros/$ROS_DISTRO/share/turtlebot3_gazebo/models

ros2 launch nav2_bringup tb3_simulation_launch.py headless:=False
```
## wsl可能存在的gazebo黑屏
有可能是因为wsl2的版本过低，在新版wsl2中已经支持WSLg
在windows命令行界面，执行
```cmd
wsl --update
```
其他可能情况，查看[https://github.com/gazebosim/gz-sim/issues/2670](https://github.com/gazebosim/gz-sim/issues/2670)