---
layout: post
comments: true
title: "Custom YOLOv4 Model on Google Colab"
excerpt: "This is a tutorial about how to utilize free GPU on Google Colab to train a custom YOLOv4 model.  I use the CrowdHuman dataset in this tutorial.  The custom trained model could detect people (head and full body) pretty accurately."
date: 2020-09-13
category: "yolo"
tags: yolo yolov4 colab
---

Quick link: [jkjung-avt/yolov4_crowdhuman](https://github.com/jkjung-avt/yolov4_crowdhuman)

I was inspired by [this post](https://chtseng.wordpress.com/2020/02/07/%E4%BD%BF%E7%94%A8google-colab%E8%A8%93%E7%B7%B4yolo/) and wanted to do a tutorial about how to train a YOLOv4 model using the FREE GPUs on Google Colab.  And here it is.

# Prerequisite

> Colaboratory (or Colab) is a free research tool from Google for machine learning education and research built on top of Jupyter Notebook.

If you are new to Google Colab or Jupyter Notebook, I suggest you to read [Basic operations in Colaboratory notebook](https://colab.research.google.com/github/pycroscopy/AICrystallographer/blob/master/Tutorials/ColabNotebooks_BasicOperations.ipynb#scrollTo=hkcI78C4Sybk) to learn how to mount your Google Drive and run cells in a Colab Notebook.

Otherwise,

* You need to have a Google account to use [Google Colab](https://colab.research.google.com/notebooks/intro.ipynb) and [Google Drive](https://drive.google.com/drive/my-drive).

* Make sure you have at least 2GB of available storage spaces on your Google Drive.  You'll need it to store training logs and trained models (weights).

# Training the model

I've made the [yolov4_crowdhuman.ipynb](https://colab.research.google.com/drive/1eoa2_v6wVlcJiDBh3Tb_umhm7a09lpIE?usp=sharing) extremely easy to run.  You just need to make your own copy of the Notebook, mount your Google Drive onto your Colab virtual machine session, and run all cells.  Refer to [Training on Google Colab](https://github.com/jkjung-avt/yolov4_crowdhuman#training-on-google-colab) for detailed descriptions of the steps.

Here are some reminders and additional details:

* Do read the words of caution in [Training on Google Colab](https://github.com/jkjung-avt/yolov4_crowdhuman#training-on-google-colab).  You don't want to get locked out of Colab GPU access for a long time, do you?
* Open "Edit -> Notebook Setting".  You should see that the Notebook has been set up to use "GPU" as the hardware accelerator by default.
* After the Notebook is connected to a virtual machine session on Colab, you could use "Runtime -> Change runtime type" to verify that "GPU" accelerator is being used.

You should read the Notebook to see what it does in each step.  And you could read the README.md in my [jkjung-avt/yolov4_crowdhuman](https://github.com/jkjung-avt/yolov4_crowdhuman) repo to further understand how the training data is prepared and how to use "darknet" framework to train and test the yolov4 model.

It might take 5~8 hours for the Colab Notebook to finish all steps.  It would be faster if you are lucky and get a more modern GPU, such as Tesla T4 or P100, on the virtual machine.  When the Colab Notebook finishes, you should be able to see this test image towards the end.  The trained "yolov4-crowdhuman-416x416" model correctly predicts all "heads" and "persons" in the picture, including partially occluded objects.

![A sample prediction using the trained "yolov4-crowdhuman-416x416" model](/assets/2020-09-13-colab-yolov4/predictions_sample.jfif)

Finally, you should be able to find the trained "yolov4-crowdhuman-416x416.weights" file in the "yolov4_crowdhuman" directory on your Google Drive, and the corresponding cfg file on GitHub: [yolov4-crowdhuman-416x416.cfg](https://github.com/jkjung-avt/yolov4_crowdhuman/blob/master/cfg/yolov4-crowdhuman-416x416.cfg).  You'd use these 2 files as a people/head detector and run inference.

# About the design and customization

* While developing the Colab Notebook, I decide to put training data on local storage of the virtual machine.  More specifically, the original dataset files are downloaded and unzipped into local drive in Step 2.  (There are over 30GB of local storage space avaialble on the virtual machines, enough to hold all training/validation data of the CrowdHuman dataset.)  Since the training data is read repeatedly and extensively during training, it is crucial to optimize its loading so as to train the model efficiently.  I think the training would have been much more slowly if the training data is on a remote storage such as Google Drive or Google Cloud Storage.

* When generating training labels, I filter out objects that are too small to be learned.  You could refer to the source code [here](https://github.com/jkjung-avt/yolov4_crowdhuman/blob/master/data/gen_txts.py#L58).  By excluding objects that are too small, I have made the learning objectives clearer for the model and the resulting model performs better in terms of accuracy.

* YOLO models use k-means algorithm to determine sizes of "anchors" ([reference](https://stackoverflow.com/questions/56442413/generating-anchor-boxes-using-k-means-clustering-yolo)).  I use my own script, [data/gen_txts.py](https://github.com/jkjung-avt/yolov4_crowdhuman/blob/master/data/gen_txts.py#L143), to determine these anchors.  Alternatively, you could refer to [official documentation](https://github.com/AlexeyAB/darknet#how-to-use-on-the-command-line) and use "darknet" to calculate anchor sizes.  Based on my own testing, both produce similar results (in terms of mAP).

   ```
   $ ./darknet detector calc_anchors data/crowdhuman-416x416.data -num_of_clusters 9 -width 416 -height 416
   ```

* For shortening overall training time (to fit in the 7~8 hour Colab session window), I set a smaller [batch size](https://github.com/jkjung-avt/yolov4_crowdhuman/blob/master/cfg/yolov4-crowdhuman-416x416.cfg#L3) and [total number of training iterations](https://github.com/jkjung-avt/yolov4_crowdhuman/blob/master/cfg/yolov4-crowdhuman-416x416.cfg#L20) in the "yolov4-crowdhuman-416x416.cfg" file.  This does have a slightly negative impact on accuracy (mAP) of the trained model.  If you could afford longer training time, you could consider the following modifications in the cfg file in order to train a model with better mAP:

   - increase "batch_size" and "subdivisions",
   - adjust "learning_rate" accordingly (usually set larger "learning_rate" for larger "batch_size"),
   - increase "max_batches" and "steps".

* For training the model for a longer time, you could try to split the training across multiple Colab sessions.  The key would be in Step 7.  Instead of using "yolov4.conv.137" as the pre-trained weights, you could use the "last" saved weights from the previous Colab session.  This way, the model would be fine-tuned based on checkpoint from the last training run.  For example,

   ```
   %cd /content/yolov4_crowdhuman/darkne
   !./darknet detector train data/crowdhuman-{INPUT_SHAPE}.data cfg/yolov4-crowdhuman-{INPUT_SHAPE}.cfg backup/yolov4-crowdhuman-{INPUT_SHAPE}_last.weights -gpu 0 -map -dont_show 2>&1 | tee "{DRIVE_SAVE_DIR}"/train.log | grep -E "hours left|mean_average"
   ```

   Be sure to also adjust "learning_rate", "max_batches", and "steps" in the cfg file if you do this.

# Next steps

* If you are adapting the training code to another dataset, you should definitely read the README.md in my [jkjung-avt/yolov4_crowdhuman](https://github.com/jkjung-avt/yolov4_crowdhuman) repo to understand how the training data is prepared and then develop your own code to prepare the training data.

* I plan to deploy the trained "yolov4-crowdhuman-416x416" model with TensorRT on my Jetson Nano DevKit.  I shall share that in my next blog post.
