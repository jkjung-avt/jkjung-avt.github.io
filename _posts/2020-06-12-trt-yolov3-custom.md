---
layout: post
comments: true
title: "TensorRT YOLOv3 For Custom Trained Models"
excerpt: "I updated the TensorRT ONNX YOLOv3 demo code to better support custom trained models."
date: 2020-06-12
category: "tensorrt"
tags: tensorrt
---

Quick link: [jkjung-avt/tensorrt_demos](https://github.com/jkjung-avt/tensorrt_demos)

Ever since I published the [TensorRT ONNX YOLOv3](https://jkjung-avt.github.io/tensorrt-yolov3/) demo, I received quite a few questions regarding how to adapt the code to custom trained YOLOv3 models.  I figured that I'd update the code to make such requests easier.  So I did it.  Let me explain the relevant parts of the TensorRT YOLOv3 code in this post.

# How to run the updated TensorRT YOLOv3 code

The pre-trained (downloaded) YOLOv3 models are for the [COCO dataset](http://cocodataset.org/#home) and would output 80 categories of objects.  However, a YOLOv3 model trained with custom datatset usuaully has a different number of object categories.  To cope with this, I've modified the TensorRT YOLOv3 code to take "--category_num" as a command-line option.

Based on my original step-by-step guide of [Demo #4: YOLOv3](https://github.com/jkjung-avt/tensorrt_demos#yolov3), you'd need to specify "--category_num" when building TensorRT engine and doing inference with your custom YOLOv3 model.

For example, if your custom trained YOLOv3 model is with only 1 category, you'd do the following.  Note that "--category_num" is used at 2 places.

```
### Installation of "pycuda" and "onnx" (still required) omitted here
$ cd ${HOME}/project/tensorrt_demos/yolov3_onnx
$ ./download_yolov3.sh
$ python3 yolov3_to_onnx.py --model yolov3-416 --category_num 1
$ python3 onnx_to_tensorrt.py --model yolov3-416
$ cd ${HOME}/project/tensorrt_demos
$ python3 trt_yolov3.py --model yolov3-416 \
                        --category_num 1 \
                        --image --filename ${HOME}/Pictures/dog.jpg
```

# YOLOv3 output shapes

For people who want to learn the underlying details of "--category_num" and the related source code, please read on.

First, check out this very nice article which explains the YOLOv3 architecture clearly: [What’s new in YOLO v3?](https://towardsdatascience.com/yolo-v3-object-detection-53fb7d3bfe6b)  Shown below is the picture from the article, courtesy of the author, Ayoosh Kathuria.

![YOLOv3 architecture](/assets/2020-06-12-trt-yolov3-custom/yolov3_architecture.jpg)

In short, the YOLOv3 model outputs detection results in 3 scales.  The outputs are at layers #82, #94 and #106, with spatial dimensions of the original image height/width divided by 32, 16 and 8 respectively.  Furthermore, the number of channels in the outputs is (B * (4 + 1 + C)), where B is "number of anchor boxes per grid", 4 is "number of bounding box regression values", 1 is for "confidence score of objectness", and C is the "number of object categories".  For the COCO dataset with C=80 (object categories) and B=3 (anchor boxes per grid), the number of output channels per grid is thus (3 * (4 + 1 + 80)) = 255.

The above mentioned calculations are already implemented in the TensorRT YOLOv3 code, as shown below:

* When building the TensorRT engine:
    - calculating the number of output channels: [yolov3_onnx/yolov3_to_onnx.py, line #796](https://github.com/jkjung-avt/tensorrt_demos/blob/master/yolov3_onnx/yolov3_to_onnx.py#L796)
    - applying spatial dimension divisors 32, 16 and 8: [yolov3_onnx/yolov3_to_onnx.py, line #802~804](https://github.com/jkjung-avt/tensorrt_demos/blob/master/yolov3_onnx/yolov3_to_onnx.py#L802)
    - specifying layer numbers (082, 094, 106) of the outputs: [yolov3_onnx/yolov3_to_onnx.py, line #802~804](https://github.com/jkjung-avt/tensorrt_demos/blob/master/yolov3_onnx/yolov3_to_onnx.py#L802)

* When doing inference with the TensorRT engine:
    - calculating the number of output channels: [utils/yolov3.py, line #407](https://github.com/jkjung-avt/tensorrt_demos/blob/master/utils/yolov3.py#L407)
    - applying spatial dimension divisors 32, 16 and 8: [utils/yolov3.py, line #412~414](https://github.com/jkjung-avt/tensorrt_demos/blob/master/utils/yolov3.py#L412)

# YOLOv3-Tiny models

Outputs of the YOLOv3-Tiny models are very similar to YOLOv3.  The main differences are just number of output scales (2 vs. 3) and the output layers.  You should be able to find such differences in the source code easily.

Feel free to let me know if some of the descriptions above are not clear enough.  I'll try to amend it accordingly.  Otherwise, thanks for reading.
