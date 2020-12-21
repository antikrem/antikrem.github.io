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

\\[p_t = p_0+vt\\]

\\[q_t = \{q_x + s t \cos{\theta}, q_y+s t \sin{\theta}\}\\]

As the goal is to solve for \\(p_t=q_t\\) for some \\(t=t^\*\\), the system of equations can be set up as follows:

\\[q_x+t^{\*} s \cos{\theta} = p_x+t^{\*} v_x\\]

\\[q_y+t^{\*} s \sin{\theta} = p_y+t^{\*} v_y\\]

Which can be rearranged as so:

\\[\cos{\theta} = \frac{a+t^{\*} v_x}{t^{\*} s}\\]

\\[\sin{\theta} = \frac{b+t^{\*} v_y}{t^{\*} s}\\]

With \\(a = p_x-q_x\\) and \\(b = p_y-q_y\\).

The unknown \\(\theta\\) can be removed by equating these expressions without loss of generality:

\\[\arccos{ \bigg{(} \frac{a+t^{\*} v_x}{t^{\*} s} \bigg{)}} = \arcsin{ \bigg{(} \frac{b+t^{\*} v_y}{t^{\*} s} \bigg{)} } \\]

The trigonometric functions can be simplified out with the identity \\( \cos{(\arccos{x})} = \sqrt{1-x^2} \\):

\\[\sqrt{ 1-\bigg{(}\frac{a+t^{\*} v_x}{t^{\*} s} \bigg{)}^2 } = \frac{b+t^{\*} v_y}{t^{\*} s} \\]

\\[ 1-\bigg{(}\frac{a+t^{\*} v_x}{t^{\*} s} \bigg{)}^2  = \bigg{(} \frac{b+t^{\*} v_y}{t^{\*} s} \bigg{)}^2 \\]

Multiplying both sides by \\({t^{\*}}^2 s^2\\)

\\[ {t^{\*}}^2 s^2-(a+t^{\*} v_x)^2  = (b+t^{\*} v_y)^2 \\]

\\[ {t^{\*}}^2 s^2-a^2-2 a t^{\*} v_x - {t^{\*}}^2 {v_x}^2 = b^2 + 2 b t^{\*} v_y + {t^{\*}}^2{v_y}^2\\]
