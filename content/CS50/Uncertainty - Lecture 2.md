---
aliases: 
date created: 2023-05-23 22:05:00
date modified: 2024-03-20 11:03:14
title: Uncertainty - Lecture 2
tags: [input]
---

## [[Bayesian Network]]
>A Bayesian network is a data structure that represents the dependencies among random variables. Bayesian networks have the following properties.

### Instance
For this image, Rain is dependent on nothing while Maitenance is depentent on the Rain. 
As a result, the possibility of Maintenance is a conditional possibility which needs to be discussed in the certain Rain situation.
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230523223001.png)

### Inference
- Query **X**: the variable for which we want to compute the probability distribution.
- Evidence variables **E**: one or more variables that have been observed for event **e**. For example, we might have observed that there is light rain, and this observation helps us compute the probability that the train is delayed.
- Hidden variables **Y**: variables that aren’t the query and also haven’t been observed. For example, standing at the train station, we can observe whether there is rain, but we can’t know if there is maintenance on the track further down the road. Thus, Maintenance would be a hidden variable in this situation.
- The goal: calculate **P**(_X | e_). For example, compute the probability distribution of the Train variable (the query) based on the evidence **e** that we know there is light rain.

## Sampling
Make a sample based on the distribution of the Variable. If the result matches what we want, then we will keep this. We must sample from the top of the Bayesian Network, or we cannot decide the possibility of the other Variable.

## [[Markov Models]]
### Markov Assumption
The current state depends on only a finite fixed number of previous states.
Usingt he Markov assumption, we restrict our previous states, thereby making the task manageable. What we get from this is a more rough approxiamation of the probabilities of interest, but this is often good enough for our needs.

### Markov Chain
> A sequence of random variables where the distribution of each variable follows the Markov assumption.
> Each event in the chain occurs based on the probability of the event before it.

#### How to use?
##### A transition model
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230524134829.png)

## [[Hidden Markov Models]]
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


Based on hidden Markov models, multiple tasks can be achieved:
- Filtering: given observations from start until now, calculate the probability distribution for the current state. For example, given information on when people bring umbrellas form the start of time until today, we generate a probability distribution for whether it is raining today or not.
- Prediction: given observations from start until now, calculate the probability distribution for a future state.
- Smoothing: given observations from start until now, calculate the probability distribution for a past state. For example, calculating the probability of rain yesterday given that people brought umbrellas today.
- Most likely explanation: given observations from start until now, calculate most likely sequence of events.
