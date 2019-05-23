---
layout: post
comments: true
title: "Installing and Testing SSD Caffe on Jetson Nano"
excerpt: "I created a script for easily installing SSD caffe onto Jetson Nano."
date: 2019-05-16
category: "nano"
tags: jetson nano caffe
---

Quick link: [jkjung-avt/jetson_nano](https://github.com/jkjung-avt/jetson_nano)

In this post, I'm documenting how I install and test SSD caffe on my Jetson Nano DevKit.  The same steps/procedure should work for BVLC or other flavors of caffe.

# Reference

* [How to Install Caffe and PyCaffe on Jetson TX2](https://jkjung-avt.github.io/caffe-on-tx2/)
* [Measuring Caffe Model Inference Speed on Jetson TX2](https://jkjung-avt.github.io/caffe-time/)
* [Single Shot MultiBox Detector (SSD) on Jetson TX2](https://jkjung-avt.github.io/ssd/)
* [How to Do Real-time Object Detection with SSD on Jetson TX2](https://jkjung-avt.github.io/camera-ssd-threaded/)

# Prerequisite

Please make sure you have opencv-3.x properly installed on the Jetson Nano DevKit.  For example, you could follow my [Installing OpenCV 3.4.6 on Jetson Nano](https://jkjung-avt.github.io/opencv-on-nano/) post and have opencv-3.4.6 installed by just executing a script I developed.  (Note that SSD caffe cannot be compiled against opencv-4.x.)

Also, as stated in my opencv-3.4.6 post, you should set up a swap file on the Jetson Nano to avoid out-of-memory issues during the build process.

In addition, I'd suggest you to read the "Disclaimer" section of my [Setting up Jetson Nano: The Basics](https://jkjung-avt.github.io/setting-up-nano/) post to understand my use cases of Jetson Nano.  You might want to make adjustments to the script or use a different version of a specific library, based on your own requirements.

# Building and Installing SSD Caffe

The script for building SSD caffe is in my GitHub 'jetson_nano' repository.  You could clone it from GitHub.

```
$ cd ${HOME}/project
$ git clone https://github.com/jkjung-avt/jetson_nano.git
```

Make sure Jetson Nano is in 10W (maximum) performance mode so the building process could finish as soon as possible.  Later on when we test caffe inferencing performance of Jetson Nano, we'd also want to test it in this 10W performance mode.

```shell
$ sudo nvpmodel -m 0
$ sudo jetson_clocks
```

Then execute the `install_ssd-caffe.sh` script.  Note the script would clone [SSD caffe](https://github.com/weiliu89/caffe/tree/ssd) source files into `${HOME}/project/ssd-caffe`, and build the code from there.

During execution of the script, some of the steps (for example, installation of certain python3 modules) might take very long.  If sudo timeout is an issue for you, you could consider using `visudo` to set a longer `timestamp_timeout` value.  (Refer to my post [Installing OpenCV 3.4.6 on Jetson Nano](https://jkjung-avt.github.io/opencv-on-nano/) if needed.)

```
$ cd ${HOME}/project/jetson_nano
$ ./install_ssd-caffe.sh
```

The building process would take quite a while.  When it is done, you should see python3 reporting the version number of 'caffe' module: `1.0.0-rc3`.

# First Benchmarking of Performance Against Jetson TX2

As you might have noticed, I've put a `caffe time --gpu 0 --model ./models/bvlc_alexnet/deploy.prototxt` command towards the end of the installation script.  That serves as a simple test which gives us confidence the built `caffe` binary works properly.  In addition, we could compare the numbers against Jetson TX2.

|             | JetPack |  CUDA | cuDNN |  Forward Time  | Backward Time |
| :---------- | :-----: | :---: | :---: | :------------: | :-----------: |
| Jetson TX2  |   3.3   |   9.0 |   7   | **47.4552 ms** |   71.7691 ms  |
| Jetson Nano |   4.2   |  10.0 |   7   | **128.965 ms** |   218.347 ms  |

Doing inferening of a batch (10) of 227x227 images with Caffe/AlexNet, **Jetson Nano takes ~3 times longer than TX2**.  I have to say I was a little bit disappointed when I first saw the numbers...

Anyway, I would do some more testing/benchmarking (especially with TensorRT) and document my findings along the way.

# Testing SSD Caffe

Now that we have SSD caffe working on the Jetson Nano, we could run some Single-Shot Multibox Detector model and do real-time object detection with a camera.  For that, I use a googlenet_fc_coco_SSD_300x300 model which I trained with MS-COCO dataset before.  That model could recognize the 80 classes of objects defined in COCO.  Its accuracy (mAP) is slightly better than the original VGGNet based SSD_300x300, while it runs faster.  You could download the model from [my GoogleDrive link](https://drive.google.com/file/d/1Ypt6H0sWI3W9I9dai5BXo2Uqsqs6tv5K/view?usp=sharing).

For [real-time object detection](https://jkjung-avt.github.io/camera-ssd-threaded/), I'd be using my [camera-ssd-threaded.py](https://gist.github.com/jkjung-avt/605904dc05691e44a26bc57bb50d3f04) script.

Assuming the `models_googlenet_fc_coco_SSD_300x300.tar.gz` file has been downloaded and saved at `${HOME}/Downloads`.  Then execute the following.

```shell
$ cd ${HOME}/project/ssd-caffe
$ tar xzvf ${HOME}/Downloads/models_googlenet_fc_coco_SSD_300x300.tar.gz
$ wget https://gist.githubusercontent.com/jkjung-avt/605904dc05691e44a26bc57bb50d3f04/raw/2076ea526adb7fdf8c25a89fa006ccefd9501263/camera-ssd-threaded.py
$ PYTHONPATH=`pwd`/python python3 camera-ssd-threaded.py --usb --vid 0 --prototxt models/googlenet_fc/coco/SSD_300x300/deploy.prototxt --model models/googlenet_fc/coco/SSD_300x300/deploy.caffemodel --width 1280 --height 720
```

As you could see below, I used a USB webcam for testing.  The SSD model recognized Steph Curry on my laptop screen, but did not do too well for the other players.  Otherwise, it also recognized the laptop computer and a few books.  It took the googlenet_fc_coco_SSD_300x300 model about 200ms to inference each frame of image from the webcam.  So it was running at roughly **5fps**.

![Inferencing with the googlenet_fc/coco/SSD_300x300 model](/assets/2019-05-16-ssd-caffe-on-nano/googlenet_fc_ssd.png)
