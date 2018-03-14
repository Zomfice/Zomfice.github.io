---
title: 'Hexo博客搭建(五):日常维护命令与配置'
date: 2018-02-26 14:35:44
tags:
	- Hexo
categories:
	- Hexo博客
---

# 日常维护与主题配置

## 日常维护命令

##### 1. 博客创建，文件提交，部署

	hexo new "博客文章题目"	//创建文章
	git add . 				//注意最后的 . ,这个表示当前目录
	git commit -m "....."	//本地commit
	git push origin hexo   	//提交网站相关文件
	git pull origin hexo	//拉取分支
	hexo g -d 				//执行生成网站并部署到GitHub上

##### 2. 其他可能用到命令

	git branch hexo   		//创建hexo分支
	git checkout hexo   	//转换到hexo分支
	rm -rf *   				//删除所有文件，隐藏文件不会删除
	git status
 
##### 3. 本地zomfice.github.io文件下执行命令
	
	npm install hexo 
	hexo init 
	npm install 
	npm install hexo-deployer-git   ＃此时当前分支应显示为hexo
	
##### 4. Tags和分类样式

	---
	title: 'Hexo博客搭建 (一):部署到Github'
	date: 2018-02-24 14:39:54
	tags:
		- Hexo
	categories:
		- Hexo博客
	---

##### 5. 站点配置文件与主题配置文件

[站点文件与主题配置文件区别](https://juejin.im/post/5a6ee00ef265da3e4b770ac1)

[官方站点文件与主题配置文件](http://theme-next.iissnan.com/getting-started.html)

	├── _config.yml
	├── package.json
	├── scaffolds
	├── source
	|   ├── _drafts
	|   └── _posts
	└── themes
	
	站点配置文件：_config.yml
	主题配置文件:themes/next/_config.yml
	
##### 6.语言修改
	
	站点配置文件下:
	# Site
	title: Zomfice’s blog
	subtitle:
	description: 巧xxxxxxx
	author: Zomfice
	language: zh-Hans
	timezone:

##### 7. 代码框序号

	站点配置文件
	highlight:
	enable: true
	line_number: false(true->false)
	auto_detect: false
	tab_replace:

##### 8. 仓库配置

	站点配置文件
	# Deployment
	## Docs: https://hexo.io/docs/deployment.html
	deploy:
	type: git
	repo: git@github.com:Zomfice/Zomfice.github.io.git
	branch: master

##### 9. Next主题首页文章显示摘要，显示阅读全文

[Hexo之next主题设置首页不显示全文(只显示预览)](https://www.jianshu.com/p/393d067dba8d)

	主题配置文件
	# Automatically Excerpt. Not recommand.
	# Please use <!-- more --> in the post to control excerpt accurately.
	auto_excerpt:
	enable: false(false->true)
	length: 150
	
* [hexo主页以摘要方式显示](https://ohmyarch.github.io/2014/12/24/Hexo%E4%B8%BB%E9%A1%B5%E6%98%BE%E7%A4%BA%E6%91%98%E8%A6%81/)
	
##### 10. 怎么让百度和谷歌搜到你的文章呢

* [怎么让百度和谷歌搜到你的文章呢](http://lijialalala.github.io/2016/04/05/hexoxo-usage/)
* [｜Hexo优化｜如何向google提交sitemap（详细）](http://fionat.github.io/blog/2013/10/23/sitemap/)
* [博客推广——提交搜索引擎](http://selfboot.cn/2014/12/21/add_blog_to_google/)

##### 11. 搭建遇到的问题

* [个人搭建HEXO并部署到GitHub的期间遇到的问题](http://lijialalala.github.io/2016/04/05/hexoxo-usage/)

##### 12. 分类标签怎么删除 

* [hexo 分类 标签怎么删除](https://segmentfault.com/q/1010000007070284)

##### 13. 分类标签无法显示

分类和标签只要的.md文档中按照格式写，就会自动在Tags和categories中显示

* [hexo 下的分类和表签无法显示，怎么解决？](https://www.zhihu.com/question/29017171)

##### 14. hexo发布博客

	hexo clean  //每次有大变动，比如更换主题等操作时，可以清理以下之前的缓存；  
	hexo g  	//即hexo generate，生成静态文件；
	hexo s  	//即hexo server，启动服务器。默认情况下，访问网址为： http://localhost:4000/； 
	hexo d  	//即hexo deploy，部署网站到Github。

hexo常用命令

	hexo n "我的博客" == hexo new "我的博客" #新建文章
	hexo p == hexo publish 				  #发布草稿
	hexo g == hexo generate               #生成静态网页
	hexo s == hexo server 				  #启动服务预览
	hexo d == hexo deploy                 #部署
	
##### 15. 主题优化

* [HEXO +下一步个人博客主题优化](https://www.jianshu.com/p/efbeddc5eb19)

##### 16. 购买域名

* [购买域名](https://www.jianshu.com/p/05289a4bc8b2)

##### 17. 统计、分享、RSS、更新、域名

* [hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)

##### 18. 主题配置与更换

* [hexo主题配置](http://theme-next.iissnan.com/theme-settings.html)
* [hexo更换主题、删除文章](http://oakland.github.io/2016/04/30/hexo-%E5%A6%82%E4%BD%95%E6%9B%B4%E6%8D%A2%E4%B8%BB%E9%A2%98%E3%80%81%E5%88%A0%E9%99%A4%E6%96%87%E7%AB%A0/)

## 主要参考文档

* [创建](https://www.jianshu.com/p/dd9244bbc550)
* [多设备同步](https://www.jianshu.com/p/6fb0b287f950)
* [SHH key](https://www.cnblogs.com/ayseeing/p/3572582.html)
* [Markdown](https://www.jianshu.com/p/1e402922ee32)
* [使用](http://dontcry2013.github.io/2016/03/02/hexo-change-workstation/)
* [Hexo官方](https://hexo.io/docs/)
* [Hexo-Next主题](https://hexo.io/zh-cn/docs/themes.html)


