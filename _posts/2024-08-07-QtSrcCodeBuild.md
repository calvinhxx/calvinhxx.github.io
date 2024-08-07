---
layout: post
title:  "Qt SrcCode Build"
date:   2024-08-07 00:00:00 +0000
categories: Qt
excerpt_image: /assets/images/QtSrcCodeBuild/image.png
tags: [Qt]
---
- [VS2019编译qt源码](#vs2019编译qt源码)
  - [🙆🏻‍♂️️ 动机](#️️-动机)
  - [🏛️️ 准备](#️️-准备)
  - [🏗️️ build](#️️-build)
  - [☝🏻 使用](#-使用)

# VS2019编译qt源码 

## 🙆🏻‍♂️️ 动机

qt5.15早期开源版本存在部分windows设备渲染卡顿、崩溃问题需要升级qt版本解决。然而qt6涉及到GPU渲染部分不支持win7和32位所以需要升级qt5.15后期fixed版本。

**note:** 从qt5.15后opensrc版本不再提供offcial binary, qt官方online installer只能安装qt6.7.1和qt5.15.2之类的早期小版本官方二进制。

1. 自己手动编译qt源码。并且遵守[qt license相关说明](https://www.qt.io/download-open-source)
![alt text](/assets/images/QtSrcCodeBuild/image.png)

2. 花钱购买[Commercial版本](https://www.qt.io/blog/commercial-lts-qt-5.15.17-released)。

---

## 🏛️️ 准备

1. 安装好vs2019确保相关组件正确安装

2. 下载[qt5.15.14源码](https://download.qt.io/official_releases/qt/5.15/5.15.14/single/)

3. 安装[jom](https://wiki.qt.io/Jom) 

    `nmake`是mscv构建系统, 历史原因只能单核build速度太慢, `jom`支持多核并发编译.

4. 查看qt官方[build on windows doc](https://doc.qt.io/qt-5/windows-building.html)

## 🏗️️ build

1. 根据想要编译的版本32/64 打开对应的终端(一定得确认) 32位打开x86 64位打开x64, 编译过程中可以查看任务管理器cl.exe是否匹配32/64

    ![alt text](/assets/images/QtSrcCodeBuild/image-3.png)
    ![alt text](/assets/images/QtSrcCodeBuild/image-2.png)

2. cd到源码解压目录

3. 修改qmake工程文件 (**note:** 不需要符号可以skip)

    添加qmake配置 Release版本附带符号
    ```code
    QMAKE_CXXFLAGS_RELEASE += $$QMAKE_CFLAGS_RELEASE_WITH_DEBUGINFO
    QMAKE_LFLAGS_RELEASE += $$QMAKE_LFLAGS_RELEASE_WITH_DEBUGINFO
    ```

4. 运行

    ```shell

    configure -platform win32-msvc -prefix "../5.15.14" -nomake examples -nomake tests -debug-and-release -confirm-license -opensource -skip qtconnectivity -skip qtwebengine -skip qtvirtualkeyboard -qt-zlib -qt-libpng -qt-libjpeg -opengl desktop -mp

    ```

    可以`configure -h`查看命令说明或者查看[options](https://doc.qt.io/qt-5/configure-options.html) ,模块信息可以对比online installer勾选自行skip 用到哪些编译哪些没有使用的模块不编译提高编译速率.

5. configure没有报错后运行jom -j8 开始编译

6. build无误后运行jom install,运行完毕后configure配置的prefix路径下就是生成的产物

    ![alt text](/assets/images/QtSrcCodeBuild/image-4.png)

## ☝🏻 使用

1. `vs2019使用` 

    添加qt环境变量到path, qtvstools导入路径即可offcial binary使用方式一致

3. `qtcreator使用`

    需要手动关联qtversion和手动添加编译器到qtcreator
    ![alt text](/assets/images/QtSrcCodeBuild/image-5.png)
    ![alt text](/assets/images/QtSrcCodeBuild/image-6.png)

