---
title: windows下vscode配置c++环境
date: 2021-04-15 15:56:20
tags:
---

# Windows C++配置教程
## 配置环境
操作系统：Windows7
vscode版本号：1.48.2

## 下载工具
### 安装cpptools工具
* 打开vscode，点击Extensions；
* 搜索c++，选择第一个c/c++(1.0.1)
* 点击install安装
![安装cpptools工具](/images/c++配置1.PNG)

### 安装code runner工具
在VScode中编译文件，结束后并不会像我们经常使用的IDE一样，终端会停留在面前然后告诉你“按任意键继续”，在VScode中，编译运行完成后往往cmd会一闪而过，然后直接：
`The program ‘d:\MinGW\Projicts\test\test.exe’ has exited with code 0 (0x00000000).`
这个时候我们往往会使用在结束的return语句前加上getchar() 或
`system(“pause”)`（注意：使用这个的时候需要加上头文件`#include<stdlib.h>`）

我们可以使用code runner工具来解决这个问题
首先和上面一样 找到这个插件并安装它
![安装code runner工具](/images/c++配置2.PNG)

### 下载MinGW
下载地址：https://sourceforge.net/projects/mingw-w64/files/
下载的文件：进入网站后不要点击 “Download Lasted Version”，往下滑，找到最新版的`x86_64-posix-seh`。
安装MinGW：下载后是一个7z的压缩包，解压后移动到你想安装的位置即可。

## 配置MinGW环境变量
配置对象：WinGW，就是把刚刚安装WinGW的路径拷贝一下
右键点击，计算机 → 属性 → 高级系统设置 → 环境变量 → 双击系统变量Path → 复制MinGW安装路径的bin文件夹
例如我的是：`D:\MinGW\mingw64\bin`
记得每一步都要点确定，配置完成后我们使用命令行检验一下是否配置成功
`win+R` 输入`cmd`后回车打开命令行，输入`g++ -v`
出现下图界面说明配置成功
![配置MinGW环境变量](/images/c++配置2.PNG)
然后打开以下界面进行操作
![](/images/CodeRunner.png)
![](/images/CodeRunner1.png)
![](/images/CodeRunner2.png)
即可
## 配置C++环境
### 创建一个c/cpp文件
首先随便打开或者新建一个文件夹CppCode, 然后在文件夹里创建一个c或cpp文件
```Cpp
#include<iostream>

int main()
{
    std::cout<<"Hello World"<<std::endl;
    getchar();
    return 0;
}
```
### 创建json文件
先创建一个你打算存放代码的文件夹（称作工作区），路径不能含有中文和空格和引号。c语言和c++需要建立不同的工作区（除非你懂得下面json文件的某些选项，则可以做到一个工作区使用不同的build task）。

打开VS Code，选打开文件夹（不要选“添加工作区文件夹”，理由见上一句），选择刚才那个文件夹，点VS Code上的新建文件夹，名称为.vscode（这样做的原因是Windows的Explorer不允许创建的文件夹第一个字符是点），然后创建`launch.json，tasks.json，settings.json，c_cpp_properties.json`放到.vscode文件夹下。
`特别注意：C/C++文件放在与.vscode 的所在的同级目录中，.vscode 只放置4个json文件。`
#### launch.json配置文件
stopAtEntry可根据自己喜好修改；cwd可以控制程序运行时的相对路径，如有需要可以改为${fileDirname}。其他无需更改，除非你不用windows，则可以用lldb调试（需要自己装）。type和request不变色是正常现象。
```Json
{
    "version": "0.2.0",
    "configurations": [

        {
            "name": "g++.exe build and debug active file", // 配置名称，将会在启动配置的下拉菜单中显示
            "type": "cppdbg", // 配置类型，这里只能为cppdbg
            "request": "launch", // 请求配置类型，可以为launch（启动）或attach（附加）
            "program": "${fileDirname}/${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径
            "args": [], // 程序调试时传递给程序的命令行参数，一般设为空即可
            "stopAtEntry": true, // 设为true时程序将暂停在程序入口处，我一般设置为true
            "cwd": "${workspaceFolder}", // 调试程序时的工作目录
            "environment": [], // （环境变量？）
            "externalConsole": true, // 调试时是否显示控制台窗口，一般设置为true显示控制台
            "internalConsoleOptions": "neverOpen", // 如果不设为neverOpen，调试时会跳到“调试控制台”选项卡，你应该不需要对gdb手动输命令吧？
            "MIMode": "gdb", // 指定连接的调试器，可以为gdb或lldb。但目前lldb在windows下没有预编译好的版本。
            "miDebuggerPath": "D:/MinGW/mingw64/bin/gdb.exe", // 调试器路径。
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": false
                }
            ],
            "preLaunchTask": "task g++" // 调试会话开始前执行的任务，一般为编译程序。与tasks.json的label相对应
        }
    ]
}
```

