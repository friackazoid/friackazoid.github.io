---
layout: post
title: Yet Another Way to Play with Your Robot; LoRa fine tunning
categories: [Robots, LLM]
---

In my [previous post](https://friackazoid.github.io/gpt-assistant/), I showed how you can use the OpenAI GPT API as a command interface.
In this post, I will show how to fine-tune the model and train the robot a new tricks.


#In the previous episode...

In my previous post (https://friackazoid.github.io/gpt-assistant/), I demonstrated how to use the OpenAI GPT API as a command interface for controlling a robotic dog. In my post, I explained that even though the API was reliable, it could not handle simple commands like "sit" (despite specifying it in the example). The model could not create robot poses that matched the robot's kinematic model.

<gif with example commands Sit, Die, Give a pow>

So let's see what results can be achieved by tuning base model with specific dataset. Here, I'll show what I did and chat a bit about why I picked the settings I did.

# What's the plan?

Fine-tuning is basically about updating all the base model's parameters so it can do a specific job better.
A model that already knows a lot about the world (like language rules, common facts, and human logic) learns the new task way quicker than one starting from scratch with random settings.
However, the process is still really resource-hungry.
Fine-tuning a model like Llama-1B (1B = 1 billion parameters) could take several days on several GPUs - resources that not every (me included) enthusiast has.  

Luckily, LoRa (Low-Rank Adaptation of Large Language Model link https://arxiv.org/abs/2106.09685?utm_source=chatgpt.com) helps a lot by cutting down the number of trainable parameters by up to 10,000 times.
That basically means I can take a solid model and fine-tune it using just my humble one-GPU setup (a trusty GeForce RTX). 
