---
layout: post
title: "今天都折腾了些什么"
date: 2012-10-11 00:24
comments: true
published: true
categories: Daily
---

##zsh

在看Ruby的时候，发现有人在提zsh，于是打算再次尝试。之前装过，但是重装系统后觉得麻烦就没再装了，今天看到一个不错的项目[`oh-my-zshell`](https://github.com/robbyrussell/oh-my-zsh)。它是一个帮助我们安装、管理和配置zsh的工具，使用起来也很简单。按主页上的说明自动安装即可：

> curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh

安装好了以后会在`~`目录创建一个`.zshrc`的zsh配置文件。其中前面的部分是`oh-my-zsh`的配置部分，可以对`配色`、`插件`进行配置。zsh的一个优势是和bash兼容，能执行`.bash_profile`或`.bashrc`中的配置，所以把bash的配置拷到zshrc的最后即可完成对已有配置的修改。

<!--more-->

###插件

zsh的插件看上去比较好用，我首先看到的是git插件，就是当切换到git目录的时候，提示符后面会显示当前所在的branch名。

除此之外还有很多插件，都可以进行配置使用。

###配色

自带了很多配色方案，可以在`~/.oh-my-zsh/theme`里看到支持的各种配色方案，在zshrc修改配色方案后就可以看到效果。这个比bash的方便些。

###自动补全

这可能是zsh相对于bash的一大优势，不仅会补全命令，还能补全命令的参数，补全的时候是大小写`不敏感`的。还有就是`cd`切换目录的时候，可以使用简略写法，比如打算切换到`~/Documents/github/goagent`目录中，在zsh里就可以写成`do/gi/go`，然后按一下`tab`就会自动补全所有路径了。

##关于Ruby

发现了一个很好的社区[Ruby-China](http://ruby-china.org/)，里面有很多牛人，社区的wiki也写的很详细，很适合新手入门。浏览了一下最活跃和最受欢迎的帖子，看看大家的讨论，觉得ruby社区氛围很好，个人很喜欢这种开放的精神，这可能和Ruby本身就源于开放有关。

之前一直觉得配置Ruby和Rails很麻烦，今天也发现了一个比较好的项目：[`Laptop`](https://github.com/thoughtbot/laptop)。它的作用就是输入一个命令，就可以把你的电脑变成一台开发用机，配置好Ruby和Rails的环境：

> zsh < <(curl -s https://raw.github.com/thoughtbot/laptop/master/mac)

##iOS

今天再次搜索了一下ARC相关的问题和讨论，之前一直认为在iOS4中缺少了对weak的支持，ARC就几乎没什么用了。其实这种想法不对，ARC主要的任务就是减少代码量，降低代码出错的概率，就算没有对`weak`的原生支持，`strong`还是可以起到很大的优化作用。另外，目前发现有两种方法可以实现对iOS4的`weak`代码兼容：

1. 通过一系列宏定义，根据当前运行系统对ARC的支持情况，将`weak`声明成`weak`、`unsafe_unretained`或者`assign`。

2. 有一个第三方库可以实现对`weak`的完美支持：[PLWeakCompatibility](https://github.com/plausiblelabs/PLWeakCompatibility)，当然可以使用`CocoaPods`对项目进行配置。

##StackOverflow

今天发现我在StackOverflow上的声望值又上涨了，现在是121，又有人顶了一下我的问题回复，感觉很有成就感，还是之前研究Android的时候的一些成果分享呢。希望iOS什么时候也能去回答别人的一些问题，由此获得一些声望 😄。