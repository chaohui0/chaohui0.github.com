---
layout: post
title: Tensorflow的一些trick
description: Tensorflow
category: blog
---
## kaldi与Tensorflow模型
kaldi的sigmoid节点计算公式为: W*X + b

Tensorflow的模型计算公式为：  X*W + b

tf模型做奇异值分解推导：

X*W + b = X*U*V + b

分解为两层：

y' = X*U

y = y'*V + b



## 修改variable值

model.py示例

    import os
    import tensorflow as tf
    from tensorflow.python.lib.io import file_io
    from tensorflow.contrib.session_bundle.exporter import Exporter
    from tensorflow.contrib.session_bundle.exporter import generic_signature
    from tensorflow.python.util import compat
    from template_model import *
    import numpy as np

    def read_vec(infile):

        tensors = []
        while 1:
            line  = infile.readline().replace(" \n","").strip()
            if not line:
                break

            ls = line.split(" ")
            if len(ls) == 4:
                layer = ls[0]
                name  = ls[1]
                line_num = int(ls[2])
                columns = int(ls[3])
                vec = []

                for i in range(line_num):
                    line  = infile.readline().replace(" \n","").strip()
                    ts  = line.split(" ")
                    ts = map(lambda x:float(x),ts)
                    if line_num == 1:
                        vec = ts
                    else:
                        vec.append(ts)
                if line_num > 1:
                    vec = np.transpose(vec)

                tensors.append([layer,name,vec])
                line  = infile.readline().replace(" \n","").strip()
                print("break:%s"%line)
        #print vec
        return tensors


    tf.app.flags.DEFINE_string("volumes", "tf_data/train,tf_data/val", "volumes info")
    #tf.app.flags.DEFINE_string("volumes", "./tf_data/train,./tf_data/val", "volumes info")
    tf.app.flags.DEFINE_string("checkpoint_file", "./saver/export", "")
    tf.app.flags.DEFINE_integer('model_version', 0, 'version number of the model.')
    tf.app.flags.DEFINE_integer('batch_size', 256, 'batch_size')

    FLAGS = tf.app.flags.FLAGS
    print(FLAGS)

    #input_size = 720
    input_size = 440
    output_size = 411
    dim_vec = 24
    frame_num = 40
    #hidden_size = 128
    hidden_size = 32

    class TreasureModel(ModelTemplate):
        def __init__(self):
            self.hls = [512,512,512,512]
            super(TreasureModel,self).__init__()

        def fc_layer(self, inputs, channels_in, channels_out, name="fc", active="sigmoid",norm=True,is_training=False,reuse=False):
            with  tf.variable_scope(name):
                w = self.add_param("%s_w"%name,[channels_in,channels_out],tf.float32,tf.truncated_normal_initializer(stddev=0.1))
                b = self.add_param("%s_b"%name,[channels_out],tf.float32,tf.zeros_initializer()) 

                Wx_plus_b = tf.matmul(inputs, w) + b
                self.params_dic["%s_full"%name] = Wx_plus_b
                if active == "sigmoid":
                    return tf.nn.sigmoid(Wx_plus_b)
                return tf.nn.relu(Wx_plus_b)

        def softmax_layer(self, inputs, channels_in, channels_out, name="softmax"):
            with  tf.variable_scope(name):
                w = self.add_param("%s_w"%name,[channels_in,channels_out],tf.float32,tf.truncated_normal_initializer(stddev=0.1))
                b = self.add_param("%s_b"%name,[channels_out],tf.float32, tf.zeros_initializer()) 
                return tf.nn.softmax(tf.matmul(inputs,w) + b)

        def rnn_layer(self, rnn_size, n_classes,x,name="rnn",re_use=True):
            with  tf.variable_scope(name,reuse=re_use):

                lstm_cell = tf.contrib.rnn.BasicLSTMCell(rnn_size, forget_bias=0)
                outputs, states = tf.contrib.rnn.static_rnn(lstm_cell, x, dtype=tf.float32)
                w = self.add_param("%s_w"%name, [rnn_size,n_classes], tf.float32, tf.random_normal_initializer())
                b = self.add_param("%s_b"%name, [n_classes], tf.float32, tf.random_normal_initializer())
                output = tf.matmul(outputs[-1], w) + b
                return tf.nn.softmax(output),outputs

        def scores(self, features,keep_prob,phase_train,re_use):
            features = tf.reshape(tf.to_float(features),[-1,input_size])
            #fc layers
            fc1 = self.fc_layer(features, input_size, self.hls[0],name="fc1",is_training=phase_train,reuse=re_use)
            fc2 = self.fc_layer(fc1, self.hls[0], self.hls[1],name="fc2",is_training=phase_train,reuse=re_use)
            fc3 = self.fc_layer(fc2, self.hls[1], self.hls[2],name="fc3",is_training=phase_train,reuse=re_use)
            fc4 = self.fc_layer(fc3, self.hls[2], self.hls[3],name="fc4",is_training=phase_train,reuse=re_use)
            y = self.softmax_layer(fc4, self.hls[3], output_size, name="fc5")
            return y

        def loss(self,labels, features):
            print("in loss")
            y_ = tf.to_float(labels)
            y  = self.scores(features,0.5,True,False)
            with tf.name_scope("cost_function") as scope:
                cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_*tf.log(tf.clip_by_value(y,1e-10,1.0)), reduction_indices=[1]))

            return cross_entropy

        def accuracy(self, labels, features):
            print("in accuracy")
            y_pred = self.scores(features,1.0,False,True)
            correct_prediction = tf.equal(tf.argmax(y_pred, 1), tf.argmax(labels, 1))
            accu = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
            return accu

        def signature(self):
            print("in signature")
            y_pred = self.scores(features,1.0,False,True)
            feature = tf.placeholder(tf.float32, [None,input_size],name='input')
            scores = self.scores(feature,1.0,False,True)
            signature = {'feature': feature, 'label': scores}
            return signature

        def init_values(self):
            kaldi_model_file = open("rbm.final.txt")
            self.tensors = read_vec(kaldi_model_file)

            #self.params_dic
            assign_tensors = []
            print("in init values") 
            for item in self.tensors:
                assign_tensors.append(tf.assign(self.params_dic[item[1]],item[2]))

            return assign_tensors

    class RecordReader(ReaderTemplate):

        def extract_feature(self,example):
            features = tf.parse_single_example(
                example,
                features={
                  'feature' : tf.FixedLenFeature([input_size], tf.float32),
                  'label': tf.FixedLenFeature([], tf.int64),
                  })
            feature = tf.cast(features["feature"], tf.float32)
            label = tf.cast(features['label'], tf.int32)
            one_hot_labels = tf.to_float(tf.one_hot(label, output_size, 1, 0))
            return feature, one_hot_labels, label

    if __name__ == '__main__':
        print('Started building graph...')
        with tf.Graph().as_default():

            print('Started building graph...')
            inputs = [v.strip() for v in FLAGS.volumes.split(',') if v != '']

            data_reader = RecordReader(inputs[0],inputs[1])
            print(data_reader)

            with tf.name_scope("read_train_data") as scope:
                train_features, train_labels, train_ori_label = data_reader.read_trainset(batch_size=FLAGS.batch_size)
            with tf.name_scope("read_val_data") as scope:
                test_features, test_labels, test_ori_label = data_reader.read_valset(batch_size=FLAGS.batch_size)

            model = TreasureModel()

            with tf.name_scope("train") as scope:
                loss = model.loss(train_labels,train_features)
                global_step = tf.Variable(0, trainable=False)
                starter_learning_rate = 5e-5
                #learning_rate = tf.train.exponential_decay(starter_learning_rate, global_step,20000,0.9, staircase=True)
                learning_rate = starter_learning_rate
                train_step = tf.train.AdamOptimizer(learning_rate).minimize(loss)

            y = model.scores(train_features,0.5,False,True)

            with tf.name_scope("accuracy") as scope:
                accuracy = model.accuracy(test_labels,test_features)
                accuracy_train = model.accuracy(train_labels,train_features)
            #init = tf.global_variables_initializer()
            saver = tf.train.Saver()
            #init = tf.initialize_all_variables()
            num_steps = 300000000/FLAGS.batch_size

            #writer = tf.summary.FileWriter("../../../tensorboard_log")
            with tf.Session() as sess:
                sess.run([tf.global_variables_initializer(), tf.local_variables_initializer()])
                saver.restore(sess, FLAGS.checkpoint_file)
                #sess.run(model.init_values())

                t_vars = tf.trainable_variables()
                coord = tf.train.Coordinator()
                threads = tf.train.start_queue_runners(sess=sess, coord=coord)
                for i in range(num_steps):
                    #fe,la = sess.run([test_features,test_labels])
                    #print fe
                    #print sess.run(y)
                    sess.run(train_step)
                    if ((i + 1) % 100 == 0):
                        #print la
                        print("step:", i + 1, "accuracy:", sess.run(accuracy)," train accuracy:",sess.run(accuracy_train))

                    if ((i + 1) % 10 == 0):
                        path = saver.save(sess, FLAGS.checkpoint_file)
                ''' 
                for var in t_vars:
                    if var.name.find("b") < 0:
                        continue
                    print(var.name,var.get_shape())
                    var = tf.transpose(var)
                    t = sess.run(tf.reshape(var,[-1]))
                    print ("len:",len(t) )
                    for item in t:
                        print item
                '''
                coord.request_stop()
                coord.join(threads)
                #signature = model.signature()
                #model.export(sess, FLAGS.checkpointDir,signature)

                path = saver.save(sess, FLAGS.checkpoint_file)
                print(path)


