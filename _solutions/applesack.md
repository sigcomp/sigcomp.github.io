---
layout: page
title: Johnny Applesack - Difficulty Level 3.7
---

## Problem Breakdown
> You are given N apples and a sack that can hold K apples. Before you lies a series of toll booths spaced 1 kilometer apart. If it costs 1 apple to pass a toll booth (moving away from the initial position), how many kilometers can you travel?

The problem states that we are allowed to deposit apples anywhere we like and retrieve them at any time. This is the key to progressing further than K kilometers. What we must optimize is the distance we travel with each trip. Going a further distance will allow us to make fewer returning trips, but will cost us more apples per trip.


## Solution Explanation
Obviously, we do not want to travel K kilometers with each trip and deposit no apples. Let us then consider traveling the maximum distance (K-1) versus traveling only 1 km per trip. As the illustration below demonstrates, these approaches often produce the same result (N=6, K=3 depicted)


![small trips for N=6, K=3](/assets/solution_img/applesack/one_six_demo.gif "N=6, K=3")
![large trips for N=6, K=3](/assets/solution_img/applesack/three_six_demo.gif "N=6, K=3")

One pitfall that we must avoid, however, is not having enough apples to fill our sack on a return trip. In the worst case, this could strand us from our remaining apples. In the best case, we simply waste the amount of apples that it cost to make the trip. Below we see the same two approaches with N=5 apples instead 

![small trips for N=5, K=3](/assets/solution_img/applesack/one_five_demo.gif "N=5, K=3")
![large trips for N=5, K=3](/assets/solution_img/applesack/three_five_demo.gif "N=5, K=3")

From this it clear to see that making repeated 1 kilometer trips is the most efficient, since it can waste at most one apple per trip and it is impossible to strand yourself from your remaining apples. Notice that we can "cheat" one extra kilometer by walking all the way up to the edge of the next marker when we run out of apples

## Source Code

```python
import math

#input "N K" -> apples = N, sack = K
apples, sack = map(int, input().split())

km = 1 #we can cheat one km by walking to the edge of the marker

while apples > sack:
    #the amount of trips necessary to move all apples 1 km
    apples -= math.ceil(apples/sack) 
    km += 1

km += apples #no more return trips

print(km)
```
