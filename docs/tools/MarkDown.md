# Markdown 入门
[TOC]

## 0. Markdown是什么

Markdown 是一种轻标记文字编辑语言，可以使文字编辑工作更加专注于内容。


**为什么是Markdown？**

相比Word这一类富文本编辑器，Markdown简洁而不失其结构，让人脱离繁琐的格式化工作，主要优点如下：

* `高效` 
使用者可以专注于写作内容与结构，而非样式
* `轻排版`
语法简单易学，随时进行排版
* `全兼容`
纯文本格式，不用考虑兼容性问题

## 1. Markdown客户端利器

在学习Markdown语法前，首先需要一款顺手的编辑器，个人比较推荐 Vscode。
Vscode原生集成了Markdown的预览页面，可以在编写的同时对文档的效果进行实时预览。此外Vscode的插件库中上架了多款有关Markdown的扩展应用插件，例如`Markdown Preview Enhanced`，可以将Markdown在浏览器中进行预览，或者导出为PDF/HTML/图片等形式。

## 2. Markdown核心语法

元素 | 语法
- | -
标题 | # H1<br> ## H2<br> ### H3<br>
加粗 | \*\*bold text\*\*
斜体 | \*italicized text\*
引用 | > blockquote
有序列表 | 1. First item<br>2. Second item<br>3. Third item
无序列表 | - First item<br>- Second item<br>- Third item
单行代码段 | \`code\`
多行代码段 | \```java <br>mutiline code<br>```
水平分隔线 | ---
链接 | \[title](https://www.example.com)
图片 | \!\[title](image.jpg)


## 3. Markdown高级语法

元素 | 语法
- | -
表格 |  Syntax \| Description <br>- \| -<br>Header \| Title <br>Paragraph \| Text<br>
颜色 | \$\color{colorCode}{text}$
注脚 | Here's a sentence with a footnote. \[^1] <br> \[^1]: This is the footnote.
标题id | ### My Great Heading {#custom-id}
定义列表 | term<br>: definition
删除线 | \~~The world is flat.~~
任务列表 | - [x] Write the press release<br> - [ ] Update the website<br>- [ ] Contact the media
emoji | `:joy:`
转义字符 | \

---
参考地址：[Markdown Cheat Sheet](https://www.markdownguide.org/cheat-sheet/)
