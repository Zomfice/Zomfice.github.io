---
title: 'Hexo博客搭建(三):多设备同步'
date: 2018-02-25 23:31:06
tags:
	- Hexo
categories:
	- Hexo博客
---

# Hexo搭建通过GitHub版本控制思想多端同步

#### 导语

> hexo 是一个静态博客工具，唯一的不足就是源文件无法同步,几乎只能在一台电脑上写博客。而解决方法之一是使用Github 来管理我们的 hexo 源文件。

#### 思路

> 在博客的远程仓库即GitHub的博客youraccount.github.io仓库新建一个分支，用这个分支来存储博客的源文件。Master是hexo deploy的自动生成和管理、hexo分支存储博客的源文件。一个负责管理hexo发布的自动管理，一个用于存储本地的博客源文件，用于实现多端源文件相同。

## 一：在搭建博客开始就将多端同步同时建立

### 博客搭建：一[详情参考](https://www.jianshu.com/p/6fb0b287f950)

同步思路与Github推拉源码思路相同，使用git指令，保持本地的博客文件与Github上的博客文件相同即可，其步骤如下：

##### 1. 使用hexo搭建部署Github博客

	// 在本地博客根目录下安装hexo
	npm install hexo
	// 初始化hexo
	npm init
	// 安装依赖
	npm install
	// 安装部署相关的配置
	npm install hexo-deployer-git
##### 2. 上传博客工程

上一步部署博客到Github以后，我们可以在Github仓库的master分支上看到我们上传的博客文件。

