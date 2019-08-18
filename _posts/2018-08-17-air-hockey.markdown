---
layout: post
title: Air Hockey Phase Space
date: 2018-08-17 01:47:20 -0600
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img:  air_hockey_average_score.png # Add image post (optional)
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

which can be solved for the intercepts at $$x=0$$, $$x=s_y$$, $$y=0$$, or $$y=s_y$$, depending on the value of $$\theta$$. The algorithm for the function ``nextPoint``, which computes the location of the next intersection point of the puck with the border of the table, as well as the rebound angle, is displayed below:

{% include pseudocode.html id="1" code="
\begin{algorithm}
\caption{Next Point}
\begin{algorithmic}
\PROCEDURE{nextPoint}{$x_0, y_0, \theta_0, x_1, y_1, \theta_1, s_x, s_y$}
    \STATE \textbf{Input:} $x_0, y_0, \theta_0 \in \mathbb{R}$
    \STATE \textbf{Output:} $x_1, y_1, \theta_1 \in \mathbb{R}$
    \STATE \textbf{Parameters:} $s_x, s_y \in \mathbb{R}$
    \STATE \textbf{Auxiliary:} $\vec{x}_{\mathrm{int}}, \vec{y}_{\mathrm{int}} \in \mathbb{R}^{2}$
    \IF{$\sin \theta > 0$}
        \STATE $\vec{y}_{\mathrm{int}} = \big[0, x_0 \frac{1}{\tan \theta} + y_0 \big]^T $ \COMMENT{\texttt{crosses $\mathtt{x=0}$}}
    \ELSE
        \STATE $\vec{y}_{\mathrm{int}} = \big[s_x, (x_0 - s_x) \frac{1}{\tan \theta} + y_0 \big]^T $ \COMMENT{\texttt{crosses $\mathtt{x=s_x}$}}
    \ENDIF
    \IF{$\cos \theta > 0$}
        \STATE $\vec{x}_{\mathrm{int}} = \big[ ( y_0 - s_y ) \tan \theta + x_0, s_y \big]^T $ \COMMENT{\texttt{crosses $\mathtt{y=s_y}$}}
    \ELSE
        \STATE $\vec{x}_{\mathrm{int}} = \big[ y_0 \tan \theta + x_0, 0 \big]^T $ \COMMENT{\texttt{crosses $\mathtt{y=0}$}}
    \ENDIF
    \IF{$\big\vert \big\vert \vec{x}_{\mathrm{int}} - \big[ x_0, y_0 \big]^T\big\vert \big\vert > \big\vert \big\vert \vec{y}_{\mathrm{int}} - \big[ x_0, y_0 \big]^T\big\vert \big\vert$}
        \STATE $x_1 = \vec{x}_{\mathrm{int}}[0]$ \COMMENT{\texttt{puck hits horizontal wall first ($\mathtt{x = 0}$ or $\mathtt{x = s_x}$)} }
        \STATE $y_1 = \vec{x}_{\mathrm{int}}[1]$
        \STATE $\theta_1 = \mathrm{atan2}( \sin \theta_0, -\cos \theta_0)$
    \ELSE
        \STATE $x_1 = \vec{y}_{\mathrm{int}}[0]$ \COMMENT{\texttt{puck hits vertical wall first ($\mathtt{y = 0}$ or $\mathtt{y = s_y}$)} }
        \STATE $y_1 = \vec{y}_{\mathrm{int}}[1]$
        \STATE $\theta_1 = \mathrm{atan2}( -\sin \theta_0, \cos \theta_0)$
    \ENDIF
\ENDPROCEDURE
\end{algorithmic}
\end{algorithm}
" %}

<img align="right" src="/assets/img/posts/2018-08-17-air-hockey/air_hockey_animation.gif" width="300">

The animation to the left shows the paths a puck takes for a series of angles, when its initial location is the center of the table. Small changes in $$\theta$$ can result in significant changes to the end state of the puck, as evidenced by the rapidly changing lines. This sensitive dependence on initial conditions is a hallmark of chaotic systems and suggests that the behavior or the air hockey table system is more complicated than its simple geometry suggests.

In order to study the phase space of the air hockey table in a more systematic way, we define a new function, ``bounceScore``, which computes the number of bounces the puck engages in before entering into one of the two goals. To distinguish between the two goals and maintain continuity, we define the function such that a value of -1 indicates the puck went into your goal before the first bounce, a value of 1 indicates the puck went into your opponent's goal before the first bounce, and a value of 0 indicates the puck did not end up in either goal after a set number of bounces.

{% include pseudocode.html id="2" code="
\begin{algorithm}
\caption{Bounce Score}
\begin{algorithmic}
\PROCEDURE{bounceScore}{$x_0, y_0, \theta_0, b, b_{\mathrm{max}}, s_x, s_y, w$ }
    \STATE \textbf{Input:} $x_0, y_0, \theta_0 \in \mathbb{R}$
    \STATE \textbf{Output:} $b \in \mathbb{Z}$
    \STATE \textbf{Parameters:} $b_{\mathrm{max}} \in \mathbb{Z}, s_x, s_y, w \in \mathbb{R}$
    \STATE \textbf{Auxiliary:} $i \in \mathbb{Z}$, done $\in$ [True, False]

    \STATE $i = 0$
    \STATE done = False
    \WHILE{not done}
        \IF{$y_0 == s_y$}
            \IF{$\frac{s_x - w}{2} < x_0 < \frac{s_x + w}{2}$}
                \STATE $b = b_{\mathrm{max}} - i$ \COMMENT{\texttt{puck went into opponent's goal}}
                \STATE done = True
            \ENDIF
        \ELSIF{$y_0 == 0$}
            \IF{$\frac{s_x - w}{2} < x_0 < \frac{s_x + w}{2}$}
                \STATE $b = -b_{\mathrm{max}} + i$ \COMMENT{\texttt{puck went into your goal}}
                \STATE done = True
            \ENDIF
        \ENDIF
        \STATE \CALL{nextPoint}{$x_0, y_0, \theta_0, x_0, y_0, \theta_0, s_x, s_y$}
        \STATE $ i= i+1$
    \ENDWHILE
\ENDPROCEDURE
\end{algorithmic}
\end{algorithm}
" %}

[reg-air-hockey]: https://gametableplanet.com/regulation-air-hockey-table-dimensions/
[build-air-hockey]: https://www.instructables.com/id/Air-Hockey-Table-2/
