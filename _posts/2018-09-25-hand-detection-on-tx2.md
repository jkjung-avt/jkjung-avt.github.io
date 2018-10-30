---
layout: post
comments: true
title: "Deploying the Hand Detector onto Jetson TX2"
excerpt: "Building upon my previous tutorials, I demonstrate how to optimize a custom trained object detection model with TensorRT, and deploy it onto Jetson TX2."
date: 2018-09-25
category: "tensorrt"
tags: tensorrt tensorflow
---

Quick link: [jkjung-avt/tf_trt_models](https://github.com/jkjung-avt/tf_trt_models)

In previous posts, I've shared how to apply TF-TRT to optimize pretrained object detection models, as well as how to train a hand detector with TensorFlow Object Detection API.  Let's now put things together by optimizing and deploying the hand detector on Jetson TX2.

# Prerequisite

1. Follow my previous post, [TensorFlow/TensorRT Models on Jetson TX2](https://jkjung-avt.github.io/tf-trt-models/).  Make sure your Jetson TX2 has been set up and could run `camera_tf_trt.py` successfully.

   Note that I've just fixed a couple of bugs, as well as added support for the 'ssd_mobilenet_v1_egohands' model, in my [jkjung-avt/tf_trt_models](https://github.com/jkjung-avt/tf_trt_models) repo.  So if you have cloned the repository previously, do pull the latest code from GitHub again.

2. Follow my other post, [Training a Hand Detector with TensorFlow Object Detection API](https://jkjung-avt.github.io/hand-detection-tutorial/).  Make sure you have successfully trained the 'ssd_mobilenet_v1_egohands' model.  We would be using the checkpoint (saved model weights) file for the demonstration below.

# Changes needed from previous tf_trt_model

The following description refers to code in `tf_trt_models`.

* `tf_trt_models` would need the config and checkpoint files of the 'ssd_mobilenet_v1_egohands' model, to be able to compile an optimized tensorflow graph for inferencing.  I made a copy of `data/ssd_mobilenet_v1_egohands.config` from the `hand-detection-tutorial` repo, while changing the `score_threshold` value from 1e-8 to 0.3 within.  However, I did not put a copy of the model checkpoint files in the repository, since those files were too big.  Please copy them over as instructed below.

* In `utils/ssd_util.py`, I added `get_egohands_model()` for locating the model config and checkpoint files.

* I made a copy of `data/egohands_label_map.pbtxt`.  This prototxt file is used for converting class IDs to class names.

# Running TF-TRT optimized ssd_mobilenet_v1_egohands model on Jetson TX2

Copy your 'ssd_mobilenet_v1_egohands' checkpoint files from your `hand-detection-tutorial` to your `tf_trt_models` directory.  For example, I have both project code checked out under my `~/project/` folder, so I'd do the following.  Note that we have trained the model for a total of 20,000 steps.  That's why the checkpoint files are marked with that number.

```shell
$ cd ~/project/tf_trt_models
$ cp ~/project/hand-detection-tutorial/ssd_mobilenet_v1_egohands/model.ckpt-20000.* ./data/ssd_mobilenet_v1_egohands/
```

When inferencing the model with `camera_tf_trt.py`, you'd need to specify the label map file and set number of classes to 1.  Refer to the commands below for details.

Example #1: build TensorRT optimized 'ssd_mobilenet_v1_egohands' model and test with a still image.  I took a picture of my son's hands along with his newly assembled LEGO car for testing.  The result looked good.  `camera_tf_trt.py` ran at ~24.5 fps in this case.

```
$ python3 camera_tf_trt.py --image \
                           --filename jk-son-hands.jpg \
                           --model ssd_mobilenet_v1_egohands \
                           --labelmap data/egohands_label_map.pbtxt \
                           --num-classes 1 \
                           --build
```

![Tested with a photo of my son's hands](/assets/2018-09-25-hand-detection-on-tx2/son-hands-detected.png)

Example #2: verify the optimized 'ssd_mobilenet_v1_egohands' model with live video feed from a USB webcam.  Here I myself was acting as the model in front of the webcam :-)

```
$ python3 camera_tf_trt.py --usb \
                           --model ssd_mobilenet_v1_egohands \
                           --labelmap data/egohands_label_map.pbtxt \
                           --num-classes 1
```

![Tested with my own hand in front of a USB webcam](/assets/2018-09-25-hand-detection-on-tx2/jk-hand-detected.png)

In addition to image file and USB webcam, the `camera_tf_trt.py` script also supports video file, IP CAM, or Jetson onboard camera as input source.  I'm not going to re-iterate those here.  You can refer to `--help` message for more details.
