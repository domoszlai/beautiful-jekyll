---
layout: post
title: 3D printed mini ball launcher
tags: [kid friendly robots, 3d print]
description: This post describes the design of a simple 3D printed mini ball launcher. 
excerpt: I made a 3D printed mini ball launcher for my kids as part of a bigger project. 
image: /img/ball_launcher_design_small.png
show-avatar: true
---

It is one part of a bigger project I have been thinking about for some time now, a remote controlled turret for my kids. A video is worth 1000 words, so let's see the result first:

[![Mini ball launcher in action](/img/3d_printed_mini_ball_launcher_video.jpg){: style="display:block; margin:auto;"}](https://www.youtube.com/watch?v=nHBG8JRPoDQ)

The design is straightforward, harnesses centrifugal force with a specially designed rotor:

![Design of the ball launcher](/img/ball_launcher_design.png){: style="display:block; margin:auto;"}

The 3D files, along with the OpenSCAD source, can be downloaded from [Thingiverse](http://www.thingiverse.com/thing:1645025).

![R140 size motor](/img/r140_sized_motors.png){: style="float: right; margin-left: 10px"}
I used an R140 sized electric motor salvaged from some old toy. You can use a different motor, it is easy to change the OpenSCAD file, but you should consider the RPM of the motor.
I do not want to provide any specific number here, pick up an RPM of your taste (1. be careful, if the motor is too fast, the launcher may be harmful 2. you do not really need to consider the torque, the ball should be very light) , but I can help a bit with the calculations :

For a specific range, you can use [this site](http://www.calctool.org/CALC/phys/newtonian/projectile) to calculate the necessary initial velocity. The radius of the rotor is 35mm, that is the ball runs ~0.22m every revolution....

Have fun, and don't let the kids play with it alone! 

