---
layout: post
title: "安装配置Powerline"
date: 2013-06-26 16:32
comments: true
categories: 
---

近期查看vim-powerline的Github库发现该库已经不赞成使用了，作者转向了另一个powerline的库: [这里](https://github.com/Lokaltog/powerline)。所以最好是现在就重新配置吧！

## 安装

首先`poerline`依赖的是`Python`， 官方文档说是要用Mac自带的Python安装，但几次实验发现不能正常使用，所以测试用`Homebrew`安装，结果成功了。

`Homebrew`安装`python`会自动安装`pip`包管理工具，

	brew install python

安装好后用`pip`安装powerline，但用`Homebrew`安装的`Python`不能使用`--user`的参数安装包([参考](https://github.com/mxcl/homebrew/wiki/Homebrew-and-Python))，所以使用下面的命令即可：

	pip install https://github.com/Lokaltog/powerline/tarball/develop
	
装好以后默认的默认安装路径是 `/usr/local/lib/python2.7/site-packages`

## 配置

### `zshrc`

在`.zshrc`最后加上下面一行即可

	. /usr/local/lib/python2.7/site-packages/powerline/bindings/zsh/powerline.zsh
	
### `vimrc`

之前使用的`vim-powerline`插件可以禁用掉了，然后在`.vimrc`的最后添加如下一行即可

	set rtp+=/usr/local/lib/python2.7/site-packages/powerline/bindings/vim
	
## 字体

之前使用的`Monaco for Powerline`貌似不好使了，特殊符号无法显示，且`powerline`开发者也更新了字库文件，[这里](https://github.com/Lokaltog/powerline-fonts)

选择一款喜欢的字体下载并在`iTerm2`中设置使用的字体即可。

### gvimrc

如果还使用了`MacVim`，还需要配置一下`.gvimrc`文件，修改一下默认的字体，如：

	set guifont=Menlo\ for\ Powerline:h16
	
这样就修改完成了！

--EOF--