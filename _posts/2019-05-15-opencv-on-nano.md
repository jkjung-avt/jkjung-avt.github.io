---
layout: post
comments: true
title: "Installing OpenCV 3.4.6 on Jetson Nano"
excerpt: "I created a script for easily building and installing opencv-3.4.6 onto Jetson Nano."
date: 2019-05-15
category: "nano"
tags: jetson nano opencv
---

Quick link: [jkjung-avt/jetson_nano](https://github.com/jkjung-avt/jetson_nano)

As a follow-up on [Setting up Jetson Nano: The Basics](https://jkjung-avt.github.io/setting-up-nano/), my next step of setting up Jetson Nano's software development environment is to build and install OpenCV.  I aggregate all steps of building/installing OpenCV into a shell scripts, so that it could be done very conveniently.

In this post, I'd explain why I choose a certain version and configuration of OpenCV.  Then I'd show how I use the script to build, install and test OpenCV on my Jetson Nano.

I don't want to repeat all information I've written in this previous post: [How to Install OpenCV (3.4.0) on Jetson TX2](https://jkjung-avt.github.io/opencv3-on-tx2/).  So be sure to check that post out if you'd like to know more details about my OpenCV configurations.

# Considerations and Choices

I tried to explain my considerations about picking the version of OpenCV and its configurations below.

1. The pre-installed opencv-3.3.1 on Jetson Nano does not support gstreamer functionalities (cannot utilize hardware H.264/H.265 codec on Jetson Nano).  That's the reason why I need to build and install OpenCV by myself.  I actually purge the pre-installed opencv-3.3.1 packages at the beginning of my `istall_opencv-3.4.6.sh` script.

2. Although NVIDIA provides a [script](https://github.com/AastaNV/JEP/blob/master/script/install_opencv4.0.0_Nano.sh) for installing opencv-4.0.0 on Jetson Nano, I'm sticking with 3.4.x because [Caffe does not build with opencv-4.x](https://github.com/BVLC/caffe/issues/6652).  In my script I use opencv-3.4.6 since that's the latest 3.4.x release as of the time of this writing.

3. **I don't want my opencv-3.4.6 build to have any dependencies on 'protobuf'** for the following reasons.  So I choose to use Qt (with OpenGL) backend instead of the default GTK+ backend.
   * [Double free issue of protobuf (due to protobuf code in opencv)](https://github.com/BVLC/caffe/issues/5282) would cause caffe to crash.
   * [TF-TRT (TensorFlow) might require a newer version of protobuf to work well.](https://devtalk.nvidia.com/default/topic/1046492/tensorrt/extremely-long-time-to-load-trt-optimized-frozen-tf-graphs/)  If I build opencv-3.4.6 with dependencies on the older version (3.0.0) of protobuf in Ubuntu 18.04, I'd likely run into trouble when trying to upgrade protobuf libraries later on.

# Prerequisite

Please go through the steps I described in [Setting up Jetson Nano: The Basics](https://jkjung-avt.github.io/setting-up-nano/).  I'd strongly suggest you to set up a swap file on the Jetson Nano DevKit since its memory is quite limited.

# Building and Installing opencv-3.4.6

Installing opencv-3.4.6 on Jetson Nano using my script is extremely simple.  I'd like to highlight a few things first:

* If you'd like to remove previously built and installed OpenCV libraries, just search all files with names containing 'opencv' or 'cv2' and delete all of them.  You can do such a search by the following command.

  ```shell
  $ find /usr/local -name "*opencv*" -o "*cv2*"
  ```

* If you'd like to modify some configuration and rebuild opencv-3.4.6, it's recommended to remove the whole `build` directory and then redo the `cmake`.  For example,

  ```shell
  $ cd ${HOME}/src/opencv-3.4.6
  $ rm -rf build/
  $ mkdir build
  $ cd build
  $ cmake XXXXXX ..  # details omitted
  $ make -j3
  $ sudo make install
  ```

* The `install_opencv-3.4.6.sh` script should work for Jetson TX2 or Jetson AGX Xavier (with JetPack-4.2) too.  Just remember to set 'CUDA_ARCH_BIN' (in the `cmake` command in the script) to the right value for the platform.

  - Jetson TX2: `CUDA_ARCH_BIN="6.2"`
  - Jetson AGX Xavier: `CUDA_ARCH_BIN="7.2"`

* During execution of the `install_opencv-3.4.6.sh` script, the sudo username/password would likely time out a few times.  You'll have to re-enter your password every time when prompted.  If this is an issue for you and you don't have security related concerns, you could [use visudo to set a longer timeout (or even no timeout) for sudo sessions](https://lifehacker.com/make-sudo-sessions-last-longer-in-linux-1221545774).

OK.  Here is the part about executing the script and building/installing opencv-3.4.6.

If you haven't cloned my GitHub 'jetson_nano' repository.  Do so now.

```
$ cd ${HOME}/project
$ git clone https://github.com/jkjung-avt/jetson_nano.git
```

I'd suggest to set Jetson Nano to Max-N mode (maximum performance mode) so the building process could be as short as possible.

```shell
$ sudo nvpmodel -m 0
$ sudo jetson_clocks
```

Then just execute the `install_opencv-3.4.6.sh` script.

```
$ cd ${HOME}/project/jetson_nano
$ ./install_opencv-3.4.6.sh
```

The building and installing process would take a couple of hours.  When done, you should see both python3 and python2 reporting the correct version number for 'cv2' module: 3.4.6.

# Testing opencv-3.4.6 with tegra-cam.py

I'd use my [tegra-cam.py](https://jkjung-avt.github.io/tx2-camera-with-python/) script to test my opencv-3.4.6 build.  Note that I use a USB webcam for the testing.  Just plug the USB webcam into one of the USB ports on Jetson Nano and:  (Adjust image width/height for the camera you're using if necessary)

```shell
$ wget https://gist.githubusercontent.com/jkjung-avt/86b60a7723b97da19f7bfa3cb7d2690e/raw/9561b135c1b40f82afe46afcc1f326a2e913e08c/tegra-cam.py
$ python3 tegra-cam.py --usb --vid 0 --width 1280 --height 720
```

And voila, the Jason Nano DevKit in action, running with the freshly built and installed opencv-3.4.6, and shot from the webcam.

![Jetson Nano in Action](/assets/2019-05-15-opencv-on-nano/nano_in_action.png)

