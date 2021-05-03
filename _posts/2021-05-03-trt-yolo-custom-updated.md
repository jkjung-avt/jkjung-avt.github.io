---
layout: post
comments: true
title: "TensorRT YOLO For Custom Trained Models (Updated)"
excerpt: "Once again, I updated my TensorRT YOLO demo code to better support custom trained models."
date: 2021-05-03
category: "tensorrt"
tags: tensorrt
---

Quick link: [jkjung-avt/tensorrt_demos](https://github.com/jkjung-avt/tensorrt_demos)

Time flies.  It's been almost a year since I last wrote the post: [TensorRT YOLOv3 For Custom Trained Models](https://jkjung-avt.github.io/trt-yolov3-custom/).  I myself learned quite a bit since then, largely by replying questions on [jkjung-avt/tensorrt_demos GitHub Issues](https://github.com/jkjung-avt/tensorrt_demos/issues) and through emails.  I applied what I've learned and updated my tensorrt_demos code to better support custom trained DarkNet yolo models yet again.

This is going to be a short blog post about what you need to do optimize and run your own custom DarkNet yolo models with TensorRT, using the latest [jkjung-avt/tensorrt_demos](https://github.com/jkjung-avt/tensorrt_demos) code.

# What's in the "52a699" (2021-05-03) code update

Link: [commit 52a699dac94041b28074179d093be6e5669c8741](https://github.com/jkjung-avt/tensorrt_demos/commit/52a699dac94041b28074179d093be6e5669c8741)

* The updated code can determine input width and height of the yolo models automatically, so users no longer need to put those in model names.  More specifically, "yolo_to_onnx.py" and "onnx_to_tensorrt.py" would use information in the DarkNet cfg file, while "trt_yolo.py" from the TensorRT engine (i.e. dimension of the input binding).

* The updated code can also determine number of object categories automatically, so "yolo_to_onnx.py" and "onnx_to_tensorrt.py" no longer requires the "-c" (or "--category_num") command-line option.

* "yolo_to_onnx.py" doesn't contain [hard-coded output tensor names](https://github.com/jkjung-avt/tensorrt_demos/blob/0016973315d1f3f6eaed70a5abd03d6309fe4730/yolo/yolo_to_onnx.py#L925-L958) anymore.  The updated code just parses the DarkNet cfg file and finds those automatically.

# Naming of the custom trained model

Your custom yolo models could have arbitrary names.  Just make sure the cfg file and weights file match each other.  For example,

* "yolov3-608.cfg" and "yolov3-608.weights"
* "yolov4-tiny-custom.cfg" and "yolov4-tiny-custom.weights"
* "my_awesome_model.cfg" and "my_awesome_model.weights"
* "whateveryoulike.cfg" and "whateveryoulike.weights"
* ......

# How to run the updated TensorRT YOLO code

Although "-c" (or "--category_num") is no longer required for "yolo_to_onnx.py" and "onnx_to_tensorrt.py", it is **still required for "trt_yolo.py"**.

For example, to convert a custom yolov4 model ("yolov4-custom.cfg" and "yolov4-custom.weights") to a TensorRT engine, do:

```
$ cd ${HOME}/project/tensorrt_demos/yolo
$ python3 yolo_to_onnx.py -m yolov4-custom
$ python3 onnx_to_tensorrt.py -m yolov4-custom
```

Then to run the TensorRT "yolov4-custom" engine (assuming "category_num" is 1):

```
$ cd ${HOME}/project/tensorrt_demos
$ python3 trt_yolo.py --image for_testing.jpg -m yolov4-custom -c 1
```

# Conclusion

That's it.

I hope you find my updated TensorRT YOLO code very easy to work with.  And do **star** my [jkjung-avt/tensorrt_demos](https://github.com/jkjung-avt/tensorrt_demos) project if you find it useful.
