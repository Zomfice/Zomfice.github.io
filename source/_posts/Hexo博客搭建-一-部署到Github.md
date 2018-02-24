---
title: 'Hexo博客搭建 (一):部署到Github'
date: 2018-02-24 14:39:54
tags:
	- Hexo
categories:
	- Hexo博客
---
# Hexo搭建部署到github

#### 导语

> 基于hexo简洁的特点，最近花了点时间搭建了自己的博客，中间遇到些问题，也处理了一些细节。把自己的这个博客搭建过程整理出来，方便自己查看，也希望能帮到其他博客搭建者理清思路。

## Hexo

> [Hexo](https://hexo.io/) 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

## 准备阶段

### Homebrew

> [Homebrew](https://brew.sh/index_zh-cn.html)是Mac OSX上的软件包管理工具，能在Mac中方便的安装软件或者卸载软件，相当于linux下的apt-get、yum神器；Homebre可以在Mac上安装一些OS X没有的UNIX工具，Homebrew将这些工具统统安装到了 /usr/local/Cellar 目录中，并在 /usr/local/bin 中创建符号链接。

我们只需要打开终端，输入以下命令，确定执行后就可以安装Homebrew 了。

` ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" `

为什么需要安装Homebrew？是为了接下来方便的安装Node.js

## 安装Git

由于Mac端已经安装Git，windows需要前往[Git官网](https://git-scm.com/)下载安装.

## 安装Node.js

由于我们使用Hexo框架来搭建生成博客，而Hexo是基于Node.js的。所以，还需要安装Node.js。
安装Node.js只需要在终端中输入：

```
brew install node
```
安装完成后，输入：

```
node -v
```
显示版本号表示已经安装完成，如下：

```
MacBook-Pro:~ ******$ node -v
v7.4.0
```
Node.js中有一个叫做npm(Node Package Manager)的工具，用于搜索、下载、管理Node的相关套件。有了这个东西，我们就能够方便的安装Hexo了。

## 安装Hexo

安装Hexo也十分简单。在终端输入以下命令即可：

```
sudo npm install -g hexo-cli
```
安装完成后，输入hexo -version 如果输出以下类似信息表示安装完成。

```
hexo-cli: 1.0.2
os: Darwin 16.3.0 darwin x64
http_parser: 2.7.0
node: 7.4.0
v8: 5.4.500.45
uv: 1.10.1
zlib: 1.2.8
ares: 1.10.1-DEV
modules: 51
openssl: 1.0.2j
icu: 58.2
unicode: 9.0
cldr: 30.0.3
tz: 2016j
```

## hexo-deployer-git自动部署发布工具

这个工具能够帮助我们快速部署博客bing并发布，安装它只需要在终端输入：

```
npm install hexo-deployer-git --save
```
## Github仓库

我们需要到自己的[Github](https://github.com/)中创建一个仓库，注意名字一定要是这种格式：

```
你的Github账号的注册名称.github.io 
```
注册名：https://github.com/Zomfice(注册名)(创建仓库的地址一定要和注册名一致，后面要通过注册名直接访问)

比如：[https://zomfice.github.io/](https://zomfice.github.io/)

这个名称将成为以后你的博客地址。

## 配置博客

1.	创建本地博客仓库
	
	使用命令:
	
	```
	hexo init 你的Github账号的注册名称.github.io 
	```
	然后会在用户目录下生成一个名为 “你的Github账号的注册名称.github.io” 的文件夹，这个目录就是我们的博客目录。
2. 设置主题
	
	首先，我们一路cd到刚刚创建的目录下。先下载一个主题，这是一个示例，都是一个套路。
	
	```
	git clone https://github.com/iissnan/hexo-theme-next themes/next
	```
	这里我使用的是[NexT](http://theme-next.iissnan.com/)这个主题，文档十分详细。
	
	然后我们在“你的Github账号的注册名称.github.io”目录下找到_config.yml文件，打开它(我是用Sublim Text打开的)。command + F搜索到theme:，在后面加上next，这是我们刚刚存放NexT主题的文件夹名称。
	
	接着需要把这个博客和刚刚创建的Github仓库关联。
command + F搜索deploy: 然后按照以下格式设置：

```
deploy: 
  type: git  //我们使用Git管理
  repo: git@github.com:****/****.github.io.git  //刚刚创建的Github仓库的SSH类型地址，使用HTTPS很有可能遇到权限问题
  branch: master  //你所期望的分支名称
```
注意，以上操作中，:号后面的空格一定不能省略。

`注意：`使用SHH而不使用https，因为https每次同步时候需要输入账号密码，SHH需要提前配置好SHH Key。有关SHH生成配置在下一篇博客。

3. 写文章
	
	我们的文章存放在博客根目录下的source/_posts文件夹下。我们可以在这个文件下建立子文件夹管理文章分类。文章类型被要求必须是.md格式的，就是Markdown。
这里写Markdown与平时稍有不同，需要在头部添加：

```
---
文章配置信息。
这里的配置信息与我们所使用的主题有关。
title: My second blog
date: 2018-02-24 00:05:53
tags:
	- iOS
categories: 
	- Test
---
从这里开始写正文内容...
```

## 发布博客

注意，以下操作需要cd到博客目录下。

```
hexo clean  //每次有大变动，比如更换主题等操作时，可以清理以下之前的缓存；  

hexo g  //即hexo generate，生成静态文件；

hexo s  //即hexo server，启动服务器。默认情况下，访问网址为： http://localhost:4000/； 

hexo d  //即hexo deploy，部署网站到Github。

```

## 参考

* [使用Hexo + Github搭建自己的私人博客](https://www.jianshu.com/p/dd9244bbc550)

* [在Github上面搭建Hexo博客（一）：部署到Github](http://mungo.space/2015/10/12/create-hexo-on-github-1/#more)

* [关于HEXO搭建个人博客的点点滴滴](https://juejin.im/post/5a6ee00ef265da3e4b770ac1)

* [Hexo 入门指南（一） - 简介 & 准备](https://wizardforcel.gitbooks.io/markdown-simple-world/hexo-tutor-2.html)

* [hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)

* [手把手教你使用Hexo + Github Pages搭建个人独立博客](https://linghucong.js.org/2016/04/15/2016-04-15-hexo-github-pages-blog/)

* [如何搭建一个独立博客——简明Github Pages与Hexo教程](https://www.jianshu.com/p/05289a4bc8b2)

* [GitHub Pages + Hexo搭建博客](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/#more)

* [Next主题](http://theme-next.iissnan.com/getting-started.html)

* [Hexo](https://hexo.io/docs/)


