---
layout: post
comments: true
title: "How I built TensorFlow 1.8.0 on Jetson TX2"
excerpt: "This is a note for myself, documenting the nifty details about how I built tensorflow 1.8.0 pip wheel with TensorRT support on a Jetson TX2 flashed with JetPack-3.3."
date: 2018-09-20
category: "tensorflow"
tags: tensorrt tensorflow
---

I was testing TensorFlow/TensorRT (TF-TRT) models on Jetson TX2 and found the pre-built 1.9.0, 1.10.0, and 1.10.1 wheels provided by NVIDIA did not work too well (Reference: [TensorFlow/TensorRT Models on Jetson TX2](https://jkjung-avt.github.io/tf-trt-models/)).  My main TX2 sofwtare development environment was based on JetPack-3.3 (TensorRT 4.0 GA), and I didn't find a pre-built TensorFlow 1.8.0 pip wheel for it.  Thus, I decided to build TensorFlow 1.8.0 by myself.

Although there were already quite a few guides on the internet about how to build TensorFlow on Jetson TX2, I still found it tricky to actualy build one for a recent release.  One obstacle was the versioning inter-dependency between bazel and TensorFlow.  The other nifty details lied in the `configure` and `bazel build` commands.

Anyway, I managed to build and verify TensorFlow 1.8.0 on my Jetson TX2 flashed with JetPack-3.3. Here is how I built the python3 tensorflow-1.8.0 pip wheel.

# Steps

1. Uninstall whatever tensorflow version that has previously been installed on the system, otherwise the `bazel build ...` process in step #10 might get messed up and fail towards the end.

   ```shell
   sudo pip3 uninstall tensorboard tensorflow*
   ```

2. Start with an environment flashed with JetPack-3.3:
   * CUDA 9.0
   * cuDNN 7.1.5
   * TensorRT 4.0 GA (4.1.3?)

3. Set the following environment variable.

   ```shell
   $ export LD_LIBRARY_PATH=/usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH
   ```

4. Get java.

   ```shell
   $ sudo apt-get install openjdk-8-jdk
   ```

5. Assuming pyhton3-dev, pip3, wheel and numpy are present already, install these additional pip3 packages.

   ```
   $ sudo pip3 install six mock h5py enum34
   ```

6. Get bazel.  I tested the latest version (0.17.1) of bazel and it was no good.  So I downloaded and used **bazel 0.15.2** instead.

   ```
   $ cd ~/Downloads
   $ wget https://github.com/bazelbuild/bazel/releases/download/0.15.2/bazel-0.15.2-dist.zip
   $ mkdir -p ~/src
   $ cd ~/src
   $ unzip ~/Downloads/bazel-0.15.2-dist.zip -d bazel-0.15.2-dist
   $ cd bazel-0.15.2-dist
   $ ./compile.sh
   $ sudo cp output/bazel /usr/local/bin
   $ bazel help
   ```

7. Download tensorflow-1.8.0 source code.

   ```
   $ cd ~/Downloads
   $ wget https://github.com/tensorflow/tensorflow/archive/v1.8.0.tar.gz -O tensorflow-1.8.0.tar.gz
   $ cd ~/src
   $ tar xzvf ~/Downloads/tensorflow-1.8.0.tar.gz
   ``` 

8. Apply [this patch](https://github.com/peterlee0127/tensorflow-nvJetson/blob/master/patch/tensorflow1.8.patch) to tensorflow-1.8.0 source code (`~/src/tensorflow-1.8.0/third_party/png.BUILD`).

   ```
   diff --git a/third_party/png.BUILD b/third_party/png.BUILD
   index 76ab32d..bc4551a 100644
   --- a/third_party/png.BUILD
   +++ b/third_party/png.BUILD
   @@ -35,6 +35,7 @@ cc_library(
        ],
        includes = ["."],
        linkopts = ["-lm"],
   +    copts = ["-DPNG_ARM_NEON_OPT=0"],
        visibility = ["//visibility:public"],
        deps = ["@zlib_archive//:zlib"],
    )
   ```

9. Configure tensorflow.  Note that I've disabled cloud platform (GCP, AWS, etc.) support stuffs to save build time and make the pip package smaller.  And do pay attention to the library paths for cuDNN and TensorRT (`/usr/lib/aarch64-linux-gnu`).

   ```shell
   $ cd ~/src/tensorflow-1.8.0
   $ ./configure
   WARNING: ignoring http_proxy in environment.
   WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
   You have bazel 0.15.2- (@non-git) installed.
   Please specify the location of python. [Default is /usr/bin/python]: /usr/bin/python3
   
   
   Found possible Python library paths:
     /usr/local/lib/python3.5/dist-packages
     /usr/lib/python3/dist-packages
   Please input the desired Python library path to use.  Default is [/usr/local/lib/python3.5/dist-packages]
   
   Do you wish to build TensorFlow with jemalloc as malloc support? [Y/n]: 
   jemalloc as malloc support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with Google Cloud Platform support? [Y/n]: n
   No Google Cloud Platform support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with Hadoop File System support? [Y/n]: n
   No Hadoop File System support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with Amazon S3 File System support? [Y/n]: n
   No Amazon S3 File System support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with Apache Kafka Platform support? [Y/n]: n
   No Apache Kafka Platform support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with XLA JIT support? [y/N]: 
   No XLA JIT support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with GDR support? [y/N]: 
   No GDR support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with VERBS support? [y/N]: 
   No VERBS support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: 
   No OpenCL SYCL support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with CUDA support? [y/N]: y
   CUDA support will be enabled for TensorFlow.
   
   Please specify the CUDA SDK version you want to use, e.g. 7.0. [Leave empty to default to CUDA 9.0]: 
   
   
   Please specify the location where CUDA 9.0 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: 
   
   
   Please specify the cuDNN version you want to use. [Leave empty to default to cuDNN 7.0]: 7.1.5
   
   Please specify the location where cuDNN 7 library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:/usr/lib/aarch64-linux-gnu
   
   
   Do you wish to build TensorFlow with TensorRT support? [y/N]: y
   TensorRT support will be enabled for TensorFlow.
   
   Please specify the location where TensorRT is installed. [Default is /usr/lib/aarch64-linux-gnu]:
   
   
   Please specify the NCCL version you want to use. [Leave empty to default to NCCL 1.3]: 
   
   
   Please specify a list of comma-separated Cuda compute capabilities you want to build with.
   You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
   Please note that each additional compute capability significantly increases your build time and binary size. [Default is: 3.5,5.2]6.2
   
   
   Do you want to use clang as CUDA compiler? [y/N]: 
   nvcc will be used as CUDA compiler.
   
   Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]: 
   
   
   Do you wish to build TensorFlow with MPI support? [y/N]: 
   No MPI support will be enabled for TensorFlow.
   
   Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]: 
   
   
   Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: 
   Not configuring the WORKSPACE for Android builds.
   
   Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See tools/bazel.rc for more details.
   	--config=mkl         	# Build with MKL support.
   	--config=monolithic  	# Config for mostly static monolithic build.
   Configuration finished
   ```

10. Build tensorflow.  Note that I did not create a swap space during the build process.  Instead, I used the `--local_resources` flag to limit the amount of memory/CPU resources used by bazel.  Even though I'd close the web browser and all other applications (to make sure bazel has maximum amount of memory at its disposal) during the build process, bazel still stopped with 'internal compiler errors' or got 'killed' sometimes.  When that happened, I'd just repeat the `bazel build ...` command which would resume the build process.  (If the error persited, I'd further reduce memory limit in the `--local_resources` option.)  This `bazel build ...` process would take 4~5 hours on Jetson TX2.  So be patient...

    ```shell
    $ bazel build --config=opt --config=cuda --local_resources 8192,2.0,1.0  //tensorflow/tools/pip_package:build_pip_package
    ```

11. Make the pip wheel file.

    ```
    $ bazel-bin/tensorflow/tools/pip_package/build_pip_package wheel/tensorflow_pkg
    ```

12. Install tensorflow with pip3.

    ```
    $ sudo pip3 install ~/src/tensorflow-1.8.0/wheel/tensorflow_pkg/tensorflow-1.8.0-cp35-cp35m-linux_aarch64.whl
    ```

# Reference

1. [Install TensorFlow from Sources](https://www.tensorflow.org/install/install_sources)

2. [peterlee0127/tensorflow-nvJetson](https://github.com/peterlee0127/tensorflow-nvJetson).

# Download link

I've shared the tensorflow-1.8.0 pip wheel I built on Google Drive. You could download it from [here](https://drive.google.com/file/d/1bAUNe26fKgGXuJiZYs1eT2ig8SCj2gW-/view).
