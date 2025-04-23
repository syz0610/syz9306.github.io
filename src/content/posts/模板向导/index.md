---
title: Markdown语法指南
published: 2025-04-23
description: '合并原博客的几个教程,作为基础的Markdown语法使用指南以供参考'
image: ''
tags: [Markdown, Guide,语法]
category: 'Cookbook'
draft: false 
lang: ''
---

# 这是一级标题

每一个自然段落都由空行分隔。

第二段。 _斜体_, **粗体**, 以及 `强调`. 

这是列表:

- 第一点
- 第二点
- 第三点

不考虑前面的符号的话，文本实际从第四列开始

> 块引用的写法如下
>
> 如果你不介意的话
> 它们可以跨越多个段落

 
使用三条破折号---表示长破折号。使用两条破折号表示范围（例如，“所有内容都位于
12--14 章”）。 三个点 ... 将被转换为省略号。

支持 Unicode。 ☺

## 这是二级标题

这是一个数字列表：

1. 第一点
2. 第二点
3. 第三点

请再次注意实际文本自左起第4个字符开始，下面是代码块示例:

    # Let me re-iterate ...
    for i in 1 .. 10 { do-something(i) }


你可能已经猜到了，缩进 4 个空格。顺便说一句，如果你愿意，也可以使用分隔块，而不是
缩进整个块：

```
define foobar() {
    print "Welcome to flavor country!";
}
```

（这使得复制和粘贴更容易）。您可以选择标记分隔块，以便 Pandoc 对其进行语法高亮：

```python
import time
# Quick, count to ten!
for i in range(10):
    # (but not *too* quick)
    time.sleep(0.5)
    print i
```

### 这是一个三级标题

现在有一个嵌套列表：

1. 第一点:

    - 1-1
    - 1-2
    - 1-3

2. 第二点

3. 第三点:

        find wooden spoon
        uncover pot
        stir
        cover pot
        balance wooden spoon precariously on pot handle
        wait 10 minutes
        goto first step (or shut off burner when done)

    以上。


再次注意文本始终以 4 个空格的缩进排列（包括延续上面第 3 项的最后一行）。


这里是指向[一个网站](http://foo.bar)、[本地文档](local-doc.html)和[当前文档中的章节标题](#an-h2-header)的链接。

这里是脚注[^1]。

[^1]: 脚注的文本写在这里


一个比较简单的表格形式：
| 列1| 列2|
|---|---|
| `第1行` | xxxxxxxxxxxxxxxxxxxxxxxxx|
| `支持列表形式,换行使用<br/>标签` | xxxxx：<br/>1.aaaa<br/>2.bbbbb<br/>3.ccccc|
| `第3行`| xxxxx 
| `第4行`| xxxxx|

<figcaption style="text-align: center; margin-top: 0.5rem; font-style: italic;">
表3-1 一个比较简单的表格
</figcaption>


| 功能描述   | 示例内容演示 |
|------------|--------------|
| `文本换行` | 第一行内容 <br/> 第二行内容 <br/> 第三行 |
| `超链接`   | <a href="https://doodles.google/" target="_blank">试试手气</a> |
| `插入图片` | <img src="/head.png" alt="图例" width="100" /> |
| | |

<figcaption style="text-align: center; margin-top: 0.5rem; font-style: italic;">
表3-2 带图片的表格
</figcaption>

图像的引用方式:
-  直接引用


<!-- <div class="img-column">
  <img src="/head.png" alt="图1" />
  <img src="/head.png" alt="图2" />
</div> -->

<div class="img-grid-caption">
  <div class="img-item">
    <img src="/head.png" alt="图1" />
    <div class="img-caption">图片标题 1</div>
  </div>
  <div class="img-item">
    <img src="/head.png" alt="图2" />
    <div class="img-caption">图片标题 2</div>
  </div>
  <div class="img-item">
    <img src="/head.png" alt="图3" />
    <div class="img-caption">图片标题 3</div>
  </div>
  <!-- 更多图项 -->
</div>


- 带格式引用
<div align="center">  
<img src="https://last9.ghost.io/content/images/2023/05/python-golang--1-.jpg" width="50%" title="可调整大小的网络图片" alt="可调整大小的网络图片" >
</div>

</br>

这是行内公式：$\omega = d\phi / dt$

这是块级公式，自动居中，注意以双美元符号形式包围输入的公式:

$$
I = \int \rho R^{2} dV
$$


$$
\begin{equation*}
\pi = 3.14159\ldots
\end{equation*}
$$

请注意，如果你希望按实际字符显示,你可以使用反斜杠转义任何标点符号：

例: \`foo\`, \*bar\*, etc.


这是一个任务列表：

- [x] task1
- [ ] task2
  - [ ] 2-1
    - [ ] 2-3
- [ ] 3
- [ ] ffsad

