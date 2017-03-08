---
layout: post
date: 2016-12-05
title: How to install JDK
tags: Java
---

# Ubuntu

## 1.官网下载JDK

*地址：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html*

![](../assets/blog/oracle-download.png)

这里我下载的是jdk8，注意勾选上Accept License Agreement。

## 2.解压缩

![](../assets/blog/extract-jdk.png)

## 3.拷贝JDK

把文件拷贝到~/opt文件夹下。


先进去这个路径中，右键点击Open in Terminal。

![](../assets/blog/copy-jdk-1.png)

将解压缩后的jdk拷贝到~/opt路径下

```
sudo cp -r '/home/yan/Desktop/jdk1.8.0_111' ./
```

![](../assets/blog/copy-jdk-2.png)

## 4.配置环境变量

键入命令：

```
sudo nano /etc/profile
```

在最后一行添加以下内容：

```
export JAVA_HOME=/opt/jdk1.8.0_111
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```

![](../assets/blog/set-jdk.png)

ctrl+x退出编辑，输入y确认保存，再回车。

重新执行配置文件，键入命令：

```
source /etc/profile
```

最后，验证jdk是否已安装成功，键入以下命令：

```
java -version
```

![](../assets/blog/complete-jdk.png)

