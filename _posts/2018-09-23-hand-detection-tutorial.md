---
layout: post
comments: true
title: "Training a Hand Detector with TensorFlow Object Detection API"
excerpt: "This is a tutorial on how to train a 'hand detector' with TensorFlow Object Detection API.  All code used in this tutorial are open-sourced on GitHub.  Just follow ths steps in this tutorial, and you should be able to train your own hand detector model in less than half a day."
date: 2018-09-23
category: "tensorflow"
tags: tensorflow
---

Quick link: [jkjung-avt/hand-detection-tutorial](https://github.com/jkjung-avt/hand-detection-tutorial)

I came accross this very nicely presented post, [How to Build a Real-time Hand-Detector using Neural Networks (SSD) on Tensorflow](https://towardsdatascience.com/how-to-build-a-real-time-hand-detector-using-neural-networks-ssd-on-tensorflow-d6bac0e4b2ce), written by Victor Dibia a while ago.  Now that I'd like to train an TensorFlow object detector by myself, optimize it with TensorRT, and deploy it on Jetson TX2, I immediately thought about following Victor's example and train a hand detector.  However, when I started to follow Victor's post and do the training, I immediately found not only were there quite a few missing dots, but some of the information was also out-dated.

So I decided to work on the hand-detector problem by myself.  And I wanted to create a tutorial which is up-to-date and easy to follow.  After quite some hard work, I was finally able to put together all necessary code/scripts and create this tutorial.

Note that, not like TensorFlow's Quick Start documentation (which starts by describing how to train an object detection model on Google Cloud Platform), my goal was to train the model locally using my own PC/server.  Let's dive in.

# Reference

* [Tensorflow Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection)
  * [Installation](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md)
  * [Quick Start: Distributed Training on the Oxford-IIIT Pets Dataset on Google Cloud](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/running_pets.md)
  * [Configuring the Object Detection Training Pipeline](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/configuring_jobs.md)
  * [Preparing Inputs](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/preparing_inputs.md)
  * [Running Locally](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/running_locally.md)
  * [Tensorflow detection model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md)


# Prerequisite

Training an object detector is more demanding than training an image classifier.  Ideally, you should have a decent NVIDIA GPU for this task.  As stated in my [jkjung-avt/hand-detection-tutorial/README.md](https://github.com/jkjung-avt/hand-detection-tutorial/blob/master/README.md), I used a good desktop PC with an NVIDIA GeForce GTX-1080Ti, running Ubuntu Linux 16.04, to do the training.

Make sure you have your training PC/server ready and a recent version of TensorFlow is properly installed on it.  (Reference: [Install TensorFlow](https://www.tensorflow.org/install/))

# Step-by-step

Please clone my GitHub repository: [jkjung-avt/hand-detection-tutorial](https://github.com/jkjung-avt/hand-detection-tutorial).  And refer to the README.md within.  All steps required to train the hand detector are listed there already.  I'd just add a few words about some of the steps here.

* The 'models/' submodule.  I added [tensorflow/models](https://github.com/tensorflow/models) as a submodule of this project.  And I used the same SHA version as in my [tf_trt_models](https://github.com/jkjung-avt/tf_trt_models) repository (which was forked from [NVIDIA-Jetson/tf_trt_models](https://github.com/NVIDIA-Jetson/tf_trt_models)).  This is to ensure I would train and deploy the model onto TX2 using the same version of Object Detection API.

* The `install.sh` script implements what's been specified in the official 'Installation' document.  I also patched a few lines of python code in the script.  Those are fixes required to run the code with python3.  (The original code only works for python2.)  Note that some python code under the `models/` directory still contains python2-specific code like `<dict>.iteritems()`.  I did not fix all occurrences of those yet.

* The `prepare_egohands.py` script downloads the [egohands dataset](http://vision.soic.indiana.edu/projects/egohands/).  Annotations in this dataset are 'polygons' stored in MATLAB files.  The script also converts the annotations into 'KITTI' format. Note that NVIDIA's DIGITS/DetectNet also uses KITTI formatted data for object detection models.  You can refer to [this document](for an explanation) of the format.  I chose this format since its annotation is relatively simple and could be easily reviewed/modified by hand.

* The `create_tfrecords.sh` script calls `create_kitti_tf_record.py`.  And the `create_kitti_tf_record.py` script was modified from [the same script](https://github.com/tensorflow/models/blob/master/research/object_detection/dataset_tools/create_kitti_tf_record.py) from tensorflow's object_detection repository.  I changed the code to fit file paths in this project.  I also shuffled train/val images randomly before creating the TFRecord files.

* The `configs/ssd_mobilenet_v1_egohands.config` was modified from tensorflow object_detection's sample [ssd_mobilenet_v1_coco.config](https://github.com/tensorflow/models/blob/master/research/object_detection/samples/configs/ssd_mobilenet_v1_coco.config).  I modified `num_classes` to 1, put in the correct file paths, and adjusted a few hyper-parameters in this file.  I plan to discuss more about this file in a later post.

* The `train.sh` implements what's been specified in the official 'Running Locally' document.

# Result

So far I have implemented and tested `ssd_mobilenet_v1_egohands` and `ssd_inception_v2_egohands`.  The `ssd_mobilenet_v1_egohands`, set to train for 20,000 steps, took a little bit over 2 hours to train on my desktop PC (GTX-1080Ti).  Its loss was around 2.5 at the end of training, and the 'coco_detection_metrics' evaluation result was as follows.  The IoU 0.50 mAP value was 0.968, which was very good.  It basically means, if we use the trained model to inference on images coming from the same distribution, the model could detect hands at both very high precision and very high recall!

```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.680
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.968
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.813
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.118
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.329
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.713
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.253
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.739
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.744
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.250
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.471
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.773
```

The high mAP result could be verified if you use TensorBoard to check the output of the `eval.sh` script.

![TensorBoard showing evaluation result of ssd_mobilenet_v1_egohands](https://github.com/jkjung-avt/hand-detection-tutorial/raw/master/doc/eval.png)

# What's next?

1. I shall deploy my trained hand detector (SSD) models onto Jetson TX2, and verify the accuracy and inference speed.

2. I shall write something about how to adapt code in this tutorial to other datasets.

3. I wanted to test other object detection models, including Faster R-CNN and Mask R-CNN, from [Tensorflow detection model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md).  Hopefully, I would be able to do that and share more soon.

# Final words

I made every effort in coding and writing this tutorial, so that it could be very easy to follow.  It should be straightforward to adapt this tutorial/code to other object detection models and other datasets.

After all, I really spent a lot of time reading/writing code and developing this tutorial.  If you find it useful, please help to share it with more people who might be interested.  Meanwhile, I do appreciate people giving me stars on GitHub.  That motivates me to write more posts/sharing.

![Stars on my GitHub repo](/assets/2018-09-23-hand-detection-tutorial/github-stars.png)
