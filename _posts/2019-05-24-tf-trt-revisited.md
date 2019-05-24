---
layout: post
comments: true
title: "TensorFlow/TensorRT (TF-TRT) Revisted"
excerpt: "I have a solution to the 'extremely long model loading time problem' of TF-TRT now!"
date: 2019-05-24
category: "tx2"
tags: tx2 tensorrt tensorflow
---

Quick links:
1. [install_protobuf-3.6.0.sh](https://github.com/jkjung-avt/jetson_nano/blob/master/install_protobuf-3.6.0.sh)
2. [jkjung-avt/tf_trt_models](https://github.com/jkjung-avt/tf_trt_models)

When I first tried out TensorRT integration in TensorFlow (TF-TRT) a few months ago, I encountered this "extremely long model loading time problem" with tensorflow versions 1.9, 1.10, 1.11 and 1.12.  This problem has since been reported multiple times on NVIDIA Developer Forum: for example, [here](https://devtalk.nvidia.com/default/topic/1037019/jetson-tx2/tensorflow-object-detection-and-image-classification-accelerated-for-nvidia-jetson/post/5281960/#5281960), [here](https://devtalk.nvidia.com/default/topic/1046492/tensorrt/extremely-long-time-to-load-trt-optimized-frozen-tf-graphs/), and [here](https://devtalk.nvidia.com/default/topic/1051546/optimizing-tf-trt-load-time/).  As a result, I was forced to use an older version of tensorflow which could suffer from incompatibility with models trained with newer version of tensorflow...

Thanks to [Dariusz](https://jkjung-avt.github.io/tf-trt-models/#comment-4290171498), one of the readers, I now have a solution to the problem.

# Reference

* [Extremely long time to load TRT-optimized frozen TF graphs](https://devtalk.nvidia.com/default/topic/1046492/tensorrt/extremely-long-time-to-load-trt-optimized-frozen-tf-graphs/)
* [Comments on Disqus about my previous TF-TRT post](https://jkjung-avt.github.io/tf-trt-models/#comment-4290171498)
* [TensorFlow/TensorRT Models on Jetson TX2](https://jkjung-avt.github.io/tf-trt-models/)

# How to fix the "extremely long model loading time problem of TF-TRT"

Please check out the reference links above for more details about the problem and how a solution was found.  In short, I think root cause of the problem is: *the default 'python implementation' of python3 'protobuf' module runs too slowly on the Jetson platforms*.  And the solution is simply to *replace it with 'cpp implementaion' of that same module*.

For that, I've written a script to automate all steps needed to update python3 'protobuf' module on your Jetson device (TX2 or else).  Just download and run the script.  Note that I picked version '3.6.0' of protobuf, because that's [the matching version](https://github.com/tensorflow/tensorflow/blob/r1.12/tensorflow/workspace.bzl#L383) in tensorflow-1.12 source code.  (Excuse me for putting this script in my 'jetson_nano' repository.  It does work for Jetson TX2.  It should work on other Jetson platforms too.)

Note the script would download and unzip protobuf-3.6.0 into `${HOME}/src/` directory.

```shell
$ wget https://raw.githubusercontent.com/jkjung-avt/jetson_nano/master/install_protobuf-3.6.0.sh
$ ./install_protobuf-3.6.0.sh
```

The script would take a while to finish.  When done, you could do `pip3 show protobuf` to verify its version number is `3.6.0`.

Then, you could use either my [jkjung-avt/tf_trt_models](https://github.com/jkjung-avt/tf_trt_models) repository or NVIDIA's original tf_trt_models code to verify the result.  Refer to [my earlier post](https://jkjung-avt.github.io/tf-trt-models/) for details.

I did the following on my Jetson platforms.  With the solution applied, **the optimized trt pb file (e.g. ssd_mobilenet_v1_coco.pb) loads in just a matter of seconds**.

```shell
$ cd ${HOME}/project/tf_trt_models
### Need to build the trt pb the 1st time around
$ python3 camera_tf_trt.py --model ssd_mobilenet_v1_coco \
                           --image \
                           --filename examples/detection/data/huskies.jpg \
                           --build
### Hit ESC to break out of the program.  Then run with the optimized
### model directly (without `--build`) the 2nd time.  Observe how soon
### it takes to load the optimized ssd_mobilenet_v1_coco.pb model.
$ python3 camera_tf_trt.py --model ssd_mobilenet_v1_coco \
                           --image \
                           --filename examples/detection/data/huskies.jpg
```

For the record, I've verified this solution with the following 2 platforms.

* tensorflow-1.12.0 on a Jetson TX2 with JetPack-3.3.
* tensorflow-1.12.2 on a Jetson Nano with JetPack-4.2.

# Discussions

Here are some additional notes about the problem and the solution.

1. Again, my guess about root cause of the problem is "inefficiency of python implementation of the protobuf module".  With the python code, it just takes very long to deserialize the trt pb file.  My solution is to use "C++ implementation (with Cython wrapper)" of the python3 protobuf module.

2. Ubuntu Linux does provide pre-built libprotobuf as apt packages.  But they are rather old versions ('2.6.1' in Ubuntu 16.04 or '3.0.0' in Ubuntu 18.04).  You could verify the apt-installed libprotobuf by `apt --installed list | grep libprotobuf` or `which protoc; protoc --version`.  I chose libprotobuf-3.6.0 for the sake of compatibility with tensorflow-1.12.x.

3. Some other important apt packages in Ubuntu have dependencies on libprotobuf (the older version which gets installed by `sudo apt-get install`).  It is not easy to remove them all.  My approach is to keep the older libprotobuf apt package installed on the system, while installing the newly built libprotobuf-3.6.0 into `/use/local/lib`.  This way, the new libprotobuf would take precedence over the apt one when I build new code.  And I would not break dependencies for the other apt stuffs either.  (*I haven't encountered any problem so far with this approach.  But do be mindful that this might cause problems for things related to libprotobuf...*)

4. Normally opencv libraries (when using the default GTK+ backend) would be linked to libprotobuf too.  That means opencv libraries might be broken if we switch version of libprotobuf.  Since I have [built my opencv-3.4.6 without dependencies on libprotobuf](https://jkjung-avt.github.io/opencv-on-nano/), I don't suffer from such a problem.

5. If your python3 protobuf module gets overwritten by another version by accident (for example, if you do `sudo pip3 install -U protobuf` unintentionally), you could install the "3.6.0 cpp implementation" back by doing `setup.py install` in the `protobuf-3.6.0` source directory again.

   ```shell
   $ cd ${HOME}/src/protobuf-3.6.0
   $ sudo python3 setup.py install --cpp_implementation
   ```

Finally, I'd like to thank Dariusz once again for sharing his finding.  This is a valuable solution and should benefit a lot of our fellow developers in the NVIDIA Jetson community.
