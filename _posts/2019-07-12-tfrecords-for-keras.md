---
layout: post
comments: true
title: "Training Keras Models with TFRecords and The tf.data API"
excerpt: "One of the challenges in training CNN models with a large image dataset lies in building an efficient data ingestion pipeline.  Without that, the GPU's could be constantly starving for data and thus training goes slowly.  In this post, I'm sharing my experience in training Keras image classification models with tensorflow's TFRecords and tf.data API.  I think I train the models much more efficiently this way than reading original jpg files from the file system."
date: 2019-07-12
category: "nano"
tags: keras tensorflow
---

Quick link: [jkjung-avt/keras_imagenet](https://github.com/jkjung-avt/keras_imagenet)

One of the challenges in training CNN models with a large image dataset lies in building an efficient data ingestion pipeline.  Without that, the GPU's could be constantly starving for data and thus training goes slowly.  In this post, I'm sharing my experience in training Keras image classification models with tensorflow's TFRecords and tf.data API.  I think I train the models much more efficiently this way than reading original jpg files from the file system.

More specifically, I share the code I used to train Keras ImageNet (ILSVRC2012: 1,000 classes) image classification models.  And I try to explain my use of [TFRecords](https://www.tensorflow.org/tutorials/load_data/tf_records) and the [tf.data.TFRecordDataset API](https://www.tensorflow.org/api_docs/python/tf/data/TFRecordDataset) below.

# Reference

* [Official TensorFlow guide on 'Importing Data'](https://www.tensorflow.org/guide/datasets)
* [tf.data.TFRecordDataset](https://www.tensorflow.org/api_docs/python/tf/data/TFRecordDataset)
* [Working with TFRecords and tf.train.Example](https://towardsdatascience.com/working-with-tfrecords-and-tf-train-example-36d111b3ff4d) -> This tutorial explains `tf.train.Example` very well.  I do recommned reading it.
* [Data Input Pipeline Performance](https://www.tensorflow.org/guide/performance/datasets)
* [how can I use Dataset to shuffle a large whole dataset?](https://github.com/tensorflow/tensorflow/issues/14857#issuecomment-347261151)
* [Training and Serving ML models with tf.keras](https://medium.com/tensorflow/training-and-serving-ml-models-with-tf-keras-fd975cc0fa27)
* [Three Ways of Storing and Accessing Lots of Images in Python](https://realpython.com/storing-images-in-python/)

# About TFRecords

  Quote from [tensorflow documentation](https://www.tensorflow.org/tutorials/load_data/tf_records):

  > To read data efficiently it can be helpful to serialize your data and store it in a set of files (100-200MB each) that can each be read linearly. This is especially true if the data is being streamed over a network. This can also be useful for caching any data-preprocessing.
  >
  > The TFRecord format is a simple format for storing a sequence of binary records.

  Check out the last linked article in the 'Reference' section.  In short, image data (especially large amount of data) could be read from disk much more efficientlt if the data is stored as aggregated and serialized database/records file(s), rather than as separate jpg files.

  So TFRecords would be the format I use for training Keras models discussed in this post.

# Creating TFRecords for ImageNet (ILSVRC2012) training data

  Please check out the 'Step-by-step' guide in my [jkjung-avt/keras_imagenet](https://github.com/jkjung-avt/keras_imagenet) repository for how to create the TFRecords files.  I mainly took and modified the `build_imagenet_data.py` script from tensorflow's [inception model code](https://github.com/tensorflow/models/tree/master/research/inception/inception/data).

  The script splits the training set (1,281,167 images) into 1,024 *shards*, and the validation set (50,000 images) into 128 *shards*.  When done, each *shard* file would contain roughly the same number of jpg files.  The image data in the shard files stays jpg encoded, otherwise the TFRecords files would take too much space.

  When done, contents of my `${HOME}/data/ILSVRC2012/tfrecords/` directory are:

  ```
  train-00000-of-01024
  train-00001-of-01024
  ...
  train-01023-of-01024
  validation-00000-of-00128
  validation-00001-of-00128
  ...
  validation-00127-of-00128
  ```

# Reading TFRecords and creating randomly shuffled data while training

  To be more precise, we would want to parallelize the tasks of reading data from TFRecords files, randomizing the data, and data augmentation efficiently.  TensorFlow's [Data Input Pipeline Performance](https://www.tensorflow.org/guide/performance/datasets#input_pipeline_structure) documentation roughly describes how to do this.  However, I found the code samples in that document a little bit confusing.

  After some research, I found [mrry's suggestion on GitHub](https://github.com/tensorflow/tensorflow/issues/14857) most helpful of achieving what I'd like to do.  I ended up doing the following:

  1. Create a `tf.data.Dataset` which is a list of the TFRecords (shard) file names: either 'train-xxxxx-of-01024' or 'validation-xxxxx-of-00128'.
  2. Next, `shuffle()` and `repeat()` the `shards` Dataset.  So `shards` would generate shard file names *in random order and indefinitely*.
  3. Feed and `interleave()` (randomize more) the `shards` Dataset into `tf.data.TFRecordsDataset`.  This results in a `TFRecordsDataset` which reads from shard files in random order.
  4. Shuffle the TFRecordsDataset so that the order of training images within a shard is randomized in each epoch.
  5. Parse and deserialize the TFRecords, with 'prefetching'.  I set `num_parallel_calls` to 4 on my desktop.  You could adjust the value you want more parallel wrokers to do data generation and augmentation.  And finally note that I implement jpg decoding and data augmentation in the deserialization function (`parse_fn_train`).

  You could find my full python code implementation [here](https://github.com/jkjung-avt/keras_imagenet/blob/master/utils/dataset.py#L119).

  ```python
  def get_dataset(tfrecords_dir, subset, batch_size):
      """Read TFRecords files and turn them into a TFRecordDataset."""
      files = tf.matching_files(os.path.join(tfrecords_dir, '%s-*' % subset))
      shards = tf.data.Dataset.from_tensor_slices(files)
      shards = shards.shuffle(tf.cast(tf.shape(files)[0], tf.int64))
      shards = shards.repeat()
      dataset = shards.interleave(tf.data.TFRecordDataset, cycle_length=4)
      dataset = dataset.shuffle(buffer_size=8192)
      parser = parse_fn_train if subset == 'train' else parse_fn_valid
      dataset = dataset.apply(
          tf.data.experimental.map_and_batch(
              map_func=parser,
              batch_size=batch_size,
              num_parallel_calls=config.NUM_DATA_WORKERS))
      dataset = dataset.prefetch(batch_size)
      return dataset
  ```

# Training Keras CNN model with TFRecordsDataset

  According to [official documentation](https://www.tensorflow.org/api_docs/python/tf/keras/Model#fit), tf.keras.Model's `fit()` method could take "a tf.data dataset or a dataset iterator" as input.  The dataset or iterator "should return a tuple of either (inputs, targets) or (inputs, targets, sample_weights)."

  So after compiling the training model, I could just call `model.fit()` with the TFRecordsDataset implemented as described above.  You could reference my source code [here](https://github.com/jkjung-avt/keras_imagenet/blob/master/train.py#L88).

  ```python
      # get training and validation data
      ds_train = get_dataset(dataset_dir, 'train', batch_size)
      ......

      model.fit(
          x=ds_train,
          ......
  ```

# Preliminary Results

  To recap, I've explained how I use sharded TFRecords for efficient I/O on the disk, as well as how to use `tf.data.TFRecordDataset` to ingest training data when training Keras CNN models.  I take advantage of tf.data's capabilities of processing data with multiple workers and shuffling/prefetching data on the fly.  Furthermore, I do online data augmentation when deserializing TFRecords.  This again takes advantage of multiple workers doing data fetching and processing in parallel.  I think I achieve very good data ingestion performance this way.

  I'm still in the process of training a Keras MobileNetv2 and a Keras ResNet50 models with the code.  I hope to share the results when the trainings are done.  Although I haven't done proper benchmarking, I'm pretty sure that using TFRecordsDataset (with 4 parallel data workers) speeds up the training quite a bit comparing to using original jpg files.

  2019-11-24 Update:  I've written a new post about how I visualized and verified training images in TensorBoard: [Displaying Images in TensorBoard](https://jkjung-avt.github.io/tensorboard-images/)
