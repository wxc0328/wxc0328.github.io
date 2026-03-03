---
title: 基于vscode的C++调试方法
date: 2026-03-03 15:51:00
tags: 
  - Debug
  - vscode
categories:
  - C++
---

在vscode中，启动调试依赖于launch.json。
该文件位于工程目录下的.vscode文件夹中。如果不存在可以自己创建。
# 以gdb启动程序
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Program",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/your_exe", // 可执行文件的完整路径
            "args": [], //程序的输入参数
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "/usr/bin/gdb", // gdb的路径
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```
# 通过附加方式调试程序
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Attach",
            "type": "cppdbg",
            "request": "attach",
            "processId": "${command:pickProcess}", // 会弹出进程选择列表
            "program": "${workspaceFolder}/your_exe", // 可执行文件的完整路径(不是动态库)
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "启用pretty-print",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "symbolLoadInfo": {
                "loadAll": true
            }
        },
    ]
}
```
# Python调用C++库的联合调试
这里推荐安装vscode插件`Python C++ Debugger`。
安装完成后，在launch.json中设置
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python C++ Debug",
            "type": "pythoncpp",
            "request": "launch",
            "pythonLaunchName": "Python: Current File",
            "cppAttachName": "(gdb) Attach",
        },
        {
            "name": "(gdb) Attach",
            "type": "cppdbg",
            "request": "attach",
            "processId": "",
        },
        {
            "name": "Python: Current File",
            "type": "debugpy",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal"
        }
    ]
}
```
打开python脚本，然后在左侧的调试项中选择`Python C++ Debug`启动调试，即可同时应用C++和Python的断点。