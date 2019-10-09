---
layout: post
comments: true
title: "Building TensorFlow 2.0.0 on Jetson Nano"
excerpt: "Check out my latest script for building and installing tensorflow-2.0.0 on Jetson Nano and other Jetson platforms."
date: 2019-10-07
category: "tensorflow"
tags: tensorrt tensorflow
---

Quick link: [jkjung-avt/jetson_nano](https://github.com/jkjung-avt/jetson_nano)

The TensorFlow team officially released version 2.0 a week ago.  I wanted to test drive it on Jetson Nano.  So I modified my previous scripts and built/installed tensorflow-2.0.0 for testing.  As always, I shared the script on GitHub.


# Reference

* [How I built TensorFlow 1.8.0 on Jetson TX2](https://jkjung-avt.github.io/build-tensorflow-1.8.0/)

* [Building TensorFlow 1.12.2 on Jetson Nano](https://jkjung-avt.github.io/build-tensorflow-1.12.2/)

# Prerequisite

Please refer to my previous TensorFlow 1.12.2 post.  The prerequisite is the same.

# Step-by-Step

1. Uninstall tensorboard and tensorflow if a previous version has been installed.

   ```shell
   $ sudo pip3 uninstall -y tensorboard tensorflow
   ```

   It is also a good idea to clean up 'libprotobuf', the python 'protobuf' module and 'bazel' if you have installed some older versions previously.

   ```shell
   $ sudo rm /usr/local/lib/libproto*
   $ sudo pip3 uninstall -y protobuf
   $ sudo rm /usr/local/bin/bazel
   ```

2. Clone my 'jetson_nano' repository from GitHub, which contains all the scripts.

   ```shell
   $ cd ${HOME}/project
   $ git clone https://github.com/jkjung-avt/jetson_nano.git
   $ cd jetson_nano
   ```

3. Update 'libprotobuf' (3.8.0).  [This solves the "extremely long model loading time problem" of TF-TRT](https://jkjung-avt.github.io/tf-trt-revisited/).

   ```shell
   $ ./install_protobuf-3.8.0.sh
   ```

   This script takes 1 hour or so to finish on the Jetson Nano.

   If you do not care about the TF-TRT problem and you do not want to compile libprotobuf by yourself, you might simply do:

   ```shell
   ### Alternative, but not recommended
   $ sudo pip3 install protobuf==3.8.0
   ```

4. Install 'bazel' (0.26.1), the build tool for tensorflow.

   ```shell
   $ ./install_bazel-0.26.1.sh
   ```
   
5. Build and install tensorflow-2.0.0 by executing the following script.  More specifically, this script would install requirements, download tensorflow-2.0.0 source, configure/build the code, build the pip3 wheel and install it on the system.

   ```shell
   $ ./install_tensorflow-2.0.0.sh
   ```

   Note this script would take a very long time (>12 hours) to run.  Since bulding tensorflow requires a lot resources (memory & disk I/O), it is suggested all other applications (such as the web browser) and tasks terminated while tensorflow is being built.

   When the script finishes successfully, you'd see the following output:

   ```shell
   ......
   tensorflow version: 2.0.0
   tensorflow.test.is_built_with_cuda(): True
   tensorflow.test.is_gpu_available(): True
   ```

# Testing the installation

* The API changed quite a bit in tensorflow-2.0.0 from 1.x.  For example, there is no longer `placeholder()` and `feed_dict()`, and all the `contrib.*` stuffs are gone.  Most of the libraries relying on tensorflow have not been updated for 2.x API.  This makes testing a little bit difficult.

  So I referenced the official [TensorFlow 2 quickstart for experts](https://www.tensorflow.org/tutorials/quickstart/advanced) article and created a script to train/test a MNIST model using the 2.x API.  (I reduced batch size from 32 to 16 in the code, otherwise the program seemed to hit RAM limit frequently.)

  ```shell
  $ cd ${HOME}/project/jetson_nano/tensorflow
  $ python3 mnist.py
  Fetching MNIST dataset...
  Start training...
  Epoch 1, Loss: 0.12405917793512344, Accuracy: 96.29833221435547, Test Loss: 0.05720750242471695, Test Accuracy: 98.16999816894531
  Epoch 2, Loss: 0.03967945650219917, Accuracy: 98.74333190917969, Test Loss: 0.06687281280755997, Test Accuracy: 98.04999542236328
  Epoch 3, Loss: 0.020061999559402466, Accuracy: 99.34832763671875, Test Loss: 0.05041925981640816, Test Accuracy: 98.48999786376953
  Epoch 4, Loss: 0.011295714415609837, Accuracy: 99.61000061035156, Test Loss: 0.07434192299842834, Test Accuracy: 98.1199951171875
  Epoch 5, Loss: 0.007720560301095247, Accuracy: 99.7550048828125, Test Loss: 0.07604508101940155, Test Accuracy: 98.18000030517578
  MNIST training done.
  ```

  The resulting test accuracy (~98%) matched what was stated in the original article.

# Additional notes

* I chose 'protobuf' version "3.8.0" since it is [the matching version](https://github.com/tensorflow/tensorflow/blob/r2.0/tensorflow/workspace.bzl#L420) in tensorflow-2.0 source code.

* I chose 'bazel' version "0.26.1" for tensorflow-2.0.0 based on tensorflow's official documentation: [Tested build configurations](https://www.tensorflow.org/install/source#tested_build_configurations).

* My previous notes about `--local_resources`, environment variable settings, `TF_CUDA_COMPUTE_CAPABILITIES` and `TF_TENSORRT_VERSION`, etc. still apply.  And I think my scripts would likely also work for Jetson TX1/TX2 and AGX Xavier.  Please refer to my TensorFlow 1.12.2 post for details.

I haven't done too much testing on this newly built tensorflow-2.0.0, so I'm not yet confident it's without issues.  Feel free to give it a try and report problems to me.  I will try my best to find a solution and update the scripts if necessary.
