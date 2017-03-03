---
layout: post
title: "再次整理一下在Github上创建个人站点"
---

<p>
首先把需要的工具理顺:
</p>
- **WebStorm** 
(`作为jekell网站结构的IDE编辑器`) 
- **MarkdownPad2** (`作为发布blog日志的编辑器`，(http://markdownpad.com))
- **GitHub for windows** (`windows桌面版的github，非常方便，只要本地有更新，就能检测到`，(https://desktop.github.com/))
- **已注册域名** (`本人早在很早以前就注册了本人姓名的域名`，（https://wanwang.aliyun.com/domain/）)
- **N多reference doc** 
([jekyll bootstrap](http://jekyllbootstrap.com/), [jekyll themes](https://jekyllthemes.io/), [hexo](https://hexo.io/), [基于GitHub和Jekyll的博客搭建](http://phoenixwu.cn/2017/02/20/share-start-blog/),[jekyll 中文官网](http://jekyll.com.cn),[jekyll 官网](http://jekyllrb.com/))

接下来就比较简单：

First 1：
	找到一个一个jekelly的模板，download下来。然后根据所在的操作系统平台搭建本地的web blog 站点（其实完全没有必要，只要下载下来设置好github的**GitHub Pages**然后再把这个站点上传上去即可。具体操作看最原版的官方教程，中文翻译来，抄来抄去的无聊至极。[GitHub pages 官网教程](https://pages.github.com/)

First 2：申请自己想要的域名，并绑定到GitHub自己的空间上。如下图
<p>
![pages的空间名称设置](![image](https://github.com/jiangzerui/picture_blog/raw/master/repository%20setting.png))


![申请成功域名后在pages的设置](![image](https://github.com/jiangzerui/picture_blog/raw/master/domain%20setting.png))

First 3：用webstome打开本地和Github共享的个人空间目录，在根目录上创建html文档的文件，命名“CNAME”，文件里面加入个人申请的域名。<p>
![CNAME目录结构及内容](![image](https://github.com/jiangzerui/picture_blog/raw/master/blog目录结构及CNAME内容.png))

First 4：接下来就踏实在posts目录下面用mdp写文章吧