Sen's Blog
======

> 该博客使用 jekyll+bootstrap-3+Markdown 搭建。
> 
> 使用本主题，请先修改以下内容：
> 
> - 修改 _config.yaml
> - 修改 CNAME
> - 修改 about.md个人简介中的内容
> - 修改 _data/blogroll.json 中的博客链接

## Thanks To

主题从 <https://github.com/javachen/javachen-blog-theme> 修改而来，感谢原作者！[http:/blog.javachen.com](http://blog.javachen.com)。

## 改进的地方
主要改进以下地方：

* 修改部分css
* 加入[TOC](https://github.com/ghiculescu/jekyll-table-of-contents)（table of content）功能
* 加入对[mathjax](https://www.mathjax.org/)的支持
* 加入知乎图标
* 加入对[Disqus](https://disqus.com/)的支持，Disqus和多说二选一，默认为多说。

## 使用方法
### 创建博文
在此文件夹内输入如下命令行：

```bash
 rake post title = "you blog title"
```

### 网页预览
在次文件夹内输入如下命令行：

```bash
 rake preview
```

或者：

```bash
 jekyll build -w
```

然后用浏览器打开 [127.0.0.1:4000](127.0.0.1:4000)
