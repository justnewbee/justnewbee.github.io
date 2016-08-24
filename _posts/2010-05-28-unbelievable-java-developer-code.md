---
layout: post
title: 一个令人无语的 Java 类 一种令人发指的编程方式
date: 2010-05-28 08:05:41
categories: code
tags: shit
---

从 2009 年 9 月份开始，在 Artemis 一直被编排在前端做开发。目前为止，对项目的前端代码，已经有了 60% 的满意度，但后端的 Java... 恕我直言，只能用「恶心」二字来形容了。

平时实在不敢往后端看，因为只要一看，就有抑制不住的冲动想重写。

今天，由于 Email 的 bug，不得不往后端看。看到了一个令人恶心到发指的 Java 类 - `SendEmailHelper.java`

![本应该不可见的成员属性](/images/posts/java_members_should_have_been_private.png)

首先是对属性的可见性完全不理解，很明显这些属性只会在本类里面用到；当然，如果你眼够尖的话，应该能看到那个多余的 `TAB`。这其实还好，不算什么，毕竟我也是接触 Java 后，第二个礼拜才对 `private` 和 `public` 有认识的。

最为令人不解的是参数个数，简直令人抓狂：多数在 8 个以上，最多的可以达到 10 个以上... 使用这样的借口的人心得多大啊。

![一个方法如此多的参数](/images/posts/java_method_with_10_args.png)

图太小 好吧 我贴一下这段代码

```java
public static void sendEmailForFileShare(ArtemisUser srcUser, // 1
	List recipentListArtemisUser, // 2
	List recipentListNonArtemisUser, // 3
	List fileInfoList, // 4
	String signUpURL, // 5
	String filesTabURL, // 6
	String artemisImagePath, // 7
	String serverPath, // 8
	String customizedMessage, // 9
	String accountId) throws WbxException { // 10
}
```

我知道 Java 里面传复合参数不似 JS 那么 easy，但仔细看看这代码，那些 url 和 path 其实是可以从系统的某个地方拿到的。