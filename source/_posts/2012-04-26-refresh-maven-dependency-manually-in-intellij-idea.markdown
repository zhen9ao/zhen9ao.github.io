---
layout: post
title: "IntelliJ IDEA中手动强制更新Maven Dependencies"
date: 2012-04-26 12:01
comments: true
categories: 
---

Intellj IDEA 的自动载入Mave依赖的功能虽然好用，但有时候依赖多了或者缓存出问题，就会导致修改pom文件却没有触发自动重新载入的动作，这个时候就需要手动强制让Intellj IDEA更新依赖了。主要需要以下两步：

1. 手动删除`Project Settings`里面的`Libraries`内容
2. `mvn clean`删除之前编译过的文件
3. `Reimport all maven projects`