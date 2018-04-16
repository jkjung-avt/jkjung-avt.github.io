---
layout: post
comments: true
title: "Keras Cats Dogs Tutorial"
excerpt: "In this post I demonstrated how to train a very powerful Keras image classifier with just a few lines of Python code. The code could be adapted to handle other image classification tasks very easily."
date: 2018-04-14
category: "keras"
tags: keras
---

Quick link to my GitHub code: [https://github.com/jkjung-avt/keras-cats-dogs-tutorial](https://github.com/jkjung-avt/keras-cats-dogs-tutorial)

Recently I started to learn [Keras](https://keras.io/). Overall I found Keras did live up up to its fame. It is quite user friendly and easy to learn, while also being extensible for handling more recent and advanced deep learning tasks.

While there are many good examples online to get you started tackling image classification tasks using Keras, most of them are lacking in terms of how to take advantage of Keras' built-in image augmentation functionalities to achieve best classification accuracy. Nonetheless, the following article on 'The Keras Blog' serves as a good starting point in that direction.

[Building powerful image classification models using very little data](https://blog.keras.io/building-powerful-image-classification-models-using-very-little-data.html)

I wanted to build on it and show how to do better.

# Prerequisite

* Have Keras with TensorFlow banckend installed on your deep learning PC or server.

In my own case, I usd `tensorflow-gpu (1.7.0)` and the Keras package within. Reference: [Installing TensorFlow on Ubuntu](https://www.tensorflow.org/install/install_linux)

# Step-by-step

1. Prepare train/validation data

   * Download `train.zip` from the [Kaggle Dogs vs. Cats page](https://www.kaggle.com/c/dogs-vs-cats/data)
   * Following the (Keras Blog) example above, we would be working on a much reduced dataset with only 1,000 pictures of cats and 1,000 of dogs. In addition, we would take some additional 400 pictures of cats and 400 of dogs as the validation set.

   ```
   $ cd /data
   $ mkdir catsdogs
   $ cd catsdogs
   $ unzip ~/Downloads/train.zip
   $ cd train
   $ mkdir -p ../sample/train/cats
   $ cp cat.?.jpg cat.??.jpg cat.???.jpg ../sample/train/cats/
   $ mkdir -p ../sample/train/dogs
   $ cp dog.?.jpg dog.??.jpg dog.???.jpg ../sample/train/dogs/
   $ mkdir -p ../sample/valid/cats
   $ cp cat.1[0-3]??.jpg ../sample/valid/cats/
   $ mkdir -p ../sample/valid/dogs
   $ cp dog.1[0-3]??.jpg ../sample/valid/dogs/
   ```

   After the steps above we ended up with the folllwing, which is how Keras' [ImageDataGenerator.flow_from_directory()](https://keras.io/preprocessing/image/) expects data to be organized.

   ```
   /data/catsdogs/sample/
       train/
           cats/
               cat.0.jpg
               ...
               cat.999.jpg
           dogs/
               dog.0.jpg
               ...
               dog.999.jpg
       valid/
           cats/
               cat.1000.jpg
               ...
               cat.1399.jpg
           dogs/
               dog.1000.jpg
               ...
               dog.1399.jpg
   ```

2. Download the code from my GitHub repository

   ```
   $ cd ~/project
   $ git clone https://github.com/jkjung-avt/keras-cats-dogs-tutorial.git
   $ cd keras-cats-dogs-tutorial
   ```

3. Train a ResNet50 based classifier on the dataset

   Run the following command. Note that I was getting over 98% accuracy on the validation set with only the 1st epoch of training. And after 20 epochs I got a classifier with **99% validation accuracy**. That was significantly better than the Keras Blog example.

   ```
   $ python3 train_rest50.py
   ......
   Found 2000 images belonging to 2 classes.
   Found 800 images belonging to 2 classes.
   ****************
   Class #0 = cats
   Class #1 = dogs
   ****************
   ......
   Epoch 1/20
   250/250 [==============================] - 41s 163ms/step - loss: 0.3811 - acc: 0.8155 - val_loss: 0.0647 - val_acc: 0.9825
   Epoch 2/20
   250/250 [==============================] - 35s 140ms/step - loss: 0.1790 - acc: 0.9290 - val_loss: 0.0513 - val_acc: 0.9850
   ......
   Epoch 20/20
   250/250 [==============================] - 35s 140ms/step - loss: 0.0192 - acc: 0.9940 - val_loss: 0.0317 - val_acc: 0.9900
   ```

4. Test the trained model

   The trained model should have been saved as `model-resnet50-final.h5` in the previous step. Here I picked a picture from the original dataset for testing. And the model predicted this picture being a dog (with 1.000 probability), which was good~

   ```
   $ python3 predict /data/catsdogs/train/dog.12499.jpg
   ......
   /data/catsdogs/train/dog.12499.jpg
       1.000  dogs
       0.000  cats
   ```

   ![dog.12499.jpg](/assets/2018-04-14-keras-tutorial/dog.12499.jpg)

# Discussion

The Keras Blog example used a pre-trained VGG16 model and reached ~94% validation accuracy on the same dataset. I think my code was able to achieve much better accuracy because:

* I used a stronger pre-trained model, **ResNet50**.
* I trained the classifier with **larger images** (224x224, instead of 150x150) images.
* I did pretty heavy **data augmentation** on the training images. For this, I took advantage of Keras' ImageDataGenerator's built-in image augmentation functionalities, including random rotation, shift in the x and y directions, shearing, zooming, adding noise (channel shift), and horizontal flipping, etc.

```python
    train_datagen = ImageDataGenerator(preprocessing_function=preprocess_input,
                                       rotation_range=40,
                                       width_shift_range=0.2,
                                       height_shift_range=0.2,
                                       shear_range=0.2,
                                       zoom_range=0.2,
                                       channel_shift_range=10,
                                       horizontal_flip=True,
                                       fill_mode='nearest')
```

My `train_resnet50.py` and `predict_resnet50.py` scripts could be very easily adapted to handle other image classification datasets. Feel free to give it a try and leave a comment below if any.
