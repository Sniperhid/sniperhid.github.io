---
author: "Snippers"
title: "Escape the Ambush - Case study of coordinated AI attack"
date: "2019-03-11"
description: "A mission centered around AI encirclement of escaping players"
tags: ["Arma"]
categories: ["themes", "syntax"]
series: ["Arma Missions"]
ShowToc: true
TocOpen: true
weight: 2
---

Escape the ambush was an attempt to revisit the concept of the players being pursued as they attempt to escape. The core concept in the mission is a wave manager that will produce waves of enemies that attempt to coordinate an attack on the players.

The idea was to attempt to surround the players at the same time from all sides to help provide something for every player to shoot at. In essence I wanted a dynamic system to enact the role of a macro level AI commander.

## Choosing the mission area
In order for this to work I needed a terrain that would offer plenty of concealment allowing troops to move into positions to stage an attack before getting caught in a firefight. I chose an area in the Anizay terrain that offers average visibility of what I estimate to be around 200-300m which is great for infantry engagements.

![map](map.jpg)

### Picking spawn points
The next step is to determine potential locations for the enemies to spawn.

![map](map_spawn.jpg)

As shown above I placed game logic objects (blue squares with a flag icon). These are relatively easy to find via code and the system can then pick randomly from these points.

# Setting up the AI Orders


### Destination: Calculating Engagement positions
A simple idea is that ultimately the AI should encircle the players. A simple mathematical tool is producing a [convex hull](https://en.wikipedia.org/wiki/Convex_hull) around the players. This is just a polygon line that surrounds all players (shown as a blue line in the below image).
![map](convex-hull.jpg)
This convex hull can then be transformed to be a bigger polygon of where we want the AI to be when engaging the players. I'll call this the "Engagement polygon" moving forwards. It is visualized above as a red line in which all points from the convex hull are pushed another 100m away from the center of the polygon.

## Attacking Forces
To use the engagement polygon effectively we need to dispatch multiple groups (Fireteams/Squads) of AI to attack the players.

Arma exposes the ability to add waypoints to groups of AI which the ingame AI will follow. So to get this to work the initial piece of work is to figure out where the units should attack from.

To provide a challenge for the players we need to space out the AI units around the engagement polygon. As opposed to having them in a big blob which is ill advised in military doctrine. An effective way to do this is  to spread out the troops across a line formation. Shown below is 4 groups of 5 soldiers each in a line formation:


![map](line-formation.jpg)

To ease things for the AI I also created waypoints for each group behind their final engagement waypoints. As this forces the AI to already be advancing in the direction they should engaging the players. In addition the spacing between the groups is calculated by the size of the group to ensure there is enough room for all the soldiers to be in line without groups overlapping.

![map](waypoints0.jpg)

There is a potential synchronization issue where some groups may reach their waypoints before other groups (see below where the right most group is closer to its engagement point than the others). This can be mitigated by having some code monitor the groups progress. Then ordering groups that are ahead of others to slow down by switching to a walking pace whilst the other groups catching up move faster by jogging.

![map](group-synch.jpg)

## Scaling up
To spice things up for the players we want to send multiple groups after the players at the same time. The groups claim a part of the perimeter of the engagement polygon and prevent other groups from it, resulting in a encirclement of final waypoints around the players as visualized below:
![map](waypoints1.jpg)
![map](waypoints2.jpg)

Now finally to package this as a fully playable mission. There is code that will spawn multiple waves periodically using this strategy. Then engagement polygons and waypoints are rebalanced as the players move. In practice "No plan survives first contact with the enemy". The players keep moving and the AI lose units. Both of these present their own issues and I have done little to address that but for the time being the coordinated approach seemed cool enough to release as a mission.

# Appendix 

## Videos

## R3 Recordings
Some selected R3 recordings of playthroughs at 1Tac sessions of the mission:
- [12th Jan 2019 - 13 Players](https://1tac.tk/r3/1583/escape-the-ambush-v4)
- [5th March 2019 - 18 Players](https://1tac.tk/r3/1700/escape-the-ambush-i7-v2)
- [30th August 2019 - 19 players](https://1tac.tk/r3/2213/escape-the-ambush-i7-v2)