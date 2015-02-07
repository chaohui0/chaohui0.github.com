---
layout: post
title: ios中UINavigationController的使用
description: 理解、使用、传播
category: blog
---

##一些误解

今天封装了下淘宝的开放平台sdk，结果这货偷懒的紧，就一个view，人新浪的sdk注册页面大小都固定的死死的。所以考虑了下自己做个UINavigationController来修饰下淘宝的注册页面，平常自己写nib，页面跳转逻辑，一直觉得UINavigationController是个鸡肋，等实际要用的时候才发现，原来我还是too young too simple啊。

##UINavigationController不只是一个控制跳转的控制器

UINavigationController其实很常用，之前做过一个“应用猎手”的app，里面的应用内购买就是基于此的，只不过这是系统实现的，所以还没有真正的理解，怎么才能实现那样的自己构造一个UINavigationController并自定义title，左右键呢？好吧，废话不多，讲下实现流程。

###创建并使用一个UINavigationController

	UINavigationController *aNav = [[UINavigationController alloc] init];

然后添加一个自定义的视图进去

	UIViewController *aView = [[UIView alloc] initWithNibName: (*xib文件名*)];
	[aNav pushViewController:aView animated:NO];

###设置导航栏的左右按钮和标题

这里其实有个坑：设置导航栏的按钮并不是去设置导航栏本身，而是当时被导航的视图控制器，比如我们对aView作设置，so

设置其标题：
	aView.title = @"标题";

设置按钮：
	UIBarButtonItem *callModalViewButton = [[UIBarButtonItem alloc]
							initWithTitle:@"点我"
							style:UIBarButtonItemStyleBordered
							target:self
							action:@selector(callModalList)];
	aView.navigationItem.leftBarButtonItem = callModalViewButton;
	[callModalViewButton release];

其他常用方法和属性：
	本地视图.navigationItem.leftBarButtonItem //左边栏项目
	本地视图.navigationItem.rightBarButtonItem //右边栏项目
	本地视图.navigationItem.backBarButtonItem //后退栏项目
	本地视图.navigationItem.hidesBackButton //隐藏后退按钮（YES or NO）

that's all
