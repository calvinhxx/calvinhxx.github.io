---
layout: post
title:  "BuildSystem Intro"
date:   2024-07-30 00:00:00 +0000
excerpt_image: /assets/images/BuildSystem/企业微信截图_1728547552582.png
categories: BuildSystem
tags: [BuildSystem]
---

## 构建系统

构建系统是用于自动化运行编译、链接, 简化构建流程的软件开发工具. 为什么需要构建系统参考:
> [Why We Need Build Systems](https://blog.feabhas.com/2021/06/why-we-need-build-systems/)

通常, 我们使用构建系统来管理软件开发的以下部分或全部方面:

- source code organisation
源代码组织
- source code inter-dependecies
源代码相互依赖
- managing third party libraries
管理第三方库
- compilation options
编译选项
- code generation options
代码生成选项
- program linker configuration
程序链接器配置
- post build processing
构建后处理
- managing testing
管理测试

接下来我们来聊一聊 C++技术栈比较成熟的构建系统CMake

## CMake

`CMake`简介参考[Modern Cmake](https://blog.feabhas.com/2021/07/cmake-part-1-the-dark-arts/), [Cmake Tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/), 通过Cmake`IDE Build Tool Generators` 跨平台生成`native build system`解耦原生系统`build-system`, 基于CMake的三方开源组件更丰富、对现代CI、CD兼容更友好, 我们采用3.20以上版本的CMake更好地兼容现代CMake语法与功能

### CMake一般分3个阶段

- 1.make阶段: 运行CMakeLists.txt生成`native build system`工程文件
- 2.build阶段: 生成二进制文件
- 3.install阶段: 将二进制文件导入到指定目录
- 4.test阶段(可选): 运行ut测试

`IDE Build Tool Generators` 大致分两种:

- 1️⃣`single-configuration`: 如Ninja, 在make阶段通过`CMAKE_BUILD_TYPE`确定二进制版本(Debug\Release)
- 2️⃣`multi-configuration`: 如vs2017/2019/2022, 在build阶段通过`CMakeCache.txt`中`CMake Generator Expressions`(如:`$<CONFIG:DEBUG>`)判断版本

### CMake实操一般3种方式

- 1️⃣ `terminal CMD`(以vs为例, **重要**: 后续集成到CI/CD 需要调通所有命令行操作)

    以`管理员权限`打开cmd, 切换到Project `Git workspace`运行命令

  🖥️ 编译debug版本

    ```shell
    cmake -S ./ -B ./build -G "Visual Studio 15 2017" -A Win32 -DCMAKE_CONFIGURATION_TYPES="Debug;Release"

    cmake --build .\build\  --config Debug --target INSTALL
    ```

  🖥️ 编译release版本

    ```shell
    cmake -S ./ -B ./build -G "Visual Studio 15 2017" -A Win32 -DCMAKE_CONFIGURATION_TYPES="Debug;Release"

    cmake --build .\build\  --config Release --target INSTALL
    ```

    `${CMAKE_BINARY_DIR}/${CMAKE_CONFIGURATION_TYPES}`路径下会生成指定exe和pdb, 运行双击运行exe即可如:

- 2️⃣ `native build system`
  
    也可以先make生成native工程文件然后以`管理员权限`打开工程文件在IDE内操作, 如VS(multi-configuration)

  🖥️ 生成工程文件

    ```shell
    cmake -S ./ -B ./build -G "Visual Studio 15 2017" -A Win32 -DCMAKE_CONFIGURATION_TYPES="Debug;Release"
    ```
    
    点击批生成勾选target(用于编译exe和pdb)和`INSTALL`(用于拷贝依赖)

    🙆‍♂️ 编译完成后`设置启动项`为运行exe即可

- 3️⃣ `cross platform system`
  
  也可以用跨平台IDE打开,如`qtcreator`、qtcreaotr默认CMake `Build Tool Generators` 为Ninja(single-configuration)相对于vs简单在make阶段确定`BuildType`, 以管理员权限打开`qtreator`修改默认二进制生成路径、添加install target即可

