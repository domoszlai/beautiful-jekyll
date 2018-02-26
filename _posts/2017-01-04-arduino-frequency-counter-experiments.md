---
layout: post
title: Arduino frequency counter experiments
description: In this post I summarize my recent experiments with different frequency counting approaches using the Arduino platform. My original goal is to develop a small, capacitive proximity sensor, that is more reliable than the default charge time measurement based one
excerpt: In this post I summarize my recent experiments with different frequency counting approaches using the Arduino platform. My original goal is to develop a small, capacitive proximity sensor, that is more reliable than the default charge time measurement based one
tags: [arduino, proximity sensor]
show-avatar: true
image: /img/Frequency-counter-kit.jpg
gh-repo: domoszlai/arduino-frequency-counter
gh-badge: [star, fork, follow] 
---

## Overview

In this post, I would like to summarize my recent experiments with different frequency counting approaches using the Arduino platform. My original goal is to develop a small, capacitive proximity sensor, that is more reliable than the default [charge time measurement based](http://playground.arduino.cc/Main/CapacitiveSensor?from=Main.CapSense) one (and just as importantly, works on a battery). The idea is, similarly to a [theremin](https://en.wikipedia.org/wiki/Theremin#Operating_principles), to use an oscillator to generate square waves and detect small changes in the frequency caused by the proximity of a person. For this, I needed a reliable frequency counter first. My requirements were the following:

- it should be as precise as possible for at least up to 100KHz
- it should work on an ATtiny (even on an ATtiny13)

{% include update.html content="UPDATE: check out the proximity sensor this library is developed for: [http://dlacko.org/2017/01/08/arduino-capacitive-proximity-sensor](http://dlacko.org/2017/01/08/arduino-capacitive-proximity-sensor)" %}

## Results

I ended up with a [small library](https://github.com/domoszlai/arduino-frequency-counter) that is able to count frequency up to the MHz range on the more complex platforms (e.g. Uno, Mega), and I also got it working on most of the ATtiny ones (I tested it on a ATtiny85), but not on the ATtiny13 (technically it would be possible, but I could not fit the proximity sensor code into the 1K flash memory, so I decided not to waste more time on it). On an ATTiny, the inaccuracy of the internal oscillator may affect the measurement with a constant factor, but this is fine for my application (why would you count frequency with an ATTiny anyway?)

The library uses fixed 100ms [gate time](https://en.wikipedia.org/wiki/Frequency_counter#Operating_principle) (the time period of counting pulses), which introduces some error (the pulse count is multiplied by 10 to get the frequency, thus, the last digit is always 0). Because of this, it is also worth to mention that this library does not work well for very low frequencies.

The frequency counter library is available at [https://github.com/domoszlai/arduino-frequency-counter](https://github.com/domoszlai/arduino-frequency-counter). It implements two different approaches as they both have pros and cons, and a separate one for ATTiny. Implementation details can be found in the source code and on the github page.

## Approaches

I identified three basic frequency counting methods based on the number of  required hardware timers:

- **No timer**: 

This is a very naive approach, and it is not even entirely timer less as it implicitly uses `Timer0` for counting time. Theses methods are usually based on one of the `pulseIn()`, `millis()`, `micros()` functions to measure gate time or pulse width. I found them too imprecise for my purposes.

- **1 timer**: 

This method uses one timer for gate time measurement and a pin-change interrupt (PCI) for counting the pulses. It is very generally usable as pin-change interrupts are available for many/most pins, but it may work well for lower frequencies only (it worked perfectly with ~60KHz in my tests). This is the method used by the ATTiny counter as the smallest ones, e.g. ATTiny85, has only one available timer (two but, `Timer0` is used by the Arduino core). 

See: [frequency_counter_PCI.cpp](https://github.com/domoszlai/arduino-frequency-counter/blob/master/frequency_counter_PCI.cpp)

- **2 timers**:

I found this the most reliable method that works for up to [several MHz](http://interface.khm.de/index.php/lab/interfaces-advanced/theremin-as-a-capacitive-sensing-device/). It uses one timer for gate time measurement and a **hardware timer/counter (TC)** for counting the pulses. However, the hardware counter requires the usage of one specific pin, what can be very impractical in some situations. It must be the `T1` pin (usually pin 5) for most boards, but `T5` (pin 47) in the case of an Arduino Mega.

See: [frequency_counter_TC.cpp](https://github.com/domoszlai/arduino-frequency-counter/blob/master/frequency_counter_TC.cpp)

Do you know any other/better way to measure frequency reliably? Maybe measuring pulse width using external interrupts?



 
