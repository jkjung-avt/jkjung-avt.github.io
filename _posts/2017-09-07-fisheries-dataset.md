---
layout: post
comments: true
title: "Training a Fish Detector with NVIDIA DetectNet (Part 1/2)"
excerpt: "I took fish image data from Kaggle's 'The Nature Conservancy Fisheries Monitoring' competition, and wanted to train a 'fish detector' with it. I chose NVIDIA DetectNet as the underlying object detector. In this post I documented how I prepared fish image training data for DetectNet."
date: 2017-09-07
category: "training"
tags: DL training
---

A while ago Kaggle held a very interesting competition: [The Nature Conservancy Fisheries Monitoring](https://www.kaggle.com/c/the-nature-conservancy-fisheries-monitoring). In this competition the participants were requested to develop machine learning models which could look at camera footages from fishing boats and tell which of the 8 classes (6 types of specific fishes, some other kind, or no fish) each image belongs to. I enrolled myself into this competition, but did not spend time working on it and thus did not make any submissions. It's a really a pity in retrospect.

After the Kaggle Fisheries competition finished, I checked out its leaderboard, results and contestants' experience sharings. I was in particular inspired by Felix Yu's article: [Detect and Classify Species of Fish from Fishing Vessels with Modern Object Detectors and Deep Convolutional Networks](https://flyyufelix.github.io/2017/04/16/kaggle-nature-conservancy.html). Since recently I've been focusing on the tasks/techniques of object detection, I took the chance to work on this Kaggle Fisheries dataset. I thought I could develop a 'fish detector' with this training data. (This is a somewhat challenging task, by referring to Felix Yu's blog post.)

I chose NVIDIA's DetectNet as the first object detector for this task, since it's relatively easy to use. To learn how to train NVIDIA DetectNet with [DIGITS](https://developer.nvidia.com/digits), you could reference the following links:

* [Deep Learning for Object Detection with DIGITS](https://devblogs.nvidia.com/parallelforall/deep-learning-object-detection-digits/)
* [DetectNet: Deep Neural Network for Object Detection in DIGITS](https://devblogs.nvidia.com/parallelforall/detectnet-deep-neural-network-object-detection-digits/)
* [Object detection example walkthrough (with KITTI dataset)](https://github.com/NVIDIA/DIGITS/tree/master/examples/object-detection)
* [DetectNet label file format](https://github.com/NVIDIA/DIGITS/blob/master/digits/extensions/data/objectDetection/README.md)
* [DetectNet online course on NVIDIA QuikLab (non-free)](https://nvidia.qwiklab.com/focuses/1204)
* [Exploring the SpaceNet Dataset Using DIGITS](https://devblogs.nvidia.com/parallelforall/exploring-spacenet-dataset-using-digits/)

In order to train a DetectNet model to detect fishes, I had to first convert the Kaggle Fisheries images to labeled data according to DetectNet file formats. Note that the original data, [train.zip](https://www.kaggle.com/c/the-nature-conservancy-fisheries-monitoring/data), downloaded from Kaggle web site does not contain bounding box labels. Fortunately previous participants of the competition had made such labels and shared them on the [discussion forum](https://www.kaggle.com/c/the-nature-conservancy-fisheries-monitoring/discussion). Specifically I downloaded [the labels posted by shuai](https://www.kaggle.com/c/the-nature-conservancy-fisheries-monitoring/discussion/25902) as my starting point.

I used ['sloth'](https://github.com/cvhciKIT/sloth) labeling tool to verify the downloaded label (.json) files. You can find installation guide and docuemtntation of 'sloth' [here](http://sloth.readthedocs.io/en/latest/index.html). Among all 3,777 training images in the Kaggle Fisheries dataset, I looked through a few hundreds of them and made quite a few corrections. I put my updated label files in [my own GitHub repository](https://github.com/jkjung-avt/dataset-tools/tree/master/Kaggle_Fisheries/train).

Here's a screenshot of sloth, showing 1 exmaple image `img_07911.jpg` from the `train\YFT\` subfolder.

![sloth screenshot](/assets/2017-09-07-fisheries-dataset/YFT_07911_sloth.png)

Then I referenced [alpop's python code](https://github.com/alpop/Fish) for how to convert sloth JSON files to DetectNet label files. I adapted the code for Jupyter Notebook, and put my modified code in:

[https://github.com/jkjung-avt/dataset-tools/tree/master/Kaggle_Fisheries](https://github.com/jkjung-avt/dataset-tools/tree/master/Kaggle_Fisheries)

To follow along with my ipynb code, you'd have to download the original training images from Kaggle and put the JPG files under the `train\` subfolder. Note that you should keep the JSON label files along the process. Next you'd just run `hist_train.ipynb` and then `prepare_detectnet.ipynb`. The resulting `for_detectnet\` subfolder would contain all data and labels you need to train your DetectNet model.

I've put quite a few comments in the Python Notebook code already. But here are some additional notes about the dataset and about how I processed it:

* The original dataset contains images with different sizes. I scaled all images to 1280x720, while keeping aspect ratio with respect to each individual training image. I pad black bars either at right or at bottom of the image, depending which side was shorter than expected after scaling.
* When resizing an image, I also needed to resize the bounding boxes with the same scale.
* Some of the original bounding box labels were out of bound, i.e. with x < 0, x >= 1280, y < 0, or y >= 720. So in the ipynb code I'd check and restrict bounding box coordinates to be within range.
* I put resized images (1280x720) and updated JSON labels into a `train_1280x720\` subfolder, so I can use sloth to easily check correctness again.
* I split the original Kaggle Fisheries training images into roughly 3/4 as DetectNet training data and 1/4 as DetectNet validation data. Note that images in the 'NoF' (No Fish) class were always put into training data side.
* Converting JSON labels to DetectNet label txt files was straightforward. I did not make any change to DetectNet's default class name, 'Car'. This should not matter since we would be training the detector to detect only 1 single class of object.
* In the end, I made a scatter plot of bounding box sizes, in order to verify most boudning boxes were with the desired sizes. According to documentation, DetectNet works best for bounding boxes with size ranging from [50, 50] up to [400, 400]. I actually spent some time to fix bounding boxes which were too small, and iterated through this step a few times. Here's my final result. I trained my DetectNet model with this exact data.

![Scatter plot of bounding box sizes](/assets/2017-09-07-fisheries-dataset/fisheries_bbox_scatter.png)

Check out how I trained the DetectNet with this data by reading the 2nd part of this post: [Training a Fish Detector with NVIDIA DetectNet (Part 2/2)](https://jkjung-avt.github.io/detectnet-training/).
