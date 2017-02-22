---
layout: post
comments: true
title: "Accessing Hardware GPIO in Torch7"
excerpt: "I developd code to control GPIO outputs from Torch7. This should work for not only Jetson TX1 but other Linux platforms as well. The code could be extended to handle GPIO inputs too."
date: 2017-02-16
category: "tx1"
tags: tx1 gpio
---

Since I already had the experience of [coding with Lua FFI](https://jkjung-avt.github.io/vidcap-in-torch7/) and I already had a solution for [accessing hardware general-purpose I/O pins (GPIOs) as a non-root user](https://jkjung-avt.github.io/gpio-non-root/), it became straightforward for me to develop a Lua FFI module to access GPIOs directly from Torch7.

I adapted [JetsonHacks' code](https://github.com/jetsonhacks/jetsonTX1GPIO), hooked it up with FFI, and wrote a test script to verify it. The resulting code is in: [https://github.com/jkjung-avt/dqn-tx1-for-nintendo/tree/master/gpio](https://github.com/jkjung-avt/dqn-tx1-for-nintendo/tree/master/gpio).

In the API I only exposed GPIO output functionalities, since that's all I needed for the Nintendo AI project. But I think the code could be easily extended to support GPIO inputs as well.

The test script could be run from the project root directory. On Jetson TX1, it toggles 'gpio38' (pin #13 on header 'J21' of Jetson TX1) a few times with 1 second delay in-between.

```shell
 $ qlua test/test_gpio.lua
```

I made a small circuit with a transistor and a LED, similar to what's described in [JetsonHacks' article](http://www.jetsonhacks.com/2015/12/29/gpio-interfacing-nvidia-jetson-tx1/) (the 'Jetson TX1 GPIO LED Interface' diagram), to test 'gpio38'. It worked as expected.

!['gpio38' toggles a LED](/assets/2017-02-16-gpio-in-torch7/test_gpio38.gif)
