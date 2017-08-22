---
layout: post
date: 2017-07-20
title: System.getProperty()
tags: Java
---

使用Java提供的System.getProperty()方法可以获取JVM以及操作系统的一些参数。

|参数名|描述|
|---|---|
|java.version|Java的运行时环境版本|
|java.vendor|Java 运行时环境供应商|
|java.vendor.url|Java 供应商的 URL|
|java.home|Java 安装目录|
|java.vm.specification.version|Java 虚拟机规范版本|
|java.vm.specification.vendor|Java 虚拟机规范供应商|
|java.vm.specification.name|Java 虚拟机规范名称|
|java.vm.version|Java 虚拟机实现版本|
|java.vm.vendor|Java 虚拟机实现供应商|
|java.vm.name|Java 虚拟机实现名称|
|java.specification.version|Java 运行时环境规范版本|
|java.specification.vendor|Java 运行时环境规范供应商|
|java.specification.name|Java 运行时环境规范名称|
|java.class.version|Java 类格式版本号|
|java.class.path|Java 类路径|
|java.library.path|加载库时搜索的路径列表|
|java.io.tmpdir|默认的临时文件路径|
|java.compiler|要使用的 JIT 编译器的名称|
|java.ext.dirs|一个或多个扩展目录的路径|
|os.name|操作系统的名称|
|os.arch|操作系统的架构|
|os.version|操作系统的版本|
|file.separator|文件分隔符（在 UNIX 系统中是“/”）|
|path.separator|路径分隔符（在 UNIX 系统中是“:”）|
|line.separator|行分隔符（在 UNIX 系统中是“/n”）|
|user.name|用户的账户名称|
|user.home|用户的主目录|
|user.dir|用户的当前工作目录|

示例代码：

```java
System.getProperty("java.vendor.url"); //java官方网站  
System.getProperty("java.vm.nam"); //java虚拟机名称  
System.getProperty("user.country"); //国家或地区  
System.getProperty("user.dir"); //工程的路径  
System.getProperty("java.runtime.version");//java运行环境版本  
System.getProperty("os.arch"); //操作系统位数（32或64）  
System.getProperty("os.name"); //操作系统名称  
System.getProperty("sun.jnu.encoding"); //编码格式  
System.getProperty("os.version"); //操纵系统版本  
System.getProperty("java.version"); //java版本版本
```
