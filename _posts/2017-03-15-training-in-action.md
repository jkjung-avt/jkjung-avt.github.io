---
layout: post
comments: true
title: "Nintendo AI Agent Training in Action, Finally..."
excerpt: "I finally managed to put everything together, and started training my AI Agent (DeepMind's DQN) to play 'Galaga' on Nintendo Famicom Mini. Hopefully the AI could learn in a couple of weeks to play significantly better than the random player so that I could report the progress then..."
date: 2017-03-15
category: "torch7"
tags: torch7 nintendo dqn
---

All my hard work in the past few weeks has led to this moment. I'm finally able to put everything together and start training the DQN AI agent (on Jetson TX1) to play 'Galaga'!

<iframe width="560" height="315" src="https://www.youtube.com/embed/j-JKPok_1os" frameborder="0" allowfullscreen></iframe>

(For a quick recap of what AI/RL/DQN agent I was building, you can refer to this earlier post of mine: [DQN (RL Agent) on TX1 for Nintendo Famicom Mini](https://jkjung-avt.github.io/dqn-tx1-for-nintendo/).)

This final step of integrating my 'gamenv' module with DeepMind's DQN did not complete without some twists though. Here's how I did it.

* I copied all DQN code from my [DeepMind-Atari-Deep-Q-Learner](https://github.com/jkjung-avt/DeepMind-Atari-Deep-Q-Learner) repository into [dqn-tx1-for-nintendo](https://github.com/jkjung-avt/dqn-tx1-for-nintendo). Note that I have [applied cuDNN on this DQN code](https://jkjung-avt.github.io/dqn-cudnn/) so that its training efficiency is at least 50% better than the original DeepMind implementation (i.e. training time reduced by 1/3).
* Then I created a `train-deepmind.lua` script based on the original `train_agent.lua`. I removed all validation related stuffs in `train-deepmind.lua` and other dqn code. In my case I think I'll have to manually validate/test how well the DQN AI agent has learned.
* DeepMind's DQN takes 84x84 grayscale images (and also stacking 4 consecutive frames together) as input to the convolution based neural network. My 'vidcap' module produces 640x360 grayscale images at 30 frames per second. So I did proper cropping and rescaling on nintendo galaga images before sending them into DQN. I also normalized pixel values from the range [0, 255] to [0, 1], so that the neural network weights could all be working in roughly (-1, 1) range. The neural network should be able to learn better this way.
* If the agent does not get any score at the moment, I'd assign a small negative reward as default. I did this as an attempt to discourage the AI agent to: (1) dodge at the corner for a long time without trying to take out any enemies; (2) intentionally collide with enemies to get some score. (The agent would get some score, i.e. positive reward, when colliding with an enemy and losing a life. But then it'd have to wait for a long time for the next fighter to become active, during which time it would have collected a bunch of negative rewards.)
* When I hooked up my 'gameenv' module with the DQN code and started to train the AI agent, the image.display() memory leak issue manifested itself. So I decided that [I had to fix it](https://jkjung-avt.github.io/imshow/).
* Next, I found during training the workload concentrated heavily on one of the 4 CPU cores of Jetson TX1. This should not be surprising since Torch7/Lua is single-threaded by nature. This did cause problem because the amount of training I could do in-between game steps (30 fps, or 33 ms per step) was heavily limited by how much work a single CPU core could do. I was not satisfied with that, and I spent time to re-write my 'gamenv' module with [torch/threads](https://github.com/torch/threads). More specifically, I created a supporting thread which handles all the work of capturing video, displaying video, parsing images (for game state/score/reward), and cropping/rescaling/normalizing images. This way the main thread could focus on useful work such as agent:perceive() (calculating forward path of the CNN) and training (backprop).
* Since [a random player (agent) gets best average score when 'actrep' is set to 2](https://jkjung-avt.github.io/random-agent/) (i.e. repeat an action for every 2 video frames), I let the DQN select ('perceive') a different action every 2 video frames/steps as well. By doing this, the DQN agent has more time (66 ms instead of 33 ms) to make a decision on the action. In addition, I could try to stuff more training work in-between.
* Finally I spent a lot of time fine-tuning how much training workload I could assign to the DQN agent during a game. I wanted to load the DQN agent as much as possible (train as much as possible), but could not overload it so that it fell bebind and didn't take actions in real-time. I ended up asking the DQN agent to do 1 minibatch of training for every 6 video frames/steps, with size of minibatch set to 8.

Whew, that's about it.

Now that I have the DQN working and learning how to play the Nintendo console game, I'll keep it running for a good while and probably validate/test it once every week or so. Since [a random agent could get an average score of 6314.3](https://jkjung-avt.github.io/random-agent/), I'm hoping the learned DQN could do much better than that, or even close to human expert level. I just have to give the DQN agent some time and hope for the best...
