---
layout: post
comments: true
title: "Deep Learning Cats Dogs Tutorial on Jetson TX2"
excerpt: "I ran the Deep Leanring Cats Dogs Tutorial code to train an AlexNet on Jetson TX2. This is the best Caffe and Pyhton tutorial I've come across so far. I highly recommend it to people who wants to learn Caffe."
date: 2017-08-11
category: "caffe"
tags: caffe
---

In general it's not recommended to train neural nets on an embedded platform like Jetson TX2. I did it for the sake of learning. In fact, this example works OK on Jetson TX2, and I do recommend it to people who wants to learn Caffe.

(Well, I've already been training the Nintendo DQN directly on Jetson TX1 before...)

The original tutorial: [A Pactical Introduction to Deep Learning with Caffe and Python](http://adilmoujahid.com/posts/2016/06/introduction-deep-learning-python-caffe/)

This tutorial demonstrates how to train an AlexNet to do image classification between 2 classes: cats and dogs. The dataset, which contains 25,000 training images, comes from Kaggle. The tutorial walks you through how to do data preparation, how to define a caffe model and the solver, how to do training, and how to use the trained model to do prediction. The model is first trained from scratch (with random initialization). Then transfer learning is applied to improve the training result. Overall I think this is the best Caffe tutorial I've come across so far.

The original source code provided along with this tutorial seemed for python2 only. When I tried to run the code on Jetson TX2 with python3, I had to fix a few things, namely:

* I checked out the code into /home/nvidia/project folder, so I had to modify all file paths in the code to match my set-up.
* I added () to all print functions, which is required for python3.
* In `code/create_lmdb.py`, map_size of lmdb.open() needed to be reduced.
* Also in `code/create_lmdb.py`, I added encode('ascii') in in_txn.put(). This is also required for python3.

You can get a copy of the modified code from my GitHub repository: [https://github.com/jkjung-avt/deeplearning-cats-dogs-tutorial](https://github.com/jkjung-avt/deeplearning-cats-dogs-tutorial).

With the modifications mentioned above, I was able to train the models on my Jetson TX2 in a reasonable time frame. More specifically, 'max_iter' was set to 40,000 in the Caffe solver files, and it took roughly **40 hours** for the 40,000 iterations of training to complete on my Jetson TX2. (I've used 'sudo nvpmodel -m 0' to set the Jetson TX2 to maximum performance mode.)

As a final note, I noticed that the deeplearing-cats-dogs-tutorial code could take up to **6.5GB** of memory during training. So most likely the code would not work on a Jetson TX1 (with 4GB of RAM only). If you'd really like to run this code on a Jetson TX1, try to reduce the data 'batch_size' defined in say `caffe_models/caffe_model_1/caffenet_train_val_1.prototxt`.


