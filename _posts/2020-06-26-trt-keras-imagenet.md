---
layout: post
comments: true
title: "Applying TensorRT on My tf.keras ImageNet Models"
excerpt: "This post explains how I optimize my trained tf.keras ImageNet models with TensorRT.  The main steps involve converting the tf.keras models to ONNX, and then to TensorRT engines."
date: 2020-06-25
category: "tensorrt"
tags: tensorrt
---

Quick link: [jkjung-avt/keras_imagenet](https://github.com/jkjung-avt/keras_imagenet)

Last year I wrote about [Training Keras Models with TFRecords and The tf.data API](https://jkjung-avt.github.io/tfrecords-for-keras/).  And I've since trained my own tf.keras "googlenet_bn", "inception_v2", "mobilenet_v2", "efficientnet_b0", and "osnet" models.  I shared those models (including trained weights) on my GoogleDrive, as linked in [keras_imagenet/README.md](https://github.com/jkjung-avt/keras_imagenet/blob/master/README.md).

Recently, I've also tried to optimized those trained tf.keras models with TensorRT.  I think it's a good idea to share my experience in this regard, thus this blog post.

# How to optimize the tf.keras models with TensorRT

In order to follow along, you'd need to have a trained tf.keras model (.h5).  You could either use such a model trained by yourself, or you could download one of the models I've shared on GoogleDrive.  I used my own trained [googlenet_bn-acc_0.7091.h5](https://drive.google.com/open?id=1EW-ShppeSkaaqSDiaHIojWEil0jMR93k) in my step-by-step guide.

The procedure of optimizing the tf.keras model with TensorRT could be broken down into:

1. Convert the tf.keras (.h5) model to a tensorflow frozen inference graph (.pb),
2. Convert the frozen inference graph (.pb) to ONNX (.onnx) by using "tf2onnx",
3. Use TensorRT's ONNX parser to read the ONNX (.onnx) file, optimize the model, and save it as the final TensorRT engine (.engine).

I mainly referenced NVIDIA's blog post, [Speeding up Deep Learning Inference Using TensorFlow, ONNX, and TensorRT](https://developer.nvidia.com/blog/speeding-up-deep-learning-inference-using-tensorflow-onnx-and-tensorrt/), for how to do all these steps.

For a detailed step-by-step guide, please refer to my [keras_imagenet/README_tensorrt.md](https://github.com/jkjung-avt/keras_imagenet/blob/master/README_tensorrt.md).  It should work on either a Linux x86_64 PC or a NVIDIA Jetson system, provided that you have the environment (prerequisites) set up properly.

In addition to "googlenet_bn", I also verified the procedure working for my own tf.keras "inception_v2" and "mobilenet_v2" models.  In fact, I think it would work for any model which does not require a plugin.  (Note that it most likely won't work for "efficientnet_b0" because the "swish" activation is not natively supported by TensorRT.)

# Discussions

* As shown in my step-by-step logs, I used tensorflow-1.14.0 and onnx-1.4.1 on my x86_64 PC for testing.  I think the code should work for both TensorRT 6 and TensorRT 7.  There could be version dependencies among these packages.  You might try different versions of onnx or tensorflow if you encounter some problems by following my step-by-step guide.

* I extended my [predict_image.py](https://github.com/jkjung-avt/keras_imagenet/blob/master/predict_image.py) code to support inferencing with TensorRT engines, in addition to tf.keras models.  Please refer to the source code for details.

* In my "googlenet_bn" example, I did not see a difference between inference results of the tf.keras model and the TensorRT engine, i.e. the softmax outputs seem to match.  But since I've applied FP16 optimization on the TensorRT engine, it might actually produce slightly different results from the original model.

    ```
    ### First test with the tf.keras model
    $ python3 predict_image.py saves/googlenet_bn-acc_0.7091.h5 \
                               huskies.jpg
    0.96   n03218198 dogsled, dog sled, dog sleigh
    0.03   n02109961 Eskimo dog, husky
    0.00   n02110185 Siberian husky
    0.00   n02110063 malamute, malemute, Alaskan malamute
    0.00   n02114367 timber wolf, grey wolf, gray wolf, Canis lupus
    ### Then run the TensorRT engine and compare
    $ python3 predict_image.py tensorrt/googlenet_bn.engine \
                               huskies.jpg
    0.96   n03218198 dogsled, dog sled, dog sleigh
    0.03   n02109961 Eskimo dog, husky
    0.00   n02110185 Siberian husky
    0.00   n02110063 malamute, malemute, Alaskan malamute
    0.00   n02114367 timber wolf, grey wolf, gray wolf, Canis lupus
    ```

* Finally, "trtexec" is a great tool for profiling TensorRT engines.  Refer to step #3 in my [keras_imagenet/README_tensorrt.md](https://github.com/jkjung-avt/keras_imagenet/blob/master/README_tensorrt.md).  When I specified the "--dumpProfile" command-line option, I got the following output for my [googlenet_bn-acc_0.7091.h5](https://drive.google.com/open?id=1EW-ShppeSkaaqSDiaHIojWEil0jMR93k) model.  It contained some very useful information, such as:

    - which layers of the original model have been merged into one operation (or one CUDA kernel),
    - execution time/percentage of each of the operations in the TensorRT engine,
    - which operations/layers should be first considered if we'd like to speed up the TensorRT engine.

    ```
    [06/26/2020-13:30:20] [I] === Profile (3786 iterations ) ===
    [06/26/2020-13:30:20] [I]                                                                                                                    Layer   Time (ms)   Avg. Time (ms)   Time
    %
    [06/26/2020-13:30:20] [I]                                                                                                            Logits/MatMul        5.56             0.00      0.2
    [06/26/2020-13:30:20] [I]                                                                          Logits/BiasAdd + (Unnamed Layer* 199) [Shuffle]        5.70             0.00      0.2
    [06/26/2020-13:30:20] [I]                                                                                                         conv2d/Conv2D__6       19.35             0.01      0.8
    [06/26/2020-13:30:20] [I]                                                                      conv2d/Conv2D + activation/Relu input reformatter 0       17.63             0.00      0.7
    [06/26/2020-13:30:20] [I]                                                                                          conv2d/Conv2D + activation/Relu       90.56             0.02      3.7
    [06/26/2020-13:30:20] [I]                                                                                                    max_pooling2d/MaxPool       21.01             0.01      0.8
    [06/26/2020-13:30:20] [I]                                                                  conv2d_1/Conv2D + activation_1/Relu input reformatter 0       15.40             0.00      0.6
    [06/26/2020-13:30:20] [I]                                                                                      conv2d_1/Conv2D + activation_1/Relu       31.14             0.01      1.3
    [06/26/2020-13:30:20] [I]                                                                                      conv2d_2/Conv2D + activation_2/Relu       72.64             0.02      2.9
    [06/26/2020-13:30:20] [I]                                                                              max_pooling2d_1/MaxPool input reformatter 0       21.02             0.01      0.8
    [06/26/2020-13:30:20] [I]                                                                                                  max_pooling2d_1/MaxPool       18.50             0.00      0.7
    [06/26/2020-13:30:20] [I]                                                                                                average_pooling2d/AvgPool       16.54             0.00      0.7
    [06/26/2020-13:30:20] [I]        conv2d_3/Conv2D + activation_3/Relu || conv2d_6/Conv2D + activation_6/Relu || conv2d_4/Conv2D + activation_4/Relu       44.64             0.01      1.8
    [06/26/2020-13:30:20] [I]                                                                                      conv2d_8/Conv2D + activation_8/Relu       31.56             0.01      1.3
    [06/26/2020-13:30:20] [I]                                                                                      conv2d_7/Conv2D + activation_7/Relu       27.27             0.01      1.1
    [06/26/2020-13:30:20] [I]                                                                                      conv2d_5/Conv2D + activation_5/Relu       51.50             0.01      2.1
    [06/26/2020-13:30:20] [I]                                                                                                 activation_3/Relu:0 copy       12.46             0.00      0.5
    [06/26/2020-13:30:20] [I]                                                                                              average_pooling2d_1/AvgPool       17.43             0.00      0.7
    [06/26/2020-13:30:20] [I]    conv2d_10/Conv2D + activation_10/Relu || conv2d_9/Conv2D + activation_9/Relu || conv2d_12/Conv2D + activation_12/Relu       66.32             0.02      2.7
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_14/Conv2D + activation_14/Relu       33.15             0.01      1.3
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_11/Conv2D + activation_11/Relu       67.77             0.02      2.7
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_13/Conv2D + activation_13/Relu       45.58             0.01      1.8
    [06/26/2020-13:30:20] [I]                                                                                                 activation_9/Relu:0 copy       12.68             0.00      0.5
    [06/26/2020-13:30:20] [I]                                                                                                  max_pooling2d_2/MaxPool       19.04             0.01      0.8
    [06/26/2020-13:30:20] [I]                                                                                              average_pooling2d_2/AvgPool       17.28             0.00      0.7
    [06/26/2020-13:30:20] [I]  conv2d_15/Conv2D + activation_15/Relu || conv2d_18/Conv2D + activation_18/Relu || conv2d_16/Conv2D + activation_16/Relu       52.25             0.01      2.1
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_20/Conv2D + activation_20/Relu       43.07             0.01      1.7
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_19/Conv2D + activation_19/Relu       22.86             0.01      0.9
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_17/Conv2D + activation_17/Relu       48.05             0.01      1.9
    [06/26/2020-13:30:20] [I]                                                                                                activation_15/Relu:0 copy       12.07             0.00      0.5
    [06/26/2020-13:30:20] [I]  conv2d_21/Conv2D + activation_21/Relu || conv2d_24/Conv2D + activation_24/Relu || conv2d_22/Conv2D + activation_22/Relu       55.50             0.01      2.2
    [06/26/2020-13:30:20] [I]                                                                                              average_pooling2d_3/AvgPool       17.02             0.00      0.7
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_26/Conv2D + activation_26/Relu       44.86             0.01      1.8
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_25/Conv2D + activation_25/Relu       31.34             0.01      1.3
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_23/Conv2D + activation_23/Relu       50.01             0.01      2.0
    [06/26/2020-13:30:20] [I]                                                                                                activation_21/Relu:0 copy       11.78             0.00      0.5
    [06/26/2020-13:30:20] [I]                                                                                              average_pooling2d_4/AvgPool       17.16             0.00      0.7
    [06/26/2020-13:30:20] [I]  conv2d_30/Conv2D + activation_30/Relu || conv2d_27/Conv2D + activation_27/Relu || conv2d_28/Conv2D + activation_28/Relu       55.53             0.01      2.2
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_32/Conv2D + activation_32/Relu       41.42             0.01      1.7
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_31/Conv2D + activation_31/Relu       30.93             0.01      1.2
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_29/Conv2D + activation_29/Relu       52.67             0.01      2.1
    [06/26/2020-13:30:20] [I]                                                                                                activation_27/Relu:0 copy       12.07             0.00      0.5
    [06/26/2020-13:30:20] [I]                                                                                              average_pooling2d_5/AvgPool       17.24             0.00      0.7
    [06/26/2020-13:30:20] [I]  conv2d_33/Conv2D + activation_33/Relu || conv2d_36/Conv2D + activation_36/Relu || conv2d_34/Conv2D + activation_34/Relu       53.78             0.01      2.2
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_38/Conv2D + activation_38/Relu       44.60             0.01      1.8
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_37/Conv2D + activation_37/Relu       27.91             0.01      1.1
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_35/Conv2D + activation_35/Relu       56.88             0.02      2.3
    [06/26/2020-13:30:20] [I]                                                                                                activation_33/Relu:0 copy       12.19             0.00      0.5
    [06/26/2020-13:30:20] [I]                                                                                              average_pooling2d_6/AvgPool       17.21             0.00      0.7
    [06/26/2020-13:30:20] [I]  conv2d_39/Conv2D + activation_39/Relu || conv2d_42/Conv2D + activation_42/Relu || conv2d_40/Conv2D + activation_40/Relu       73.35             0.02      3.0
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_44/Conv2D + activation_44/Relu       65.87             0.02      2.7
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_43/Conv2D + activation_43/Relu       30.30             0.01      1.2
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_41/Conv2D + activation_41/Relu       60.40             0.02      2.4
    [06/26/2020-13:30:20] [I]                                                                                                activation_39/Relu:0 copy       11.88             0.00      0.5
    [06/26/2020-13:30:20] [I]                                                                                                  max_pooling2d_3/MaxPool       15.55             0.00      0.6
    [06/26/2020-13:30:20] [I]                                                                                              average_pooling2d_7/AvgPool       16.25             0.00      0.7
    [06/26/2020-13:30:20] [I]  conv2d_45/Conv2D + activation_45/Relu || conv2d_48/Conv2D + activation_48/Relu || conv2d_46/Conv2D + activation_46/Relu       53.48             0.01      2.2
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_50/Conv2D + activation_50/Relu       55.56             0.01      2.2
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_49/Conv2D + activation_49/Relu       26.57             0.01      1.1
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_47/Conv2D + activation_47/Relu       56.91             0.02      2.3
    [06/26/2020-13:30:20] [I]                                                                                                activation_45/Relu:0 copy       12.00             0.00      0.5
    [06/26/2020-13:30:20] [I]                                                                                              average_pooling2d_8/AvgPool       16.82             0.00      0.7
    [06/26/2020-13:30:20] [I]  conv2d_51/Conv2D + activation_51/Relu || conv2d_54/Conv2D + activation_54/Relu || conv2d_52/Conv2D + activation_52/Relu       63.29             0.02      2.6
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_56/Conv2D + activation_56/Relu       56.19             0.01      2.3
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_55/Conv2D + activation_55/Relu       31.95             0.01      1.3
    [06/26/2020-13:30:20] [I]                                                                                    conv2d_53/Conv2D + activation_53/Relu       65.61             0.02      2.7
    [06/26/2020-13:30:20] [I]                                                                                                activation_51/Relu:0 copy       12.00             0.00      0.5
    [06/26/2020-13:30:20] [I]                                                                                                           Transpose__648       16.14             0.00      0.7
    [06/26/2020-13:30:20] [I]                                                                        global_average_pooling2d/Mean input reformatter 0       14.01             0.00      0.6
    [06/26/2020-13:30:20] [I]                                                                                            global_average_pooling2d/Mean       32.28             0.01      1.3
    [06/26/2020-13:30:20] [I]                                                                                   (Unnamed Layer* 197) [Matrix Multiply]       47.47             0.01      1.9
    [06/26/2020-13:30:20] [I]                                                                                       (Unnamed Layer* 200) [ElementWise]       14.91             0.00      0.6
    [06/26/2020-13:30:20] [I]                                                                                           (Unnamed Layer* 209) [Softmax]       16.41             0.00      0.7
    [06/26/2020-13:30:20] [I]                                                                                                                    Total     2475.34             0.65    100.
    0
    [06/26/2020-13:30:20] [I]
    ```
