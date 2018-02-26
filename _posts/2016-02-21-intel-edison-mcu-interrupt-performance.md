---
layout: post
title: Intel Edison MCU interrupt performance issues
tags: [intel edison]
description: The Intel Edison microcontroller has serious interrupt performance issues as it is illustrated by ultrasonic sensor (HCSR04) use case.
excerpt: I have been thinking a while about moving from Arduino to a platform with more computational power. I wanted something more powerful, but having real time capabilities in the same time. I tried inted Edison, and my experience was seriously disappointing. 
image: /img/intel_edison.jpg
show-avatar: true
---

{% include update.html content="Intel Edison was discontinued on June 19, 2017." %}

## Overview

I have been thinking a while about moving from Arduino to a platform with more computational power. I want something more powerful, but having real time capabilities in the same time. My first candidate was a [Raspberry Pi](https://www.raspberrypi.org/), it is fast and could be used along with an Arduino connected by [I2C](https://en.wikipedia.org/wiki/I%C2%B2C). Then I learned about [Intel Edison](http://www.intel.com/content/www/us/en/do-it-yourself/edison.html); it is more expensive, but has lower power consumption and, more importantly, has an embedded microcontroller (aka MCU), an [Intel Quark](http://www.intel.com/content/www/us/en/embedded/products/quark/overview.html), that 

> adds deterministic behavior to Linux applications as a service 

Thus I acquired one, but, just after my very first test, I'm seriously disappointed.

## The test

As a first test, I wanted something which requires real time scheduling while easy to perform. Programming the ubiquitous HCSR04 ultrasonic range sensor seemed a trivial choice; obviously Intel also thought so as they already had a [tutorial](https://software.intel.com/en-us/articles/using-an-mcu-on-the-intel-edison-board-with-the-ultrasonic-range-sensor) for it.

Programming such a sensor is quite straightforward. It has a trigger and an echo pin. A ping can be triggered by setting the trigger pin HIGH for more than 10 microseconds. Right after the ping impulse is sent by the device, the echo pin goes HIGH until the physical echo impulse is received. So all we have to do is to measure how long the echo pin is HIGH. The elapsed time is directly proportional to the distance. For this, we need real time scheduling as it is written in the tutorial as well:

> It’s not possible to measure the duration of such a short pulse with estimated accuracy without microsecond real-time delays. For example, the scheduler could preempt the measurement process and the measurement result will become invalid.

The only problem is that the tutorial develops a synchronous toy example what is insufficient if you want to use this expensive processor for doing more than one thing in the same time.

## Interrupts on the Edison MCU

For asynchronous behavior you need to utilize interrupts on a microcontroller. After a quick search on how to do that on an Edison MCU, I also learned that there are [serious performance issues](https://communities.intel.com/thread/76232?start=0&tstart=0) related to interrupt handling (by the discussion, the MCU can flawlessly handle only interrupt rates up to 100Hz). First warning sign, but I could live with that as HCSR04 measurements can be safely performed on every ~40-50 milliseconds only (according to the datasheet). Much less than 100Hz, hopefully I do not want to use interrupts for other purposes as well in the future...

On an Arduino, you may want to use a timer to periodically check the state of the *echo pin*, because [external interrupts](https://www.arduino.cc/en/Reference/AttachInterrupt) are available only for specific pins (although pin change interrupts work on most of the pins). This is how the NewPing library works. However, there are no timer interrupts on the Edison MCU. Second warning sign, but I can live with that as I do not like this solution anyway.

Fortunately, Edison MCU enables external interrupts on all the pins. That sounds great, all I have to do is to attach an interrupt handler on the *echo pin* which triggers at both falling edge and rising edge and measure the elapsed time in between. It turns out, however, that an interrupt can be either falling edge or rising edge on Edison MCU. No problem, then I attach two interrupt handlers... Not possible on the same pin... Third warning sign.

Ok. Then I'll wait until the echo pin goes HIGH, save the current microsecs, and calculate the distance in the falling edge interrupt handler. Not optimal, but most time is spent while the *echo pin* is HIGH, anyway. This is how the final program looks like, it is a slightly modified version of the code developed in the Intel tutorial:

```c
#include "mcu_api.h"
#include "mcu_errno.h"

#define TRIGGER_PIN 49
#define ECHO_PIN 48

// From HCSR04 datasheet
#define MIN_DISTANCE 2
#define MAX_DISTANCE 400

#define MAX_WAIT 10000

volatile unsigned long ping = 0;

int echo_irq(int req) {
   unsigned long echo = time_us();

   int distance = (echo - ping) / 58;
   if(distance > MIN_DISTANCE && distance < MAX_DISTANCE) {
      debug_print(DBG_INFO, "DISTANCE: %d\n", distance);
   }

   return IRQ_HANDLED;
}

void send_ping() {
   // Trigger a ping 
   gpio_write(TRIGGER_PIN, 1);
   mcu_delay(10);
   gpio_write(TRIGGER_PIN, 0);

   int i = 0;

   // Poor man's method of detecting "rising edge"
   while ((gpio_read(ECHO_PIN) == 0) && (i++ < MAX_WAIT)) {
      mcu_delay(1);
   }

   ping = time_us();
}

void mcu_main() {
   gpio_setup(ECHO_PIN, 0);
   gpio_setup(TRIGGER_PIN, 1);
   gpio_write(TRIGGER_PIN, 0);
   gpio_register_interrupt(ECHO_PIN, 0, echo_irq); // 0 means falling edge

   while(1) {
      send_ping();
      mcu_delay(500000); // wait half a second
   } 
}
```

## The latency

At first sight it looked like working perfectly, but at second sight I realized a tiny flaw: **everything appeared four centimeters further away**. Something is very wrong with the program. Four centimeters means ~200 microseconds, I believe that this is caused by a such a big latency between the occurrence of the event and the execution of the interrupt handler.

## Interrupts on Arduino

To double check that not I made a mistake, I ported the code above to Arduino by simply replacing the names of the API functions; no structural changes.  Needless to say, it works perfectly (I tested it on an Arduino Nano).

```c
#define TRIGGER_PIN 3
#define ECHO_PIN 2

// From HCSR04 datasheet
#define MIN_DISTANCE 2
#define MAX_DISTANCE 400

#define MAX_WAIT 10000

volatile unsigned long ping = 0;

void echo_irq() {
   unsigned long echo = micros();

   int distance = (echo - ping) / 58;
   if(distance > MIN_DISTANCE && distance < MAX_DISTANCE) {
      Serial.print("DISTANCE: ");
      Serial.println(distance);
   }
}

void send_ping() {
   // Trigger a ping 
   digitalWrite(TRIGGER_PIN, HIGH);
   delayMicroseconds(10);
   digitalWrite(TRIGGER_PIN, LOW);

   int i = 0;

   // Poor man's method of detecting "rising edge"
   while ((digitalRead(ECHO_PIN) == LOW) && (i++ < MAX_WAIT)) {
      delayMicroseconds(1);
   }

   ping = micros();
}

void setup() {
   Serial.begin(9600);

   pinMode(ECHO_PIN, INPUT);  
   pinMode(TRIGGER_PIN, OUTPUT);
   digitalWrite(TRIGGER_PIN, LOW);    
   attachInterrupt(digitalPinToInterrupt(ECHO_PIN), echo_irq, FALLING);
}

void loop() {
   send_ping();
   delay(500);  // wait half a second
}
```

What do you use/want to use an Intel Edison?



