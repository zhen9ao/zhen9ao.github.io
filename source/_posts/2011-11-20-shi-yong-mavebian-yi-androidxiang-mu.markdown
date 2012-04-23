---
layout: post
title: "使用mave编译Android项目"
date: 2011-11-20 17:10
comments: true
categories: 
---
## 为何打算使用Maven

1. maven可以让目录更加简洁和清晰

2. 利用maven，不用手动下载依赖的各种jar文件，可以减小源码大小，所需的依赖库都又maven自动处理。

3. 主要使用了开源的maven-android-plugin（3.0版本改名为[android-maven-plugin](http://code.google.com/p/maven-android-plugin/wiki/PluginRenamed)）插件


本文主要介绍在Mac OS X下使用maven进行android编译需要的做的工作。

<!--more-->

## 开始折腾

1. 安装maven

我使用的是Homebrew，执行以下命令：

{% codeblock %}
brew install maven
{% endcodeblock %}

2. 配置系统环境

主要需要配置的是ANDROID_HOME这个环境变量，在我这里配置如下：

{% codeblock %}
export ANDROID_HOME=/Users/zheng/Documents/Android/android-sdk-mac_x86
{% endcodeblock %}

可以将环境变量写到.bash_profile（如果使用bash）或者.zshrc（如果使用zsh）中。

3. 下载例子程序

在官方的[wiki页面](http://code.google.com/p/maven-android-plugin/wiki/Samples)可以下载例子程序，我选择了3.0.0-alpha版的，看了一下，差别不是特别大，主要是集成了一些新的命令。

4. 编辑例子程序

主要需要编辑的是pom.xml文件，也是maven的核心文件。

本文件展示的是修改后的helloflashlight文件夹下的pom.xml文件

{% codeblock pom.xml lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.simpligility.android</groupId>
    <artifactId>helloflashlight</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>apk</packaging>
    <name>HelloFlashlight</name>

    <dependencies>
        <dependency>
            <groupId>com.google.android</groupId>
            <artifactId>android</artifactId>
            <version>2.3.3</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>xom</groupId>
            <artifactId>xom</artifactId>
            <version>1.2.5</version>
        </dependency>
        <dependency>
            <groupId>xom</groupId>
            <artifactId>xom</artifactId>
            <version>1.2.5</version>
            <exclusions>
                <exclusion>
                    <groupId>xml-apis</groupId>
                    <artifactId>xml-apis</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>xerces</groupId>
                    <artifactId>xercesImpl</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>xalan</groupId>
                    <artifactId>xalan</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.caacsri.mobile</groupId>
            <artifactId>mobile-client</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>${project.artifactId}</finalName>
        <sourceDirectory>src</sourceDirectory>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                    <artifactId>android-maven-plugin</artifactId>
                    <version>3.0.0-alpha-14</version>
                    <extensions>true</extensions>
                </plugin>
            </plugins>
        </pluginManagement>
        <plugins>
            <plugin>
                <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                <artifactId>android-maven-plugin</artifactId>
                <configuration>
                    <sdk>
                        <path>/Users/zheng/Documents/Android/android-sdk-mac_x86</path>
                        <platform>10</platform>
                    </sdk>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
{% endcodeblock %}

需要注意的主要是以下几个部分：

{% codeblock lang:xml %}
<dependency>
  <groupId>com.google.android</groupId>
  <artifactId>android</artifactId>
  <version>2.3.3</version>
  <scope>provided</scope>
</dependency>
{% endcodeblock %}

这里需要将scope设置为provided

{% codeblock lang:xml %}
<plugin>
  <groupId>com.jayway.maven.plugins.android.generation2</groupId>
  <artifactId>android-maven-plugin</artifactId>
  <configuration>
     <sdk>
     <path>/Users/zheng/Documents/Android/android-sdk-mac_x86</path>
     <platform>10</platform>
     </sdk>
  </configuration>
</plugin>
{% endcodeblock %}

这里需要设置path，指向sdk的安装目录。"10"是对应的"android2.3.3"的版本号

{% codeblock lang:xml %}
<dependency>
  <groupId>xom</groupId>
  <artifactId>xom</artifactId>
  <version>1.2.5</version>
  <exclusions>
    <exclusion>
      <groupId>xml-apis</groupId>
      <artifactId>xml-apis</artifactId>
    </exclusion>
    <exclusion>
      <groupId>xerces</groupId>
      <artifactId>xercesImpl</artifactId>
    </exclusion>
    <exclusion>
      <groupId>xalan</groupId>
      <artifactId>xalan</artifactId>
    </exclusion>
  </exclusions>
</dependency>
{% endcodeblock %}

使用了第三方的依赖，这里添加了exclusions，因为xom的依赖可能较老或者在较老的jvm虚拟机上编译的原因，所以如果不添加exclusion，可能在在转换成dex文件的时候会报错

{% codeblock %}
[INFO] Ill-advised or mistaken usage of a core class (java.* or javax.*)
[INFO] when not building a core library.
[INFO] 
[INFO] This is often due to inadvertently including a core library file
[INFO] in your application's project, when using an IDE (such as
[INFO] Eclipse). If you are sure you're not intentionally defining a
[INFO] core class, then this is the most likely explanation of what's
[INFO] going on.
[INFO] 
[INFO] However, you might actually be trying to define a class in a core
[INFO] namespace, the source of which you may have taken, for example,
[INFO] from a non-Android virtual machine project. This will most
[INFO] assuredly not work. At a minimum, it jeopardizes the
[INFO] compatibility of your app with future versions of the platform.
[INFO] It is also often of questionable legality.
[INFO] 
[INFO] If you really intend to build a core library -- which is only
[INFO] appropriate as part of creating a full virtual machine
[INFO] distribution, as opposed to compiling an application -- then use
[INFO] the "--core-library" option to suppress this error message.
[INFO] 
[INFO] If you go ahead and use "--core-library" but are in fact
[INFO] building an application, then be forewarned that your application
[INFO] will still fail to build or run, at some point. Please be
[INFO] prepared for angry customers who find, for example, that your
[INFO] application ceases to function once they upgrade their operating
[INFO] system. You will be to blame for this problem.
[INFO] 
[INFO] If you are legitimately using some code that happens to be in a
[INFO] core package, then the easiest safe alternative you have is to
[INFO] repackage that code. That is, move the classes in question into
[INFO] your own package namespace. This means that they will never be in
[INFO] conflict with core system classes. JarJar is a tool that may help
[INFO] you in this endeavor. If you find that you cannot do this, then
[INFO] that is an indication that the path you are on will ultimately
[INFO] lead to pain, suffering, grief, and lamentation.
{% endcodeblock %}

提示中说明可以通过添加"--core-library"参数解决这个问题，具体在pom文件中就是在configuration部分加入如下代码：

{% codeblock lang:xml %}
<dex>
  <jvmArguments>
    <jvmArgument>-Xms256m</jvmArgument>
    <jvmArgument>-Xmx512m</jvmArgument>
  </jvmArguments> 
  <coreLibrary>true|false</coreLibrary>
  <noLocals>true|false</noLocals>
  <optimize>true|false</optimize>
</dex>
{% endcodeblock %}

这也是除了添加exclusion以外的另一种解决方案，但是经我的测试，这种方案会导致很长时间的build和一大堆提示信息，说明某个类不能被编译什么的，但是最后还是能通过编译，不过我不喜欢用这种方式，还是添加exclusions吧，那种方式的输出结果比较简洁。

5. 编译及运行

切换到目录下，编译代码使用命令：

{% codeblock %}
mvn clean install
{% endcodeblock %}

部署和运行代码使用以下命令：

{% codeblock %}
mvn android:deploy android:run
{% endcodeblock %}

6. 如何创建新项目

最后，说一下如何直接创建新项目。按着[官方wiki](https://github.com/akquinet/android-archetypes/wiki)里的说法，有三种创建方式"quickstart"，"with-test"和"release"三种方式，一般使用quickstart即可，其他的配置可以自己添加。

quickstart的创建方式如下：

{% codeblock %}
mvn archetype:generate \
  -DarchetypeArtifactId=android-quickstart \
  -DarchetypeGroupId=de.akquinet.android.archetypes \
  -DarchetypeVersion=1.0.5 \
  -DgroupId=your.company \
  -DartifactId=my-android-application
{% endcodeblock %}

默认情况下package名和groupId相同，可以使用'-Dpackage=...'的参数创建自定义的包名。创建的项目默认使用android2.2，可以通过'-Dplatform=...'设置使用的android sdk。
