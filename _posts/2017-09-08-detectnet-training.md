---
layout: post
comments: true
title: "Training a Fish Detector with NVIDIA DetectNet (Part 2/2)"
excerpt: "I trained a DetectNet model with the data I prepared from Kaggle's 'The Nature Conservancy Fisheries Monitoring' dataset. Here's the list of steps I employed for the training, as well as a brief discussion about the result."
date: 2017-09-08
category: "training"
tags: DL training
---

As described in my previous post, [Training a Fish Detector with NVIDIA DetectNet (Part 1/2)](https://jkjung-avt.github.io/fisheries-dataset/), I've prepared Kaggle Fisheries image data with labels ready for DetectNet training. It's time to load the data to my DIGITS server and do the training.

Training a DetectNet model with DIGITS is mostly straightforward, **except that I had to modify image width and height correctly (1280x720) in the prototxt file** (more on this later). I basically followed [the Object Detection example (with KITTI dataset)](https://github.com/NVIDIA/DIGITS/tree/master/examples/object-detection) in the NVIDIA/DIGITS GitHub repository.

I first loaded the Object Detection dataset into DIGITS.

![New Object Detection Dataset in DIGITS](/assets/2017-09-08-detectnet-training/new-dataset.png)

Next I created an Object Detection model to be trained with the dataset. Following the DIGITS Object Detection KITTI example, I **set `Subtract Mean` to `None`**, set `Solver type` to `Adam`, set `Base Learning Rate` to `0.0001` with (advanced) `Exponential Decay` Policy and `0.95` Gamma value, set `Batch size` to `2` and set `Batch Accumulation` to `5`. Note that training the DetectNet on a GTX-1080 with 8GB memory, I was only able to fit at most `2` 1080x720 input images as a batch to the GPU.

![New Object Detection Model on DIGITS](/assets/2017-09-08-detectnet-training/new-model.png)

![Learning Rate Decay](/assets/2017-09-08-detectnet-training/lr-decay.png)

I then copied and pasted the example [detectnet_network.prototxt](https://raw.githubusercontent.com/NVIDIA/caffe/caffe-0.15/examples/kitti/detectnet_network.prototxt) as my `Custom Network`. And I did 2 important modifications here.

1. I **modified all image sizes from 1248x352 (or 1248x384) to 1280x720 in the prototxt**. There are 6 occurences in total. Refer to the screenshot below for 2 of such occurences.
2. I used a caffemodel (DNN weights) which had been pre-trained with KITTI dataset. With this transfer learning trick, I think the network should be able to learn faster.

![Custom Network definition](/assets/2017-09-08-detectnet-training/custom-network.png)

I trained the DetectNet model for 300 epochs in the first round. As a result I got a model with validation precision 75.3%, recall 76.0% and mAP 64.4. (By the way, training this model for 300 epochs on my GTX-1080 desktop PC took roughly 21 hours.)

![Training 1st Round](/assets/2017-09-08-detectnet-training/training-1st-round.png)

I took the result of the first round, fine-tuned a few parameters, lowering the learning rate a little bit, and trained the model again for 300 epochs. The result improved a little bit. In the end I had a DetectNet model with validation precision 86.77%, recall 87.12% and **mAP 78.6**.

![Training 2nd Round](/assets/2017-09-08-detectnet-training/training-2nd-round.png)

Finally I tested my trained DetectNet model with the `Test Many` function in DIGITS. I could see that the model indeed had about 80% accuracy in detecting fishes on newly unseen test images (from [test_stg1.zip](https://www.kaggle.com/c/the-nature-conservancy-fisheries-monitoring/data)).

Here is an example for which the model made a correct prediction.

![Correct Prediction](/assets/2017-09-08-detectnet-training/prediction-ok.png)

And here is an example for which the model had clearly missed a fish.

![Incorrect Prediction](/assets/2017-09-08-detectnet-training/prediction-ng.png)
