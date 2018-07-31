---
layout: post
comments: true
title: "How to Do Real-time Object Detection with SSD on Jetson TX2"
excerpt: "In this post, I'm demonstrating how to do real-time object detection with Single-Shot Multibox Detector (SSD) on Jetson TX2. Note that SSD runs much faster than my previous similar example of Faster R-CNN."
date: 2018-07-30
category: "object detection"
tags: opencv caffe SSD
---

Quick link: [camera-ssd-threaded.py](https://gist.github.com/jkjung-avt/605904dc05691e44a26bc57bb50d3f04)

## Prerequisite:

* Refer to my previous post ["Multi-threaded Camera Caffe Inferencing"](https://jkjung-avt.github.io/camera-caffe-threaded/). But install all dependencies for **python2** instead of python3. Make sure `tegra-cam-threaded.py` runs OK with **python2** on the Jetson TX2.
* Refer to my previous post ["Single Shot MultiBox Detector (SSD) on Jetson TX2"](https://jkjung-avt.github.io/ssd/). Build SSD Caffe and make sure `./examples/ssd/ssd_pascal_webcam.py` runs OK on the Jetson TX2. I assume SSD Caffe has been downloaded and built at `/home/nvidia/project/ssd-caffe`.
* Download the pre-trained SSD model for testing. By default, my `camera-ssd-threaded.py` script would use the 'COCO' SSD300 model. The download link of  this model could be found at the bottom of the [SSD GitHub page](https://github.com/weiliu89/caffe/tree/ssd). Just follow the `SSD300` link under `2. COCO models:`. I made a copy of the link [here](https://drive.google.com/file/d/0BzKzrI_SkD1_dUY1Ml9GRTFpUWc/view). Go ahead and download `models_VGGNet_coco_SSD_300x300.tar.gz` from there, and untar the file into the SSD Caffe folder.

```shell
$ cd /home/nvidia/project/ssd-caffe
$ tar xzvf ~/Downloads/models_VGGNet_coco_SSD_300x300.tar.gz
### Verify the prototxt and caffemodel files are present
$ ls -l models/VGGNet/coco/SSD_300x300/
```

## How to run the camera SSD sample code:

* Download the `camera-ssd-threaded.py` source code from my GitHubGist: [https://gist.github.com/jkjung-avt/605904dc05691e44a26bc57bb50d3f04](https://gist.github.com/jkjung-avt/605904dc05691e44a26bc57bb50d3f04)
* To dump help messages:

  ```shell
  $ python camera-ssd-threaded.py --help
  usage: camera-ssd-threaded.py [-h] [--rtsp] [--uri RTSP_URI]
                                [--latency RTSP_LATENCY] [--usb]
                                [--vid VIDEO_DEV] [--width IMAGE_WIDTH]
                                [--height IMAGE_HEIGHT] [--cpu]
                                [--prototxt CAFFE_PROTOTXT]
                                [--model CAFFE_MODEL] [--labelmap LABELMAP_FILE]
                                [--confidence CONF_TH]
  
  This script captures and displays live camera video, and does real-time object
  detection with Single-Shot Multibox Detector (SSD) in Caffe on Jetson TX2/TX1.
  
  optional arguments:
    -h, --help            show this help message and exit
    --rtsp                use IP CAM (remember to also set --uri)
    --uri RTSP_URI        RTSP URI, e.g. rtsp://192.168.1.64:554
    --latency RTSP_LATENCY
                          latency in ms for RTSP [200]
    --usb                 use USB webcam (remember to also set --vid)
    --vid VIDEO_DEV       device # of USB webcam (/dev/video?) [1]
    --width IMAGE_WIDTH   image width [1280]
    --height IMAGE_HEIGHT
                          image height [720]
    --cpu                 run Caffe in CPU mode (default: GPU mode)
    --prototxt CAFFE_PROTOTXT
                          [/home/nvidia/project/ssd-
                          caffe/models/VGGNet/coco/SSD_300x300/deploy.prototxt]
    --model CAFFE_MODEL   [/home/nvidia/project/ssd-caffe/models/VGGNet/coco/SSD
                          _300x300/VGG_coco_SSD_300x300_iter_400000.caffemodel]
    --labelmap LABELMAP_FILE
                          [/home/nvidia/project/ssd-
                          caffe/data/coco/labelmap_coco.prototxt]
    --confidence CONF_TH  confidence threshold [0.3]
  ```

* To do real-time object detection with the default COCO SSD model, using the Jetson onboard camera (default behavior of the python script), do the following. According to my own testing, it takes ~180ms for SSD to process each image frame on JTX2 this way. That equates to **5~6 fps**.

  ```shell
  $ python camera-ssd-threaded.py
  ```

* To use USB webcam `/dev/video1` instead, while setting video resolution to 1920x1080:

  ```shell
  $ python camera-ssd-threaded.py --usb --vid 1 --width 1920 --height 1080
  ```

* Or, to use an IP CAM:

  ```shell
  $ python camera-ssd-threaded.py --rtsp --uri rtsp://admin:XXXXXX@192.168.1.64:554
  ```

* We could also do real-time object detection with a different Caffe model. For example, download and untar the pre-trained VOC0712Plus SSD300 model from [here](https://drive.google.com/file/d/0BzKzrI_SkD1_TkFPTEQ1Z091SUE/view). And execute the following:

  ```
  $ python ./camera-ssd-threaded.py --usb \
                                    --prototxt /home/nvidia/project/ssd-caffe/models/VGGNet/VOC0712Plus/SSD_300x300_ft/deploy.prototxt
                                    --model /home/nvidia/project/ssd-caffe/models/VGGNet/VOC0712Plus/SSD_300x300_ft/VGG_VOC0712Plus_SSD_300x300_ft_iter_160000.caffemodel
                                    --labelmap /home/nvidia/project/ssd-caffe/data/VOC0712/labelmap_voc.prototxt
  ```

## Additional Notes:

* PIXEL_MEAN ([B, G, R] = [104.0, 117.0, 123.0]) has been hardcoded in the python script.
* Input image size (300, 300) has also been hardcoded. Just modify the [`preprocess()`](https://gist.github.com/jkjung-avt/605904dc05691e44a26bc57bb50d3f04#file-camera-ssd-threaded-py-L142) function if a SSD512 (or other sizes) is used.
* You can adjust `--confidence` (confidence threshold value: 0.0~1.0) depending on whether you prefer better 'precision' (less false positives) or better 'recall' (less missed detections).
* I only tested the code with python3 (with my own modified version ssd-caffe). Feel free to report issues to me if you have trouble running with code with python2 or else.
