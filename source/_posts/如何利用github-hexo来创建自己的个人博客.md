title: 如何利用github+hexo来创建自己的个人博客
date: 2015-12-30 20:59:17
tags: 
- git
categories: 
- git

---


# 开始

首先你得需要以下几样东西，缺一不可。

* [Node.js](https://nodejs.org/en/)
* Github 账号
* hexo

## 安装环境

### 安装node.js
去node.js.js的官网下载自己平台对应的最新版本，只要安装成功，后来便不需要管它了。

### 安装git
git的客户端很多，关于git的使用我在这里不方便展开说，不清楚的同学我推荐大家去* [廖雪峰老师的网站](http://www.liaoxuefeng.com/)*，上面的git教程对于新手来说是十分友好的，我git的入门也是在这上面看的，比较清晰易懂。

### 安装Sublime
我自己的电脑是mac，所有有些代码的格式在xcode上面打开还是不太方便查看，sublime text 支持多种编程语言，还有很多种特色配色方案，对于这个软件的代码高亮我非常喜欢。
当然这个软件也是支持markdown语法的，把它当做一个文本编辑器也是一个很好的选择，我现在正在写的这篇日志也是用它编写的。
<!-- more -->
## githubpage设置

![](/images/github截图.png)

在github的页面新建一个仓库，也就是点击图片上绿色的按钮。

![](/images/github截图2.png)
特别要注意的是，你的项目名字必须是username.github.io这样的格式。特别项目名中的username要和*用户名*一直，开始我起了其他的名字导致一直不能正常使用。现在我们就完成了github上面的配置了。

![](/images/github截图3.png)

现在我们进入到我们新建的那个项目里面去，我们点击setting，设置我们的githubpage，基本上第一次申请，过几分钟后就成功了，然后我们访问username.github.io就可以访问我们申请的githubpage 页面了。（*将username替换成你得用户名*）这个githubpage也就是我们在github上面申请的免费的个人空间，大小有300m，作为个人博客是足够用了。

现在我们回到本地来，我们从命令行界面，通过下列命令将远程库clone到本地来。之前我们首先新建一个文件夹，如myblog，然后切换到这个文件内，执行下列命令。
```
git clone https://github.com/chouchang/chouchang.github.io
```
好了，此时会自动在*myblog*目录下创建子文件夹chouchang.github.io,那里就是我们博客的代码，以后的操作都在git的管理之下了，此时默认的branch为master。

## 安装hexo
当node.js安装和git安装好以后变可以安装hexo了，输入以下命令。
```
npm install -g hexo-cli
```

### 建站
安装完成后，输入以下命令，hexo会再指定文件夹中新建所需要的文件。
```
hexo init <folder>
cd <folder>
npm install
```
init命令中的<folder>就是文件夹chouchang.github.io。初始化后，chouchang.github.io里面就已经有完整的Hexo框架了。
创建完毕后，文件夹中会存在一些新的文件，每个文件夹的具体作用，请参考[hexo官网](https://hexo.io/zh-cn/docs/setup.html)。

### 主题
我博客用的hexo的主题是next主题。

#### 评论
因为next主题自带多说评论设置，具体主题配置方面大家可以参考next安装多说评论。

## 域名
我的域名是在万网上面购买的，其实也可以不用购买域名，但是有了域名才让人更好的记住啊。像我的*vaexiaochang.cn*多么炫酷。
去万网上购买域名成功后，我们要做个域名解析，能让人么访问你的域名时自动跳转到githubpage页面。所以我们在域名解析里添加两条设置。下面是我的配置。

![](/images/域名解析.png)

解析完成以后我们还需要在githubpage里面新建一个CNAME文件，里面写入自己申请的域名。因为githubpage不是什么页面都能指向的，需要自己在里面配置CNAME。
可以参考下我的[CNAME](https://github.com/chouchang/chouchang.github.io/blob/master/CNAME)
这里还是有一点要注意的，因为每次hexo发表新文章的时候都会把之前的东西都清空，然后上传最新的上去。所以我们最好再hexo框架里面的public的source文件夹里放一个CNAME文件。

## 总结
还有一些细节方面没有补充清楚，这两天自己病了，等回去后再把这篇博文补充清楚，如果有什么疑问欢迎大家在下面评论，由于自己也是刚刚建好，很多地方写的可能也有问题,如有错误欢迎斧正。

个人邮箱：xiaochangvv@gmail.com

