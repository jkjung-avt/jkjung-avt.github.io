---
layout: post
comments: true
title: "Setting up Jetson Nano: The Basics"
excerpt: "I finally have time to try out NVIDIA Jetson Nano.  I plan to document all the steps of setting up my Jetson Nano, starting from this post."
date: 2019-05-14
category: "nano"
tags: jetson nano
---

Quick link: [jkjung-avt/jetson_nano](https://github.com/jkjung-avt/jetson_nano)

I've just been too busy at work the past few months.  Today I finally have time to try out my [Jetson Nano Developer Kit](https://developer.nvidia.com/embedded/buy/jetson-nano-devkit).  I think I'd document all steps I apply to set up the software development environment on the Jetson Nano, which could probably save time for people who are new to NVIDIA Jetson platforms.

# Disclaimer

My use cases for Jetson Nano are described below.  In case your use cases differ from mine, you might want to make some adjustments to the set-up steps I describe in the blog posts.  For example, you might want to install a different version of certain SDK/libraries, or you might need to modify the config settings slightly when you build the packages.  Anyway, I believe my posts would be very relevant.  As always, I welcome comments and discussions.  So feel free to leave a comment on Disqus below.

My primary use cases for Jetson Nano (and Jetsonn TX2):

* I use the Jetson platforms to process images.  The images mainly come from camera feeds, but could also be from video/image files.  I do not use the Jetson platforms to process audio, text (NLP) and other kinds of data.
* I use Caffe and TensorFlow (with Keras) primarily.  I almost don't use other deep learning frameworks.  So I don't care much about compatibility issues for, say, PyTorch or MXNet.
* I do use TensorRT to optimize trained Caffe and TensorFlow models.
* I use python3 as the primary programming language.  My set-up works good for python3, but does not necessarily work for python2.
* I use OpenCV mainly for "preprocessing" (image capturing, resizing, color space conversion, etc.) and "postprocessing" (drawing image recognition results) of images.  Sometimes I use other image processing and feature detection functionalities in OpenCV.  But since OpenCV's python API doesn't support CUDA accelerated image processing functions, I care much less about those.
* I reserve the GPU for neural network computations only.  I almost don't use CUDA code for other tasks.
* I use the hardware (H.264/H.265) codecs for video decoding/encoding as much as possible, so that the GPU and CPU could be fully utilized for other tasks.

# Basic Set-up of Jetson Nano

The Jetson Nano DevKit does not come with any eMMC (storage space).  The package does not include a power adapter either.  So you'll have to prepare a microSD card and a power adapter, as well as a set of keyboard/mouse/monitor, by yourself.  Refer to [Getting Started With Jetson Nano Developer Kit](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit) about how to download the image (zip file) and write it to the microSD card.

As of the time of this writing, the official image file for Jetson Nano is "jetson-nano-sd-r32.1-2019-03-18.zip".

I myself use a *DC (5V @ 4A) adapter* (note: not USB) to power the Jetson Nano.  According to the official [User Guide](https://developer.download.nvidia.com/assets/embedded/secure/jetson/Nano/docs/Jetson_Nano_Developer_Kit_User_Guide.pdf?dxoGGzP6MofZCwmovbuSH92tg40Vo_1d7OdnxeDDzicG2hCUW-z1bNlUzM3flKuJJQvWK65Ix1vMTCBU3U0v30tIbRnaBD037LbRpiuKfMgxuhtfJ2ZjDtCdda266rYUD2PTLCzA-8j4TGSWkkeFnu__OnTif_PnGGvy7FXlcY7MJ_5fvBMDKrSynps-u7U), I need to put *a jumper on J48* to be able to power the Jetson Nano DevKit through the J25 power jack.  It took me quite a while to figure that out the first time around.  In addition, when working with the Jetson Nano DevKit, I noticed the board might not reboot properly if I power cycled it too quickly.  My suggestion is to completely *power down the board for at least 5 seconds*, before powering it up again.

In addition to the above-mentioned documents, the following web sites also contain great resources about how to get started with Jetson Nano.  So be sure to check them out as well.

* [https://elinux.org/Jetson_Nano](https://elinux.org/Jetson_Nano)
* [NVIDIA Developer Forum](https://devtalk.nvidia.com/default/board/371/jetson-nano/)

# Basic Set-up of the Software

I'd skip basic set-up of the Ubuntu 18.04 Linux here.  Out of convenience, I just created the account "nvidia" with password "nvidia" on my Jetson Nano.

After the Jetson Nano DevKit boots up, I'd open a termial (Ctrl-Alt-T) and check what software packages are already available on the system.  Much to my delight, I find that CUDA Toolkit 10.0, cuDNN 7 and TensorRT libraries are all readily installed in the microSD image.  That means we could almost jump right in and start building/installing OpenCV and the deep learning frameworks right away.

However, there are a couple of things I'd fix before moving on:

1. CUDA toolkit related paths are not set in the environment variables.  For example, "nvcc" is not in ${PATH} by default.  So I'd fix that in my `~/.bashrc`.  I created a script ([install_basics.sh](https://github.com/jkjung-avt/jetson_nano/blob/master/install_basics.sh)) for doing that.  It would add CUDA stuffs into the `PATH` and `LD_LIBRARY_PATH` variables.

   ```shell
   $ mkdir -p ${HOME}/project
   $ cd ${HOME}/project
   $ git clone https://github.com/jkjung-avt/jetson_nano.git
   $ cd jetson_nano/
   $ ./install_basics.sh
   ```

   At the next log-in, the environment variables should have been set up properly.

2. Since memory (4GB) on the Jetson Nano is rather limited, I'd create and mount a swap file on the system.  I referenced [Create a Linux swap file](https://support.rackspace.com/how-to/create-a-linux-swap-file/) for that.  And I made a 8GB swap file on my Jetson Nano DevKit.  (You could adjust the size based on your own use cases.)

   ```shell
   $ sudo fallocate -l 8G /mnt/8GB.swap
   $ sudo mkswap /mnt/8GB.swap
   $ sudo swapon /mnt/8GB.swap
   ```

   Once the above is working, add the following line into `/etc/fstab` and reboot the system.  Make sure the swap space gets mounted automatically after reboot.

   ```
   /mnt/8GB.swap  none  swap  sw 0  0
   ```

   I think the swap space is necessary for my applications since I'd build/install OpenCV, Caffe, and TensorFlow on the platform.  Anyway, I'd leave those for later posts and end here for now.
