---
layout: post
comments: true
title: "Building TensorFlow 1.12.2 on Jetson Nano"
excerpt: "I wrote a script for building and installing tensorflow-1.12.2 on Jetson Nano.  It should work for Jetson TX2 and other Jetson platforms as well."
date: 2019-05-26
category: "tensorflow"
tags: tensorrt tensorflow
---

Quick link: [jkjung-avt/jetson_nano](https://github.com/jkjung-avt/jetson_nano)

I wrote a script for building and installing tensorflow-1.12.2 on Jetson Nano.  It should work for Jetson TX2 and other Jetson platforms (requiring some adjustments if not JetPack-4.2) as well.

# Reference

* [How I built TensorFlow 1.8.0 on Jetson TX2](https://jkjung-avt.github.io/build-tensorflow-1.8.0/)

* [peterlee0127/tensorflow-nvJetson](https://github.com/peterlee0127/tensorflow-nvJetson)

* [Build from source \| TensorFlow](https://www.tensorflow.org/install/source)

# Prerequisite

Setting up a swap file on Jetson Nano is essential, otherwise the tensorflow building process would likely fail due to out-of-memory.  You could refer to my [Setting up Jetson Nano: The Basics](https://jkjung-avt.github.io/setting-up-nano/) post for how to do that.

I'm assuming that "pip3" is already properly installed on the Jetson Nano.  If you've followed my [Installing OpenCV 3.4.6 on Jetson Nano](https://jkjung-avt.github.io/opencv-on-nano/) and built/installed opencv-3.4.6 on the Jetson Nano, then you're good.  Otherwise, you could install pip3 on the system by doing either `wget https://bootstrap.pypa.io/get-pip.py; sudo python3 get-pip.py` or `sudo apt-get install -y python3-pip`.

In case you are building/installing tensorflow on Jetson TX2 or another Jetson platform, it's still a good idea to use some swap.  In addition, you could adjust the `--local_resources` setting (since more RAM is available) in the installation script.

# Step-by-Step

1. Uninstall tensorboard and tensorflow if a previous version has been installed.

   ```shell
   sudo pip3 uninstall -y tensorboard tensorflow
   ```

2. Clone my 'jetson_nano' repository from GitHub, which contains all the scripts.

   ```shell
   $ cd ${HOME}/project
   $ git clone https://github.com/jkjung-avt/jetson_nano.git
   $ cd jetson_nano
   ```

3. (Optional, but highly recommended) Update libprotobuf (3.6.0).  [This solves the "extremely long model loading time problem" of TF-TRT](https://jkjung-avt.github.io/tf-trt-revisited/).

   ```shell
   $ ./install_protobuf-3.6.0.sh
   ```

   This script takes 1 hour or so to finish on the Jetson Nano.

4. Install [bazel](https://docs.bazel.build/versions/0.25.0/bazel-overview.html) (0.15.2), the build tool for tensorflow.

   ```shell
   $ ./install_bazel-0.15.2.sh
   ```
   
5. Build and install tensorflow-1.12.2 by executing the script.  More specifically, this script would install requirements, download tensorflow-1.12.2 source, configure/build the code, build the pip3 wheel and install it on the system.

   ```shell
   $ ./install_tensorflow-1.12.2.sh
   ```

   Note this script would take a very long time (>5 hours) to run.  Since bulding tensorflow requires a lot resources (memory & disk I/O), it is suggested all other applications (such as the web browser) and tasks terminated while tensorflow is being built.

6. Test.  (To be updated...)

# Additional notes

* Thanks to [peterlee0127](https://github.com/peterlee0127/tensorflow-nvJetson) for publishing [tensorflow1.12.patch](https://github.com/peterlee0127/tensorflow-nvJetson/blob/master/patch/tensorflow1.12.patch).

* I chose protobuf version "3.6.0" since that's [the matching version](https://github.com/tensorflow/tensorflow/blob/r1.12/tensorflow/workspace.bzl#L383) in tensorflow-1.12 source code.

* I chose bazel version "0.15.2" for tensorflow-1.12.2 based on tensorflow's official documentation: [Tested build configurations](https://www.tensorflow.org/install/source#tested_build_configurations).

* In case you encounter problem (e.g. out-of-memory or bazel crashing) when running the `install_tensorflow-1.12.2.sh`, you could try to set `--local_resources` ([reference](https://docs.bazel.build/versions/master/user-manual.html)) to lower values.  For example, replace `3072` with `2048` (MB of RAM used by bazel for building code).

* In the `install_tensorflow-1.12.2.sh` script, I enabled GPU, CUDNN and TENSORRT settings, while disabling most of the other features in tensorflow.  This is for reducing size of the compiled tensorflow binary and only enabling functionalities I do use.  You could refer to the environment variable settings (starting from the line `PYTHON_BIN_PATH`) in the script for details.  In case one of those disabled features matters to you, you could turn it on by just setting the environment variable to 1 (instead of 0).

* In the `install_tensorflow-1.12.2.sh` script, I set `TF_CUDA_COMPUTE_CAPABILITIES` to `5.3,6.2,7.2`.  That covers all of Jetson Nano, TX1, TX2 and AGX Xavier.

* In case you are not using JetPack-4.2 (for example, if you are on JetPack-3.3), you likely only need to adjust the following environment variables in the script for it to work: `TF_CUDA_VERSION`, `TF_TENSORRT_VERSION`.
