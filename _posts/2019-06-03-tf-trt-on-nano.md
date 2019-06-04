---
layout: post
comments: true
title: "Testing TF-TRT Object Detectors on Jetson Nano"
excerpt: "I tested TF-TRT object detection models on my Jetson Nano DevKit.  I also compared model inferencing time against Jetson TX2."
date: 2019-06-03
category: "nano"
tags: nano tensorrt tensorflow
---

I tested TF-TRT object detection models on my Jetson Nano DevKit.  I also compared model inferencing time against Jetson TX2.  This post documents the results.

# Reference

* [TensorFlow/TensorRT Models on Jetson TX2](https://jkjung-avt.github.io/tf-trt-models/)
* [Training a Hand Detector with TensorFlow Object Detection API](https://jkjung-avt.github.io/hand-detection-tutorial/)
* [Deploying the Hand Detector onto Jetson TX2](https://jkjung-avt.github.io/hand-detection-on-tx2/)
* [TensorFlow/TensorRT (TF-TRT) Revisited](https://jkjung-avt.github.io/tf-trt-revisited/)
* [TensorFlow object detection and image classification accelerated for NVIDIA Jetson](https://devtalk.nvidia.com/default/topic/1037019/jetson-tx2/tensorflow-object-detection-and-image-classification-accelerated-for-nvidia-jetson/post/5288250/#5288250)

# Prerequisite

* Set up software development environment on the Jetson Nano DevKit.  Reference: [Setting up Jetson Nano: The Basics](https://jkjung-avt.github.io/setting-up-nano/)

* Build and install opencv on the Jetson Nano.  Reference: [Installing OpenCV 3.4.6 on Jetson Nano](https://jkjung-avt.github.io/opencv-on-nano/)

* **Important:** Install 'cpp_implementation of protobuf-3.6.1 for python3', so as to resolve the extremely long TF-TRT model loading time problem.  Reference: [TensorFlow/TensorRT (TF-TRT) Revisted](https://jkjung-avt.github.io/tf-trt-revisited/)

* Install a proper version of tensorflow.  I'm using tensorflow-1.12.2.  Reference: [Building TensorFlow 1.12.2 on Jetson Nano](https://jkjung-avt.github.io/build-tensorflow-1.12.2/)

* If you'd also like to test the hand (egohands) detection models, you'd need to train those models by following my [Training a Hand Detector with TensorFlow Object Detection API](https://jkjung-avt.github.io/hand-detection-tutorial/) post.

# How to test

I'm only highlighting the major steps here.  Please refer to my earlier posts as listed in the 'Reference' section for more details.

1. Clone my 'tf_trt_models' code and run the `install.sh`.

   ```shell
   $ cd ${HOME}/project
   $ git clone --recursive https://github.com/jkjung-avt/tf_trt_models
   $ cd tf_trt_models
   $ ./install.sh
   ```

2. Set `MEASURE_MODEL_TIME = True` in the source code: [utils/od_utils.py](https://github.com/jkjung-avt/tf_trt_models/blob/master/utils/od_utils.py#L15)

3. Copy over the trained 'egohands' models.  For example, I copied the tensorflow model checkpoint files from my training PC to Jetson Nano.  (Replace account name and IP address with your own settings.)

   ```shell
   $ scp jkjung@10.1.2.3:/home/jkjung/project/hand-detection-tutorial/ssd_mobilenet_v1_egohands/model.ckpt-20000.* data/ssd_mobilenet_v1_egohands/ 
   $ scp jkjung@10.1.2.3:/home/jkjung/project/hand-detection-tutorial/ssdinception_v2_egohands/model.ckpt-20000.* data/ssd_inception_v2_egohands/ 
   $ scp jkjung@10.1.2.3:/home/jkjung/project/hand-detection-tutorial/ssd_mobilenet_v2_egohands/model.ckpt-20000.* data/ssd_mobilenet_v2_egohands/ 
   $ scp jkjung@10.1.2.3:/home/jkjung/project/hand-detection-tutorial/ssdlite_mobilenet_v2_egohands/model.ckpt-20000.* data/ssdlite_mobilenet_v2_egohands/ 
   $ scp jkjung@10.1.2.3:/home/jkjung/project/hand-detection-tutorial/rfcn_resnet101_egohands/model.ckpt-50000.* data/rfcn_resnet101_egohands/ 
   $ scp jkjung@10.1.2.3:/home/jkjung/project/hand-detection-tutorial/faster_rcnn_resnet50_egohands/model.ckpt-50000.* data/faster_rcnn_resnet50_egohands/ 
   $ scp jkjung@10.1.2.3:/home/jkjung/project/hand-detection-tutorial/faster_rcnn_resnet101_egohands/model.ckpt-50000.* data/faster_rcnn_resnet101_egohands/ 
   $ scp jkjung@10.1.2.3:/home/jkjung/project/hand-detection-tutorial/faster_rcnn_inception_v2_egohands/model.ckpt-50000.* data/faster_rcnn_inception_v2_egohands/ 
   ```

4. Set Jetson Nano to 10W mode before testing.

   ```shell
   $ sudo nvpmodel -m 0
   $ sudo jetson_clocks
   ```

5. For each of the models, repeat the 'build' and 'test' steps as below.  Record the numbers in the 'tf_sess.run() took XXX ms on average' messages.  (Hit ESC to break out of the `camera_tf_trt.py` script.)

   It's a good idea to run `sudo tegrastats` in another terminal while testing.  You could monitor RAM occupancy while the TF-TRT model is being built, loaded and run.

   ```shell
   $ python3 camera_tf_trt.py --model ssd_mobilenet_v1_coco \
                              --image \
                              --filename examples/detection/data/huskies.jpg \
                              --build
   $ python3 camera_tf_trt.py --model ssd_mobilenet_v1_coco \
                              --image \
                              --filename examples/detection/data/huskies.jpg
   ......
   $ python3 camera_tf_trt.py --model ssd_mobilenet_v1_egohands \
                              --image \
                              --filename jk-son-hands.jpg \
                              --labelmap data/egohands_label_map.pbtxt \
                              --num-classes 1 \
                              --build
   $ python3 camera_tf_trt.py --model ssd_mobilenet_v1_egohands \
                              --image \
                              --filename jk-son-hands.jpg \
                              --labelmap data/egohands_label_map.pbtxt \
                              --num-classes 1
   ### repeat the same steps for the other models...
   ```

# Results

Please note that the results might depend a lot on the software environment.  For example, if you're using a different version of tensorflow, you could get different measurements from mine.

I summarize my test results in the table below.  The 'TX2' numbers are from my previous test results done on a Jetson TX2 with JetPack-3.3 and tensorlow-1.12.0.  And the 'Nano' numbers are done on my Jetson Nano DevKit with JetPack-4.2 and tensorflow-1.12.2.

The results are somewhat disappointing since the larger TF-TRT object detection models simply do not work on Jetson Nano...

'OOM': Out of Memory.

'SegFault": Segmentation Fault, which was also caused by system running out of memory.  

|                                   |  # class  |    TX2    |   Nano   |
| :-------------------------------- | :-------: | :-------: | :------: |
| ssd_mobilenet_v1_coco             |     90    |  43.5 ms  |  62.0 ms |
| ssd_inception_v2_coco             |     90    |  45.9 ms  |    OOM   |
| ssd_mobilenet_v1_egohands         |      1    |  24.5 ms  |  48.6 ms |
| ssd_inception_v2_egohands         |      1    |  25.9 ms  |  55.9 ms |
| ssd_mobilenet_v2_egohands         |      1    |  28.7 ms  |  58.0 ms |
| ssdlite_inception_v2_egohands     |      1    |  28.9 ms  |    OOM   |
| rfcn_resnet101_egohands           |      1    |   351 ms  | SegFault |
| faster_rcnn_resnet50_egohands     |      1    |   226 ms  | SegFault |
| faster_rcnn_resnet101_egohands    |      1    |   317 ms  | SegFault |
| faster_rcnn_inception_v2_egohands |      1    |   117 ms  | SegFault |
