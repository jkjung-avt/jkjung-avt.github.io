---
layout: post
comments: true
title: "Nintendo Famicom Mini!"
excerpt: "The Nintendo Famicom Mini, Japan's Version Of The NES Classic Edition."
date: 2017-02-06
category: "nintendo"
tags: nintendo
---

After I got DeepMind's DQN code and the Atari game emulator working on Jetson TX1, I really wanted to adapt the DQN AI agent to work on a real game console!

Note that video resolution of the 'xitari' emulator is RGB 160x210, and DeepMind's DQN actually scale it down to 84x84 before feeding the image data into the convolutional neural network (CNN). So to make it work I'd need to find a game console whose video resolution is in roughly the same range (160x210). For example, modern game console systems like PS4 or Xbox One would not cut it.

Meanwhile Nintendo launched the [NES Classic Edition](http://www.nintendo.com/nes-classic/) in Novemeber 2016. That became my targeted game console immediately. Although video output of NES Classic Edition is deafult to 1280x720 at 60 fps, under the hood the gaming system is probably only processing 256x240 images. So hopefully it would work out.

I live in Taiwan, so it's easier for me to purchase Japan's version of Nintendo NES Class, which is formally named **Famicom Mini**.

![Famicom Mini](https://www.nintendo.co.jp/clv/images/spec/spec_top_main.png)

Official web site: [https://www.nintendo.co.jp/clv/](https://www.nintendo.co.jp/clv/)

<iframe width="560" height="315" src="https://www.youtube.com/embed/3GQ02nXQQiM" frameborder="0" allowfullscreen></iframe>

It was actually quite difficult to purchase this Nintendo Famicom Mini console, which was out of stock all the time. I finally managed to get one a few days ago.

In order to let the DQN learn to play Nintendo Famicom Mini games, I'll have to implement at least the following:

* capturing the HDMI video output from Famicom Mini and converting the data to Torch7 Tensors,
* picking a game for which I'd need to parse game screens (image data) to determine the score (reward signal for reinforcement learning) and whether the game is over (terminal state),
* using GPIO to control Fimicom Mini's joystick from Torch7/Lua,
* adjusting DQN code and integrating everything together.

Those would be topics for my subsequent posts...
