---
title: 基于Docker+QEMU部署交叉编译环境
date: 2026-03-02 16:39:00
tags: 
  - Docker
  - QEMU
categories:
  - [Docker]
---

# 安装QEMU
这一步是为了使系统可以识别到arm格式的文件，将其发送给qemu模拟器执行
```shell
sudo apt install -y qemu-user-static binfmt-support

echo ':aarch64:M::\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\xb7\x00:\xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff:/usr/bin/qemu-aarch64-static:F' | sudo tee /proc/sys/fs/binfmt_misc/register

# qemu-user-static 静态编译的QEMU用户模式模拟器，允许在x86系统上运行为ARM64等其他架构编译的程序
# binfmt-support 提供内核的二进制格式处理支持，用于注册可执行格式的处理程序
# 向Linux内核的binfmt_misc机制注册一个新的二进制格式处理规则
```
注册指令会创建一个内核规则
```
# 查看规则方式
cat /proc/sys/fs/binfmt_misc/qemu-aarch64

# 内容
enabled
interpreter /usr/libexec/qemu-binfmt/aarch64-binfmt-P
flags: PF
offset 0
magic 7f454c460201010000000000000000000200b700
mask ffffffffffffff00fffffffffffffffffeffffff
```
## binfmt_misc内核规则
[官方文档](https://www.kernel.org/doc/html/latest/admin-guide/binfmt-misc.html)
```
# 字段规则

:name:type:offset:magic:mask:interpreter:flags

# name 名称
# type 类型 M表示魔数 E表示基于扩展名
# offset 对魔数有意义，默认是0
# magic M下是二进制字节序列，E下是要识别的文件扩展名(不含.)。需要用16进制编码传入
# mask M下有意义，二进制字节序列，为1表示需要位相同，16进制编码传入
# interpreter 二进制文件首先传入的程序，这里是/usr/bin/qemu-aarch64-static
# flags F 表示 fix binary，会要求模拟器立即打开目标二进制文件并持有其文件描述符，确保无论环境如何变化，模拟器始终可用。（因为默认情况下是惰性加载）
```
[ARM64识别规则来源](https://github.com/qemu/qemu/blob/master/scripts/qemu-binfmt-conf.sh)
```
aarch64_magic='\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\xb7\x00'
aarch64_mask='\xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff'
aarch64_family=arm
```
# 生成交叉编译环境
## docker生成并运行容器
```
docker run -itd --name ub22_arm64 --platform linux/arm64 -v $(pwd)/workspace:/workspace ubuntu:22.04
```
## 启动已存在的容器
```
docker start ub22_arm64
```
## 停止容器
```
docker stop ub22_arm64
```
## 进入编译环境
```
docker exec -it ub22_arm64 /bin/bash
```
在编译环境内，就可以正常执行arm格式的程序，同时可以通过apt这些安装arm下的执行环境，生成的内容可以直接给到树莓派5这类使用。
## 导出容器为镜像
```
docker commit ub22_arm64 ubuntu22.04-arm64:v1
```