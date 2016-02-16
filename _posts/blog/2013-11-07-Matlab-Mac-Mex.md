---
layout: post
title: Mac版matlab的c++编译库设置
description: 做先锋者，总会遇到很多问题
category: blog
---

刚把mac升级到maverick，然后装了个2013a的matlab，结果在用mex编译c++文件的时候，不停的报错：

	xcodebuild: error: SDK "macosx10.7" cannot be located.
	xcrun: error: unable to find utility "clang++", not a developer tool or in PATH

查了半天，也不知道怎么回事，幸得大牛指导，原来升级了os和xcode，现在的sdk已经是macosx10.9了，所以改一下matlab的编译连接库就行了

修改/Applications/MATLAB_R2013a.app/bin/ 下面的mexopts.sh 和 mbuildopts.sh
将所有macosx10.7改为macosx10.9

然后打开matlab，在命令行中输入

mbuild -setup 重安装mbuild
mex -setup 重安装mex


### 后台运行matlab

在服务器上跑个程序跑了一天了，结果早上把电脑电源踢了。。我勒个去，还是要用后台启动的方式啊

文件：
nohup matlab -nojvm -nodisplay -nosplash-nodesktop < matlabscript.m 1>running.log 2>running.err &

函数：
nohup matlab -nojvm -nodisplay -nosplash -nodesktop -r "func(1,2)"  1>running.log 2>running.err &


[LinChaohui]:    http://www.linchaohui.com  "LinChaohui"