![icon](https://upload-images.jianshu.io/upload_images/291600-9c5faf3a622cf574.png)

但是这个博客文件是不包含hexo配置的，所以我们需要新建分支，使用git指令将带hexo配置的Github工程文件上传到新建的分支上。

![icon2](https://upload-images.jianshu.io/upload_images/291600-fd8d2be4578c9aa4.png)

在本地博客根目录下使用git指令上传项目到Github:
	
	// git初始化
	git init
	// 添加仓库地址
	git remote add origin https://github.com/用户名/仓库名.git
	// 新建分支并切换到新建的分支
	git checkout -b 分支名
	// 添加所有本地文件到git
	git add .
	// git提交
	git commit -m ""
	// 文件推送到hexo分支
	git push origin hexo
	
其他设备上clone下Github上新建的分支的文件到本地
在另一台设备上使用git指令下载Github新建分支上的文件:
	
	// 克隆文件到本地
	git clone -b 分支名 https://github.com/用户名/仓库名.git
	
##### 3. 部署到Github

	// 安装hexo
	npm install hexo
	// 注意这里不需要hexo初始化：hexo init；否则之前的hexo配置参数会重置
	// 安装依赖库
	npm install
	// 安装部署相关配置
	npm install hexo-deployer-git
	
##### 4. 同步项目源文件到Github

	// 添加源文件
	git add .
	// git提交
	git commit -m ""
	// 先拉原来Github分支上的源文件到本地，进行合并
	// 分支名后面的“--allow-unrelated-histories”是为了弹出“fatal: refusing to merge unrelated histories.”的错误
	git pull origin 分支名 --allow-unrelated-histories
	// 比较解决前后版本冲突后，push源文件到Github的分支
	git push origin 分支名
	
	
至此多设备同步到此为止

##### 5. 问题

* Deployer not found: git
	在终端执行命令：
	
	```
	npm install hexo-deployer-git --save
	```
	然后继续执行hexo deploye指令进行部署。
* fatal: could not read Username for ‘ https://github.com ‘: Invalid argument
	由于使用的是https协议，安全性较高，所以系统终端不允许部署，所以我们该用 ssh，修改本地博客hexo配置文件_config.yml，将repository参数修改如下：
	![icon3](https://upload-images.jianshu.io/upload_images/291600-d237476f9941b30c.png)
	继续执行hexo deploye指令进行部署。
* Could not read from remote repository
	这是因为系统没有添加github的ssh信任到本机，所以我们要在命令行执行：
	
	```
	ssh -T git@github.com
	yes
	```
[文章详情参考](https://www.jianshu.com/p/6fb0b287f950)
	

### 博客搭建：二[详情参考](http://zealscott.com/2018/02/21/hexo%E5%A4%9A%E8%AE%BE%E5%A4%87%E5%90%8C%E6%AD%A5%E4%B8%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E5%AE%9E%E7%8E%B0/#more)

	1. 创建仓库，youraccount.github.io
	2. 创建两个分支：master 与 hexo
	3. 设置hexo为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件，master分支为hexo deploy自动生成和管理）
	4. 使用git clone git@github.com:youraccount/youraccount.github.io.git拷贝仓库
	5. 在本地youraccount.github.io文件夹下通过Git bash依次执行npm install hexo、hexo init、npm install 和 npm install hexo-deployer-git（此时当前分支应显示为hexo）
	6. 修改_config.yml中的deploy参数，分支应为master
	7. 依次执行git add .、git commit -m “…”、git push origin hexo提交网站相关的文件
	8. 执行hexo generate -d生成网站并部署到GitHub上

这样一来，在GitHub上的youraccount.github.io仓库就有两个分支，一个hexo分支用来存放网站的原始文件，一个master分支用来存放生成的静态网页

<font color=red>注意：</font>
 
* 分支的创建一定要在Git Bash(博客目录下)。	切不可在GitHub仓库直接点击生成新的分支，我在仓库直接生成分支，导致一直上传失败，然后需要先pull仓库文件，导致本地出现源文件和自动生成管理文件混合 
* 当不知道当前分支的时候查看当前分支 git branch -v
  
  ```
MacBook-Pro:Zomfice.github.io WZY$ git branch -v 
* hexo   2d90e51 修改
  master 7b6392f [ahead 1, behind 14] 提交分支hexo
  ```
  
<font color=red>补充(遇到的问题)：</font>

##### 问题1：由于我是先配置好博客，才处理多端同步问题，然后直接在GitHub新建hexo分支，然后对本地文件夹和仓库关联，然后执行push,发现无法提交，查看GitHub仓库自动生成管理文件存在，然后拉取分支，本地文件和自动生成管理文件混合。

解决：

1. 首先进行版本回滚，回滚到[之前版本](https://ws1.sinaimg.cn/large/ad3a9ce5ly1fot5w7h6eij210q0b0n0t.jpg)

2. 远程分支存在需要删除,[git怎么删除远程分支](https://segmentfault.com/q/1010000008841093)

3. 本地也初始化了一个分支(注:  git checkout -b 分支名 新建分支并切换到新建的分支),删除本地分支

4. [删除掉没有与远程分支对应的本地分支](https://segmentfault.com/q/1010000008841093)

	查看远程分支
	
	```
	git remote -v
	```
	查看本地分支
	
	```
	git branch  -v
	```
	新建并切换到新的分支
	
	```
	git checkout -b 分支名
	```
	删除远程分支
	
	```
	git branch -r -d origin/branch-name
	```
	删除本地分支
	
	```
	git branch -d xxxxx
	```
	删除掉没有与远程分支对应的本地分支
	
	```
	git fetch -p
	```
	切换分支
	
	```
	 git checkout -b hexo
	```
	[git 记录你的每一个命令](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013744142037508cf42e51debf49668810645e02887691000)
	
	```
	git reflog
	```
	git 状态
	
	```
	git status
	```
##### 问题2： modified:   ../themes/next (modified content)

[解决](https://segmentfault.com/q/1010000010287878)：hexo g 生成静态页面
hexo d 部署hexo之后提交代码

##### 问题3：[Hexo deploy 发布不成功](https://github.com/hexojs/hexo/issues/67)

##### 问题4：[npm WARN enoent ENOENT: no such file or directory, open '<root>/node_modules/supertest/package.json'](https://github.com/visionmedia/debug/issues/261)本地文件package丢失

解决：重新对npm配置，但是不要hexo init，重新对npm和hexo建立依赖,如果不行，执行npm cache clean之后再执行

## 二：(推荐)在搭建完成博客之后将多端同步加入

通过教程一搭建完成博客，然后通过加入多端同步，只需对前文 博客一步骤进行精简就好.将本地仓库与远程仓库建立连接，创建远程分支，并切换到新建分支。

	// git初始化
	git init
	// 添加仓库地址
	git remote add origin https://github.com/用户名/仓库名.git
	// 新建分支并切换到新建的分支
	git checkout -b 分支名
	// 添加所有本地文件到git
	git add .
	// git提交
	git commit -m ""
	// 文件推送到hexo分支
	git push origin hexo
	//执行生成网站并部署到GitHub上
	hexo g -d  
<font color=red>参考：</font>

* [多设备同步hexo搭建的Github博客](https://www.jianshu.com/p/6fb0b287f950)

* [HEXO换电脑麻烦解决](http://dontcry2013.github.io/2016/03/02/hexo-change-workstation/)
## 三：配置完成多端同步，新设备如何配置使用

##### 1. 在新设备中安装node.js和Git

无需多说，直接点击链接安装：[node.js](https://nodejs.org/en/)，[Git](https://gitforwindows.org/)。

##### 2. 给新设备添加SSH KEYS

在Git Bash中输入：

> ssh-keygen -t rsa -C “你的邮箱地址”

按三次回车（密码为空），生成密匙。

在C:\Users\Administrator.ssh中（Administrator为自己对应的管理员账号），得到两个文件，分别为id_rsa和id_rsa.pub。

打开id_rsa.pub，复制全文。进入GitHub中的[SSH设置](https://github.com/settings/keys) ，Add SSH key，粘贴进去。

##### 3. 新设备同步

使用git clone git@github.com:youraccount/youraccount.github.io.git拷贝仓库（默认分支为hexo）

在本地得到的youraccount.github.io文件夹下通过Git bash依次执行下列指令：
npm install -g hexo、npm install、npm install hexo-deployer-git即可将最新的博客文件全部同步。

<font color=red>参考:</font>

* [hexo多设备同步与版本控制实现](http://zealscott.com/2018/02/21/hexo%E5%A4%9A%E8%AE%BE%E5%A4%87%E5%90%8C%E6%AD%A5%E4%B8%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E5%AE%9E%E7%8E%B0/#more)


## 四：日常维护使用

在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理：
	
	1. hexo new "文章名"
	2. git add .
	3. git commit -m “…”
	4. git push origin hexo
	5. hexo g -d

<font color=red>参考:</font>

* [HEXO换电脑麻烦解决](http://dontcry2013.github.io/2016/03/02/hexo-change-workstation/)

* [hexo多设备同步与版本控制实现](http://zealscott.com/2018/02/21/hexo%E5%A4%9A%E8%AE%BE%E5%A4%87%E5%90%8C%E6%AD%A5%E4%B8%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E5%AE%9E%E7%8E%B0/#more)



## 参考

##### Git
* [Git命令](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)
* [Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
* [Git代码回退](https://www.jianshu.com/p/f7451177476a)
* [git 查看远程分支、本地分支、创建分支、把分支推到远程repository、删除本地分支](http://blog.csdn.net/arkblue/article/details/9568249)
* [删除掉没有与远程分支对应的本地分支](https://segmentfault.com/q/1010000008841093)
* [处理合并冲突](https://www.git-tower.com/learn/git/ebook/cn/command-line/advanced-topics/merge-conflicts)

##### 多设备

* [多设备同步hexo搭建的Github博客](https://www.jianshu.com/p/6fb0b287f950)
* [hexo多设备同步与版本控制实现](http://zealscott.com/2018/02/21/hexo%E5%A4%9A%E8%AE%BE%E5%A4%87%E5%90%8C%E6%AD%A5%E4%B8%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E5%AE%9E%E7%8E%B0/#more)
* [hexo 博客利用 github 分支同步源文件](http://imweb.io/topic/5848d4259be501ba17b10a9a)
* [在Github上面搭建Hexo博客（四）:使用不同电脑维护](http://mungo.space/2015/10/14/create-hexo-on-github-4/)
* [HEXO换电脑麻烦解决](http://dontcry2013.github.io/2016/03/02/hexo-change-workstation/)
* [使用hexo，如果换了电脑怎么更新博客？](https://www.zhihu.com/question/21193762)

##### 配置

* [多设备同步hexo搭建的Github博客](https://www.jianshu.com/p/6fb0b287f950)
* [HEXO换电脑麻烦解决](http://dontcry2013.github.io/2016/03/02/hexo-change-workstation/)
* [hexo多设备同步与版本控制实现](http://zealscott.com/2018/02/21/hexo%E5%A4%9A%E8%AE%BE%E5%A4%87%E5%90%8C%E6%AD%A5%E4%B8%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E5%AE%9E%E7%8E%B0/#more)
* [在Github上面搭建Hexo博客（四）:使用不同电脑维护](http://mungo.space/2015/10/14/create-hexo-on-github-4/)

##### 使用

* [HEXO换电脑麻烦解决](http://dontcry2013.github.io/2016/03/02/hexo-change-workstation/)
* [hexo多设备同步与版本控制实现](http://zealscott.com/2018/02/21/hexo%E5%A4%9A%E8%AE%BE%E5%A4%87%E5%90%8C%E6%AD%A5%E4%B8%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E5%AE%9E%E7%8E%B0/#more)
