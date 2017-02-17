---
layout: post
comments: true
title: "Accessing TX1 GPIO as a Non-root User (ubuntu)"
excerpt: "I set up a 'gpio' group and some udev rules on Jetson TX1 so that I can access GPIO without 'sudo'."
date: 2017-02-15
category: "tx1"
tags: tx1 gpio
---

To make the DQN AI agent on Jetson TX1 able to control Nintendo Famicom Mini (take actions), I have to let the Torch7 program access GPIO. Normally, that would then require the Torch7 program to run with root privilege. I certainly don't want to run my whole Torch7 program as root, so I need a solution to allow a non-root user ('ubuntu' on JetsonTX1) to access GPIO.

By the way, there is already a good article ["GPIO Interfacing – NVIDIA Jetson TX1"](http://www.jetsonhacks.com/2015/12/29/gpio-interfacing-nvidia-jetson-tx1/) by JetsonHacks.com about how to test and use GPIO pins on Jetson TX1. I don't find a better solution to access GPIO on TX1 than the /sys/class/gpio interface, so I'd just stick to it. I just need to do this without sudo (or being root).

After some googling, I found the trick is just to use a 'gpio' group to manage access to /sys/class/gpio files, and to set up some udev rules to automatically assign proper file permissions. So I wrote the solution as shell scripts and put them into my repository.

[https://github.com/jkjung-avt/dqn-tx1-for-nintendo/tree/master/gpio/non_root](https://github.com/jkjung-avt/dqn-tx1-for-nintendo/tree/master/gpio/non_root)

To set up Jetson TX1, just run `./setup_gpio.sh` once and then reboot the system. Here's the content of 'setup_gpio.sh'.

```shell
 #!/bin/bash
 #
 # This script is used to enable the user "ubuntu" to access GPIO through
 # /sys/class/gpio interface. It only needs to be executed once. Make sure
 # you have "fix_udev_gpio.sh" and "99-com.rules" in the same directory
 # when you run this script.

 sudo groupadd gpio
 sudo usermod -a -G gpio ubuntu
 sudo cp fix_udev_gpio.sh /usr/local/bin/
 sudo chmod a+x /usr/local/bin/fix_udev_gpio.sh

 sudo chown -R root.gpio /sys/class/gpio
 sudo chmod 0220 /sys/class/gpio/export
 sudo chmod 0220 /sys/class/gpio/unexport

 sudo cp 99-com.rules /etc/udev/rules.d/
```

Afterwards, we can use 'gpio.sh' to do simple tests.

```shell
 ### set gpio38 as output with value 1
 $ ./gpio.sh 38 out 1
```
