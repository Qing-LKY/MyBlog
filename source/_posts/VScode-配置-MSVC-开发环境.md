---
title: VScode 配置 MSVC 开发环境
date: 2022-10-24 10:02:10
tags:
- vscode
categories: 项目经验
---

## 前言

软安实验要求使用 Windows SDK。但是电脑硬盘有点吃紧，而且用习惯了 VScode，不想用 Visual Studio。于是就有了这篇文章。

在这篇文章中，你可能会感兴趣的是:

1. 不安装 VS 的情况下安装 MSVC 开发环境，并修改安装盘
2. 配置 VScode，使得调试生成不需要在 Developer Command Prompt 下打开 VScode

参考资料: [VScode 文档](https://code.visualstudio.com/docs/cpp/config-msvc)。

## 下载、安装 Make Tools

我们可以选择下载 Vistual Studio 生成工具，而不是安装整个 Vistual Studio。

在[官网下载页面](https://visualstudio.microsoft.com/zh-hans/downloads/)下拉，找到 "所有下载 -> 适用于 Vistual Studio 2022 的工具 -> Visual Studio 2022 生成工具"。

如果在运行下载的安装包时出现闪退，请不要反复双击安装包，因为它其实是 Installer 的安装包。可以在 Windows 搜索中手动打开 Vistual Studio Installer 后选择 VS 生成工具进行安装。

安装时根据需要，选中 "使用 C++ 的桌面开发" 即可。

安装时，可以选择安装位置到 D 盘。如果有一些选项是灰色的，可以打开注册表 (Windows + R 输入 regedit)，删除 "HKEY_LOCAL_MACHINE > SOFTWARE > Microsoft > Visual Studio > setup"，再重新打开 Installer 进行安装。

## VScode 配置

### 在终端列表中添加 Developer Command Prompt

在设置中搜索: `terminal.integrated.profiles.windows`，点击在 settings.json 中编辑 (args 中的路径需要自己修改):

```json
{
    "terminal.integrated.profiles.windows": {
        "VS 2019": {
            "path": "C:\\Windows\\System32\\cmd.exe",
            "args": [
                "/k",
                "D:\\Microsoft Visual Studio\\2019\\Community\\Common7\\Tools\\VsDevCmd.bat"
            ]
        },
        ...
    },
}
```

### 配置 C/C++

按需修改即可:

```json
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${default}"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ],
            "windowsSdkVersion": "10.0.19041.0",
            "compilerPath": "D:/Microsoft Visual Studio/2019/Community/VC/Tools/MSVC/14.29.30133/bin/Hostx64/x64/cl.exe",
            "cStandard": "c17",
            "cppStandard": "c++17",
            "intelliSenseMode": "windows-msvc-x86"
        }
    ],
    "version": 4
}
```

### 配置 Build Task

修改 task.json 如下 (args 中的路径需要自己修改):

```json
{
    "version": "2.0.0",
    "windows": {
        "options": {
            "shell": {
                "executable": "cmd.exe",
                "args": [
                    "/C",
                    "\"D:/Microsoft Visual Studio/2019/Community/Common7/Tools/VsDevCmd.bat\"",
                    "&&"
                ]
            }
        }
    },
    "tasks": [
        {
            "type": "shell",
            "label": "cl.exe build active file",
            "command": "cl.exe",
            "args": [
                "/Zi",
                "/EHsc",
                "/Fe:",
                "${fileDirname}\\${fileBasenameNoExtension}.exe",
                "${file}"
            ],
            "problemMatcher": [
                "$msCompile"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

在 "终端 -> 运行生成任务" (Ctrl + Shift + B) 时选择 "cl.exe build active file"，即可不通过 Developer Command Prompt 打开 VScode 来生成活动文件了。

### 配置 Debug

在 launch.json 中添加 (注意: preLaunchTask 要与 task.json 中的 label 一致):

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "C/C++: cl.exe build and debug active file",
            "type": "cppvsdbg",
            "request": "launch",
            "program": "${fileDirname}\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "console": "externalTerminal",
            "preLaunchTask": "cl.exe build active file"
        }
    ]
}
```

配置完后，就可以通过: "终端 -> 运行任务" 中选择 "C/C++: cl.exe build and debug active file" 来生成并运行了。

初次运行完成 Default 的设置后，就可以通过 F5 启动调试了。

console 项是在运行时生成一个新的窗口，而不是使用调试控制台。它默认为 `internalConsole`。较低版本中配置方法不同，如下:

```json
"externalConsole": true,
```
