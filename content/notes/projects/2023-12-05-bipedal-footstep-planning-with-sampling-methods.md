---
share: "True"
categories: Note
layout: post
title: Bipedal Footstep Planning with Sampling Methods
tags:
  - -motion_planning/sampling/RRT-star
---
For a bipedal robot to start planning joint states and control trajectories, it has to know where to place its feet sequentially, from start to goal. This [[Projects|project]] starts off as the course project for RBE 550 Motion Planning at WPI and aims to plan the footsteps using RRT*, a sampling-based motion planning algorithm that samples, expands, and reorganizes its waypoint tree.

The environment studied is assumed to be closed, stationary, and fully known. Dynamics are ignored in planning footsteps, and are assumed to be handled by the trajectory planner one level below this planner.

To cope with bipedal robots, several modifications are made:

1. Instead of planning a point robot, RRT* is now planning for two rectangular 3DOF Omnidirectional robots, corresponding to the two feet of a bipedal robot;
2. A ‘Left’ foot placement node must be followed by a ‘Right’ node and vice versa;
3. Each ‘Left’ and ‘Right’ connection must fall within a kinematic constraint, since human beings can’t put our other foot anywhere we want when we already have one foot placed on the ground. Bipedal robots are even more stiff and are more limited in terms of how freely their legs can operate;
4. Nodes should not be placed within a safety clearance of the obstacles, to accommodate for wider shoulders on robots.

Some Results are pasted below. The legs are marked with green and red respectively.

I’m not satisfied with these results due to several observations:

1. This version of RRT* is very sensitive to workspace size, number of nodes to sample, and kinematic range of robot. Increasing these parameters will decrease the chance of finding the solution within a reasonable time.
2. Trajectory quality–meaning how likely the robot is able to step on the planned locations without postprocessing–can be improved at the expense of computation time by increasing the above mentioned parameters. In the pictures above, I relaxed kinematic range for performance, so the quality is not great.
3. It’s hard to plan the footsteps when the robot turns, mainly because the Gaussian sampling strategy I used poorly guided the search. The order of footsteps as well as its configuration must both be guided.
4. Too many samples are rejected, preventing speedy solutions. Effective samples cluster around existing ones and the tree expands very slowly. Looser collision check yields better result, but is not a fundamental solution.

More detailed results can be found in my final presentation:

[RBE 550 final presentation](https://drive.google.com/file/d/1VN1X4AsiWoPm_6C_C9ojCVLfJfNtWOLG/view?usp=drive_link)


Apparently there are a lot areas to improve on top this work. Robustness, speed, customizability, to name a few. I’m investigating this topic in my free time and I look forward to updating this page in the future.