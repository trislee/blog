---
layout: post
title: Air Hockey Phase Space
date: 2018-08-17 01:47:20 -0600
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img:  low_quality_render.png # Add image post (optional)
tags: [Blog, Chaos]
author: # Add name author (optional)
---
<img align="right" width="250" src="/assets/img/posts/2018-08-17-air-hockey/air_hockey_diagram.svg">

After a night out at a barcade a few years back, I came home and began to think about the phase space of an air hockey game. To be precise, given a position on the table $$(x_0, y_0)$$ and a velocity vector with direction specified by an angle $$\theta$$, will the puck go into my goal, my opponent's goal, or just bounce between walls forever? For simplicity, I'm assuming a perfectly frictionless table surface and perfectly elastic collisions. A diagram of the idealized air hockey table, with a coordinate system and relevant variables and parameters, is shown to the right. These parameters are:

* $$s_x$$ : length of table in $$x$$ dimension, taken to be 1.0 in non-dimensionalized length units

* $$s_y$$ : length of table in $$y$$ dimension, taken to be 1.8 in non-dimensionalized length units

* $$w$$ : width of goal, taken to be 0.16 in non-dimensionalized length units,

with values of these parameters taken from [these][reg-air-hockey] [websites][build-air-hockey].

The strategy employed for finding the final destination of the puck consists of calculating the intersection point of the puck with a particular wall and changing the angle consistent with the law of reflection, repeating the process until the puck lands within the goal, or the maximum number of bounces is reached.
The intersection point can be calculated explicitly using basic algebra:
based on the definition of $$\theta$$, the slope at which the puck will travel is $$-\frac{1}{\tan \theta}$$. By rearranging the expression for point-slope form, we get the equations

$$
\begin{aligned}
y &= \frac{1}{\tan \theta} (x_0 - x) + y_0\\
x &= \tan \theta (y_0 - y) + x_0,
\end{aligned}
$$

which can be solved for the intercepts at $$x=0$$, $$x=s_y$$, $$y=0$$, or $$y=s_y$$, depending on the value of $$\theta$$.

<img align="left" src="/assets/img/posts/2018-08-17-air-hockey/air_hockey_animation.gif" width="300">

The animation to the left shows the paths a puck takes for a series of angles, when its initial location is the center of the table. Though the air hockey table system and its dynamics appear simple at first glance, the animation demonstrates that the emergent behavior is fairly complex. A small change in $$\theta$$ can result in a significant change to the end state of the puck. This sensitive dependence on initial conditions is a hallmark of chaotic systems and suggests

[reg-air-hockey]: https://gametableplanet.com/regulation-air-hockey-table-dimensions/
[build-air-hockey]: https://www.instructables.com/id/Air-Hockey-Table-2/
