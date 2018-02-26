---
layout: post
title: 3D printer as a beginners DIY project
subtitle: and the things I've learned from building one
tags: [3D printing, delta robot]
description: The story of what I learned from building a 3D printer as my first DIY project. 
excerpt: I wanted a 3D printer. I built a Rostock Mini and learned many unexpected things. 
image: /img/Rostock_Mini_Pro.jpg
show-avatar: true
---

[I wanted a 3D printer](http://dlacko.blogspot.nl/2015/10/about-blog.html). After some hesitation I decided to build a Rostock Mini for two main reasons:

* delta robots looks very high-tech and elegant
* the Rostock Mini is just [awesome](http://reprap.org/wiki/File:Deltaprogress.jpg)

Later on it became a [Rostock Mini Pro](http://reprap.org/wiki/Rostock_Mini_Pro); that is build with universal joints used for model helicopters (the original Rostock Mini uses 3D printed u-joints) what I found much nicer.

If you've never build something complicated like a delta robot, even a single look at the part list is frightening and extremely demotivating. It really looked like rocket science (conceptual complexity can be very high if one knows nothing about a subject), so I decided that I do not try to understand it at once, but take a bottom-up approach, and break it into small subsystems. As a consequence, I concentrated the delta robot for a while, and forgot the 3D printing parts. Many people builds this 3D printer at home, it is easy to find detailed information on how to build one, I rather highlight the most important things I learned from this first project.

The first lesson to learn is that on a low budget, parts are coming from China. It is cheap and shipping is free most of the time. The downside is that it usually takes 2-3 weeks to deliver (the quality is also very erratic, slightly biased towards the low end). However, I found it a great fun to wait for the parcels and put the parts together week by week (literally), I was not in a hurry.

Second lesson, you need tools. Probably one has a screwdriver, but not necessarily a soldering station, a thread cutter, a multimeter, a [Dremel](https://en.wikipedia.org/wiki/Dremel), etc. You will get these items in no time.

Third lesson. 3D printers suck. A lot. Some parts of the Rostock Mini must be 3D printed, and I found out there is a printer at the university I work for. It is a [BigBuilder Dual Feed](http://builder3dprinters.com/products/big-builder-dual-feed-overview/), which is not a low end printer  (but not very expensive either). Still, it is painful to calibrate it (regularly): to level the bed (the head is supposed to be 0.2mm from the bed, you have to adjust it manually by turning some screws); to find the proper printing parameters; to replace the adhesive tape on the bed; ... and so on. The quality of the result is also at least questionable some times. [This article](https://www.techrepublic.com/article/why-desktop-3d-printing-still-sucks/) is speaking from my heart:

> The way it's advertised, you'd think you could set up your 3D printer, hit print, leave for the day, and return home later to an item made out of thin air that you can use immediately. It's like Christmas morning, right? 
>
> Unfortunately, that's not exactly the case. You may come back to a tangled mess of plastic, or an extruder printing with no plastic coming out at all. It might be jammed, paused, beeping, or flashing lights. It may have never started at all.

I learnt to hate it even if I find it very useful when I need a custom object (and I still use it to print parts for my other projects).

Anyway, this made me not to finish the 3D printer. The delta robot is finished, fully functional and was a lot of fun to build. One has never been happier than me when my stepper motor had moved the first time. My kids spent hours controlling the robot from a simple application, and explained to everyone how it is supposed to work.

[![Playing around with Rostock Mini](/img/playing_around_with_rostock_mini.jpg)](http://www.youtube.com/watch?v=w41N7MBomaw "Playing around with Rostock Mini"){: style="display:block; margin:auto; width: 400px"} 

One can also learn a great deal from it:  the basic building parts of the mechanics, bearings, pulleys, belts, screws, nuts and so on; materials; electronics basics and prototyping; the types of electric motors used in robotics; microcontrollers; firmwares, developing embedded software; kinematics; just a few things off the top of my head.

What did you learn from your first DIY project?
 