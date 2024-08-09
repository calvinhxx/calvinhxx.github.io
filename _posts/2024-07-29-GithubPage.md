---
layout: post
title:  "GithubPage Intro"
date:   2024-07-29 00:00:00 +0000
categories: GithubPage
excerpt_image: /assets/images/GithubPage/企业微信截图_17231842973490.png
tags: [GithubPage]
---

## 记录搭建StaicBlog

想搭建一个静态博客, 用于工作上的知识总结. 但又不想折腾服务相关的内容. 
希望能更关注于内容, 遂花了点时间了解了一下`GithubPage`和`jeklly`.实践下来确实简介、方便好用.

### 核心流程:

1. 明确`GithubPage`和`jeklly`概念
   - [GithubPage](https://docs.github.com/zh/pages/getting-started-with-github-pages/about-github-pages)是GitHub提供的一个`静态站点托管服务`. 
   - [jeklly](https://jekyllrb.com/)是一个`静态站点生成器`. 结合自己的理解总结汉化了一个版本[jeklly文档]({% post_url 2024-07-29-Jekyll %})

    GithubPage官方支持托管jeklly站点。非常适合不熟悉前端开发的同学搭建自己的静态博客.

2. 创建站点
`<username>.github.io`为个人站点, 相关操作在Github上点击GUI即可.简单来说就是创建一个特殊的仓库.

3. 定义发布源
所谓发布源就是指定一个分支或者文件夹存储jeklly内容. 默认配置为`/`即可.

4. 配置jekyll
挑选一个自己中意的[主题](https://pages.github.com/themes/), 按照主题要求配置插件和Gem, 然后开始本地调试
本地调试会遇到`failed to load`的[问题](https://talk.jekyllrb.com/t/load-error-cannot-load-such-file-webrick/5417)
手动安装一下add webrick即可
ruby 3.0后需要手动add webrick
```shell
bundle add webrick
```

5. 开始更新博文,在本地`_post_`文件夹下创建博文即可


