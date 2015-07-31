---
layout: post
title: MCMC & Gibbs Sampling
description: 新系列(三)
category: blog
---

一般看完贝叶斯公式就应该开始Gibbs采样的相关学习了，但这个之前还要学习MCMC，所谓圆环套圆环啊。于是我看了两篇讲的不错的博客，首先可以看下[这篇](http://www.52nlp.cn/lda-math-mcmc-%E5%92%8C-gibbs-sampling1)，然后可以继续看下[这篇](http://www.cnblogs.com/xbinworld/p/4266146.html)。之前看过很多对冲基金就用的蒙特卡洛马尔柯夫模型，所以一开始看是充满了期待的。刚开始介绍了采样以及Box-Muller变换，虽然推不出来但还是可以接受的。然后就是学习马尔科夫链的收敛定理，一开始我是拒绝。为什么能发生这么神奇的事情，还不带证明过程的！

然后，我就思考了半个小时，考虑怎么让自己接受这个结果。在涂涂画画了半小时后，我突然发现了，原来是这个样子。其实，状态转移矩阵本身有一个隐含条件，那就是每一行的和值\\(\sum_{i=1}^\infty P_{ij} = 1\\)
\\(\sum_{i=1}^{\infty} P_{ij} = 1\\)



[LinChaohui]:    http://www.linchaohui.com  "LinChaohui"
