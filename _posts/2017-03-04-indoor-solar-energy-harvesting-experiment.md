---
layout: post
title: On the viability of semi-indoor solar energy harvesting
subtitle: with measurements
tags: [intel edison, IoT, powering, prototyping, python]
show-avatar: true
excerpt: This is the log of my recent experiment on the viability of solar powering of an IoT device in a semi-indoor environment (indoor, but bright enough to enable a plant growing, e.g. by a window).
description: This is the log of my recent experiment on the viability of solar powering of an IoT device in a semi-indoor environment (indoor, but bright enough to enable a plant growing, e.g. by a window).
image: /img/solar_harvest_measure_fritzing.png
gh-repo: domoszlai/OpenPlantSense
gh-badge: [star, fork, follow] 
---

## Overview

![Solar harvest measurement by the window](/img/solar_harvest_measurement_location.jpg){: style="float: right;"}

This is the log of my recent experiment on the viability of solar powering of an IoT device in a semi-indoor environment (indoor, but bright enough to enable a plant growing, e.g. by a window). I basically wanted to find out what is the absolutely minimum power that can be squeezed out from a tiny 53x30mm, 5V, 30mA solar panel. The measurement took place in the Netherlands between January 10-26, when the amount of sunshine is the lowest in the year.

## The hardware

Because I did not have a real load on the solar panel, a just measured the [short-circuit current and the no-load voltage](https://en.wikipedia.org/wiki/Theory_of_solar_cells#Open-circuit_voltage_and_short-circuit_current) as an indication of the generated energy. The current was measured by an [INA219](https://learn.adafruit.com/adafruit-ina219-current-sensor-breakout/overview) device and was logged by an [Intel Edison Arduino Breakout Kit](https://www.sparkfun.com/products/13097), the voltage was simply measured by one of the analog pins of the Edison (the used software is available at [GitHub](https://github.com/domoszlai/OpenPlantSense/tree/master/experiment/solar-harvest)).

![Solar harvest measurement equipment Fritzing diagram](/img/solar_harvest_measure_fritzing.png){: style="float: right;"}

However,  the short-circuit current and the no-load voltage cannot be measured in the same time as voltage drops to nearly zero when the short-circuit current is measured. To overcome this, I used a [RFP30N06LE](https://www.sparkfun.com/datasheets/Components/General/RFP30N06LE.pdf) TTL level N-Channel MOSFET to connect and disconnect the INA219 from the circuit.

The Edison board has a 10-bit analog to digital converter, it maps input voltages between 0 and 5 volts into integer values between 0 and 1023, having a resolution of 4.9 mV. The INA219 was used in a div8 mode to have a resolution of 0.1 mA.

## The result

During the measurement, the solar panel never gave more than 4V or 0.5mA, and it generated approximately 10 mA all together in this two weeks period.

![Solar harvest measurement result](/img/solar_harvest_measurement_result.png){: style="display:block; margin:auto;"}
![Solar harvest measurement daily result](/img/solar_harvest_measurement_result_daily.png){: style="display:block; margin:auto;"}

## Conclusion

The result sounds absolutely terrible regarding that the panel could generate 30mA in ideal conditions. However, this is an **absolutely worst case scenario**, and it still would generate 20 mA a month, 240 mA a year (probably it gives this amount of power during a sunny summer day). In theory, this is certainly enough to charge e.g. a 150 mA LiPo battery in a year, which sounds ridiculous, but can be acceptable for many low power applications.

Reality can be quite different though. The solar panel gave really small amount of power, a way below 1 mA. The question is if it is enough to drive a LiPo charger chip, and how much energy the chip itself uses up... In a next post I try to find a solar harvesting chip with a very low quiescent current to see how much of the generated energy can be actually stored in a battery.

For what project would you like to use solar harvesting?






