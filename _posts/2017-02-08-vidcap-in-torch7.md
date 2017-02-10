---
layout: post
comments: true
title: "Capturing HDMI Video in Torch7"
excerpt: "I implemented a video capturing module in Torch7. This module calls V4L2 API (in C code) to fetch raw video data from the underlying video capture device. It then uses Lua FFI interface to pass the data to Lua/Torch7. The resulting data are Torch Tensors and could be displayed on screen with Torch7's image API."
date: 2017-02-08
category: "torch7"
tags: torch7 video capture
---

I need to capture HDMI video from Nintendo Famicom Mini and convert the video data to Torch Tensors, so that I can train DeepMind's DQN AI agent to play real Nintendo games. Ideally this video capture module should not consume too much CPU and should produce image data with low latency.

I know quite a few options for capturing HDMI video on Tegra X1 platforms (some of which might require installation of custom device drivers):

1. [AVerMedia's C353 HDMI/VGA frame grabber](http://www.avermedia.com/professional/product/c353/overview).
2. HDMI video inputs on [AVerMedia's EX711-AA TX1 carrier board](http://www.avermedia.com/professional/product/ex711_aa/overview).
3. [ZHAW's HDMI2CSI adapter board](https://blog.zhaw.ch/high-performance/category/hdmi2csi/).
4. [Magewell's USB Capture HDMI](http://www.magewell.com/usb-capture-hdmi).
5. [Inogeni's HD to USB 3.0 Converter](https://inogeni.com/hd-usb3-0/).

I end up choosing option #2 because I have AVerMedia's EX711-AA carrier board on hand and it can take HDMI video input without connecting any additional adapter board (quite convenient).

The HDMI video inputs (HDMI-to-CSI) on EX711-AA carrier board are implemented as V4L2 devices and are exposed as /dev/video0 and /dev/video1. Since performance (CPU consumption and latency) is a major concern, I think it's a good idea to write most of the video data handling code in C. And I'd use [Lua FFI library](http://luajit.org/ext_ffi.html) to interface between my C code and Torch7/Lua code.

More specifically I have implemented my Torch7 video capture module this way:

* In the C code I call V4L2 API to fetch video data from /dev/video0 directly. This minimizes overhead, comparing to GStreamer or libv4l, etc.
* HDMI video from Nintendo Famicom Mini is RGB 1280x720 at 60 fps. To reduce the amount of video data for DQN to process, I'd decimate video data to grayscale 640x360 at 30 fps only. I do this video data decimation in C code as well.
* Passing video data from C code to Torch7/Lua through FFI interface is straightforward. I referenced [Atcold's torch-Developer-Guide](https://github.com/Atcold/torch-Developer-Guide) for the actual implementation.
* Raw video data from /dev/video0 is in [UYVY format](https://linuxtv.org/downloads/v4l-dvb-apis/uapi/v4l/pixfmt-uyvy.html). The above-mentioned video data decimation process would throw away U and V, as well as 3/4 of Y values. The resulting grayscale image data is ordered, pixel-by-pixel, from left to right, then from top to bottom.
* The decimated grayscale image data (640x360) would correspond to data layout for a ByteTensor(360,640,1), or **ByteTensor(H,W,C)**, where H stands for height, W for width and C for color channels. However, Torch7 image and nn/cunn libraries expect image data in the form of **ByteTensor(C,H,W)**. So after getting image data (img) through FFI I'll have to do `img:permute(3,1,2)` to format it in the right order. Then I can use `image.display()` to show the images correctly.

The resulting code is located in [my dqn-tx1-for-nintendo repository](https://github.com/jkjung-avt/dqn-tx1-for-nintendo) and manifested in these files:

```
 vidcap/Makefile
 vidcap/device.h
 vidcap/device.c
 vidcap/video0_cap.c
 vidcap/vidcap.lua
 test/test_vidcap.lua
```

The following screenshot was taken when I ran `qlua test/test_vidcap.lua` on the EX711-AA TX1 carrier board, with Nintendo Famicom Mini connected to /dev/video0 input.

![Screenshot of test_vidcap.lua](/assets/2017-02-08-vidcap-in-torch7/test_vidcap_mario.png)

From the screenshot, I observed that CPU loading for test_vidcap.lua was around 20% accross all 4 CPUs of TX1. That was after I ran the 'jetson_clocks.sh' script to boost all CPUs of TX1 to the highest clock rate. Note that capturing 1280x720p60 video and rendering at 30 fps continuously does take a tow on the TX1 system. I don't consider 20% CPU consumption ideal but it's probably OK. I should have enough remaining CPU power to run the DQN. On the other hand I also verfied latency of the captured grayscale image data by playing a few games. The result seemed satisfactory.

As mentioned in [my earlier post](https://jkjung-avt.github.io/dqn-pong/), there is memory leak in image.display(). The 'test_vidcap.lua' script also suffers from the same problem and could hog more and more memory over time.

To-do:

* Primer on V4L2 API.

