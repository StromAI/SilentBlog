title: Ubuntu下Sublime Text 2的安装
date: 2013-10-30 13:02:27
tags: [Linux,Sublime]
categories: Linux
---
作为一个程序员，不会用Linux是不应该的，所以昨天搞了一个Ubuntu 12.04装在了虚拟机上，这也算是我Linux学习的开始吧。

##Sublime Text 2的安装

Sublime Text 是一个代码编辑器（Sublime Text 2是收费软件，但可以无限期试用），也是HTML和散文先进的文本编辑器。Sublime Text是由程序员Jon Skinner于2008年1月份所开发出来，它最初被设计为一个具有丰富扩展功能的Vim。

Linux下的安装流程：
###0.环境说明
	操作系统：Ubuntu 12.04 x32
	Sublime版本：2.0.2
###1.下载sublime
百度或者google可以搜索到[Sublime Text的官网][1]在Download中找到合适自己操作系统的版本进行下载，这里我下载的是**Linux 32 bit**。
下载下来以后是.bz2文件。10M多一点，不是很大。
	
###2.解压放置

网上查到的方法是这样的：

* 找到目录后键入：
```
tar -xvf Sublime\ Text\ 2.0.1.tar.bz2
mv Sublime\ Text\ 2 /usr/lib/
```
第一行解压，第二行为移动。
其中的\为转义符
这样做是因为$PATH这个环境变量自动涵盖了/usr/lib这个目录，不用专门去修改环境变量。

* 然后键入：
```
ln -s /usr/lib/Sublime\ Text\ 2/sublime_text /usr/bin/sublime
```
这行命令是在/usr/bin/目录下建立一个名为sublime链接，这样后面可以比较方便的用命令行启动这个编辑器。其中sublime这个名字是自行定义的，用户可以定义的更加简单方便。

但是在执行tar命令时总是有问题，于是换了种方法，如下：

用系统自带的Firefox下载完成后会左边栏会显示Archive Manager，可以看到下载好的**Sublime Text 2**,尝试后发现直接把这东西从Archive Manager拖到桌面上貌似会自动解压。
然后把路径切到Desktop执行mv 命令，后面的步骤与之前的相同。

这样进入命令行输入Sublime即可运行。
打开以后是这个样子的：
![image](http://d.pr/i/s1jE.jpg)

###3.侧边条LaunchBar设置
这个到现在没有搞成功……
不知道具体是啥问题，留着以后解决。
下面是我找到的[链接][2]，想了解的可以看看,在此感谢这篇文章的作者。
	
[1]:http://www.sublimetext.com/
[2]:http://www.linuxidc.com/Linux/2013-01/77623.htm