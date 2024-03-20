---
aliases: 
date created: 2023-05-18 14:05:00
date modified: 2024-03-20 11:03:15
title: Introduction - Lecture 0
tags: [input]
---

## Search Problems
Consists of:
1. Initial state
2. Actions
3. Transition Model
4. Goal test
5. Path cost function

### Goal
Find the **Optimal Solution**

## Node
>Data structure

Keeps track of:
1. A state
2. A parent (A node that generates this node)
3. An action (action applied to parent to get this node)
4. A path cost (from initial state to node)

## Approach
Start with a frontier that contains the initial state
Repeat:
	If the frontier is empty, then no solution
	Remove a node from the frontier
	If node contains goal state, return the solution
	Expand node, add resulting nodes the frontier

### Revised Approach
Start with a frontier that contains the initial state
Start with an empty explored set
Repeat:
	If the frontier is empty, then no solution
	Remove a node from the frontier
	If node contains goal state, return the solution
	Expand node, add resulting nodes to the frontier if they aren't already in the frontier or the explored set.

>When we use stack as the frontier, it will be DFS.
>When we use queue as the frontier, it will be BFS.

### DFS
NEED Luck to find the Optimal Solution

### BFS
Must find the Optimal Solution but it will find more states

## Uninformed Search
>Search stategy that uses no problem-specific knowledge

## Informed Search
>Search strategy that uses problem-specific knowledge to find solutions more efficiently

### GBFS
Greedy best-first search

search algorithm that expands the node that is closest to the goal, as estimated by a heuristic function h(n)

### A* search
search algorithm that expands node with lowest value of g(n) + h(n)
>g(n) = cost to reach node
>h(n) = estimated cost to goal

Optimal if
- h (n) is admissible (never overestimates the true cost)
- h (n) is consistent (for every node n and successor n' with step cost c, h (n) <= h (n') + c)

## Adversarial Search
### Minimax
>Min (X) wants to minimize the score
>Max (X) wants to maximize the score

### Depth-Limited Minimax
#### evaluation function
function that estimates the expected utility of the game from a given state