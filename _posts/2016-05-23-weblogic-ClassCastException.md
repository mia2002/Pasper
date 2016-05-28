---
layout: post
date: 2016-05-23
title: Weblogic ClassCastException
tags: Oracle
---

今天手贱不小心用了超级管理员帐号去开启weblogic，导致后面用回IDE上配置的weblogic server启动失败。失败报错信息如下：

```java
java.lang.ClassCastException: com.octetstring.vde.backend.BackendRoot cannot be cast to com.octetstring.vde.backend.standard.BackendStandard
	at weblogic.ldap.EmbeddedLDAP.start(EmbeddedLDAP.java:323)
	at weblogic.server.AbstractServerService.postConstruct(AbstractServerService.java:76)
	at sun.reflect.GeneratedMethodAccessor2.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.glassfish.hk2.utilities.reflection.ReflectionHelper.invoke(ReflectionHelper.java:1262)
	at org.jvnet.hk2.internal.ClazzCreator.postConstructMe(ClazzCreator.java:332)
	at org.jvnet.hk2.internal.ClazzCreator.create(ClazzCreator.java:374)
	at org.jvnet.hk2.internal.SystemDescriptor.create(SystemDescriptor.java:471)
	at org.glassfish.hk2.runlevel.internal.AsyncRunLevelContext.findOrCreate(AsyncRunLevelContext.java:228)
	at org.glassfish.hk2.runlevel.RunLevelContext.findOrCreate(RunLevelContext.java:85)
	at org.jvnet.hk2.internal.Utilities.createService(Utilities.java:2072)
	at org.jvnet.hk2.internal.ServiceHandleImpl.getService(ServiceHandleImpl.java:114)
	at org.jvnet.hk2.internal.ServiceLocatorImpl.getService(ServiceLocatorImpl.java:698)
	at org.jvnet.hk2.internal.ThreeThirtyResolver.resolve(ThreeThirtyResolver.java:78)
	at org.jvnet.hk2.internal.ClazzCreator.resolve(ClazzCreator.java:211)
	at org.jvnet.hk2.internal.ClazzCreator.resolveAllDependencies(ClazzCreator.java:234)
	... ... 
```

一开始出现这错误时都不知道是啥回事，百度了一下，网友们说是因为有人用sudo去启动过weblogic，导致weblogic的LDAP文件的权限属性为root:root，所以使用普通用户在启动过程中会报LDAP异常，百度的结果大多数都是说使用chown命令来更改文件所有者，解决这个问题的方法有两种，一种是别人博客上的解决方案，另外一种是oracle官网上看到的解决方案，第二种会简单点。先说第一种解决方案吧，用命令行，看起来高大上一丁点。
## 解决方案一

命令：

```
#chown -R weblogic:weblogic <mydomain_dir> 
```
说明：前一个weblogic是指用户，后一个weblogic是指用户所属的组，<mydomain_dir>是weblogic安装所在的目录，必须使用sudo执行命令。

如果你不记得用户和用户组，可以使用如下命令查看用户和用户所属组：

```
ls -l <oracle_home_dir>
```

说明：oracle\_home\_dir是weblogic安装所在的目录，如果你在安装weblogic时使用oracle默认创建的文件名的话，一般都是.../Middleware/Oracle_Home，接下来你就可以看到你的用户名和用户组了。比如：

```
drwxrwx---  22 root  root   748 11 14  2015 OPatch
```
第一个root是用户名，第二个root是用户所在的组。

接下来再用sudo来执行chown命令，如下：

```
sudo chown -R root:root /Users/yan/Oracle/Middleware/Oracle_Home
```

执行成功后，此时在IDE启动weblogic即可，我这边用的IDE是IntelliJ，虽然能成功启动weblogic，但是打开不了项目的页面，重启IntelliJ，就正常了。

好了，接下来说说第二种解决方案。

## 解决方案二

找到weblogic当前使用的domain，如我这边weblogic当前使用的domain是base_domain，进入目录/base_domain/servers/AdminServer/data，删除ldap整个文件夹，这里你最好备份一下data这个文件，至少删错了，还可以重来。

删除ldap文件夹后再使用IDE打开weblogic，就这么简单粗暴的解决问题了，那么此时你就可以把刚刚备份的data文件夹彻底扔到垃圾桶里了。

如果在启动过程中有报类似说ldap文件丢失的异常，没关系，他会自动创建这个文件夹，如果你有强迫症（像我一样）不喜欢看到控制台报异常的话，也没关系，你重启一下weblogic就没有了。

## 解决方案三

如果上述两种解决方案未能解决您的问题的话，可以考虑把整个domain都删掉，重建。







