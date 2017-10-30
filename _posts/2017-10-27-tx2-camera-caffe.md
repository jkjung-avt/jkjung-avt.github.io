---
layout: post
comments: true
title: "How to Capture Camera Video and Do Caffe Inferencing with Python on Jetson TX2"
excerpt: "I extended my previous tegra-cam.py example by hooking up a Caffe image classification model into the video pipeline. The resulting code should be good for quickly verifying a newly trained Caffe image classification model, for prototyping, or for building Caffe demo programs with live camera input."
date: 2017-10-27
category: "opencv"
tags: opencv caffe
---

Quick link: [tegra-cam-caffe.py]( https://gist.github.com/jkjung-avt/d408aaabebb5b0041c318f4518bd918f)

Last week I shared a python script which could be used to capture and display live video from camera (IP, USB or onboard) on Jetson TX2. Here I extend that script and show how you can run Caffe image classification (inferencing) on the captured camera images, all done in python code. This `tegra-cam-caffe.py` sample should be good for quickly verifying your newly trained Caffe image classification models, for prototyping, or for building Caffe demo programs with live camera input.

I mainly tested the script with python3 on Jetson TX2. But I think the code also works with python2, as well as on Jetson TX1.

Prerequisite:

* Refer to my previous post ["How to Capture and Display Camera Video with Python on Jetson TX2"](https://jkjung-avt.github.io/tx2-camera-with-python/) and make sure `tegra-cam.py` runs OK on your Jetson TX2.
* Build Caffe on your Jetson TX2. You can use either [BVLC Caffe](https://github.com/BVLC/caffe), [NVIDIA Caffe](https://github.com/NVIDIA/caffe) or other flavors of Caffe of your preference, while referring to my earlier post about how to build the code: [How to Install Caffe and PyCaffe on Jetson TX2](https://jkjung-avt.github.io/caffe-on-tx2/).
* My python code assumes the built Caffe code is located at `/home/nvidia/caffe`. In order to run the script with the default `bvlc_reference_caffenet` model, you'll have to download the pre-trained weights and labels by:

```shell
$ cd /home/nvidia/caffe
$ ./scripts/download_model_binary.py ./models/bvlc_reference_caffenet
$ ./data/ilsvrc12/get_ilsvrc_aux.sh
```

Reference:

* How to classify images with Caffe's python API: [https://github.com/BVLC/caffe/blob/master/examples/00-classification.ipynb](https://github.com/BVLC/caffe/blob/master/examples/00-classification.ipynb)
* How to calculate mean pixel values for mean subtraction: [https://devtalk.nvidia.com/default/topic/1023944/loading-custom-models-on-jetson-tx2/#5209641](https://devtalk.nvidia.com/default/topic/1023944/loading-custom-models-on-jetson-tx2/#5209641)

How to run the Tegra camera Caffe sample code:

* Download the `tegra-cam-caffe.py` source code from my GitHubGist: [https://gist.github.com/jkjung-avt/d408aaabebb5b0041c318f4518bd918f](https://gist.github.com/jkjung-avt/d408aaabebb5b0041c318f4518bd918f)
* To dump help messages.

```shell
$ python3 tegra-cam-caffe.py --help
```

* To do Caffe image classification with the default `bvlc_reference_caffenet` model on USB webcam `/dev/video1`.

```shell
$ python3 tegra-cam-caffe.py --usb --vid 1
```

* To do image classification with a different Caffe model on the onboard camera.

```shell
$ python3 tegra-cam-caffe.py --prototxt XXX.prototxt \
                             --model YYY.caffemodel \
                             --labels ZZZ.txt \
                             --mean UUU.binaryproto
```

When I tested the code with a USB camera and a picture of pineapple, the default `bvlc_reference_caffenet` said it was 100% sure (probability ~ 1.0) the image was a pineapple!

![A pineapple picture shown to tegra-cam-caffe.py](/assets/2017-10-27-tx2-camera-caffe/pineapple.png)

Next, I tried to test with a Caffe model trained with NVIDIA DIGITS. More specifically, I trained an AlexNet with 'Caltech 101' dataset, as mentioned in this NVIDIA QuikLabs course: [Image Classification with DIGITS](https://nvidia.qwiklab.com/focuses/1579). One very nice thing about this free QuickLabs course is that you get 2-hour access of a K520 GPU based cloud server with NVIDIA DIGITS, with no charge at all. After successfully training an AlexNet model with 'Caltech 101' dataset (I just trained the model for 30 epochs with plain SGD and the default learning rate, 0.01), I then downloaded the model snapshot of the last training epoch from DIGITS: `20171022-025612-7b04_epoch_30.0.tar.gz`. Here's the list of files in that snapshot tarball.

```shell
info.json
labels.txt
mean.binaryproto
original.prototxt
snapshot_iter_1620.caffemodel
solver.prototxt
train_val.prototxt
```

I then verified this trained Caffe model with the following command. During training, the logs indicated this trained model has an accuracy at only around 67.5% (for classifying 101 classes of objects). When testing, I did find this model working poorly on many test images. But anyway I managed to get this model to classify a 'pegion' picture correctly.

```
$ python3 ./tegra-cam-caffe.py --usb --vid 1 --crop \
                               --prototxt alexnet/deploy.prototxt \
                               --model alexnet/snapshot_iter_1620.caffemodel \
                               --labels alexnet/labels.txt \
                               --mean alexnet/mean.binaryproto \
                               --output softmax
```

I have not done much testing of this code with various cameras or Caffe models. Feel free to let me know if you find any issue with the code, and I'll look into it as soon as I can.

