---
layout: post
comments: true
title: "How to Capture and Display Camera Video with Python on Jetson TX2"
excerpt: "In this post I share how to use python code (with OpenCV) to capture and display camera video on Jetson TX2, including IP CAM, USB webcam and the Jetson onboard camera. This sample code should work on Jetson TX1 as well."
date: 2017-10-19
category: "opencv"
tags: opencv
---

In this post I share how to use python code (with OpenCV) to capture and display camera video on Jetson TX2, including IP CAM, USB webcam and the Jetson onboard camera. This sample code should work on Jetson TX1 as well.

Prerequisite:

* OpenCV with **GStreamer** and **python** support needs to be built and installed on the Jetson TX2. I use opencv-3.3.0 and python3. You can refer to my earlier post for how to build and install OpenCV with python support: [How to Install OpenCV (3.3.0) on Jetson TX2](https://jkjung-avt.github.io/opencv3-on-tx2/).
* If you'd like to test with an IP CAM, you need to have it set up and know its RTSP URI, e.g. rtsp://admin:XXXXX@192.168.1.64:554.
* Hook up a USB webcam (I was using Logitech C920) if you'd like to test with it. The USB webcam would usually be instantiated as /dev/video1, since the Jetson onboard camera has occupied /dev/video0.

Reference:

* I developed my code based on [this canny edge detector sample code](https://devtalk.nvidia.com/default/topic/1024245/jetson-tx2/opencv-3-3-and-integrated-camera-problems-/post/5210653/#5210653).
* [ACCELERATED GSTREAMER FOR TEGRA X2 USER GUIDE](http://developer2.download.nvidia.com/embedded/L4T/r28_Release_v1.0/Docs/Jetson_TX2_Accelerated_GStreamer_User_Guide.pdf?rJ32fctxo7T2SNlckXrTndWYLO_LIQmRRJpd4Hcz2B8CQefnSGKDjQHbW7bxDRTW6OfvHzMRTdrE16hA6hj4gcLIZvPxWzXYw2Z1t88_cFVTCPDQZdiwJzsy3-hMvahbvSH23CEEca47iu-igcm7cwnCUXhvHCsNhkgouSCxjlIBxHV2iZ7i9xB42FoFpttBQw): Descriptions of *nvcamerasrc*, *nvvidconv* and *omxh264dec* could be found in this document.

How to run the Tegra camera sample code:

* Download the `tegra-cam.py` source code from my GitHubGist: [https://gist.github.com/jkjung-avt/86b60a7723b97da19f7bfa3cb7d2690e](https://gist.github.com/jkjung-avt/86b60a7723b97da19f7bfa3cb7d2690e)
* To capture and display video using the Jetson onboard camera, try the following. By default the camera resolution is set to 1920x1080 @ 30fps.

```shell
$ python3 tegra-cam.py
```

* To use a USB webcam and set video resolution to 1280x720, try the following. The '--vid 1' means using /dev/video1.

```shell
$ python3 tegra-cam.py --usb --vid 1 --width 1280 --height 720
```

* To use an IP CAM, try the following command, while replacing the last argument with RTSP URI for you own IP CAM.

```shell
$ python3 tegra-cam.py --rtsp --uri rtsp://admin:XXXXXX@192.168.1.64:554
```

Discussions:

The crux of this `tegra-cam.py` script lies in the GStreamer pipelines I use to call `cv.VideoCapture()`. In my experience, using **nvvidconv** to do image scaling and to convert color format to BGRx (note that OpenCV requires *BGR* as the final output) produces better results in terms of frame rate.

```python
def open_cam_rtsp(uri, width, height):
    gst_str = "rtspsrc location={} latency=50 ! rtph264depay ! h264parse ! omxh264dec ! nvvidconv ! video/x-raw, width=(int){}, height=(int){}, format=(string)BGRx ! videoconvert ! appsink".format(uri, width, height)
    return cv2.VideoCapture(gst_str, cv2.CAP_GSTREAMER)

def open_cam_usb(dev, width, height):
    # We want to set width and height here, otherwise we could just do:
    #     return cv2.VideoCapture(dev)
    gst_str = "v4l2src device=/dev/video{} ! video/x-raw, width=(int){}, height=(int){}, format=(string)RGB ! videoconvert ! appsink".format(dev, width, height)
    return cv2.VideoCapture(gst_str, cv2.CAP_GSTREAMER)

def open_cam_onboard(width, height):
    # On versions of L4T previous to L4T 28.1, flip-method=2
    # Use Jetson onboard camera
    gst_str = "nvcamerasrc ! video/x-raw(memory:NVMM), width=(int)1920, height=(int)1080, format=(string)I420, framerate=(fraction)30/1 ! nvvidconv ! video/x-raw, width=(int){}, height=(int){}, format=(string)BGRx ! videoconvert ! appsink".format(width, height)
    return cv2.VideoCapture(gst_str, cv2.CAP_GSTREAMER)
```

If you like this post or have any questions, feel free to leave a comment below.
