---
layout: post
title: Prototyping of an MCP73831 based battery charger
subtitle: with breadboard and perfboard diagrams
image: /img/MCP73831_fritzing_small.png
description: In this post I share the breadboard and perfboard diagrams of an MCP73831 based Li-Ion, Li-Polymer battery charger circuit
tags: [electronics, prototyping, powering]
show-avatar: true
---

One of my ongoing projects is a very low-power electronics device that supposedly runs on a single-cell Li-Ion or Li-Polymer battery for a very long time, hopefully many-many months. In my idea, the battery actually could be a rechargeable one, e.g. LIR2032 or LIR2450 [button cell](https://en.wikipedia.org/wiki/Button_cell), and I might also be able to recharge them during these months from a small solar panel, so ideally it runs "forever" with a single battery.

I looked for charger circuits and found out there are many incredibly cheap single-chip solutions available. For some reason I do not remember any more, I picked the [MCP73831](http://ww1.microchip.com/downloads/en/DeviceDoc/20001984g.pdf) chip, which is a very small, single-cell Li-Ion, Li-Polymer battery charger IC.

![MCP73831 typical application diagram](/img/MCP73831-typical-application.png){: style="float: right;"}

To be able to experiment with the chip, I just took the typical application schematics from the specification and created breadboard and perfboard diagrams. In this post I just want to share these diagrams as they maybe useful for others as well.

{% include note.html content="Please note that this circuit has no low voltage protection. On one hand, it is very practical as you can use it to restore over-discharged batteries. On the other hand, you should be careful with such batteries. Please read [this blog post](http://www.electricrcaircraftguy.com/2014/10/restoring-over-discharged-LiPos.html) about recharging over-discharged LiPo batteries." %}

![Sot23 dip10 prototype board](/img/sot23-dip10.png){: style="float: left; margin-right: 10px;"}

This chip is coming in a SOT23-5 package, so it is actually too small for prototyping; first of all, it must be converted to DIP format using a SOT23 to DIP adapter board. I used a DIP10 one, because that was at home, but there are other, smaller variants (less legs) as well.

The diagrams are slightly modified. First of all, I replaced the 2K resistor with a 10K one, so it has 100mA output instead of 500mA. I also added one more LED indicating incoming power. So, the green LED is always on, the red LED indicates the charging status.

- Blinking: no battery / battery fault
- On: battery is being charged
- Off: battery is fully charged 

I created [Fritzing](http://fritzing.org/home/) diagrams, and also a [DIY Layour Creator](http://diy-fever.com/software/diylc/) one for the perfboard diagram as I find it nicer than the Fritzing one:

![MCP73831 based battery charger Fritzing diagram](/img/MCP73831_fritzing.png){: style="display:block; margin:auto;"}

![MCP73831 based battery charger DIY Layour Creator diagram](/img/MCP73831_diylc.png){: style="display:block; margin:auto;"}

And finally, the realization. Two remarks: 1. I added some more connectors to the perfboard to make it more useful 2. A single-sided perfboard would be sufficient, and would look nicer, but I had only the double-sided at home

![MCP73831 based battery charger on breadboard](/img/MCP73831-breadboard.jpg){: style="display:block; margin:auto;"}
![MCP73831 based battery charger on perfboard](/img/MCP73831-perfboard.jpg){: style="display:block; margin:auto;"}


