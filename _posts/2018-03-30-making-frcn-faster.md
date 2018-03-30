---
layout: post
comments: true
title: "Making Faster R-CNN Faster!"
excerpt: "A while ago I wrote a post about how to set up and run Faster RCNN on Jetson TX2. In this post I demonstrate how to use a faster CNN feature extractor to speed up Faster RCNN while maintaining its object detection accuracy (mAP). More specifically, I replaced VGG16 layers with GoogLeNet in Faster RCNN and was able to reduce model inference time roughly by half."
date: 2018-03-30
category: "frcn"
tags: frcn rcnn
---

In my previous post ["Faster R-CNN on Jetson TX2"](https://jkjung-avt.github.io/faster-rcnn/), I wrote about how to set up and run Faster RCNN on Jetson TX2, as well as to use it for real-time object detection with a live camera video feed. While Faster RCNN exhibited good object detection accuracy, it didn't run fast enough on Jetson TX2. I got only ~1 fps (0.91~0.95 second per image) when running the pre-trained (VGG16 based) pascal_voc model.

To speed up Faster RCNN on a Jetson TX2, a recommended approach by NVIDIA is using [TensorRT](https://developer.nvidia.com/tensorrt). In fact, NVIDIA already provided a [sample code](http://docs.nvidia.com/deeplearning/sdk/tensorrt-developer-guide/index.html#fasterrcnn_sample) illustrating how to implement a Faster RCNN model with TensorRT. But anyway, I tried a different route and the result was OK. I replaced the slower VGG16 feature extrator with GoogLeNet layers in Faster RCNN, and was indeed able to boost its inference speed by a significant margin.

# About how Faster RCNN works

In addition to [the original paper](https://arxiv.org/abs/1506.01497), I'd also recommend reading [this presentation file](https://www.dropbox.com/s/xtr4yd4i5e0vw8g/iccv15_tutorial_training_rbg.pdf?dl=0) for understanding how Faster RCNN works.

![Faster RCNN block diagram](/assets/2018-03-30-making-frcn-faster/FRCN_architecture.png)

In a nutshell, the Faster RCNN network consists of 3 parts:

* 'CNN' feature extractor, which is used to turn the input image into a condensed feature map
* Region proposal network, which is used to generate Region Of Interests (ROIs)
* 'Classifier' network, which classifies (ROI-pooled) ROIs as either objects or backgrounds and generates the detection ouputs, i.e. bounding boxes and class probabilities/scores

# About swapping CNN feature extraction layers of Faster RCNN

In the original Faster RCNN implementation, VGG16 (and ZF Net) is used as the CNN feature extractor. As I've [checked previously](https://jkjung-avt.github.io/caffe-time/), GoogLeNet, with similar cllaissification accuracy as VGG16, runs much faster on Jetson TX2. So I wanted to replace VGG16 layers with GoogLeNet in Faster RCNN, to improve its speed.

The way I implemented this is straightforward. I took [bvlc_googlenet](https://github.com/BVLC/caffe/blob/master/models/bvlc_googlenet/deploy.prototxt) and split the neural network into 2 parts. I used all layers before (inclusive) `inception_4e/output` as the 'CNN feature extractor', and I used the rest (mainly inception_5x) layers as the 'Classifier' network. The resulting `test.prototxt` could be found [here](https://github.com/jkjung-avt/py-faster-rcnn/blob/master/models/pascal_voc/GoogLeNet/faster_rcnn_end2end/test.prototxt).

# Running the modified Faster RCNN on Jetson TX2

1. Follow my ["Faster R-CNN on Jetson TX2"](https://jkjung-avt.github.io/faster-rcnn/) post and make sure [py-faster-rcnn](https://github.com/rbgirshick/py-faster-rcnn) runs OK on the target Jetson TX2.

2. Assuming `py-faster-rcnn` has been installed at `/home/nvidia/project/py-faster-rcnn`, do the following.

   ```shell
   $ cd /home/nvidia/project/py-faster-rcnn
   $ cd tools
   $ wget https://raw.githubusercontent.com/jkjung-avt/py-faster-rcnn/master/tools/demo_camera.py
   $ cd ..
   $ mkdir -p models/pascal_voc/GoogLeNet/faster_rcnn_end2end/
   $ cd models/pascal_voc/GoogLeNet/faster_rcnn_end2end/
   # wget https://raw.githubusercontent.com/jkjung-avt/py-faster-rcnn/master/models/pascal_voc/GoogLeNet/faster_rcnn_end2end/test.prototxt
   ```

   In addition, download this [voc2007_googlenet_iter_70000.caffemodel](https://drive.google.com/open?id=1Uvvgsl-PwkOPHjmuHTwS0L7Aktz6ePmR) file and put it into the `models/pascal_voc/GoogLeNet/faster_rcnn_end2end/` directory as well.

3. Run the GoogLeNet Faster RCNN model with the demo script. Note the script uses the Jetson onboard camera by default. Specify the `--usb` or `--rtsp` command line options if a USB webcam or an IP CAM is used instead.

   ```shell
   $ cd /home/nvidia/project/py-faster-rcnn
   $ python2 tools/demo_camera.py \
             --prototxt models/pascal_voc/GoogLeNet/faster_rcnn_end2end/test.prototxt \
             --model models/pascal_voc/GoogLeNet/faster_rcnn_end2end/voc2007_googlenet_iter_70000.caffemodel
   ```

   When testing the above on my Jetson TX2, I was able to get **~2 fps (0.48s per image)** throughput. That was **roughly 2 times the speed of the original VGG16 based Faster RCNN!**

# Training the GoogLeNet Faster RCNN model

If you are interested in training your own GoogLeNet Faster RCNN model, do read on. **But note that training of the Faster RCNN model should be done on a deep learning PC or server. The steps described below most likely won't work on Jetson TX2.**

1. Follow instructions on the [py-faster-rcnn GitHub page](https://github.com/rbgirshick/py-faster-rcnn) and make sure the original VGG16 based Faster RCNN model could be successfully trained on your deep learning PC or server. More specifically, follow what's stated under `Beyond the demo: installation for training and testing models`.

   ```shell
   ### Download Pascal VOC 2007 dataset
   $ mkdir -p ~/data
   $ cd ~/data
   $ wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtrainval_06-Nov-2007.tar
   $ wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtest_06-Nov-2007.tar
   $ wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCdevkit_08-Jun-2007.tar
   $ tar xvf VOCtrainval_06-Nov-2007.tar
   $ tar xvf VOCtest_06-Nov-2007.tar
   $ tar xvf VOCdevkit_08-Jun-2007.tar
   $ cd ~/project/py-faster-rcnn/data
   $ ln -s ~/data/VOCDevkit VOCdevkit2007
   ### Download pre-trained ImageNet models
   $ cd ..
   $ ./data/scripts/fetch_imagenet_models.sh
   ### Do training and testing, using GPU 0
   $ ./experiments/scripts/faster_rcnn_end2end.sh 0 VGG16 pascal_voc
   ```

   Note that this training could take a long time. And the final caffemodel file could be found at `~/project/py-faster-rcnn/output/faster_rcnn_end2end/voc_2007_trainval/`.

2. Download additional files needed for training your own GoogLeNet Faster RCNN.

   Put these 2 files under `~/project/py-faster-rcnn/models/pascal_voc/GoogLeNet/faster_rcnn_end2end/`:
   * [train.prototxt](https://raw.githubusercontent.com/jkjung-avt/py-faster-rcnn/master/models/pascal_voc/GoogLeNet/faster_rcnn_end2end/train.prototxt)
   * [solver.prototxt](https://raw.githubusercontent.com/jkjung-avt/py-faster-rcnn/master/models/pascal_voc/GoogLeNet/faster_rcnn_end2end/solver.prototxt)

   Put this file under `~/project/py-faster-rcnn/data/imagenet_models/`:
   * [bvlc_googlenet.caffemodel](http://dl.caffe.berkeleyvision.org/bvlc_googlenet.caffemodel)

3. Train the GoogLeNet Faster RCNN with Pascal VOC 2007 dataset.

   ```shell
   $ cd ~/project/py-faster-rcnn/
   $ python2 ./tools/train_net.py --gpu 0 \
                                  --solver ./models/pascal_voc/GoogLeNet/faster_rcnn_end2end/solver.prototxt \
                                  --weights ./data/imagenet_models/bvlc_googlenet.caffemodel \
                                  --imdb voc_2007_trainval \
                                  --iters 70000 \
                                  --cfg experiments/cfgs/faster_rcnn_end2end.yml 2>&1 | \
     tee -a ./experiments/logs/faster_rcnn_googlenet.log
   ```

4. After the training is done, verify its accuracy (mAP).

   ```shell
   $ cd ~/project/py-faster-rcnn/
   $ python2 ./tools/test_net.py --gpu 0 \
                                 --def ./models/pascal_voc/GoogLeNet/faster_rcnn_end2end/test.prototxt \
                                 --net ./output/faster_rcnn_end2end/voc_2007_trainval/voc2007_googlenet_iter_70000.caffemodel \
                                 --imdb voc_2007_test \
                                  --cfg experiments/cfgs/faster_rcnn_end2end.yml 2>&1 | \
     tee -a ./experiments/logs/faster_rcnn_googlenet.log
   ```

   I tried to train this GoogLeNet Faster RCNN model quite a few times. The best **mAP** I got was around **0.69**. It was very close to that of the original VGG16 Faster RCNN, which was also around 0.69.

   Some additional notes about training the model: (based on my own experiments)

   * I had to use a higher initial learning rate (0.005 instead of 0.001) and a lower weight_decay (0.0001 instead of 0.0005) in the solver to get better result.
   * I got better mAP if I just used `SGD` solver with momentum (better than `Adam`).
   * I got better mAP if I removed the Dropout (`pool5/drop_7x7_s1`) layer in bvlc_googlenet.
