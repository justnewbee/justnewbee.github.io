---
layout: post
title: 我恨注册表之 - 文件不能共享
date: 2010-12-15 02:12:18
categories: os
tags: registry
---

领导派了个研究某个软件的任务。倒腾，装了卸，卸了装...突然发现机子有些地方就有问题了，最显著的一点就是 [DUMeter](http://www.hageltech.com/dumeter/about) 不能运行了。但是为了完成任务，不得不继续。最后在机子上共享一个目录，悲剧地发现文件属性没有「共享」这个标签页了居然！

网上说这个问题很有可能是 workstation 和 server 这两个服务都没有运行。果然，运行 `services.msc` 后发现这两个东西停着。尝试运行，发现老是报错「Error 1068 The dependency service or group failed to start」。能不能更详细点...我很菜的，究竟哪些 dependency 呢...

想通过系统还原来解决问题，悲剧地发现被禁用了，没办法只能硬着头皮找解决办法。

最后很愉快的发现每条 service 双击可以打开属性，而属性里有个 `Dependencies` 标签：

![](/images/posts/workstation_prop_deps.png)

里面有一项，从名字上看肯定是刚刚被删除的软件。于是在注册表里搜这个东西，由于操作不当删除之后，整个 Dependency 都没了...好在有同事是相同的 OS，从他那边搞了一份过来，导进去发现出了「Error 2」。瞎倒腾一番之后，错误变成 「1075 the dependency service does not exist or has been marked for deletion」。

从注册表和 service 上看，所有的 Dependency 已经是正常了，难道还有余孽？于是又搜了一把注册表，发现在 `HKEY_LOCAL_MACHINE/SYSTEM/ControlSet002/Control/Session Manager/AppCompatCache` 下还有，不过是二进制的，从这个名字上看应该可以删除，毕竟 `Cache` == 缓存。只能死马当活马医了，大不了重装系统... 删！

奇迹出现了，重启之后，发现这两个 service 起来了，Oh Yeah！

## 总结

最后结合非专业的注册表知识和非正常思维方式，来总结一下吧，以免以后再出现类似的问题。

一，运行 `services.msc` 检查 `serer` 和 `workstation` 是否运行着

二，如果没有运行且运行出错「1068」错误，双击查看「Dependency」正常如下

**Workstation**

![](/images/posts/service_workstation_prop_dependencies.png)

**Server**

![](/images/posts/service_server_prop_dependencies.png)

对应到注册表，以下项的「DependOnService」键：

```
HKEY_LOCAL_MACHINE/SYSTEM/ControlSet001/services/LanmanWorkstation
HKEY_LOCAL_MACHINE/SYSTEM/ControlSet001/services/LanmanServer
HKEY_LOCAL_MACHINE/SYSTEM/ControlSet002/services/LanmanWorkstation
HKEY_LOCAL_MACHINE/SYSTEM/ControlSet002/services/LanmanServer
HKEY_LOCAL_MACHINE/SYSTEM/CurrentControlSet/services/LanmanWorkstation
HKEY_LOCAL_MACHINE/SYSTEM/CurrentControlSet/services/LanmanServer
```
我想不明白同样的东西为什么要有这么多份...结合很多程序员的通病，我觉得这很有可能是程序垃圾。

在注册表里面改「Dependency」似乎很简单（其实我早该这么做的）：

![](/images/posts/reg_service_pendendency.png)

把不正常的 Dependency 删掉。

注意，这里对应的值在 `HKEY_LOCAL_MACHINE/SYSTEM/ControlSet001/services` 下应该都可以找到。

三，如果运行出错「1075」那应该就是缓存在捣乱了，把整个 `HKEY_LOCAL_MACHINE/SYSTEM/ControlSet002/ControlSession Manager/AppCompatCache` 删掉试试

最后，Service 的 Dependency 有多层，所以有些时候可能逐层要往下看