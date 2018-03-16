---
layout: post
title: DIY robot chassis with LEGO platform 
tags: [arduino, kid friendly robots, robot car]
excerpt: The first version of my robot car went broken in no time, the acrylic chassis was a way to fragile for my kids. I decided that this time I build a chassis myself, something more sturdy, less expensive, more interesting for the kids, and (hopefully) cool looking.
description: The first version of my robot car went broken in no time, the acrylic chassis was a way to fragile for my kids. I decided that this time I build a chassis myself, something more sturdy, less expensive, more interesting for the kids, and (hopefully) cool looking.
show-avatar: true
image: /img/robot_car_chassis_with_lego_platform.png
gh-repo: domoszlai/stitson
gh-badge: [star, fork, follow] 
---

{% include update.html content="This robot car has a new version, [Mr. Stitson](https://dlacko.org/blog/2016/01/01/mr-stitson-kid-friendly-arduino-lego)" %}

## Overview

![Robot car chassis with LEGO platform](/img/robot_car_chassis_with_lego_platform.png){: style="float: left; margin-right: 10px"}

The first version of my robot car went broken in no time, the acrylic chassis was a way to fragile for my kids. Fortunately, all the other parts survived, only the acrylic parts were needed to be replaced. There also had to be found a way of mounting the motors. This time I wanted a more kid friendly design, at least the electronics should have been hidden in the chassis, but something that was also a LEGO platform was preferred.  I came up with the simplest possible design, an undercarriage and a cover with the electronics in-between, and a LEGO baseplate fastened to the top of the cover board.

{% include update.html content="The cat finally got named as [Mr. Stitson](https://dlacko.org/blog/2016/01/01/mr-stitson-kid-friendly-arduino-lego) after the villain of the book we happened to read to the kids that time." %}

I listed over the material alternatives. The candidates had to be inexpensive, easy to work with, stiff and light. In this order; I was afraid it was already too much to ask for...

- **Aluminium**: That would be lovely, but also a way too expensive, especially for prototyping purposes.
- **Acrylic**: Not for me. It is just too rigid, and hard to work with (although it is one of the cheapest option); I did not want to risk a crack every time when a new hole needed to be drilled (it is a completely different case, though, if you can plan ahead and laser cut all the necessarily holes).
- **Plastic**: [HDPE](https://en.wikipedia.org/wiki/High-density_polyethylene) is one of the most commonly used plastic materials in robotics. I was just not sure whether it is stiff enough for such a wide chassis I wanted.
- **Wood**: I filtered it out in the first place as being too low-tech, but otherwise it fulfilled all the requirements. Then I learnt how cheap it is; since that I believe the marriage of low-tech and high-tech is actually an interesting idea...

If opting for wood, basically there are two options, [plywood](https://en.wikipedia.org/wiki/Plywood) and [MDF](https://en.wikipedia.org/wiki/Medium-density_fibreboard). Plywood is many thin sheets of wood glued together, while MDF is basically sawdust and glue, fused together under pressure and heat. Personally, I think plywood looks nicer, and also lighter a bit than MDF, but it has an annoying tendency to split among wood grain; without a laser cutter I felt more confident with MDF.

MDF comes in large tables. The smallest one I could buy was 122x61cm, enough for ten robots at least (for the price of an ice cream).  The 4mm thick version felt a bit too flexible, but the 5mm thick one seemed alright.

First, I designed an easily 3D printable simple [T-shaped motor mount](https://www.thingiverse.com/thing:1169412):

![T-shaped motor mount](/img/motor_mount_T_preview.jpg){: style="display:block; margin:auto;"}

The stem of the T-shape is supposed to be put through an oblong hole cut into the chassis (it was trivial to make with a Dremel). The motor mount then is fastened to the chassis from the top by four M3 screws, while the motor is fixed to the stem of the T-shape by another two long screws.

![Motor mounted](/img/motor_mounted.jpg){: style="display:block; margin:auto;"}

Next, I put the main components of the robot on the MDF board to estimate the necessary size of the chassis. Then I allocated same space for the motor mounts by drawing their contours on the board. Finally, the positions of the components were adjusted taking the motor mounts into account. This method took only a couple of minutes, and the result was quite accurate (with the LEGO board on the top, the positions of the hex spacers also should be chosen carefully, you want the head of the screws to be between the "bumps").

![Robot chassis with components](/img/robot_chassis_with_components.jpg){: style="display:block; margin:auto;"}

![Amsterdam acrylic paint black](/img/amsterdam_black.jpg){: style="float: left;"}

The most critical part is to cut out the chassis from the board (twice, a second one for the cover). I believe that it is just impossible to make a nice, long, precision cut with a hand saw. Anyway, the result was satisfactory, and actually much better than I expected. I finished it up with polishing the cut edges a bit.

Eventually, the chassis looked really nice, but just as low-tech as I expected. My wife came up with the idea to treat the wood with some high quality acrylic paint. There was some from the Amsterdam brand at home, we used a mini roller to apply it to the chassis as evenly as possible. I was very pleased with the result, it looked really "crafted".

![Robot chassis undercarriage](/img/robot_chassis_undercarriage.jpg){: style="display:block; margin:auto;"}

And the proof that the idea really worked out:

![Proof of work](/img/robot_lego_platform_works.jpg){: style="display:block; margin:auto;"}

First test, without the cover yet:

[![Kid friendly robot car v2 (halfway ready)](/img/robot_car_v2_video.jpg)](https://www.youtube.com/watch?v=s46vjU8FAJA){: style="display:block; margin:auto; width: 400px"} 

Lastly, the list of parts and cost:

Part | Price
--- | --- 
MDF board | < 5$
3D printed motor mount x 4 | -
6cm hex spacer x 6| ~ 2$
LEGO Building Plate (10" x 10") | ~ 10$
Amsterdam acrylic paint | ~6$ 
 | < 23$

What robot car chassis would *you* use?



