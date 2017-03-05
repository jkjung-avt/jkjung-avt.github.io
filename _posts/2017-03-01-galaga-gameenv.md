---
layout: post
comments: true
title: "Galaga Game Environment"
excerpt: "I integrated the vidcap, galaga and gpio modules and finalized a Galaga game environment API which is similar to xtari, the Atari game emulator."
date: 2017-03-01
category: "torch7"
tags: torch7 nintendo
---

In order to use DeepMind's DQN code for my Nintendo AI agent with minimum porting effort, I'd need to develop a game environment API very similar to 'xitari' and 'alewrap', the Atari game emulator used by DeepMind.

As shown in the diagram in my previous post ['DQN (RL Agent) on TX1 for Nintendo Famicom Mini'](https://jkjung-avt.github.io/dqn-tx1-for-nintendo/), the 'game environment' should provide the following functionalities:

* providing the 'state' (game image) to the agent
* providing the 'reward' (game score, remaining lives) to the agent
* receiving the 'action' from the agent and transition to next state

For my purpose (letting AI learn to play Galaga), I can get the 'state' by capturing HDMI video directly from Nintendo Famicom Mini. And I can get the 'reward' by parsing game images (['galaga' module](https://jkjung-avt.github.io/galaga/)). Then I need to define the actions.

In Galaga, there are only 6 possible actions a player can take: moving left, moving right, stay still, or the aforementioned 3 actions plus firing bullets. So I define actions 1~6 as follows.

```lua
 -- Get the list of available actions.
 -- get_actions() is now hard-coded for Galaga...
 function gameenv.get_actions()
     assert(gameenv.is_initialized, 'get_action() called while gameenv is not initialized')
     local actions = { 1, 2, 3, 4, 5, 6 }
     -- Six possible actions are defined for Galaga:
     -- 1 = Left
     -- 2 = Stay
     -- 3 = Right
     -- 4 = Left  + Fire
     -- 5 = Stay  + Fire
     -- 6 = Right + Fire
     return actions
 end
```

In addtion, I also need a reliable mechanism to restart a new game. Using a physical Nintendo console is not like using the game emulator, where we can reset the game environment at any time and start a new game. For my case I just have to wait for a certain sequence of game images and press 'Start' button at the appropriate times. Here is how it looks like.

![Galaga new game sequence](/assets/2017-03-01-galaga-gameenv/galaga_new_game.jpg)

With all the above straightened up, coding of the 'gameenv' (Galaga game environment is straightforward. And the resulting API is quite simple.

* gameenv.init() -> initialize the game environment.
* gameenv.cleanup()
* gameenv.get_actions() -> query available actions for the game.
* gameenv.new_game() -> start a new Galaga game (this function might wait a long time for the previous game to finish).
* gameenv.step(a) -> take action 'a', trasition to next state, then return the 'state', 'reward' and 'terminal' (whether the game is over); if 'a' is nil, then repeat the previous action and step forward.
* gameenv.get_score() -> read the score of the current game.

The source code has been committed to the 'gameenv' subdirectory in my project repository.

[https://github.com/jkjung-avt/dqn-tx1-for-nintendo](https://github.com/jkjung-avt/dqn-tx1-for-nintendo)
