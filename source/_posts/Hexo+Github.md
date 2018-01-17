---
title: Hexo+Github搭建个人博客
date: 2018-01-17
categories: Hexo
author: Sunny
tags:
    - Hexo
cover_picture: http://upload-images.jianshu.io/upload_images/5231076-3385be91b57dabf4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
---
![header.png](http://upload-images.jianshu.io/upload_images/5231076-3385be91b57dabf4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 安装Node.js
Windows下安装Node.js还是比较简单的，直接去[官网下载](https://nodejs.org/en/ "官网下载")安装即可，不需要任何其他的配置。

#### 安装Git
根据自己电脑的参数去[Git官网](https://git-scm.com/download/win "Git官网")下载安装对应的版本即可，安装完成之后，在cmd命令窗下输入`git version`，有输出Git版本号则说明安装成功。

**配置SSH Key**
在桌面右键打开Git Bash，输入下面命令：
```
$ cd ~/.ssh
$ ls #检查本机是否有已存在密钥
```
如果是第一次使用Git，估计是没有配置过SSH密钥，具体判断有没有配置密钥的依据是执行上面命令后有没有打印出.pub为后缀的文，有.pub为后缀的文件则说明之前配置过SSH密钥，那么我们接下来应该看看这个密钥有没有在Github上添加过，如果添加过就不用再添加了。没有.pub为后缀的文件则说明本地没有配置过SSH密钥，需要按照下面的步骤进行配置：

生成密钥(一个客户端只能有一个密钥）：
```
$ ssh-keygen -t rsa -C "邮件地址"
```
输入上面命令回车之后第一句提示是让你修改保存密钥的文件名，可以不用管，直接回车，接下来的提示是让你设置密码，如果设置了，以后每次从本地往Github上部署的时候会提示输入密码才能部署，这样的话有更好的安全性，当然了，你也可以不设置，直接回车跳过。

等待密钥生成完后，输入下面两条命令就会看到.pub后缀的文件：
```
$ cd ~/.ssh
$ ls
id_rsa  id_rsa.pub
```

我们复制`id_rsa.pub`这个文件：
```
$ clip < ~/.ssh/id_rsa.pub
```
然后再把这个密钥配置到Github上。

测试密钥是否配置成功：
```
$ ssh -T git@github.com
```
成功会有如下提示：
```
Hi Sunny8519! You've successfully authenticated, but GitHub does not provide shell access.
```

#### 安装和初始化Hexo
第一步：
全局安装Hexo:`npm install -g hexo`

第二步：
创建文件夹（比如我的是D盘下的Hexo文件夹）；
在Hexo文件夹下，右键运行Git Bash，输入命令：`hexo init`

到这儿，本地最简单的Hexo就搭建完了，然后我们在本地测试一下：
```
$ hexo g
$ hexo s
```
打开浏览器输入`localhost:4000`，回车就能看到我们本地的博客界面了。

第三步：
当看到我们的博客界面时发现不是特别的美观，那么我们可以[下载](https://hexo.io/themes/ "下载")自己喜欢的主题(这块的主题一般是进入到发布者的Github上去看具体的下载和设置方式，这里不详细表述)。
这里举一个我用的主题的例子，在前面创建的Hexo文件夹下右键打开Git Bash输入下面这条命令：
```
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```
这里附上该主题的[Github地址](https://github.com/litten/hexo-theme-yilia "Github地址")
下载完这个主题之后，我们需要修改`_config.yml`文件：
```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: yilia
```
保存之后，再输入下面两条命令：
```
$ hexo g
$ hexo s
```
打开浏览器输入`localhost:4000`就能看到我们新设置的主题了，接下来主题功能的具体配置可以到发布者的Github上详细了解。

到这里，本地博客的搭建就基本完毕了，接下来就是要把我们的博客部署到Github上。

第一步：在Github上建立如下仓库：
![建立仓库](http://upload-images.jianshu.io/upload_images/5231076-030e31bb8eb87f76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
但是要注意的是仓库的名称由你的Github用户名+github+io组成，如图中所示，因为这个名称就是你博客搭建完毕后访问的首页地址（图中报红是因为我已经搭建了这个仓库）。

第二步：仓库创建完之后，我们需要把仓库的地址设置到Hexo文件夹下的`_config.yml`文件中，如图所示：

![copy_address.png](http://upload-images.jianshu.io/upload_images/5231076-ac97e20ff02f2365.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

把地址按照如下方式设置：
```
deploy:
  type: git
  repo: git@github.com:Sunny8519/sunny8519.github.io.git
  branch: master
```
保存好了之后，我们在Hexo文件夹下右键打开Git Bash命令窗口，安装hexo-deployer-git自动部署发布工具：
```
$ npm install hexo-deployer-git  --save
```
到这儿所有工作都准备好了，我们继续在命令窗口中输入如下三条命令就能把本地博客系统部署到Github上了：
```
$ hexo clean
$ hexo g
$ hexo d
```
然后打开浏览器输入你的`Github用户名.github.io`就能够访问你的博客首页了。

#### 参考
[搭建个人博客-hexo+github详细完整步骤](https://www.jianshu.com/p/189fd945f38f)

[MiHo-主题安装和配置详情](https://blog.minhow.com/2017/08/01/blog/installation-configuration/)
