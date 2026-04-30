---
title: Factor Graph in SLAM — Trajectory and Landmark Observations
tags: [SLAM, factor-graph, robotics, localization, mapping, probabilistic]
created: 2026-04-26
---
![[Pasted image 20260426151110.png]]
## What the graph shows

The figure is a **factor graph** representing a simulated SLAM (Simultaneous Localization and Mapping) problem.

- **Teal/cyan circles** — robot poses along the trajectory over time. The chain of circles traces the robot's estimated path through the environment, forming an arc from the lower-left to the upper-right.
- **Blue squares** — landmarks (observed features) in the environment.
- **Grey edges** — factors (constraints) connecting poses to landmarks, encoding the probabilistic observation that a given pose saw a given landmark.

## What SLAM is doing here

In SLAM, the robot must simultaneously estimate:
1. **Its own trajectory** — the sequence of poses $x_1, x_2, \ldots, x_T$
2. **A map of the environment** — the positions of landmarks $l_1, l_2, \ldots, l_M$

The factor graph encodes this as a joint probability distribution. Each edge between a pose node and a landmark node represents a **measurement factor** — a constraint derived from a sensor observation (e.g. range-bearing from lidar or camera). The dense fan of edges from certain poses reflects moments where many landmarks were simultaneously visible.

The graph is solved by **maximum a posteriori (MAP) inference**, typically via nonlinear least squares (e.g. Gauss-Newton, Levenberg-Marquardt), which jointly optimises all poses and landmark positions to best explain all the observations.

## Key intuition

> The trajectory is the robot's path; the landmarks are the map. Every grey edge is a "I saw that landmark from here" constraint. SLAM fuses all of these to resolve the chicken-and-egg problem: you need a map to localise, and you need to localise to build a map.
