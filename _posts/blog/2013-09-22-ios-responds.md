---
layout: post
title: ios中的Touch消息响应
description: 理解消息传递流程，改写系统调用函数
category: blog
---

##ios中的Touch消息传递流程

先来说说响应者对象（Responder Object），顾名思义，指的是有响应和处理事件能力的对象。响应者链就是由一系列的响应者对象构成的一个层次结构。

UIResponder是所有响应对象的基类，在UIResponder类中定义了处理上述各种事件的接口。我们熟悉的UIApplication、 UIViewController、UIWindow和所有继承自UIView的UIKit类都直接或间接的继承自UIResponder，所以它们的实例都是可以构成响应者链的响应者对象。
####响应者链有以下特点：

1、响应者链通常是由视图（UIView）构成的；

2、一个视图的下一个响应者是它视图控制器（UIViewController）（如果有的话），然后再转给它的父视图（Super View）；

3、视图控制器（如果有的话）的下一个响应者为其管理的视图的父视图；

4、单例的窗口（UIWindow）的内容视图将指向窗口本身作为它的下一个响应者

需要指出的是，Cocoa Touch应用不像Cocoa应用，它只有一个UIWindow对象，因此整个响应者链要简单一点；

5、单例的应用（UIApplication）是一个响应者链的终点，它的下一个响应者指向nil，以结束整个循环。


##改写系统响应函数

看过上面一大段的话，总结无非一句话：消息从父view调用api，确认消息再哪个子view中，然后子view调子子view的，子子孙孙无穷尽也，直到最后那个view没有自己的子view了。

所以其实我们只要改写调用的判断api 和响应api，这样所有的响应方式都可以定制了。
判断api：
	- (UIView*)hitTest:(CGPoint)point withEvent:(UIEvent *)event
	{
	}

翻译一下，默认情况下，view会调用自己的hitTest:WithEvent，如果返回NO就game over，如果返回YES，就会调用自己的subView的hitTest:WithEvent，只要最后一个view 把自己返回，这样就由它处理这个消息了。看到这里,如果明白了的话，改写下这个函数已经能解决很多问题了。

当然，默认情况下，一个view是怎么判断这个消息该不该return YES呢。

	- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event
	{
	}

虽然看函数名是根据这个消息是否在view的bound中发生来决定返回值，但事实上，有一些情况，比如你添加了一个UIView，然后这个UIView的下面有一个Button，虽然UIView在最上层，也会由UIButton响应这个点击，怎么解决这个问题呢，很简单，只要再子UIView中添加
        - (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event
        {
		return YES;
        }
这样tap消息就能被这个UIView处理了
