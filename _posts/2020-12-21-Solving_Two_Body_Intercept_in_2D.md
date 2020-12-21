---
layout: post
title: Solving Two Body Intercept in 2D with Unknown Angle
---
Interesting problem: given a starting position and a constant speed, compute the angle to intercept another object moving at a constant velocity. 

This is a common issue in a few places such as real time collision physics. Some other obvious examples come to mind as well, such as AI aiming in FPS and target leads in flight action games. 

A casual google will find only [one acceptable solution](https://www.codeproject.com/Articles/990452/Interception-of-Two-Moving-Objects-in-D-Space). However, its a slightly over complicated trigonometric derivation. Also, the resulting angle will be relative to the vector from the source to destination, rather than absolute. 
A nicer solution can be done by simply solving the linear system derived from the finite difference model of the objects position.

# Solution
To begin with, consider an object with initial position \\( p_0=\{p_x, p_y\} \\) at constant speed \\(s\\) and unknown angle \\(\theta\\). The target begins at \\(q_0=\{q_x, q_y\}\\) with velocity \\(v=\{v_x, v_y\}\\). It follows each objects position at time \\(t\\) will be:

\\[p_t = p_0+t\times v\\]

\\[q_t = \{q_x + t\times s \times \cos{\theta}, q_y+t\times s\times \sin{\theta}\}\\]

As the goal is to solve for \\(p_t=q_t\\) for some \\(t=t*\\), the system of equations can be set up as follows:

\\[q_x+t*\times s \times \cos{\theta}\\]

\\[q_x+t*\times s \times \cos{\theta}\\]

\\[q_x+t*\times s \times \cos{\theta} = p_x+t\\]

\\[q_x+t*\times s \times \cos{\theta} = p_x+t* \\]

\\[q_x+t*\times s \times \cos{\theta} = p_x+t* \times v_x\\]

\\[q_y+t*\times s \times \sin{\theta} = p_y+t* \times v_y\\]

Which can be rearranged as so:

\\[\cos{\theta} = \frac{a+t*\times v_x}{t*\times s}\\]

\\[\sin{\theta} = \frac{b+t*\times v_y}{t*\times s}\\]

The unknown \\(\theta\\) can be removed by equating these expressions without loss of generality:

\\[\arccos{\frac{a+t*\times v_x}{t*\times s}} = \arcsin{\frac{b+t*\times v_y}{t*\times s}} \\]

\\[\cos{\arccos{\frac{a+t*\times v_x}{t*\times s}}} = \frac{b+t*\times v_y}{t*\times s} \\]

The trigonometric functions can be simplified out with the identity \\( \cos{\arccos{x}} = \sqrt{1-x^2} \\)
