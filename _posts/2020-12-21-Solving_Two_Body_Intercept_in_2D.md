---
layout: post
title: Solving Two Body Intercept in 2D with Unknown Angle
---

Two bodies .

This is a common issue in a few places, and a casual google will find only [one acceptable solution](https://www.codeproject.com/Articles/990452/Interception-of-Two-Moving-Objects-in-D-Space). However, its a slightly over complicated trigonometric derivation, with the resulting angle being relative to the vector from the source to destination. 
A nicer solution can be done by simply solving 

# Solution
To begin with, given initial position \\( q=\{q_x, q_y\} \\) at constant speed \\(s)\\/
