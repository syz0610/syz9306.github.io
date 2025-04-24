---
title: 本框架开发部署指南
published: 2025-04-23
description: '当前博客框架Fuwari的开发和部署指南'
image: 'cover.jpeg'
tags: [Cookbook,Astro,前端,Git]
category: 'Cookbook'
draft: false 
lang: ''
---

# 项目说明 

当前博客框架使用[Astro](https://astro.build/)构建。想要进一步了解如何使用`Astro`框架的话,请访问[Astro Docs](https://docs.astro.build/)

# 工程部署

1. [拉取一份源代码作为副本](https://github.com/saicaca/fuwari/generate),可以是`fork`或者是另起炉灶
2. 使用`git clone`指令拉取仓库代码
3. 使用`pnpm install`和`pnpm add sharp`指令完成开发环境初始化
   - 关于如何使用git和配置前端开发环境在此不做赘述
4. 在文件`src/config.ts`中个性化定制你自己的博客
5. 使用`pnpm new-post <filename>`指令创建一个新博客,之后在`src/content/posts/<filename>.md`中编辑
   - 也可以直接创建文件`src/content/posts/<filename>.md`
   - 删除文章则直接删除该md文件即可
   - 建议每一个博文都建立一个文件夹,单独存放md文件,便于归类
   - 博文配图可以和md文件在同一个文件夹下,也可以统一维护在public目录下,注意维持两侧目录结构一致,方便维护
6. 源代码采用`Github Action`自动编译部署，因此需要在Github仓库内配置`Astro`编译引擎以及`Action`脚本
   - `Action`脚本可以参考当前工程下`.github/workflows/astro.yml`文件的写法自定义触发器所在分支
   - 在`Github Pages`选项卡页面中配置`Astro`引擎、触发脚本
   - 将仓库重命名为`<自定义英文+数字>.github.io`的形式,当`Pages`服务运行起来后,就可以直接访问`<自定义英文+数字>.github.io`并显示博客内容了

# 文章维护

上一节已经提到过了如何新增文章，这一节主要是每个文章头部属性配置字段的说明：

|字段|用途|
|---|---|
|`title`| 文章标题|
|`published`|发布时间|
|`description`|摘要|
|`image`| 文章封面的路径<br/>1.在线图片必须以`http://`或`https://`开头<br/>2.本地图片如果在`public`中必须以`/`开始<br/>3.当前文件夹内的图片无需前缀|
|`tags`|当前文章的关键字|
|`category`|当前文章的分类|
|`draft`|控制生产环境中是否显示该文件|

## 草稿

当文章尚未完成需要保存时,可以将当前文章属性的`draft`选项设置为`true`，这样部署后生产环境将不会显示该文章

```markdown
---
（前略）
draft: true
---
```