template_model.py

    # -*- coding: utf-8 -*-

    import tensorflow as tf
    import threading
    from tensorflow.contrib.session_bundle.exporter import Exporter
    from tensorflow.contrib.session_bundle.exporter import generic_signature

    class ModelTemplate(object):
        def __init__(self):
            self.params_dic = {}

        def add_param(self,name,shape,m_type,init_func):
            if name not in self.params_dic:
                self.params_dic[name] = tf.get_variable(name,shape,m_type,init_func)
            return self.params_dic[name]

        def get_param(self,scope_name,name):
            with tf.variable_scope(scope_name, reuse=True):
                return tf.get_variable(name)  # reuse
            #return self.params_dic[name]

        def scores(self):
            #前向计算
            pass

        def loss(self):
            #cal cost
            pass

        def accuracy(self):
            #准确率计算
            pass

        def signature(self):
            #生成sessionbunddle需要的signature
            pass

        def export(self, sess, path, sig):
            saver = tf.train.Saver()
            exporter = Exporter(saver)
            exporter.init(sess.graph.as_graph_def(),
                    clear_devices=True,
                    default_graph_signature=generic_signature(sig))
            #导出的SessionBundle统一为00000000
            exporter.export(path, tf.constant(0), sess)

    class ReaderTemplate(object):
        def __init__(self,train_root,val_root):
            if train_root.find("odps://") >= 0 and train_root.find("cached://") < 0:
                train_root = "cached://%s"%train_root
            if val_root.find("odps://") >= 0 and val_root.find("cached://") < 0:
                val_root = "cached://%s"%val_root

            self.train_file_path_ = train_root  + '*.tfrecords'
            self.val_file_path_ = val_root   + '*.tfrecords'

            self.train_file_path_ = train_root  + '.tfrecords*'
            self.val_file_path_ = val_root   + '.tfrecords*'

            print("train file path:%s"%self.train_file_path_)
            print("validate file path:%s"%self.val_file_path_)

        def extract_feature(self,example):
            #特征解析
            pass

        def __read(self, file_path,batch_size):
            print("file_path")
            print(file_path)
            #file_queue = tf.train.string_input_producer([file_path])
            file_queue = tf.train.string_input_producer(tf.train.match_filenames_once(file_path))
            #file_queue = tf.train.string_input_producer(tf.train.match_filenames_once(file_path),num_epochs=1)
            reader = tf.TFRecordReader()
            _, example = reader.read(file_queue)

            return tf.train.batch(
                    self.extract_feature(example),
                    batch_size=batch_size,
                    capacity=batch_size*3)

        def read_trainset(self,batch_size):
            return self.__read(self.train_file_path_,batch_size)

        def read_valset(self,batch_size):
            return self.__read(self.val_file_path_,batch_size)





 



[LinChaohui]:    http://www.linchaohui.cn  "LinChaohui"

