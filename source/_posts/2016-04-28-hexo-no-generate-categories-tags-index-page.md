---
title: hexo不生成categories和tags的index页
tags:
  - hexo
categories: 技术
date: 2016-04-28 17:42:44
updated: 2016-04-28 18:09:15
---

## 问题
使用第三方主题的时候，有时候会遇到hexo不能自动生成`/categories/index.html`和`/tags/index.html`页面的问题，导致进入分类或标签主页无法正常显示该页面，如下图

<!-- more -->

![](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-28/17727126.jpg)

## 解决
通过终端执行
```bash
$ hexo new page categories
$ hexo new page tags
```
会在`source`文件夹创建两个文件夹`categories`, `tags`，并分别创建`index.md`文件

```bash
├── source
├── categories
|   ├── index.md
└── tags
    └── index.md
```

编辑`index.md`文件

`source/categories/index.md`
```markdown
---
title: 分类
type: categories
date: 2016-04-28 17:32:38
---
```

`source/tags/index.md`
```markdown
---
title: 标签
type: tags
date: 2016-04-28 17:32:38
---
```

从新生成就有了
```bash
$ hexo generate
$ hexo server
```

![](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-28/38418913.jpg)
