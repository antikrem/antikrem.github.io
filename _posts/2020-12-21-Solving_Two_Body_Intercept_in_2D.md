---
layout: post
title: Solving Two Body Intercept in 2D with Unknown Angle
---
Interesting problem: given a starting position and a constant speed, compute the angle to intercept another object moving at a constant velocity. 

This is a common issue in a few places such as real time collision physics. Some other obvious examples come to mind as well, such as AI aiming in FPS and target leads in flight action games. 

A casual google will find only [one acceptable solution](https://www.codeproject.com/Articles/990452/Interception-of-Two-Moving-Objects-in-D-Space). However, its a slightly over complicated trigonometric derivation. Also, the resulting angle will be relative to the vector from the source to destination, rather than absolute. 
A nicer solution can be done by simply solving the linear system derived from the finite difference model of the objects position.

# Solution
Consider an object with initial position \\( p_0=\{p_x, p_y\} \\) at constant speed \\(s\\) and unknown angle \\(\theta\\). The target begins at \\(q_0=\{q_x, q_y\}\\) with velocity \\(v=\{v_x, v_y\}\\). The intersect time will be 

\\[t^{\*} =\frac{(2 j v_x + 2 k v_y)\\pm \sqrt{(-2 j v_x - 2 k v_y)^2-4(s^2 - {v_x}^2 - {v_y}^2)(- j^2 - k^2)}}{2(s^2 - {v_x}^2 - {v_y}^2)} \\]

With \\(j = p_x-q_x\\) and \\(j = p_y-q_y\\).

And the angle being:

\\[\theta = \arctan{\frac{k+t^{\*} v_y}{j+t^{\*} v_x }}\\]

# Derivation
To begin with, each objects position at time \\(t\\) will be:

\\[p_t = p_0+vt\\]

\\[q_t = \{q_x + s t \cos{\theta}, q_y+s t \sin{\theta}\}\\]

As the goal is to solve for \\(p_t=q_t\\) for some \\(t=t^\*\\), the system of equations can be set up as follows:

\\[q_x+t^{\*} s \cos{\theta} = p_x+t^{\*} v_x\\]

\\[q_y+t^{\*} s \sin{\theta} = p_y+t^{\*} v_y\\]

Which can be rearranged as so:

\\[\cos{\theta} = \frac{a+t^{\*} v_x}{t^{\*} s}\\]

\\[\sin{\theta} = \frac{b+t^{\*} v_y}{t^{\*} s}\\]

With \\(j = p_x-q_x\\) and \\(j = p_y-q_y\\).

The unknown \\(\theta\\) can be removed by equating these expressions without loss of generality:

\\[\arccos{ \bigg{(} \frac{j+t^{\*} v_x}{t^{\*} s} \bigg{)}} = \arcsin{ \bigg{(} \frac{k+t^{\*} v_y}{t^{\*} s} \bigg{)} } \\]

The trigonometric functions can be simplified out with the identity \\( \cos{(\arccos{x})} = \sqrt{1-x^2} \\):

\\[\sqrt{ 1-\bigg{(}\frac{j+t^{\*} v_x}{t^{\*} s} \bigg{)}^2 } = \frac{k+t^{\*} v_y}{t^{\*} s} \\]

\\[ 1-\bigg{(}\frac{j+t^{\*} v_x}{t^{\*} s} \bigg{)}^2  = \bigg{(} \frac{k+t^{\*} v_y}{t^{\*} s} \bigg{)}^2 \\]

Multiplying both sides by \\({t^{\*}}^2 s^2\\):

\\[{t^{\*}}^2 s^2-(j+t^{\*} v_x)^2  = (k+t^{\*} v_y)^2 \\]

\\[{t^{\*}}^2 s^2-j^2-2 j t^{\*} v_x - {t^{\*}}^2 {v_x}^2 = k^2 + 2 k t^{\*} v_y + {t^{\*}}^2{v_y}^2\\]

\\[{t^{\*}}^2 (s^2 - {v_x}^2 - {v_y}^2) + t^{\*}(-2 j v_x - 2 k v_y) + (- j^2 - k^2) = 0 \\]

This gives a quadratic on \\(t^{\*}\\), which can be solved with the quadratic formula:

\\[a = s^2 - {v_x}^2 - {v_y}^2 \\]

\\[b = -2 j v_x - 2 k v_y \\]

\\[c = - j^2 - k^2 \\]

\\[t^{\*} =\frac{-b\\pm \sqrt{b^2-4ac}}{2a} \\]

## Interpreting \\(t^{\*}\\)
The quadratic formula has two solutions. Each solution has four possible states:

#### Negative
This indicates that a collision, i.e. \\(p_t=q_t\\) would have been possible at an earlier time. Generally, at most 1 solution will be negative, especially when the source's position lies almost perpendicular to the target's velocity.

#### Non-Real
Such a solution is due to parameters not allowing a solution. An example is a target thats moving away from the source faster than \\(s\\). As expected, with impossible parameters both solutions will be non-real.

#### Undefined
When \\(a = s^2 - {v_x}^2 - {v_y}^2 = 0\\), there is a division by zero. Such a case occurs when both objects have the same speed. Outside of the degenerate case, a collision would not be possible, and solutions to the quadratic fomula will be undefined.

#### Positive
A positive real solution is workable.

### Choosing solution
Negative and non-real solutions should be discarded. Given valid parameters, either one or two positive solutions will be generated. Two positive solutions occur when two angles exist that cause intercept.

In this case, generally just choose the smaller solution for consistency. For application (especially gameplay), choosing the faster collision is useful, as the target has less time to change velocity.

When no positive real solution is possible, the model is simply not solvable. No angle exists that can cause an intersection. In this case, using the direct angle to target is probably the most graceful backup, though exceptions and error codes might be more your style.

## Solving Angle
Solving for \\(theta\\) also comes from the system of equations:
\\[t^{\*} s  = \frac{j+t^{\*} v_x}{\cos{\theta}}\\]

\\[t^{\*} s = \frac{k+t^{\*} v_y}{\sin{\theta} }\\]

Which can be equated:

\\[\frac{j+t^{\*} v_x}{\cos{\theta}} = \frac{k+t^{\*} v_y}{\sin{\theta} }\\]

\\[\frac{\sin{\theta}}{\cos{\theta}} = \frac{k+t^{\*} v_y}{j+t^{\*} v_x }\\]

And recalling \\(\tan{\theta}=\sin{\theta}/\cos{\theta}\\):

\\[\tan{\theta} = \frac{k+t^{\*} v_y}{j+t^{\*} v_x }\\]

\\[\theta = \arctan{\frac{k+t^{\*} v_y}{j+t^{\*} v_x }}\\]

# Implementation
