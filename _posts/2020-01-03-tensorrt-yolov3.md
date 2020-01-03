---
layout: post
comments: true
title: "TensorRT ONNX YOLOv3"
excerpt: "I created a TensorRT ONNX YOLOv3 demo based on NVIDIA's sample code."
date: 2020-01-03
category: "tensorrt"
tags: tensorrt
---

Quick link: [jkjung-avt/tensorrt_demos](https://github.com/jkjung-avt/tensorrt_demos)

![Dog, bicycle and truck detected](https://raw.githubusercontent.com/jkjung-avt/tensorrt_demos/master/doc/dog_trt_yolov3.png)

I wrote a blog post about [YOLOv3 on Jetson TX2](https://jkjung-avt.github.io/yolov3/) quite a while ago.  As of today, YOLOv3 stays one of the most popular object detection model architectures.  Since NVIDIA already provided an [Object Detection With The ONNX TensorRT Backend In Python (YOLOv3)](https://docs.nvidia.com/deeplearning/sdk/tensorrt-sample-support-guide/index.html#yolov3_onnx) sample code, I just adapted the sample with my 'tensorrt_demos' camera/video input code and created a real-time TensorRT YOLOv3 object detector demo: [Demo #4: YOLOv3](https://github.com/jkjung-avt/tensorrt_demos#yolov3).

# Reference

* Official TensorRT sample: **'yolov3_onnx'**.  The source code (including a README.md) could be found on Jetson platforms at '/usr/src/tensorrt/samples/python/yolov3_onnx'.
* [TensorRT/YoloV3 FAQ](https://elinux.org/TensorRT/YoloV3)
* Official documentation of [TensorRT Python API](https://docs.nvidia.com/deeplearning/sdk/tensorrt-api/python_api/)
* Official documentation of [TensorRT Onnx Parser](https://docs.nvidia.com/deeplearning/sdk/tensorrt-api/python_api/parsers/Onnx/pyOnnx.html)
* Official [PyCUDA documentation](https://documen.tician.de/pycuda/)

# How to Run the Demo

For running the demo on Jetson Nano/TX2, please follow the step-by-step instructions in [Demo #4: YOLOv3](https://github.com/jkjung-avt/tensorrt_demos#ssd).  The steps mainly include: installing requirements, downloading trained YOLOv3 and YOLOv3-Tiny models, converting the downloaded models to ONNX then to TensorRT engines, and running inference with the converted engines.

Note that this demo relies on TensorRT's Python API, which is only available in TensorRT 5.0.x+ on Jetson Nano/TX2.  So you'll have to set up the Jetson Nano/TX2 with **JetPack-4.2+**.  To re-iterate, JetPack-3.x won't cut it.

As already stated in the [README.md](https://github.com/jkjung-avt/tensorrt_demos#yolov3) on my GitHub repo, you'll have to install version '1.4.1' of python3 'onnx' module instead of the latest version.  Otherwise, you'll likely encounter this error: `onnx.onnx_cpp2py_export.checker.ValidationError: Op registered for Upsample is depracted in domain_version of 10`.

In addition, the 'trt_yolov3.py' demo requires the python3 'pycuda' package.  Since `sudo pip3 install pycuda` always failed on my Jetson's, I created this [install_pycuda.sh](https://github.com/jkjung-avt/tensorrt_demos/blob/master/ssd/install_pycuda.sh) to install it from source.

After downloading darknet YOLOv3 and YOLOv3-Tiny models, you could choose one of the 5 supported models for testing: 'yolov3-tiny-288', 'yolov3-tiny-416', 'yolov3-288', 'yolov3-416', and 'yolov3-608'.  I recommend starting with **'yolov3-416'** since it produces roughly the same detection accuracy as the larger 'yolov3-608' but runs faster.

# About 'download_yolov3.py'

The [download_yolov3.py](https://github.com/jkjung-avt/tensorrt_demos/blob/master/yolov3_onnx/download_yolov3.sh) script would download trained YOLOv3 and YOLOv3-Tiny models (i.e. configs and weights) from the original [YOLO: Real-Time Object Detection](https://pjreddie.com/darknet/yolo/) site.  These models are in [darknet](https://github.com/pjreddie/darknet) format and provided by the original author of YOLO/YOLOv2/YOLOv3, Joseph Redmon.  Kudos to Jospeh!

The downloaded YOLOv3 model is for 608x608 image input, while YOLOv3-Tiny for 416x416.  But we could convert them to take different input image sizes by just modifying the `width` and `height` in the .cfg files (NOTE: input image width/height would better be multiples of 32).  I already did that in the 'download_yolov3.sh' script.  You could read the script for details.

# About 'yolov3_to_onnx.py'

First note this quote from the [official TensorRT Release Notes](https://docs.nvidia.com/deeplearning/sdk/tensorrt-archived/tensorrt-700/tensorrt-release-notes/tensorrt-7.html#rel_7-0-0):

> Deprecation of Caffe Parser and UFF Parser - We are deprecating Caffe Parser and UFF Parser in TensorRT 7. They will be tested and functional in the next major release of TensorRT 8, but we plan to remove the support in the subsequent major release. Plan to migrate your workflow to use tf2onnx, keras2onnx or TensorFlow-TensorRT (TF-TRT) for deployment.

So going forward, using ONNX as the intermediate NN model format is definitely the way to go.

My [yolov3_to_onnx.py](https://github.com/jkjung-avt/tensorrt_demos/blob/master/yolov3_onnx/yolov3_to_onnx.py) is largely based on the original 'yolov3_onnx' sample provided by NVIDIA.  NVIDIA's original code needed to be run with 'python2'.  I made necessary modifications so that it could be run with 'python3'.  In addition, I added code to handle different input image sizes (288x288, 416x416, or 608x608) as well as support of 'yolov3-tiny-xxx' models.

# About 'onnx_to_tensorrt.py'

The [onnx_to_tensorrt.py](https://github.com/jkjung-avt/tensorrt_demos/blob/master/yolov3_onnx/onnx_to_tensorrt.py) is pretty straightforward.  It just calls standard TensorRT APIs to optimize the ONNX model to TensorRT engine and then save it to file.

NVIDIA's original sample code builds default (`FP32`) TensorRT engines.  I added the following line of code so I'd be testing `FP16` (less memory consuming and faster) TensorRT engines instead.

```python
    builder.fp16_mode = True
```

# About 'trt_yolov3.py'

My [trt_yolov3.py](https://github.com/jkjung-avt/tensorrt_demos/blob/master/trt_yolov3.py) is very similar to my previous example, [trt_ssd.py](https://github.com/jkjung-avt/tensorrt_demos/blob/master/trt_ssd.py).  I took the 'preprocessing' and 'postprocessing' code from NVIDIA's original 'yolov3_onnx' sample and encapsulated them into the 'TrtYOLOv3' class.  When run, the code would: (1) deserialize/load the TensorRT engine, (2) manage CUDA memory buffers using 'pycuda', (3) preprocess input image, run inference and postprocess YOLOv3 detection output.  You could read [source code](https://github.com/jkjung-avt/tensorrt_demos/blob/master/utils/yolov3.py#L370) for details.

I tested the TensorRT optimized 'yolov3-416' model with the 'dog.jpg' (with a dog, a bicycle and a truck) from the original YOLOv3 web site, and the model successfully detected all 3 target objects as expected.

As stated in [README.md](https://github.com/jkjung-avt/tensorrt_demos#yolov3), I also verified mean average precision (mAP, i.e. detection accuracy) of the optimized YOLOv3 models with COCO 'val2017' data.  The results, e.g. yolov3-416 'mAP @ IoU=0.5:0.95' = 0.373, were good.  However, the 'yolov3-608' and 'yolov3-416' TensorRT engines did run much slowlier than the TensorRT SSD engines in [my previous demo](https://github.com/jkjung-avt/tensorrt_demos#ssd) example.

# YOLOv3-Tiny models

I added some code into NVIDIA's 'yolov3_onnx' sample to make it also support 'yolov3-tiny-xxx' models.  The main differences between the 'tiny' and the normal models are: (1) output layers; (2) 'yolo_masks' and 'yolo_anchors'.  You could check out my git [history](https://github.com/jkjung-avt/tensorrt_demos/commit/7b54fd53d001c9ee79f6f4fd14261850e6fde3ce) to find the exact changes I made in the code to support 'yolov3-tiny-xxx'.

However, when I evaluated mAP of the optimized 'yolov3-tiny-xxx' TensorRT engines, I found they were quite a lot worse (mAP much too low) than the regular 'yolov3-xxx' engines.  That why I said "I'm not sure whether the implementation is correct" in [README.md](https://github.com/jkjung-avt/tensorrt_demos#yolov3).  In case you manage to find the problems in my implementation, please do let me know.

# Thoughts

YOLOv3 (608x608), with 'mAP @ IoU=0.5' = 0.579 as reported by the original author, is a rather accurate object detection model.  However, it does not run fast on Jetson Nano even when optimized by TensorRT.  I think this would limit its applications in edge computing to cases where frames processed per second (FPS) requirement is low...
