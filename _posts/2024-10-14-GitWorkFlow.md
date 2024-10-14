---
layout: post
title:  "GitWorkFlow"
date:   2024-10-14 00:00:00 +0000
categories: Software Engineering
excerpt_image: /assets/images/GitWorkFlow/企业微信截图_17288919293286.png
tags: [Software Engineering]
---

## GitFlow Conversion

现在业内比较流行的流行的FDD(Feature-driven-development)WorkFlow有如下三种:

- [GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
- [Github Flow](https://docs.github.com/en/get-started/quickstart/github-flow)
- [Gitlab Flow](https://docs.gitlab.cn/jh/topics/gitlab_flow.html)

FDD Workflow指的是需求是开发的起点, 先有需求再有功能分支（feature branch）或者补丁分支（hotfix branch）。完成开发后，该分支就合并到主分支，然后被删除.

三种不同WorkFlow优劣点[参考](https://medium.com/@sreekanth.thummala/choosing-the-right-git-branching-strategy-a-comparative-analysis-f5e635443423)

**如果项目暂时没有完备的CI/CD基建, 需要有清晰可控的分支结构、不要求持续发布推荐采用`GitFlow WorkFlow`作为项目WorkFlow**

### Bitbucket Git WorkFlow简介

Bitbucket GitFlow 开发模型[参考](https://nvie.com/posts/a-successful-git-branching-model/)

![Alt text](/assets/images/GitWorkFlow/image-7.png)

核心概念:
  - 代码流分支(Code Flow Branches)`长期分支, 存在远端存储库永久可用`
    - origin/master`反应生产环境代码状态`
    - origin/develop`反应最新下个版本的开发代码状态、又称为集成分支(integration branch)develop分支是CI的基础`
    ![Alt text](/assets/images/GitWorkFlow/image-4.png) 
  - 临时分行(Temporary Branches)`短期分支, 一般存在开发人员本地`
    - Feature branches
      - Feature branches指新开发功能分支, 一般从develop分支拉、必须合并到develop, 开发功能分支被采纳合并到develop然后删除, 被弃用直接删除.
    ![Alt text](/assets/images/GitWorkFlow/image-2.png) 
    - Release branches
      - Release branches指新发布分支, 一般从develop分支拉、必须合并到develop和master, 发布分支支持提交bug和提交版本发布元数据(version number, build dates, etc) 提测通过合并到develop然后删除.
    - Hotfix branches
      - Hotfix branches指修复分支, 一般从master分支拉、必须合并到develop和master, 一般是对线上紧急bug进行修复改完合并到master然后删除.
    ![Alt text](/assets/images/GitWorkFlow/image-3.png)
  
### Commit Message规范

目前业内比较流行的git commit规范有两种:
- [gitemoji](https://github.com/liuchengxu/git-commit-emoji-cn)
- [google angular](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit?pli=1#heading=h.uyo6cb12dt6w)

`emoji`对commit message的归类更加详细, 第三方对changelog支持的自动化工具更好
`angular`相对更加轻量简约, 在原来通用类型上扩展即可

比如[Steam++](https://github.com/BeyondDimension/SteamTools)项目中,存在这两种提交风格的痕迹

Commit message 基本格式
```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```
`commit message的每一行都不要超过100个字符`
主要的构成元素是`a header`、`a body`、a footer(不涉及issue操作暂时弃用) 和blank line

`Header message`
简短的标题描述包含type、(scope)、subject
`Type` 类别:
- feat (添加新功能提交)
- fix (修复bug提交)
- docs (文档提交)
- style (代码格式化提交)
- refactor (代码重构提交)
- deprecate (代码弃用提交)
- test (测试相关提交)
- build (构建系统、外部依赖修改)
- ci (ci相关提交)
- pref (代码性能优化提交)
- ui (ui相关调整)
  
`Scope`: 我们以模块作为定义如:IM、SDK
`Subject`: 对本次提交的简洁描述
  - 使用命令式, 现在时态'改变'不是'已改变'也不是'改变了'
  - 不要大写首字母
  - 不在末尾添加句号

`Body Message`
`body`: 对此次提交的详细描述和主题设置类似, 使用命令式、现在时态应该包含修改的动机以及和之前行为的对比.

`Revert`
当此次提交包含回滚(revert)操作, 那么页眉以"revert:"开头, 同时在正文中添加"This reverts commit hash", 其中hash值表示被回滚前的提交, 主流gui都支持快捷操作如Fork