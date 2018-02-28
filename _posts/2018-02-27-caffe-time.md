---
layout: post
comments: true
title: "Measuring Caffe Model Inference Speed on Jetson TX2"
excerpt: "When deploying Caffe models onto embedded platforms such as Jetson TX2, inference speed of the caffe models is an essential factor to consider. I think the best way to verify whether a Caffe model runs fast enough is to do measurement on the target platform."
date: 2018-02-27
category: "caffe"
tags: caffe
---

When deploying Caffe models onto embedded platforms such as Jetson TX2, inference speed of the caffe models is an essential factor to consider. I think the best way to verify whether a Caffe model runs fast enough is to do measurement on the target platform.

Here's how I measure caffe model inference time on Jetson TX2.

# Prerequisite:

* Build and install Caffe on the target Jetson TX2. Reference: [How to Install Caffe and PyCaffe on Jetson TX2](https://jkjung-avt.github.io/caffe-on-tx2/)
* Prepare `deploy.prototxt` for the Caffe models to be measured

In the following examples I used my own fork of [ssd-caffe](https://github.com/jkjung-avt/ssd-caffe).

# Reference:

* Check out the official [Caffe 'Interfaces' documentation](http://caffe.berkeleyvision.org/tutorial/interfaces.html) for descriptions about the `caffe time` command.

# Step-by-step:

Assuming a version of Caffe has been built at ~/project/ssd-caffe, we would use the built `caffe` executable to measure inference time of the models.

**Important:** During measurement `caffe` would use whatever input batch size as specified in the `deploy.prototxt`. When you compare inference speed of 2 different Caffe models, if input batch size is set differently between the 2, you would *not* be making fair comparisons.

For practical purposes I care most about inference time of **batch size 1** (inferencing only 1 single image each time). So when measuring, I would set input batch size to 1 for all models being compared.

Take AlexNet for example. First make a copy of its deploy.prototxt.

```shell
$ cp ~/project/ssd-caffe/models/bvlc_alexnet/deploy.prototxt /tmp/alexnet_deploy.prototxt
### Set TX2 to max performance mode before measuring
$ sudo nvpmodel -m 0
$ sudo ~/jetson_clocks.sh
### Modify input batch size as described below
$ vim /tmp/alexnet_deploy.prototxt
```

Then modeify line #6 of the prototxt to specify batch size as 1.

```
-   input_param { shape: { dim: 10 dim: 3 dim: 227 dim: 227 } }
+   input_param { shape: { dim: 1 dim: 3 dim: 227 dim: 227 } }
```

Run the `caffe time` command.

```
$ cd ~/project/ssd-caffe
$ ./build/tools/caffe time -gpu 0 -model /tmp/alexnet_deploy.prototxt 
I0228 11:53:37.071836  7979 caffe.cpp:343] Use GPU with device ID 0
I0228 11:53:37.616500  7979 net.cpp:58] Initializing net from parameters: 
......
0228 11:53:41.861127  7979 caffe.cpp:412] Average Forward pass: 12.9396 ms.
I0228 11:53:41.861150  7979 caffe.cpp:414] Average Backward pass: 35.2972 ms.
I0228 11:53:41.861168  7979 caffe.cpp:416] Average Forward-Backward: 48.4081 ms.
I0228 11:53:41.861196  7979 caffe.cpp:418] Total Time: 2420.4 ms.
```

So we get inference time (forward pass only) of bvlc_alexnet on JTX2 is about 12.9396 ms.

Next, repeat the measurement for bvlc_googlenet (set input batch size to 1 as well). And the result is 24.6415 ms.

```shell
$ ./build/tools/caffe time -gpu 0 -model /tmp/googlenet_deplay.prototxt 
I0228 12:00:19.444232  8129 caffe.cpp:343] Use GPU with device ID 0
I0228 12:00:19.983999  8129 net.cpp:58] Initializing net from parameters: 
......
I0228 12:00:25.924129  8129 caffe.cpp:412] Average Forward pass: 24.6415 ms.
I0228 12:00:25.924151  8129 caffe.cpp:414] Average Backward pass: 41.9625 ms.
I0228 12:00:25.924170  8129 caffe.cpp:416] Average Forward-Backward: 66.9036 ms.
I0228 12:00:25.924201  8129 caffe.cpp:418] Total Time: 3345.18 ms.
```

I also downloaded [VGG16](https://gist.github.com/ksimonyan/211839e770f7b538e2d8) and [ResNet-50](https://github.com/KaimingHe/deep-residual-networks/blob/master/prototxt/ResNet-50-deploy.prototxt) from links on [Caffe Model Zoo](https://github.com/BVLC/caffe/wiki/Model-Zoo) and did the measurements. Here are all the results.


  | Model          | Inference TIme |
  | -------------- |:--------------:|
  | bvlc_alexnet   | 12.9396 ms     |
  | bvlc_googlenet | 24.6415 ms     |
  | VGG16          | 91.82 ms       |
  | ResNet-50      | 64.0829 ms     |

A big take-away for myself by doing these measurements is that **bvlc_googlenet**, having similar classification accuracy as VGG16, actually runs much faster than VGG16 on JTX2. So it could be a better (speedier) CNN feature extractor for the object detection models such as Faster R-CNN, YOLO, and SSD.
