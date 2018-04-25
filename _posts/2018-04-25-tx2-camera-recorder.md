---
layout: post
comments: true
title: "Tegra Camera Recorder"
excerpt: "This post is in response to a reader of my blog posts, who was trying to 'get Python OpenCV to write back to a gstreamer pipeline either into a file or into a video stream for web browsers'. As usual, I shared the full python code to demonstrate how to do that."
date: 2018-04-25
category: "opencv"
tags: opencv gstreamer
---

Quick link: [tegra-cam-rec.py](https://gist.github.com/jkjung-avt/1df91d4cedad633cd5b49a4779cb4e32)

A reader emailed me asking about how to 'get Python OpenCV to write back to a gstreamer pipeline either into a file or into a video stream for web browsers'. While working on my reply to the email, I quickly coded a sample python script. I'm sharing it here, in belief that it would benefit others as well.

# Prerequisite

* Refer to my previous post ["How to Capture and Display Camera Video with Python on Jetson TX2"](https://jkjung-avt.github.io/tx2-camera-with-python/) and make sure `tegra-cam.py` runs OK on the target Jetson TX2.
* My `tegra-cam-rec.py` script would require the `mpegtsmux` gstreamer element. It could be installed with the following apt command.

  ```shell
  $ sudo apt-get install gstreamer1.0-plugins-bad
  ```

  + **Note:** The `apt-get install` above actually also installed a bunch of **`libopencv-xxx2.4v5`** stuffs, which could potentially conflict with our own-built opencv3.x libraries. I did not run into any problems related to this so far. But I think the reader should be reminded about the risk.

# About the Code

My Tegra Camera Recorder code could be downloaded from my Gist repository: [tegra-cam-rec.py](https://gist.github.com/jkjung-avt/1df91d4cedad633cd5b49a4779cb4e32)

I think the source code is easy to understand. The `cv2.VideoWriter()` is created by:

```python
def get_video_writer(fname, fps, width, height):
    gst_str = ("appsrc ! videoconvert ! omxh264enc ! mpegtsmux ! "
               "filesink location={}.ts").format(fname)
    return cv2.VideoWriter(gst_str, cv2.CAP_GSTREAMER, 0, fps, (width, height))
```

Note in the `gst_str` above I used `omxh264enc`. This would utilize the H/W H.264 encoder of TX2 (or TX1) and thus would not burden CPU to do the video encoding.

After the `VideoWriter` is instantiated, we just call its `write()` method once for each video frame we'd like to record. Here I'm using MPEG TS encapsulation for the recorded video file. MPEG TS does not require any metadata at the end of the file, and thus is more resilient to errors, such as program crash or errors in the encoded H.264 bitstream.

If, instead of recording video to a file, we'd like to do video streaming, we could just replace `filesink` in the `gst_str` above with an appropriate gstreamer element for the intended application.

# How to Run It

Same as the other tegra camera example code I shared previously, this `tegra-cam-rec.py` script could also capture live video from either IP CAM, USB webcam, or the Tegra onboard camera. I added a few arguments for recording, which I think are all pretty self-explanatory.

```shell
jkjung@ubuntu:~/project/tegra-cam$ python3 tegra-cam-rec.py --help
usage: tegra-cam-rec.py [-h] [--rtsp] [--uri RTSP_URI]
                        [--latency RTSP_LATENCY] [--usb] [--vid VIDEO_DEV]
                        [--width IMAGE_WIDTH] [--height IMAGE_HEIGHT]
                        [--ts TS_FILE] [--fps FPS] [--rec REC_SEC]
                        [--text TEXT]

Capture and record live camera video on Jetson TX2/TX1

optional arguments:
  -h, --help            show this help message and exit
  --rtsp                use IP CAM (remember to also set --uri)
  --uri RTSP_URI        RTSP URI string, e.g. 'rtsp://192.168.1.64:554'
  --latency RTSP_LATENCY
                        latency in ms for RTSP [200]
  --usb                 use USB webcam (remember to also set --vid)
  --vid VIDEO_DEV       video device # of USB webcam (/dev/video?) [1]
  --width IMAGE_WIDTH   image width [640]
  --height IMAGE_HEIGHT
                        image height [480]
  --ts TS_FILE          output .ts file name ['output']
  --fps FPS             fps of the output .ts video [30]
  --rec REC_SEC         recording length in seconds [5]
  --text TEXT           text to be overlayed on the video ['TX2 DEMO']
```

* To record a short (5 seconds) video clip from the Jetson onboard camera, do the following. The default output video filename is `output.ts`.

  ```shell
  $ python3 tegra-cam-rec.py
  ```

* Or, to use USB webcam `/dev/video1`, while changing the overlay text to "Hello, World!", try this.

  ```shell
  $ python3 tegra-cam-rec.py --usb --vid 1 --text "Hello, World!"
  ```

* And use the `totem` video player to play back the recorded video file.

  ```shell
  $ totem output.ts
  ```

  ![A screenshot of tegra-cam-rec.py](/assets/2018-04-25-tx2-camera-recorder/tegra-cam-rec.png)
