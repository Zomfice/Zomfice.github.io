---
title: 'Hexo博客搭建 (二):配置Github的SSH key'
date: 2018-02-24 17:08:58
tags:
	- Hexo
	- SSH
categories: 
	- Hexo博客
---

# 配置Github的SSH Key

## 什么是SSH

> 很多朋友在用github管理项目的时候，都是直接使用https url克隆到本地，当然也有有些人使用 SSH url 克隆到本地。然而，为什么绝大多数人会使用https url克隆呢？

> 这是因为，使用https url克隆对初学者来说会比较方便，复制https url 然后到 git Bash 里面直接用clone命令克隆到本地就好了。而使用 SSH url 克隆却需要在克隆之前先配置和添加好 SSH key 。

> 因此，如果你想要使用 SSH url 克隆的话，你必须是这个项目的拥有者。否则你是无法添加 SSH key 的。

生成<font color=red>多个公钥</font>请点击[http://www.cnblogs.com/ayseeing/p/4445194.html](http://www.cnblogs.com/ayseeing/p/4445194.html)

## https 和 SSH 的区别：

1、前者可以随意克隆github上的项目，而不管是谁的；而后者则是你必须是你要克隆的项目的拥有者或管理员，且需要先添加 SSH key ，否则无法克隆。

2、https url 在push的时候是需要验证用户名和密码的；而 SSH 在push的时候，是不需要输入用户名的，如果配置SSH key的时候设置了密码，则需要输入密码的，否则直接是不需要输入密码的。

## 如何配置Github的SSH

### Step 1:检查SSH Keys

首先，我们要查看在你电脑上已经存在的SSH keys。运行Git Bash 然后输入：

```
$ ls -al ~/.ssh
```

这两个命令就是检查是否已经存在 id_rsa.pub 或 id_dsa.pub 文件，如果文件已经存在，那么你可以跳过步骤2，直接进入步骤3。

### Step 2:生成SSH key

#### 1. 在Git Bash中输入下面命令，引号内一定是你的Github注册邮箱地址

```
$ ssh-keygen -t rsa -b 4096 -C "your_github_email@example.com" 
#这句作用是生成一个新的SSH key
```

#### 2. 等待几秒，当提示让你输入保存地址时，官方特别推荐放在默认位置就可以了。所以这里直接输入回车，提示如下：

```
Enter file in which to save the key (/Users/you/.ssh/id_rsa):[直接输入回车]
```

#### 3. 将会提示你输入一个密码串(这里输入密码时不会显示在屏幕上的，只要输入正确按回车就好):

```
Enter passphrase （empty for no passphrase）: [输入你想设置的密码]
Enter same passphrase again：[在输入一遍密码]
＃虽然说这里可以设置为空，但是推荐用一个更加安全的密码
```

#### 4. 输完密码之后，你将会得到你的SSH的[print](https://ws1.sinaimg.cn/large/ad3a9ce5gy1foroi8j00ej217s0oojxi.jpg)或者id。

```
Your identification has been saved in /c/Users/you/.ssh/id_rsa.
# Your public key has been saved in /c/Users/you/.ssh/id_rsa.pub.
# The key fingerprint is:
# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
```
当你看到上面这段代码的收，那就说明，你的 SSH key 已经创建成功，你只需要添加到github的SSH key上就可以了。

### Step 3:添加SSH key到ssh-agent

#### 1.输入如下命令

```
$ ssh-agent -s
```
会响应:

```
echo Agent pid [端口号]
```
#### 2.把SSH key添加到ssh-agent

```
$ ssh-add ~/.ssh/id_rsa
```
如果他提示如下，说明不能打开您身份验证的代理

```
Could not open a connection to your authentication agent.
```
只需要输入如下命令即可解决：

```
ssh-agnet bash
```
更多关于ssh-agent的细节，可以用man ssh-agent 来查看

### Step 4:添加SSH key到Github账户

```
$ clip < ~/.ssh/id_ras.pub
```
添加到你的Github账户：

#### 1. 浏览器登陆你的Github账户，点击右上角你的头像，然后点击Settings

[点击Setting](http://ww4.sinaimg.cn/large/8ac969edjw1f1zt9l88fjj20di0ki0u0.jpg)

#### 2.进入Settings，点击侧栏选项SSH key

[点击SSH key](http://ww4.sinaimg.cn/mw690/8ac969edjw1f1zt9lca00j20fs0qsmyx.jpg)

#### 3.单击右边 Add SSH key 按钮

[点击Add key](http://ww3.sinaimg.cn/mw690/8ac969edjw1f1zt9m4k6tj20wm05cta4.jpg)

#### 4. 在下面输入标题（Title，这个可以自定义）和SHH Key（直接 Ctrl＋V 粘贴就可以）

#### 5. 点击下面的Add key按钮便可以添加成功了

### Step 5:测试是否连接成功

#### 1. 在Git Bash中输入：

```
$ ssh -T git@github.com
# ssh尝试连接到GitHub
```

#### 2.你可能看到下面的警告：

```
The authenticity of host 'github.com(207.97.227.239)' can't be established.
RSA key fingerprint is SHA256:nJKJFKDnDLFJDndndnfkdfldjfldldfjld.
Are you sure you want to continue connecting (yes/no)?
```
确定提示信息里的指纹（fingerprint）是否匹配，如果匹配就键入｀yes｀，将得到：

Hi [你的用户名]! You’ve successfully authenticated, but GitHub does not provide shell access.

#### 3. 如果提示信息中你的用户名是你的，那么你就成功建立了SSH key！

### 参考

* [如何配置Github的SSH key](http://mungo.space/2015/10/13/how-to-config-ssh-on-github/index.html)
* [github设置添加SSH](https://www.cnblogs.com/ayseeing/p/3572582.html)
* [设置其他SSH密钥](https://confluence.atlassian.com/bitbucket/set-up-additional-ssh-keys-271943168.html#SetupadditionalSSHkeys-Step2.(Mercurialonly)EnableSSHcompression)
* [使用Github SSH Key以免去Hexo部署时输入密码](https://xuanwo.org/2015/02/07/generate-a-ssh-key/)
* [配置 SSH keys](https://www.jianshu.com/p/05289a4bc8b2)