#### tasks.json配置文件
reveal可根据自己喜好修改，即使设为never，也只是编译时不跳转到“终端”而已，手动点进去还是可以看到，我个人设为never。
命令行参数方面，-std根据自己的需要修改。如果使用Clang编写C语言，把command的值改成clang。
如果使用MinGW，编译C用gcc，编译c++用g++，并把-target和-fcolor那两条删去。如果不想要额外警告，把-Wall那一条删去。
```Json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "task g++", // 任务名称，与launch.json的preLaunchTask相对应
            "command": "D:/MinGW/mingw64/bin/g++.exe", // 要使用的编译器
            "args": [
                "${file}",
                "-o", // 指定输出文件名，不加该参数则默认输出a.exe
                "${fileDirname}/${fileBasenameNoExtension}.exe",
                "-g", // 生成和调试有关的信息
                "-Wall", // 开启额外警告
                "-static-libgcc", // 静态链接
                //"-fcolor-diagnostics",
                //"--target=x86_64-w64-mingw", // 默认target为msvc，不加这一条就会找不到头文件
                "-std=c++17" // C语言最新标准为c11，或根据自己的需要进行修改
            ], // 编译命令参数
            "type": "shell",
            "group": {
                "kind": "build",
                "isDefault": true // 设为false可做到一个tasks.json配置多个编译指令，需要自己修改本文件，我这里不多提
            },
            "presentation": {
                "echo": true,
                "reveal": "always", // 在“终端”中显示编译信息的策略，可以为always，silent，never。具体参见VSC的文档
                "focus": false, // 设为true后可以使执行task时焦点聚集在终端，但对编译c和c++来说，设为true没有意义
                "panel": "shared" // 不同的文件的编译信息共享一个终端面板
            },
            "options": {
                "cwd": "D:/MinGW/mingw64/bin"
              },
            "problemMatcher": [
                "$gcc"
            ]
        }
    ]
}
```
#### c_cpp_properties.json配置文件
此文件内容来自于 Microsoft/vscode-cpptools ；这个json不允许有注释（其实按照标准本来就不能有）。

如果你没有合并Clang和MinGW，则该文件中的compilerPath必需修改成MinGW的完整路径，精确到gcc.exe，否则会提示找不到头文件；Linux下应该是/usr/bin/gcc。
没有该文件运行时，会报错”includepath”设置问题
记得把路径修改为你自己的路径，查找自己路径方法为打开cmd 输入:`gcc -v -E -x c++ -`
```Json
{
    "configurations": [
        {
            "name": "MinGW",
            "includePath": [
                "${workspaceRoot}",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/x86_64-w64-mingw32",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/backward",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include-fixed",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/../../../../x86_64-w64-mingw32/include"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "__GNUC__=6",
                "__cdecl=__attribute__((__cdecl__))"
            ],
            "intelliSenseMode": "msvc-x64",
            "browse": {
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": "",
                "path": [
                "${workspaceRoot}",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/x86_64-w64-mingw32",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/backward",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include-fixed",
                "D:/MinGW/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/../../../../x86_64-w64-mingw32/include"
                ]
            }
        }
    ],
    "version": 4
}
```

#### 运行文件
这个时候我们已经配置完成，可以开始运行我们刚才创建的测试代码啦
F5：VScode调试
CTRL+ALT+N:code runner调试

