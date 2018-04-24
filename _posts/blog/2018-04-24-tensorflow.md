---
layout: post
title: Tensorflow的一些trick
description: Tensorflow
category: blog
---

## 修改variable值



    def get_param(self,scope_name,name):
      with tf.variable_scope(scope_name, reuse=True):
        return tf.get_variable(name)  # reuse

    fc1b = model.get_param('fc1','fc1_b')
    new_fc1b = tf.assign(fc1b,[1]*512)

 



[LinChaohui]:    http://www.linchaohui.cn  "LinChaohui"

