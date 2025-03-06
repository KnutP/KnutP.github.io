---
layout: page
title: Policy Gradients for Aoe2 Unit Micro
description: Fun little reinforcement learning project
img: assets/img/raw_img2.png
importance: 1
category: fun
related_publications: false
---

# REINFORCE Policy Gradients for AoE2 Unit Micro

Fun little project I did for a reinforcement learning class in 2021. It implements the REINFORCE algorithm to micro crossbowmen in the RTS game Age of Empires 2.
Below is a summary of the project.


## Background Info

Age of Empires 2 (AoE2) is a popular Real-Time Strategy game where unit control can have a major impact on the game.
For instance, in battles between crossbow units, arrows can be dodged, and enemy units can be focus-fired to get cost-effective trades against
much larger groups of opposing units.
Arrow dodging is a difficult skill, but can swing engagements and is a crucial skill for good players.

The goal of this project is to answer the question:
**Can we create a reinforcement learning agent that can learn to micro like a pro?**

Specifically, can it learn to dodge arrow shots and win engagements with fewer units?
To test this, and emulate how pro players micro, the agent must follow two limitations:
<ol>
  <li>Controls all units as a group</li>
  <li>Cap on Actions Per Minute (APM) of 120</li>
</ol>


## Methodology
AoE2 doesn't have an automated interface to easily control a player via code, so I built a simple custom one in aoe2_game.py using pyautogui.
I also created a custom scenario with 6 crossbow units per player in a confined space, and gave the RL agent 89 possible actions to choose from, 
where 1-88 were to click on a different grid cell, and 89 was do nothing.

I based the reward function on the in-game score since it was possible to check changes in score lead by grabbing pixel colors from the in-game interface,
and I assigned a reward of +5 for taking the score lead, and -1 for losing it.

<figure >
    <img src="assets/img/raw_img2.png" width="50%" style="display: block; margin-left: auto; margin-right: auto;">
    <figcaption  width="40%" style="text-align: center;">
        Raw image capture
    </figcaption>
</figure>
<figure >
    <img src="assets/img/frame_diff2.png" width="50%" style="display: block; margin-left: auto; margin-right: auto;">
    <figcaption  width="40%" style="text-align: center;">
        Processed difference frame
    </figcaption>
</figure>

For the input of my model, I wanted to capture movement information while still keeping the input simple, so I took a tip from Andrej Karpathy’s 
 <a href='http://karpathy.github.io/2016/05/31/rl/'>“Pong from Pixels” article</a> on policy gradients, and calculated a difference frame to use as the input.
 For this I first captured 3 frames of gameplay, downsized each frame, and then calculated the difference frame. Then I finally flattened the image
 so the final input to the model was a list of 49686 integers representing the difference frame image.



For my model, I implemented a simple multi-layer perceptron policy gradient, using the REINFORCE policy gradient algorithm.
REINFORCE is a “Monte-Carlo Policy Gradient” approach, so it is relatively simple to implement, and it updates every episode.
This was important for me, as it means the main computation is done AFTER the CPU is no longer being used for running the game.
For my policy MLP I used an input layer of 49686 nodes (image size), and an output layer of 89 nodes (action space size).
I initially only included 1 hidden layer of size 780, but testing showed that the model struggled to learn and I expanded to 2
hidden layers for the final model.

<figure >
    <img src="assets/img/REINFORCE.png" width="50%" style="display: block; margin-left: auto; margin-right: auto;">
    <figcaption  width="50%" style="text-align: center;">
        An overview of the REINFORCE algorithm
    </figcaption>
</figure>



## Results

The model very quickly learned that killing an enemy unit led to a reward, as shown in the demo after 20 training iterations.
However, it took much longer to learn to avoid enemy arrow fire.

<figure >
    <img src="assets/img/after-20.gif" width="40%" style="display: block; margin-left: auto; margin-right: auto;">
    <figcaption  width="50%" style="text-align: center;">
        Initial model performance after 20 iterations
    </figcaption>
</figure>

By the end of training the model learned to focus enemy units and avoid arrow fire, but was not able to completely beat the 
in-game AI.

<figure >
    <img src="assets/img/after-800.gif" width="40%" style="display: block; margin-left: auto; margin-right: auto;">
    <figcaption  width="50%" style="text-align: center;">
        Final model performance after 800 iterations
    </figcaption>
</figure>

From the plot of the average reward vs. training episodes, we can see that the reward quickly increases as the model learns
to attack enemy units, and then drops as the model learns to dodge arrows. Then the reward begins to climb again as it becomes
more effective at dodging and eliminates more enemy units before losing.

<figure >
    <img src="assets/img/avg_reward.png" width="30%" style="display: block; margin-left: auto; margin-right: auto;">
    <figcaption  width="50%" style="text-align: center;">
        Plot of reward vs. training iterations
    </figcaption>
</figure>

While the RL agent did not manage to completely beat the in-game AI in the end, there are a few reasons for that. 
For one, the in-game AI does not micro like a human - it controls units individually instead of as a group,
which makes it notoriously annoying to micro against. Shot dodging is also more difficult, as multiple shots may come in at
the same time, while a human player would use a single group of units, making shots easier to dodge.

Additionally, the other main limiting factor was the number of training episodes. Each training episode was longe and
computationally intensive, as both the Aoe2 game and RL agent had to run on the same machine. As the game only ran at a 
set speed, this meant that 1 episode took ~1 minute of real time, and training all 800 episodes took ~13 hours spread
over a total of 5 days. This could be improved by expanding to an Actor-Critic policy gradient method, or by exploring
 model-based methods with prioritized replay.

In the end, the final agent was still very effective at dodging arrows, even against multiple units. While not quite
reaching the pro level, it made a lot of progress but was limited by the number of training episodes. Overall I'm pretty 
happy with the results, and may revisit this at some point to try and improve it.