---
layout: post
comments: true
title: "TensorFlow/TensorRT Models on Jetson TX2"
excerpt: "NVIDIA released tf_trt_models sample code for both image classification and object detection a while ago.  I tested it and developed a real-time object detection script using TensorRT optimized TensorFlow models based on NVIDIA's code.  I'd like to share the demo script here."
date: 2018-09-14
category: "tensorrt"
tags: tensorrt tensorflow
---

Quick link: [jkjung-avt/tf_trt_models](https://github.com/jkjung-avt/tf_trt_models)

I forked the code from [NVIDIA-Jetson/tf_trt_models](https://github.com/NVIDIA-Jetson/tf_trt_models) and developed a `camera_tf_trt.py` script which could do real-time object detection with TensorRT optimized models.

# Background information and references

* [TensorRT Integration Speeds Up TensorFlow Inference](https://devblogs.nvidia.com/tensorrt-integration-speeds-tensorflow-inference/): NVIDIA TensorRT has been integrated into TensorFlow (since v1.7.0), and the python API is extremely easy to use!

* GTC 2018 talk: "Accelerate TensorFlow Inference with New TensorRT Integration", [video](http://on-demand.gputechconf.com/gtc/2018/video/S81009/), [slides](http://on-demand.gputechconf.com/gtc/2018/presentation/s81009-accelerate-tensorflow-inference-with-new-tensorrt-integration.pdf)

* Announcement on Jetson TX2 developer forum: [TensorFlow object detection and image classification accelerated for NVIDIA Jetson](https://devtalk.nvidia.com/default/topic/1037019/jetson-tx2/tensorflow-object-detection-and-image-classification-accelerated-for-nvidia-jetson/)

* NVIDIA's tf_trt_models repository: [NVIDIA-Jetson/tf_trt_models](https://github.com/NVIDIA-Jetson/tf_trt_models)

# How to set up Jetson TX2 environment before running the demo code

Refer to my GitHub repository (forked and modified from NVIDIA's repository), and follow instructions in the 'Setup' section.

[https://github.com/jkjung-avt/tf_trt_models](https://github.com/jkjung-avt/tf_trt_models)

The steps are:

1. Flash Jetson TX2 with JetPack-3.3 (TensorRT 4.0 GA included).
2. Install OpenCV 3.4.x.
3. Install TensorFlow 1.10.0.
4. Clone the code from my GitHub repo.
5. Run the installation script.

# Description of camera_tf_trt.py

* The `camera_tf_trt.py` script supports video inputs from one of the following sources: (1) a video file, say mp4, (2) an image file, say jpg or png, (3) an RTSP stream from an IP CAM, (4) a USB webcam, (5) the Jetson onboard camera.  In case of image file, the script would do inferencing using the same image over and over again.
* The object detection models all come from [TensorFlow Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection).  Based on NVIDIA's code, this script could download the pretrained model snapshot (provided by Google) and optimize it with TensorRT (when `--build` option is specified).  The TensorRT optimized model/graph would be automatically saved as a protobuf file in one of the `./data/*` directories.  So the `--build` action only needs to be done once for each model to be used.  Later invocations of the script could load the optimized graph directly and thus saves execution time.
* Here is the help message of the script.

  ```
  $ python3 camera_tf_trt.py --help
  usage: camera_tf_trt.py [-h] [--file] [--image] [--filename FILENAME] [--rtsp]
                          [--uri RTSP_URI] [--latency RTSP_LATENCY] [--usb]
                          [--vid VIDEO_DEV] [--width IMAGE_WIDTH]
                          [--height IMAGE_HEIGHT] [--model MODEL] [--build]
                          [--tensorboard] [--labelmap LABELMAP_FILE]
                          [--num-classes NUM_CLASSES] [--confidence CONF_TH]
  
  This script captures and displays live camera video, and does real-time object detection with TF-TRT model on Jetson TX2/TX1
  
  optional arguments:
    -h, --help            show this help message and exit
    --file                use a video file as input (remember to also set
                          --filename)
    --image               use an image file as input (remember to also set
                          --filename)
    --filename FILENAME   video file name, e.g. test.mp4
    --rtsp                use IP CAM (remember to also set --uri)
    --uri RTSP_URI        RTSP URI, e.g. rtsp://192.168.1.64:554
    --latency RTSP_LATENCY
                          latency in ms for RTSP [200]
    --usb                 use USB webcam (remember to also set --vid)
    --vid VIDEO_DEV       device # of USB webcam (/dev/video?) [1]
    --width IMAGE_WIDTH   image width [1280]
    --height IMAGE_HEIGHT
                          image height [720]
    --model MODEL         tf-trt object detecion model [ssd_inception_v2_coco]
    --build               re-build TRT pb file (instead of usingthe previously
                          built version)
    --tensorboard         write optimized graph summary to TensorBoard
    --labelmap LABELMAP_FILE
                          [third_party/models/research/object_detection/data/msc
                          oco_label_map.pbtxt]
    --num-classes NUM_CLASSES
                          number of object classes [90]
    --confidence CONF_TH  confidence threshold [0.3]
  ```

# Examples of using the camera_tf_trt.py script

Example #1: build TensorRT optimized 'ssd_mobilenet_v1_coco' model and run real-time object detection with USB webcam.

```
$ python3 camera_tf_trt.py --usb --model ssd_mobilenet_v1_coco --build
```

Example #2: verify the optimized 'ssd_mobilenet_v1_coco' model with NVIDIA's original 'huskies.jpg' picture. 

```
$ python3 camera_tf_trt.py --image --filename examples/detection/data/huskies.jpg --model ssd_mobilenet_v1_coco
```

![MobileNet V1 SSD detection result on huskies.jpg]( https://raw.githubusercontent.com/jkjung-avt/tf_trt_models/master/examples/detection/data/huskies.jpg)

Example #3: build TensorRT optimized 'ssd_inception_v2_coco' model and run real-time object detection with Jetson onboard camera.

```
$ python3 camera_tf_trt.py --model ssd_inception_v2_coco --build
```

Example #4: test the optimized 'ssd_inception_v2_coco' model with a mp4 video file.

```
$ python3 camera_tf_trt.py --file --filename examples/detection/data/huskies.jpg --model ssd_inception_v2_coco
```

# Observations

Since I myself am not very familiar with TensorFlow and am still learning.  I'm not completely sure my current implementation of `camera_tf_trt.py` is correct or not.

* Documentation in NVIDIA's original repository asks the user to install tensorflow-1.8.0.  And I was using tensorflow-1.10.0 for all my testing.  I'm not sure if this would result in significant difference in performance of the TensorRT optimized model or not.
* I ran both 'ssd_inception_v2_coco' and 'ssd_mobilenet_v1_coco' on my Jetson TX2.  They did not run as fast as what NVIDIA has published on their repo.
* The process of building TensorRT optimized model and loading saved TensorRT model from protobuf takes **very long** (> 10 minutes).  I'm not sure whether it's normal.  I need to dig into it.

Otherwise I plan to test out more pretrained object detection models (including, say, Faster R-CNN ones) from [TensorFlow detection model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md).  Hopefully, I will be able to share more experience on that soon.

Do feel free to leave a comment below.
