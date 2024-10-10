---
layout: post
title:  "Expand Ubuntu Server Disk"
date:   2024-08-09 00:00:00 +0000
categories: Server 
excerpt_image: /assets/images/ExpandServerDisk/企业微信截图_17285433595141.png
tags: [Server]
---

## 扩展服务器磁盘
在日常工作中遇到了磁盘不足服务器无法正常运行的问题。记录下如何排查解决。

- 查看磁盘使用情况

```shell
df -h 
```

- 选择要扩容的盘符

```shell
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

- 从新计算磁盘大小

```shell
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv 
```

搞定！ 挂载的trilium wiki又能正常运行了。