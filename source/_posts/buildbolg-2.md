---
title: 打造更健全的博客空间 - 建站指引2
date: 2017-06-05 14:16:20
categories: 前端
comments: true
articleId: buildblog-2
---
根据建站指引1的步骤，我们搭建了一个最基本的个人博客。本人使用atom编辑器自带的markdown系统进行写作，结合atom的markdown插件，能给我们带来更舒畅的写作体验。同时，给我们的博客添加第三方服务集成能使我们站点功能得到扩展。这篇文章能为你解决下面的问题：
1. 简化 Markdown 贴图流程
2. 其他 Atom Markdown 插件
3. 集成 Gitment 评论系统

<!--more-->

## Atom Markdown 优化插件
### 1. 使用快捷键插入截图
在使用hexo时，我们可以使用网图或者本地图来作为文章图片的引用。而当我们想直接在文章中使用截图时，一般需要经过如下几个步骤：
- 截图
- 上传至图床
- 将图床markdown代码插入文章中

由此看来，插入截图时比较繁琐的，而我们希望的是截图之后能直接快捷键将其贴入我们的文章中。查阅了几篇文章，最后决定使用atom的插件`markdown-img-paste`（贴图快捷键为`ctrl+shift+v`）。

> 如非使用atom编辑器，请参考以下两篇相似内容文章
[windows版本markdown一键贴图工具](http://jverson.com/2017/05/28/qiniu-image-v2/)
[Mac版本使用alfred在markdown中愉快的贴图](http://jverson.com/2017/04/28/alfred-qiniu-upload/)

回到上文内容，`markdown-img-paste`支持直接截图上传至七牛图床和上传至本地文件夹。但是如果选择本地储存的话，是直接将图片存入assets文件夹再进行引用的，而hexo的图片资源需要souces/images下。所以我将其直接上传至七牛云。下载并安装好该插件后，对此进行七牛云的授权和设置。对于没有七牛云账号的同学们，可以到[七牛云](https://portal.qiniu.com/create)申请一个账号，__并创建储存空间__。

调出Preferences窗口，找到`markdown-img-paste`package进行设置。配置如下：
![](http://oqwh8w9hs.bkt.clouddn.com/markdown-img-paste-2017060517125265.png)

提一下几个参数：
1. 首先得勾选 use qiniu for image link 使用七牛云储存图片
2. accesskey 和 secretkey 是我们七牛账号的两个唯一秘钥，可在*个人面板-密钥管理*中查看
3. bucket 是我们创建的储存空间的名称
4. domain 是我们储存空间的域名
5. uphost 默认是华东的存储空间即 up.qiniu.com
华北的存储空间 使用 up-z1.qiniu.com
华南的存储空间 使用 up-z2.qiniu.com
（这里没注意到这个有可能会报错）

设置好之后，就可以愉快地截图>一键粘贴>上传 啦～（注意一下，gif是没有办法直接粘贴上传的）

![](http://oqwh8w9hs.bkt.clouddn.com/test.gif)


### 2. 其他优化markdown写作体验插件
我是好文章的搬运工。。。 (。・`ω´・)
[使用Atom打造无懈可击的Markdown编辑器](http://www.cnblogs.com/fanzhidongyzby/p/6637084.html)

## 集成Gitment评论系统
多说倒闭了，想必大家都了解过。比对了市面上几款评论系统，觉得他们外观都不太简约，或者色调和我选择的主题不相符。而口碑比较好的disqu则在国内会被墙掉。所以，这里选了一款较新的评论系统[gitment](https://imsun.net/posts/gitment-introduction/)。
如何将gitment嵌入hexo的主题中，是下文将要讨论的问题，对于gitment的其他Api，请自行查阅gitment的readme。

### 1. 注册 OAuth Application
首先得注册一个OAuth Application，并获得client_id和secret_id。[传送门](https://github.com/settings/applications/new)

### 2. 在配置文件添加gitment支持
找到主题文件下的_config.yml，自行添加字段：
```java
gitment: true
```
### 3. 在comments文件中加入gitment配置
找到 主题/layout/_partials/comments.swig（也可能不是swig文件，但逻辑是类似的），加入代码
```java
{% elseif theme.gitment %}
<link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
<script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
<script>
  var gitment = new Gitment({
    // 根据文章标题设置唯一评论id
    id: '{{page.articleId}}',
    owner: 'yourname',
    repo: 'your blog github repo',
    oauth: {
      client_id: 'your clientid',
      client_secret: 'your clientsecret',
    },
  })
  gitment.render('comments')
</script>
```
配置项的id若不设置的话，默认为 location.href，尝试过不设置该项，但若文章url含有hash #more 时，会拉取不到评论。

### 4. `.md`文章下的配置
在头部添加下面代码
```java
comments: true
/**
  若comments.swig中的id没配置的话，忽略该项
  注意这里的artile name是指你md文件的命名而不是文章标题
**/
articleId: your article.md's name
```

以上，我们便搭建好了一个较为健全的博客空间～

![](http://oqwh8w9hs.bkt.clouddn.com/markdown-img-paste-20170605182754871.png)
