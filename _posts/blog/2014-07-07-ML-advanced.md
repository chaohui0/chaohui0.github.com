---
layout: post
title: Machine Learning学习笔记（高阶） 
description: 对上个入门系列的一些补充
category: blog
---
###照例是一点废话
来鹅厂快两个月了，blog也一直没更新，其实是一直想写点神马的，无奈做的东西还是比较琐碎的，其实就是没啥好写，一句概括就是js、django、SSTable、mapreduce、recordio。大概明白了把，就是数据处理+网页demo的活！欲做厨师，必先打下手，人也不是一来就学做菜的是吧，准备食材、洗菜、切菜也是很重要的嘛。下面就随便写一些看法，就当对上上上篇学习笔记的一个补充。

###局部加权回归

在使用线性回归时，会碰到选择多项式特征的问题，如何去选择一个多项式特征去更好的拟合是一个棘手的问题，局部加权回归的思路是：只对一小部分的训练参数使用简单的线性回归去拟合，然后对其他训练参数用其他模型，其实也是一个分治的思想。
相对于普通线性回归，其cost function要添加一个w选项，使距离越远的点权重越小。

	J(θ) = Σ(Wi * (yi-θ·xi))
	其中 Wi = exp(-(xi - x)^2/2)

###logistic回归的一些八卦
其实当时我就问了个问题，logistic function为什么又叫sigmoid funtion，现在知道了，原来就是因为那个函数曲线是S形的。

困了，睡觉先，未完待续





[LinChaohui]:    http://www.linchaohui.com  "LinChaohui"
