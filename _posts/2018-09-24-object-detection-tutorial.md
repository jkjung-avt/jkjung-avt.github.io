---
layout: post
comments: true
title: "Adapting the Hand Detector Tutorial to Your Own Data"
excerpt: "This is a tutorial on how to adapt my 'hand detector' to other object detection tasks.  You should be able to train your own models to detect other kinds of objects with very little change to my code."
date: 2018-09-24
category: "tensorflow"
tags: tensorflow
---

Quick link: [jkjung-avt/hand-detection-tutorial](https://github.com/jkjung-avt/hand-detection-tutorial)

To follow up on my previous post, [Training a Hand Detector with TensorFlow Object Detection API](https://jkjung-avt.github.io/hand-detection-tutorial/), I'd like to discuss how to adapt the code in that tutorial and train models which could detect other kind of objects.  Make sure you have followed the previous tutorial with the training environment properly set up, and are able to train the hand detector (SSD model) successfully first.  And read on.

# Preparing the training data

In the previous tutorial, I first converted 'egohands' annotations into KITTI format.  Then I was able to use one of the [dataset_tools](https://github.com/tensorflow/models/tree/master/research/object_detection/dataset_tools) available in the original object_detection repository to convert data into TFRecord files.  The benefit here is that I could leverage existing code without creating and maintaining another program to do this.

You could reference the `convert_one_folder()` and `box_to_line()` functions in `prepare_egohands.py` to see how I generated annotations in KITTI format.  If your dataset is stored in another format, reference my code and develop your own script to convert it into KITTI format.  We would generally allocated 10~20% of all images into validation set.  But if you have a very large dataset, you might just reserve enough number of images for validation and increase the percentage of images for training.

Note that class name in the KITTI annotaions has been [hardcoded](https://github.com/jkjung-avt/hand-detection-tutorial/blob/master/prepare_egohands.py#L139) as 'hand' in `box_to_line()` in `prepare_egohands.py`.  When you are preparing training data for your own object detector, you'll need to modify that so that the correct class names are encoded in the KITTI annotations.

There are a few other modifications needed when you train your own object detection model for other classes of objects.

* In `create_tfrecords.sh`, specify all classes in `--classes_to_use=` as a comma separated list (for example, `--classes_to_use=car,pedestrian`).  Refer to comments in `create_kitti_tf_record.py` for more details.

* You need to create your own label map file.  Refer to `data/egohands_label_map.pbtxt`.  All classes should be assigned with an id (1, 2, 3, ...) in the label map.  Note the id starts at 1, as 0 is reserved for 'background'.

* You need to specify the number of classes correctly in the model config file.  Take a look at `configs/ssd_mobilenet_v1_egohands.config`.  The `model -> ssd -> num_classes` should be set to, say, 2 if you have 2 classes ('car' and 'pedestrian') to be detected.

In addition, check how many labeled image files you have in your own dataset.  In `create_tfrecords.sh` I set `--validation_set_size` to `500` so that 500 of the images in egohands dataset would go to the validation set, while the remaining (4,800 - 500 = 4,300) to the training set.  We would generally allocate 10~20% of all images to the validation set.  But if you have a large amount of training images to work with, you might just reserve a fixed number for validation and thus increase the precentage of images in the training set.

In `create_kitti_tf_record.py` I randomly shuffled the images each time before splitting the train/val sets.  If you don't like that behavior, you could remove [that line of code](https://github.com/jkjung-avt/hand-detection-tutorial/blob/master/create_kitti_tf_record.py#L113), or set a fixed random seed so that you get a fixed split every time.

# Configuring your own object detection model

The official documentation about how to do this could be found below.

* [Quick Start: Distributed Training on the Oxford-IIIT Pets Dataset on Google Cloud](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/running_pets.md)
* [Configuring the Object Detection Training Pipeline](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/configuring_jobs.md)

I will use my `configs/ssd_mobilenet_v1_egohands.config` as example.  When configuring the model for your own object detection dataset, you'll need to pay attention to the following.

1. Set `num_classes`, as stated above.

2. Set the paths to your TFRecord files.

3. In `eval_config` section, set `num_examples` to the number of images in your validation set.

4. In `train_config` section, set `num_steps` (number of iterations) to a proper value.  As a rule of thumb, we usually want to train the model for at least, say, 10~20 epochs.  For large models trained from scratch, people usually train it for 200 epochs.  In the egohands example, I set this number to 20,000.  This corresponds to (20,000 * 24 / 4,300) = 111 epochs.  The 24 is batch size.

5. In the `train_config -> optimizer -> ...`, set `decay_steps` and `decay_factor` to proper values.  Note that learning rate and its decaying schedule is a very important hyper-parameter for training.  You could reference my settings and try to find what works best for your training sessions.  You could also try other optimizers if you want.  TensorFlow Object Detection API supports 'momentum_optimizer' and 'adam_optimizer', in addition to 'rms_prop_optimizer' ([reference](https://github.com/tensorflow/models/blob/master/research/object_detection/protos/optimizer.proto)).

6. (Optional) You might also try to add some data augmentation ([reference](https://stackoverflow.com/questions/44906317/what-are-possible-values-for-data-augmentation-options-in-the-tensorflow-object)) in the config file.  This could be especially useful if you have a relatively small dataset.

# Training and Evaluating

Refer to `train.sh` and `eval.sh` in my tutorial repo.  It is straightforward to modify those scripts for your own data/model.

So, that's it.  I think it's rather easy to adapt my hand detection tutorial and code for other object detection tasks.  Give it a try and leave a comment below.  Do let me know if you find any issues.  Otherwise, I'd also be thrilled to hear from you if you adapt my tutorial and solve your own object detection problems.
