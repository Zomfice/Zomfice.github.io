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

### 博客搭建

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

## 三：配置完成多端同步，新设备如何配置使用

## 四：日常维护使用

## 参考

* [Git命令](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)
* [Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
* [Git代码回退](https://www.jianshu.com/p/f7451177476a)
* [git 查看远程分支、本地分支、创建分支、把分支推到远程repository、删除本地分支](http://blog.csdn.net/arkblue/article/details/9568249)
* [删除掉没有与远程分支对应的本地分支](https://segmentfault.com/q/1010000008841093)
