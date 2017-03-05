---
layout: post
comments: true
title: "Letting a Random Agent Play Galaga"
excerpt: "A random agent (player) playing Galaga can get an average score of 6314.3! Hopefully my AI agent could do much better than that after training."
date: 2017-03-03
category: "torch7"
tags: torch7 nintendo
---

After I got the ['gamenev' API](https://jkjung-avt.github.io/galaga-gameenv/) working, it became very easy to program an agent/player which picks complete random actions (with equal probabilities for all possible moves). As shown in the source code [`test/test_gameenv.lua`](https://github.com/jkjung-avt/dqn-tx1-for-nintendo/blob/master/test/test_gameenv.lua), all that's required is to call `gameenv.new_game()` once, followed by `gameenv.step(a)` repeatedly until the game is over.

```lua
 for i = 1, opt.games do
     local terminal = false
     local cnt = 0
     gameenv.new_game()
     while not terminal do
         cnt = cnt + 1
         if cnt % opt.actstep == 0 then
             -- take a random action
             _, _, terminal = gameenv.step(actions[math.random(#actions)])
         else
             _, _, terminal = gameenv.step()
        end
     end
     gameenv.step(0)  -- release all buttons
     print(('Game #%3d, score = '):format(i) .. gameenv.get_score())
     history[#history + 1] = gameenv.get_score()
 end
```

I experimented with different `opt.actstep` values (repeat same action for this many consecutive steps/frames before trying to take another action). The result is shown in the table below: average, minimum, maximum scores over 100 games for the 6 different configurations. As it turned out, the random player with `opt.actstep = 2` got the highest average score: **6314.3**. So that would be the score to beat for my DQN AI agent!

| opt.actstep |  Average   |    Min    |    Max    |
|:-----------:|:----------:|:---------:|:---------:|
|      1      |   5875.8   |    3380   |   14590   | 
|      2      | **6314.3** |    2540   |   14710   | 
|      3      |   5688.7   |    2820   |   14820   | 
|      4      |   5220.1   |    2200   |   12640   | 
|      5      |   4856.0   |    2070   |   10600   | 
|      6      |   4734.2   |    1680   |   12930   | 

I also made some histograms for comparison: opt.actstep = 1~4 over 100 games each.

![Scores of opt.actstep = 1](/assets/2017-03-03-random-agent/1.png)
![Scores of opt.actstep = 2](/assets/2017-03-03-random-agent/2.png)
![Scores of opt.actstep = 3](/assets/2017-03-03-random-agent/3.png)
![Scores of opt.actstep = 4](/assets/2017-03-03-random-agent/4.png)

