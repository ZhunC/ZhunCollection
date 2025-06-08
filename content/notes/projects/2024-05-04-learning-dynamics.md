---
share: "True"
postType: post
categories: Note
title: "Learning Dynamics: Some Thoughts on World Models"
tags: 
  - machine_learning
  - control_theory
---

This [[Projects|project]] started as the course projecct for MA 529 Stochastic Processes in Worcester Polytechnic Institute in SP 2024 Semester. We aim to feed a quadruped’s state and control history to a deep learning model, which outputs states in the next instance. In other words, we want a neural network that captured the dynamic of this robot.

Why do we want to do so? Part of the motivation was Large Language Models(LLM) like ChatGPT, which is able to complete a sentence with the most likely phrases. What if it’s with system states instead of sentences, we thought? Can we design some neural networks bale to complete a state trajectory just as LLMs complete sentences?

Another part is driven by modelling errors. Mathematically derived dynamic systems make assumptions which give rise to errors. For example, link center of mass is off from its geometric center. Or contact forces with the ground doesn’t follow the polynomial model precisely, etc. We’re hoping a neural network is able to learn these niche aspects of the quadruped dynamics where math formulations are not able to.

Due to the time limit, we built a Diffusion Model and a Variational Autoencoder with feedforward networks. The results are hilarious. You can see our simulation results in the playlist below. We plan to develop this idea beyond a ‘we tested this, this is what we got’ style course project by replacing feedforward networks with Recursive Neural Networks like LSTMs that specialize in tracking data as time series, continuing to test difference constructions, and experimenting with more ideas.

[AE unsorted (youtube.com)](https://www.youtube.com/watch?v=C5HenmXGH18&list=PLs3cT-s1EaOcz7azQ7wEMfVoVdm0u8nrH)

Cooperator: Pedram Rajaei

Course Final Report:

[MA529_Final_Report](https://zhuncollectedwork.com/wp-content/uploads/2024/08/ma529_final_report.pdf)[Download](https://zhuncollectedwork.com/wp-content/uploads/2024/08/ma529_final_report.pdf)

Course Final 10min Presentation:

[MA 529 Final Presentation](https://zhuncollectedwork.com/wp-content/uploads/2024/08/ma-529-final-presentation.pdf)[Download](https://zhuncollectedwork.com/wp-content/uploads/2024/08/ma-529-final-presentation.pdf)