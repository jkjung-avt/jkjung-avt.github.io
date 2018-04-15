---
layout: post
comments: true
title: "Keras InceptionResNetV2"
excerpt: "With change of only 3 lines of code from my previous example, I was able to use the more powerful CNN model, 'InceptionResNetV2', to train a Cats vs. Dogs classifier."
date: 2018-04-15
category: "keras"
tags: keras
---

Quick link to my GitHub code: [https://github.com/jkjung-avt/keras-cats-dogs-tutorial](https://github.com/jkjung-avt/keras-cats-dogs-tutorial)

In the previous post I built a pretty good Cats vs. Dogs classifier (with a pretty small training set) based on Keras' built-in 'ResNet50' model. In the post I'd like to show how easy it is to modify the code to use an even more powerful CNN model, 'InceptionResNetV2'.

One of the really nice features of Keras is it comes with quite a few pretty modern pre-trained CNN models. Referring to Keras' [Applications documentation](https://keras.io/applications/):

| Model             | Size  | Top-1 Accuracy | Top-5 Accuracy |
| ----------------- |:-----:|:--------------:|:--------------:|
| VGG16             | 528MB |      0.715     |      0.901     |
| ResNet50          |  99MB |      0.759     |      0.929     |
| InceptionV3       |  92MB |      0.788     |      0.944     |
| Xception          |  88MB |      0.790     |      0.945     |
| InceptionResNetV2 | 215MB |      0.804     |      0.953     |

I wanted to see if I could further improve accuracy of the Cats vs. Dogs classifier by adopting a more powerful CNN frontend. More specifically, I'd like to try 'InceptionResNetV2'. To do that, I only needed to change 3 lines of code from my previous `train_resnet50.py` script!

I modified the following 3 lines in `train_resnet50.py`:

```python
......
from tensorflow.python.keras.applications.resnet50 import ResNet50, preprocess_input
......
IMAGE_SIZE    = (224, 224)
......
net = ResNet50(include_top=False, weights='imagenet', input_tensor=None,
               input_shape=(IMAGE_SIZE[0],IMAGE_SIZE[1],3))
......
```

And I created another `train_inceptionresnetv2.py` script as follows:

```python3
......
from tensorflow.python.keras.applications.inception_resnet_v2 import InceptionResNetV2, preprocess_input
......
MAGE_SIZE    = (299, 299)
......
net = InceptionResNetV2(include_top=False,
                        weights='imagenet',
                        input_tensor=None,
                        input_shape=(IMAGE_SIZE[0],IMAGE_SIZE[1],3))
......
```

The resulting full script could be found [here](https://github.com/jkjung-avt/keras-cats-dogs-tutorial/blob/master/train_inceptionresnetv2.py)

I ran the `train_inceptionresnetv2.py` script with the same reduced dataset (1,000 cats + 1,000 dogs), and with the same data augmentations. At the end of 20 epochs I got a classifier with **validation accuracy at 98.12%**. While this result was not as good as ResNet50, I thought it could be reasonable. The task of distinguishing cats from dogs is probably not challenging enough for these modern CNN models. The pre-trained CNN models learns this task very quickly and basically stops improving after just a couple of epochs. And thus the more advanced InceptionResNetV2 doesn't do better than ResNet50 on this task.

```shell
$ python3 train_inceptionresnetv2.py
......
Epoch 20/20
250/250 [==============================] - 90s 358ms/step - loss: 0.0160 - acc: 0.9945 - val_loss: 0.0675 - val_acc: 0.9812
```

