---
layout: post
title: "配置Powerline"
date: 2013-08-08 13:56
comments: true
categories: MacOS Terminal Powerline
---

更新了iTerm2后，Powerline的字体出现了问题，重新到Powerline的网站上去看，才发现已经有了新的版本和安装方法。这里只介绍Mac OS下的安装和配置方法

结果写完发现之前写过一篇Powerline了，这篇算是进化版吧。。。

### 与之前版本的安装区别

* 之前的`vim-powerline`插件弃用了，现在改为一个Python脚本
* zsh的Powerline主题更新
* 字体文件发生了变化，之前的字体文件不能用，官方推出了几个path过的字体，当然也可以自己path喜欢的字体

<!--more-->

### 安装、配置Powerline

首先`poerline`依赖的是`Python`， 官方文档说是要用Mac自带的Python安装，但几次实验发现不能正常使用，所以测试用`Homebrew`安装，结果成功了。

`Homebrew`安装`python`会自动安装`pip`包管理工具，

	brew install python

安装好后用`pip`安装powerline，但用`Homebrew`安装的`Python`不能使用`--user`的参数安装包([参考](https://github.com/mxcl/homebrew/wiki/Homebrew-and-Python))，所以使用下面的命令即可：

	pip install https://github.com/Lokaltog/powerline/tarball/develop
	
装好以后默认的默认安装路径是 `/usr/local/lib/python2.7/site-packages`
	
最后还要在`vimrc`中加入一下配置：

	set rtp+=/usr/local/lib/python2.7/site-packages/powerline/bindings/vim
	
其中`/usr/local/lib/python2.7/site-packages/`目录是python的插件安装目录，如果当前使用的是系统python，则安装目录可能不同

如果还打算在`zsh`中使用`powerline`主题的话，推荐使用`oh-my-zsh-powerline-theme`，可以作为一个submodule放在dotfiles中，地址在：

	https://github.com/jeremyFreeAgent/oh-my-zsh-powerline-theme
	
然后执行一下其中的
	
	install_in_omz.sh
	
就会自动将`powerline.zsh-theme`放在指定的位置了
	
然后在`.zshrc`中配置`ZSH_THEME`参数为`powerline`，即可使用oh-my-zsh的powerline-theme了。

这时可能会有一些符号无法正常显示，现在需要做的就是给字体打补丁（也可以用powerline提供的已经打过补丁的[字体](https://github.com/Lokaltog/powerline-fonts)）

不过LZ还是喜欢用Menlo字体，所以需要自己给自己打补丁，那就开始吧

首先，将系统中的Menlo字体导出到`~/Desktop`

其次，安装`fontforge`

	brew install fontforge
	
然后，克隆`powerline`的代码

	git clone https://github.com/Lokaltog/powerline-fonts.git
	
最后使用代码库中的pather，`font/fontpather.py`

	fontforge -script /path/to/powerline/font/fontpather.py /path/to/your/font.ttf

将生成的后缀为`for powerline`的字体导入到字体库中，设置iTerm的字体，这样就可以正确显示符号了