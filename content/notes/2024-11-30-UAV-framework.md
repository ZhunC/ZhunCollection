---
share: "True"
postType: post
categories: Note
title: UAV Controller Framework
---
Tags: #control #UAV

The idea for this plan arose from two course projects–UAV Control in WUSTL, and Robotic Control in WPI. In the former, we were tasked to command a real-life quadrotor to lift off, follow a square trajectory, and return. We used an LQR controller with linearized dynamics in this case. In the latter, we were to simulate a quadrotor to follow a flying object in unknown trajectories, capture it, and return. We used a sliding mode controller on the virtual inputs(lift force and three torques). You can find our poster below:

[RBE 502 poster](https://drive.google.com/file/d/1BArOFkfy4LThffX5R8g4vH1r56HNiZ5e/view?usp=drive_link)

The problem is that as I go back and try to reuse the codes, they are too bulky and specific for that task, whereas I feel the need to revisit the UAV control problem frequently now and in the future. Hence I begin to work to write a modular Matlab framework that recycles my old code and interfaces well with its future applications.