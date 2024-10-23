---
layout: post
title:  "DoubleBuffering"
date:   2024-10-23 00:00:00 +0000
categories: Qt
excerpt_image: /assets/images/DoubleBuffering/企业微信截图_17296740734787.png
tags: [Qt]
---

在开发Qt桌面客户端过程中会遇到界面元素闪烁的问题。比如listview更新model数据.可以使用双缓冲的技术解决.

## 双缓冲(Double Buffer)
聊到双缓冲前先说说单缓冲解释一下为什么单缓冲下界面会卡顿.网上关于双缓冲的文章措辞太过学术理解起来不太直观.笔者这里使用一些"人话"希望可以表达清楚.

- 单缓冲: 图像数据既视图数据, 修改图像数据的同时界面也在刷新, 修改图像数据是一个同步操作, 生成的图像不是一下子被绘制出来的，而是按照从左到右，由上而下逐像素地绘制而成的, 图像不是在瞬间显示给用户, 越是复杂的图像数据闪屏卡顿的现象越是明显.

- 双缓存: 前缓冲保存图像数据, 后缓冲保存视图数据, 简单理解就是两块内存显示的都是第二块的内容, 第一块的内容处理完毕可以使用时, swap第二块内存. swap操作相对生成图像要快的多所有不会出现闪屏卡顿问题.

## Qt中的双缓冲
QWidget
```c++
void MyWidget::paintEvent(QPaintEvent *event) {
    QPainter painter(&QPixmap());
    // 在 pixmap 上绘制内容
    painter.fillRect(pixmap.rect(), Qt::blue)
}

paintEvent事件中的绘制就是填充前缓冲, update操作就是swap操作把数据渲染到视图.

```

QML
```qml
ApplicationWindow {
    visible: true
    width: 640
    height: 480

    Canvas {
        id: myCanvas
        width: parent.width
        height: parent.height

        onPaint: {
            var ctx = myCanvas.getContext("2d");
            ctx.fillStyle = "blue";
            ctx.fillRect(0, 0, myCanvas.width, myCanvas.height);
        }

        Component.onCompleted: {
            myCanvas.requestPaint(); // 触发绘制
        }
    }
}
```

onPaint事件中的绘制就是填充前缓冲, requestPaint操作就是swap操作把数据渲染到视图.

## 从双缓冲理解优化渲染性能

参考下图:
![alt text](/assets/images/DoubleBuffering/企业微信截图_17296740734787.png)

可以抽象出来一个高渲染性能的思维模板:

💎 `工作线程处理执行 构建复杂图像元素的耗时操作, Gui线程只负责swap操作.`

比如:
listview生成model的操作在工作线程执行, emit更新数据的操作在主线程.