注意：若出现`标记“&&”不是此版本中的有效语句分隔符`错误，
选择 File-> Preferences -> Settings，在搜索框输入setting.json。
进入setting.json，添加这一句话
```
terminal.integrated.shell.windows": "powershell.exe",
```
使用powershell的格式来进行编译执行，会自动将 && 符号换为 ;  号，即可解决。
如果code runner出现中文乱码，尝试在setting.json添加：
```
"cpp": "cd $dir && g++ -fexec-charset=GBK -std=c++11 $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
```

# VS Code构建C++远程开发环境
操作系统：win7 远程服务器：Ubuntu 16.04.4
通过过Windows远程Linux进行开发，同时也能拥有IDE一般的开发体验。
方法：
* Visual Studio支持的远程Linux开发功能：Visual Studio堪称宇宙IDE，功能强大而且Windows和Linux的开发体验一脉相承。然而程序体积庞大，并且配置较复杂，启动时间较长。如果希望使用cmake来进行项目构建，Visual Studio所提供的cmake模板与常规在Linux下构建cmake项目的模板结构有些差异。使用起来不太顺手;
* CLion的远程开发功能：配置相对简单，编程体验也非常优秀（尤其是CLion自带的很多重构方式，极大提升编程效率）。然而远程开发首先要在本地构建项目，然后通过CLion将项目文件传输到目标Linux机器，然后才开始调试。不能实现直接打开远端的项目（可能我没找到） 来进行编程。
* 通过VS Code进行远程开发：承袭VS Code的编程体验，流畅度和稳定度都非常良好。而且程序较轻量，打开速度很快。各种插件几乎涵盖日常编程需求。各种语言都可以统一在VS Code下面来编写（个人觉得这个特点非常优越）。美中不足就是毕竟不是传统IDE，在一些重构和自动生成的功能不如Visual Studio和CLion。不过瑕不掩瑜，这是当前个人最提倡的远程开发C++的方式。

## 环境配置
### Remote SSH配置
微软的PowerShell团队已经支持openssh，所以安装文件我们可以在github的powershell团队项目下进行下载
下载地址： https://github.com/PowerShell/Win32-OpenSSH/releases
根据你自己的系统对应下载。
把OpenSSH整个目录进行复制到 C:\Program Files (其实哪个目录都可以，不过建议安装在这里)
回到Windows桌面，在 计算机(windows7)或此电脑(windows10)，右键 --> 属性 --> 高级系统设置 --> 环境变量--系统变量，在此框里面找到 Path 进行编辑
使用cmd命令打开dos命令行，输入ssh或scp，出现如下情况则配置成功！
![](/images/openssh.PNG)
### VS Code添加扩展包
从VS code中的扩展商店中添加Remote Development插件，如下图所示。
![](/images/ssh.PNG)
添加完成后，我们发现多了这些插件以及Remote SSH的图标。
### 配置私钥
在.ssh目录下，用`ssh-keygen`命令生成密钥，如果.ssh目录已有id_rsa文件，可跳过。
然后将生成的id_rsa.pub文件传到远程服务器的根目录下.ssh文件夹中。
用ssh命令（ssh username@ip -p port）连接远程主机，并将idrsa.pub加入到authorized_keys
```
ssh gxk@10.103.238.162 -p 22
cd .ssh
cat id_rsa.pub >> authorized_keys  //该指令可以免密登录
```
退出连接（exit命令），改用私钥登录（ssh username@ip -p port –i id_rsa），即`ssh gxk@10.103.238.162 -p 22 –i id_rsa`
### 添加配置文件
点击Remote SSH的图标后再点击箭头所指的齿轮
![](/images/ssh2.jpg)
会弹出菜单让你选择需要编辑的配置文件，一般选第一个
![](/images/ssh3.jpg)
选择之后可以按照下图添加配置信息
![](/images/ssh4.jpg)
参数的含义分别为：
Host 连接的主机的名称，可自定
Hostname 远程主机的IP地址
User 用于登录远程主机的用户名
Port 用于登录远程主机的端口
IdentityFile 本地的id_rsa的路径
如果需要多个连接，可按照如上配置多个。配置完成并保存后，左边栏中多了远程主机的图标。
![](/images/ssh5.jpg)