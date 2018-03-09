---
layout: post
comments: true
title: "Building and Testing 'openalpr' on Jetson TX2"
excerpt: "Recently I needed to implement license plate recognition function on Jetson TX2. I managed to build 'openalpr' from source and had it working on JTX2. This post documents how I did it."
date: 2018-03-09
category: "opencv"
tags: opencv openalpr
---

I [read about openalpr](https://medium.freecodecamp.org/how-i-replicated-an-86-million-project-in-57-lines-of-code-277031330ee9) a while ago. Recently I was requested to integrate license plate recognition function into our TX2 product. ['openalpr'](https://github.com/openalpr/openalpr) came up as my go-to solution for the task.

There seems to be a pre-built 'openalpr' apt package available, but it has dependency on opencv apt packages. I didn't want 'apt install' to mess up with the opencv-3.4.0 I built and installed on JTX2, so I chose to build 'openalpr' from source. Here's how I've done it.

# Prerequisite

* Build and install opencv-3.4.0 on the target JTX2. Reference: [How to Install OpenCV (3.4.0) on Jetson TX2](https://jkjung-avt.github.io/opencv3-on-tx2/)

# Step-by-step

In addition to 'opencv', 'leptonica' and 'tesseract' are also required for 'openalpr'. The build process is broken down into several steps below. Reference: [Compile openalpr The Harder Way (Compile all dependencies manually)](https://github.com/openalpr/openalpr/wiki/Compilation-instructions-(Ubuntu-Linux))

* Step #1: build and install 'leptonica'

  I installed the latest version of 'leptonica': 1.75.3.

  First, install dependencies.

  ```shell
  $ sudo apt-get install autoconf automake libtool
  $ sudo apt-get install autoconf-archive pkg-config
  $ sudo apt-get install libpng-dev libjpeg8-dev libtiff5-dev zlib1g-dev
  $ sudo apt-get install libicu-dev libpango1.0-dev libcairo2-dev
  ```

  I'd also set TX2 to maximum performance mode in order to speed up the building process.

  ```shell
  $ sudo nvpmodel -m 0
  $ sudo ~/jetson_clocks.sh
  ```

  Then download and install 'leptonica'.

  ```shell
  $ mkdir -p ~/src
  $ cd ~/src
  $ wget http://www.leptonica.com/source/leptonica-1.75.3.tar.gz
  $ tar xzvf leptonica-1.75.3.tar.gz
  $ cd leptonica-1.75.3/
  $ ./configure --prefix=/usr/local
  $ make -j4
  $ sudo make install
  ```

* Step #2: build and install 'tesseract'

  I installed the latest stable version of 'tesseract': 3.05.01. Reference: [Compiling (tesseract)](https://github.com/tesseract-ocr/tesseract/wiki/Compiling)

  ```shell
  $ cd ~/src
  $ git clone https://github.com/tesseract-ocr/tesseract.git
  $ cd tesseract/
  $ git checkout 3.05
  $ ./autogen.sh 
  $ ./configure --prefix=/usr/local
  $ make -j4
  $ sudo make install
  ```

  In addition to the libraries and excutbales, I also had to download the language data files needed for OCR. More specifically, I downloaded and installed the 'English' and 'Chinese - Traditional' data files. Reference: [Data Files (tesseract)](https://github.com/tesseract-ocr/tesseract/wiki/Data-Files)

  ```shell
  $ cd ~/Downloads/
  $ wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/eng.traineddata
  $ wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/chi_tra.traineddata
  $ sudo cp eng.traineddata /use/local/share/tessdata/
  $ sudo cp chi_tra.traineddata /use/local/share/tessdata/
  ```

  Following tesseract's documentation I put the following into my `~/.bashrc`. Log out and log in again to let the `TESSDATA_PREFIX` environment variable take effect.

  ```
  export TESSDATA_PREFIX=/usr/local/share
  ```

* Step #3: build and install 'openalpr'

  Install dependencies and clone the 'openalpr' repo.

  ```shell
  $ sudo apt-get install curl libcurl4-openssl-dev
  $ sudo apt-get install liblog4cplus-1.1-9
  $ cd ~/src
  $ git clone https://github.com/openalpr/openalpr.git
  $ cd openalpr
  ```

  'openalpr' needs to be compiled with gcc '-std=c++11' flag. So I modified `src/CMakeLists.txt` as follows. I put a copy of my modified `src/CMakeLists.txt` [here](/assets/2018-03-09-openalpr-on-tx2/CMakeLists.txt), for reference.

  ```
  --- a/src/CMakeLists.txt
  +++ b/src/CMakeLists.txt
  @@ -5,6 +5,8 @@ set(CMAKE_BUILD_TYPE RelWithDebugInfo)
  
   cmake_minimum_required (VERSION 2.6)

  +set (CMAKE_CXX_STANDARD 11)
  +
   # Set the OpenALPR version in cmake, and also add it as a DEFINE for the code to access
   SET(OPENALPR_MAJOR_VERSION "2")
   SET(OPENALPR_MINOR_VERSION "3")
  ```

  I also needed to make some midifications to `src/daemon.cpp`, otherwise compilation of that source file would fail. Again I put a copy of my modified `src/daemon.cpp` [here](/assets/2018-03-09-openalpr-on-tx2/daemon.cpp), for reference.

  ```
  --- a/src/daemon.cpp
  +++ b/src/daemon.cpp
  @@ -156,7 +156,8 @@ int main( int argc, const char** argv )
       log4cplus::SharedAppenderPtr myAppender(new log4cplus::RollingFileAppender(logFile));
       myAppender->setName("alprd_appender");
       // Redirect std out to log file
  -    logger = log4cplus::Logger::getInstance(LOG4CPLUS_TEXT("alprd"));
  +    auto tmp_logger = log4cplus::Logger::getInstance(LOG4CPLUS_TEXT("alprd"));
  +    logger = tmp_logger;
       logger.addAppender(myAppender);
  
  
  @@ -167,7 +168,8 @@ int main( int argc, const char** argv )
       //log4cplus::SharedAppenderPtr myAppender(new log4cplus::ConsoleAppender());
       //myAppender->setName("alprd_appender");
       // Redirect std out to log file
  -    logger = log4cplus::Logger::getInstance(LOG4CPLUS_TEXT("alprd"));
  +    auto tmp_logger = log4cplus::Logger::getInstance(LOG4CPLUS_TEXT("alprd"));
  +    logger = tmp_logger;
       //logger.addAppender(myAppender);
  
       LOG4CPLUS_INFO(logger, "Running OpenALPR daemon in the foreground.");
  ```

  Here are the commands for building 'openalpr'.

  ```shell
  $ cd ~/src/openalpr/src
  $ mkdir build
  $ cd build
  $ cmake -D WITH_GPU_DETECTOR=ON ..
  $ make -j4
  $ sudo make install
  ```

* Step #4: testing

  In order to achieve better performance of 'openalpr' by utilizing the GPU on TX2, I had to modify the config file. More specifically, I made a copy of openalpr's default config file and changed `detector` from `lbpcpu` to `lbpgpu`, along with other minor modifications. I put a copy of my modified config file [here](/assets/2018-03-09-openalpr-on-tx2/tx2.conf), for reference.

  ```shell
  $ mkdir -p ~/src/openalpr/test
  $ cd ~/src/openalpr/test
  $ cp /usr/local/share/openalpr/config/openalpr.defaults.conf tx2.conf
  $ vim tx2.conf
  ```

  Finally I used a picture from my mobile phone for testing.

  ![A photo of ASM-9197](/assets/2018-03-09-openalpr-on-tx2/ASM-9197.jpg)

  Here's the result...

  ```shell
  nvidia@tegra-ubuntu:~/src/openalpr/test$ alpr --country us --config tx2.conf ASM-9197.jpg
  --(!)Loaded CUDA classifier
  plate0: 10 results
      - ASM9197    confidence: 88.5777
      - ASM9I97    confidence: 85.3164
      - A5M9197    confidence: 81.3223
      - A5M9I97    confidence: 78.0611
      - ASH9197    confidence: 76.4136
      - ABM9197    confidence: 76.0675
      - AM9197     confidence: 75.7315
      - ASH9I97    confidence: 73.1524
      - ABM9I97    confidence: 72.8063
      - AM9I97     confidence: 72.4703
  ```
