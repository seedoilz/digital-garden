---
aliases: 
date created: 2023-05-24 14:05:00
date modified: 2024-03-20 11:03:15
title: Hidden Markov Models
tags: [code/machine-learning]
---

>a type of a Markov model for a system with hidden states that generate some observed event

### Definitions
#### Hidden state
**what the AI wants to infer from the observations**
the state of the world which AI has not access to

#### Observations
**what the AI knows**
whatever data the AI has access to

#### Examples
- For a robot exploring uncharted territory, the hidden state is its position, and the observation is the data recorded by the robot’s sensors.
- In speech recognition, the hidden state is the words that were spoken, and the observation is the audio waveforms.
- When measuring user engagement on websites, the hidden state is how engaged the user is, and the observation is the website or app analytics.

### Sensor model (Emission model)
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230524140724.png)

### Sensor Markov Assumption
The assumption that the evidence variable depends only on the corresponding state. (NOT necessarily reflective of the complete truth)

A hidden Markov model can be represented in a Markov chain with two layers. The top layer, variable X, stands for the hidden state. The bottom layer, variable E, stands for the evidence, the observations that we have.
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230524140939.png)


Based on hidden [[Markov models]], multiple tasks can be achieved:
- Filtering: given observations from start until now, calculate the probability distribution for the current state. For example, given information on when people bring umbrellas form the start of time until today, we generate a probability distribution for whether it is raining today or not.
- Prediction: given observations from start until now, calculate the probability distribution for a future state.
- Smoothing: given observations from start until now, calculate the probability distribution for a past state. For example, calculating the probability of rain yesterday given that people brought umbrellas today.
- Most likely explanation: given observations from start until now, calculate most likely sequence of events.
