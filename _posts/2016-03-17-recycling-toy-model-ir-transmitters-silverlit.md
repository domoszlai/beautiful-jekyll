---
layout: post
title: Recycling toy model IR transmitters with Arduino
subtitle: reverse enginering a Silverlit Falcon Nano controller
tags: [arduino, kid friendly robots, reverse engineering]
description: Learn how to reuse a spare infrared transmitter from a toy vehicle on the example of a Silverlit Falcon Nano controller
excerpt: My kids received a Silverlit Falcon Nano toy helicopter for Christmas. They broke it it no time, so I ended up with broken helicopter, a working controller and a small challenge. 
image: /img/silverlit_falcon_nano_controller.jpg
show-avatar: true
---

## Overview

![silverlit falcon nano controller](/img/silverlit_falcon_nano_controller.jpg){: style="float: right"}

My kids received a [Silverlit Falcon Nano](http://www.silverlit.com/a/product/nano-falcon-xs/) toy helicopter for Christmas. It is a fantastic contraption, but it is certainly not for small kids, especially that there are no spare parts available. Long story short, it went broken before they actually learned how to fly it properly. The actual helicopter is nice enough to hang it from the ceiling of the kids' room, but I also felt sorry for the remote controller as it actually looks very decent compared to the usual chinese RC toys. Thus I decided to reuse it for one of my future projects.

## The IR receiver 

A brief research told me that the actual IR signal is travelling on a [PWM](http://dlacko.org/blog/2015/11/04/why-pulse-width-modulation-pwm-confusing/) carrier pulsing at a carrier frequency of (usually) 38KHz; it is nicely explained in details at [adafruit](https://learn.adafruit.com/ir-sensor/ir-remote-signals). Fortunately, common IR receivers, like what I purchased, [TSOP38238](https://www.adafruit.com/products/157), can turn such a PWM signal (top one on the following image) into regular digital one (bottom signal):

![IR signal](/img/ir_signal.png){: style="display:block; margin:auto;"}

{% include note.html content="I take the opportunity to give here some practical advice about TSOP38238: you probably want to use some bypass capacitor close to the receiver. I had serious issues when a servo was attached to the same power rail than the receiver. I could solve this issue by using a huge (1000 µF, likely too big, but I did not have smaller) bypass capacitor. The TSOP38238 [datasheet](http://www.vishay.com/docs/82491/tsop382.pdf) also recommends to use a bigger than 0.1 µF bypass capacitor along with a resistor of 33 Ω -1 kΩ for protection against electrical overstress." %}

## Decoding an IR signal

The IR receiver just passes the raw data along, it still needs to be decoded. Decoding means that you take the length of the zeros and ones and try to find out the meaning of the signal by this timing information. Most of the time these signals implement some kind of well-known protocol (it is usually the case with traditional remote controllers) and can be easily decoded; sometimes we can assign a [unique hash number](http://www.righto.com/2010/01/using-arbitrary-remotes-with-arduino.html) to a given signal without any knowledge of the actual protocol. Sometimes, like in this case, the protocol must be reverse engineered.

The following picture illustrates the the [NEC protocol](https://techdocs.altium.com/display/FPGA/NEC+Infrared+Transmission+Protocol) to give you the basic idea how such a protocol looks like:

![NEC message frame](https://techdocs.altium.com/sites/default/files/wiki_attachments/212919/NECMessageFrame.png){: style="display:block; margin:auto;"}

## Decoding the Nano's IR signal: naive approach

I was concerned about whether automatic hash generation could be useful in this case. It is not a traditional remote controller after all, you are supposed to push multiple buttons in the same time, not to mention that you want the values of the joysticks as consecutive - and not random - numbers.

Still, I gave it a go using the Arduino [IRremote](https://github.com/z3t0/Arduino-IRremote) library; I thought it might be smart enough to recognize the protocol or to generate values from the signals where the individual values of the buttons and joysticks can be read as bit fields.

I ran the [IRrevDemo](https://github.com/z3t0/Arduino-IRremote/blob/master/examples/IRrecvDemo/IRrecvDemo.ino) example application, but the result was very disappointing. I got the following numbers when I should have got the same ones:

```
D3DB175B
1927009E
6DAD5130
B08A9DC0
D3DB175B
99C4D258
D4DB18EC
F7D2AE54
2319BCD0
32E5AF23
D3DB175B
80029B2E
E6F89EA5
```

## Decoding the Nano's IR signal: the hard way

I did not have any choice, but to have a look at the timings and reverse engineer the protocol (I was not completely clueless, though, as I found [this Silverlit protocol description](http://www.jrl.cs.uni-frankfurt.de/web/projects/helicontrol/silverlit-protocol/), which gave me a basic idea about what to look for). This time I ran the IRrecvDumpV2 example; it provided every kind of useful information, but most importantly the timings.

The followings are two typical readings representing the same value. The number in the square brackets shows the length of the signal, the numbers annotated with + and - are the lengths of the consecutive 1 and 0 signs (+ annotates 1s, - annotates 0s) of the signal in nanoseconds.

```
Timing[47]: 
  +1650, -450     +250, -400     +300, -400     +300, -400
  + 250, -450     +250, -400     +300, -400     +250, -450
  + 250, -400     +950, -450     +900, -450     +900, -450
  + 900, -500     +200, -450     +850, -500     +900, -500
  + 850, -450     +900, -450     +250, -450     +200, -550
  + 150, -500     +850, -500     +150, -500     +250

Timing[47]: 
  +1750, -500     +300, -450     +300, -450     +250, -500
  + 300, -450     +250, -400     +300, -400     +250, -450
  + 250, -450     +900, -500     +950, -400     +850, -450
  + 900, -400     +250, -400     +850, -400     +950, -500
  + 950, -450     +950, -450     +300, -450     +300, -450
  + 250, -450     +900, -400     +250, -500     +300
```

There is quite a bit of fluctuation in the numbers, but we can make some observations that helps with the decoding:

- The signals seemingly always contain 47 timings. Good for identifying the signal.

- It seems that the numbers annotated with - are the same (modulo fuzziness), thus do not carry information.

- The first timing is obviously different than the others (so much bigger). By the example protocol description, I guess it is a header bit, so it can be ignored (then again, good for identifying).

- The rest of the timings should represent 1s and 0s. There are bigger numbers, around 850-950, and smaller ones around 150-300. Let's say that everything below 500 represents 0, the others represent 1.

According to these, I modified one of the example programs a bit. The gist is in the decodeNano method:

```c
#include <IRremote.h>

int recvPin = 2;
IRrecv irrecv(recvPin);

void  setup ( )
{
  Serial.begin(9600);   
  irrecv.enableIRIn();  // Start the receiver
}

unsigned long decodeNano(decode_results *results)
{
  unsigned long value = 0;
  
  // Start at 2, skip header
  for (int i = 2;  i < results->rawlen;  i++) {

    // Skip even indexes
    if (i & 1) {
      int t = results->rawbuf[i] * USECPERTICK;
      value <<= 1;
      value += t > 500;
    }
  }

  return value;
}

void  loop ( )
{
  decode_results results; // Somewhere to store the results

  if (irrecv.decode(&results)) { // Grab an IR code
     Serial.println(decodeNano(&results));
  }

  irrecv.resume(); // Prepare for the next value
}
```

## The Silverlit IR protocol for Falcon Nano

Using this method I finally got stable values. The next step was to find out which bits are related to which buttons or joysticks. It is very simple, you basically just push buttons one by one and try to identify which bits are changed in the result. At the end, I came up with the following bit pattern:

`CCTTTTTHHHHHRRRRRLLLVVV`

- `C`: channel (2 Bits)
- `T`: throttle (5 Bits)
- `H`: horizontal direction (5 Bits)
- `V`: vertical direction (3 Bits)
- `T`: trim (5 Bits)
- `L`: light (3 Bits)

Finally, I developed some helper functions to read the bit fields and shift their values when necessary (e.g. set the origins for the joysticks):

```c

int getThrottle(unsigned long value){
  value >>= 16;
  value &= 0b11111;
  return value;
}

int getDirectionH(unsigned long value){
  value >>= 11;
  value &= 0b11111;
  return value-15;
}

int getDirectionV(unsigned long value){
  value &= 0b111;
  return 4-value;
}

int getLight(unsigned long value){
  value >>= 3;
  value &= 0b111;
  return value == 0b111;
}

int getTrim(unsigned long value){
  value >>= 6;
  value &= 0b11111;
  return value-16;
}

int getChannel(unsigned long value){
  value >>= 21;
  value &= 0b11;
  return value;
}
```

Tell me if my case helped you to reverse engineer other controllers!








