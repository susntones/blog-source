---
title: hello-hexo
date: 2018-10-04 11:48:05
tags:
---

# Hello hexo #

&emsp;&emsp;今年在学习《高性能mysql》这本书的时候，产生的写博客的想法，希望能够记录自己学到的东西，只求打开自己博客的瞬间能够回忆起自己所学。当然，如果我总结的知识能够帮到别人，会更完美。<br/>
&emsp;&emsp;从知乎上了解到hexo这个博客框架，同时因为自已现在主要还是做nodejs开发，hexo用起来一定会相当顺手，就毫不犹豫的选择了。

## Hexo 的搭建过程 ##
&emsp;&emsp;hexo 的整体配置还是很简单的。

1. 所有工具最正经的学习路径一定是看 [<font color=#3399ea>官方文档</font>](https://hexo.io/zh-cn/docs/)，hexo提供了中文文档（对于我这个英语半吊子的人来说很舒服）。
2. 选择一个自己喜欢的主题。当前最火的主题毫无疑问是 [<font color=#3399ea>next</font>](https://github.com/iissnan/hexo-theme-next)（已经有13k+的star了），这也是我的选择。我最喜欢的scheme 是 pisces。
3. [<font color=#3399ea>hexo文档</font>](https://hexo.io/zh-cn/docs/writing) 学习，了解更多feature。

## Hexo 使用 ##

### layout ###

* hexo提供了三种默认布局：post、page、draft。
* 自定义的布局和post，会存储到 ***source/_posts*** 文件夹。
* 自定义的layout 都配置在 scaffolds 文件夹中。
* 可以编辑 new_post_name 参数来改变默认的文件名称（e.g. :year-:month-:day-:title.md）。
* 特殊布局：draft，这种布局会将建立的文章保存到 source/_draft 文件夹，你可以通过 hexo publish [layout] \<title\>  将草稿移动到 source/_posts 文件夹。
* 草稿默认不会显示在页面中，您可在执行时加上 --draft 参数，或是把 render_drafts 参数设为 true 来预览草稿。