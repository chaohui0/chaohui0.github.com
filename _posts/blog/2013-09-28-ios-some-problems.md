---
layout: post
title: ios路上的一些坑
description: 坑这个东西嘛，有些是本来就有的，有些是自己挖的
category: blog
---

## 学会总结

写IOS也快一年了，上了一个app，更新了N个版本，马上要上另一个app。怎么滴也算IOS初级开发者了，从一开始的一定要把NSString转成string来printf，到完全适应NS*的各种类，学习更是一个习惯的过程。将碰到的一些坑记录一下，要是有缘得见，也可以少掉几次。

## 罗里吧嗦

#### 1.关于设计

设计和开发的配合很重要，一个好的设计能帮开发省去很多事情，当然一个坑爹的也会然你痛不欲生。如果和你配合的设计喜欢使用有一定透明度的按钮，请告诉it，这绝对是个坑爹的想法。

#### 2.各种icon

发了应用后收到苹果的邮件，原来ipad需要76*76 和 152*152的，iphone72*72搞定，如果想展示在itunes上。请使用512*512的命名为iTunesArtwork的文件。注意这里没有png的后缀！

#### 3.页面跳转闪退

在ios中的页面跳转时，最好将跳转目标页面的对象全局化，不然有一定的概率会出现段错（错误代码如下）。

	-(void)jumpToDrawView
	{
    		drawViewController *drawView = [[drawViewController alloc] init];
    		[self presentModalViewController:drawView animated:YES];
	}

#### 4.架构设计

关于架构设计的一点总结：模块化，服务化，可重用化。不管是app的实现或者后台服务的实现，都应该遵守这几个原则。模块化指将基础的功能模块独立起来，比如把动画处理整成一个类，网络通信模块也独立成一个类，这样后面在做同样功能的开发才会很省力（做了两个app就发现，开发大多数时候都是在重复劳动，少给自己挖坑），服务化指后台提供的应该是一个标准协议的服务，比如http，比如使用json。使用自定义协议，那就是给自己挖了一个很深的坑；服务化后台实现上的体现则是，多个服务最好使用网络通信实现，比如两个模块有交互，采用http的方式比共享内存神马的靠谱很多，不然后期扩展的时候就等着跳坑里吧。可重用化是在模块化和服务化的基础上的一个目标，程序猿嘛，能偷懒就偷懒，苦逼兮兮的重复体力劳动不如腾出点时间多学点东西。


#### 5.静态库的那点事

在xcode中创建一个静态库文件，编译后会生成两个版本，一个是模拟器版，一个是真机版。这样对后面引入静态库来开发非常不方便。因此非常需要打包成一个通用静态库方便调试。
a.学习一个查看静态库文件信息的命令
lipo -info xxxxxxxxxx.a 
显示结果中i386是mac上的架构（模拟器） armv6/armv7是ios架构的（真机）；

b.打包命令
lipo -create "完整路径/lib.a" "完整路径/lib.a" -output "输出路径/lib.a"   
执行成功后，可使用查看命令查看。

#### 6.相册拍照的那些坑

a.UIView的contentMode属性可以调整展示图的显示比例。默认为 UIViewContentModeScaleToFill 也就是缩放变形的效果，推荐使用UIViewContentModeScaleAspectFit

b.相机旋转的问题，这个问题需要在orient调整函数中调用    
	if(UIDeviceOrientationIsLandscape([[UIDevice currentDevice]orientation])) 
	{
        
        	self.imagePickerController.cameraViewTransform = CGAffineTransformMakeRotation(M_PI/2);
    	}


#### 7.编译的一些问题

这个其实应该跟第五个坑写在一块，通常编译的静态库用c++实现的，如果还用了opencv，那编译的时候非常有可能会出现[ “list” file not found ] 这么一个奇葩的错误，这是由于在Xcode中，通常，Objective-c的后缀名位 .h/.m，C语言的后缀名为 .h/.c， C++的后缀名为.h/.cpp, 当一个文件中既有objective-c又有C++代码时，后缀名为 .h/.mm。在编写代码时要写对后缀名。其次，你可能发现，后缀名都写对了，代码也没有任何问题，编辑器也没有报错，为什么编译的时候就报错了呢？事实上，编译器和编辑器的工作是区分开来的，编辑器就是你写代码的地方，仅检查代码语法是否有错误，你语法没有错误当然不会报错了。编译器就是要编译运行在编辑器中编写好的代码，如果编辑器仅支持Objective-C，它怎么可能编译的了C++代码呢？

两个解决步骤:

a.将引用了静态库的源文件后缀改为.mm。通常情况下问题基本解决，如果还是不行。。

b.照下图找到选项“Compile Sources As”，意思是要把工程按照哪一种语言进行编译，默认是第一个“According to File Type”，将其改成Objective-C++即可。
![TARGETS->Build Settings->Apple LLVM compiler - Language](/images/oc_compile.png)
