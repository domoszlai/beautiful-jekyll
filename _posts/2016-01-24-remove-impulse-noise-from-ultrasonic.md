---
layout: post
title: Remove impulse noise from ultrasonic sonar data 
tags: [arduino, ultrasonic sonar, robot car]
description: Do you have spikes in the data read from your ultrasonic sensor, like an HCSR04? Noise can be efficiently removed by simple median filter. Arduino code is provided.
excerpt: Mr. Stitson has an ultrasonic sonar mounted in the front. namely a HC-SR04; it is inexpensive, and ubiquitous, but unfortunately not quite reliable. In this post I try to fix it by means of software. 
image: /img/ultrasonic-sensor-HCSR04.jpg
show-avatar: true
gh-repo: domoszlai/stitson
gh-badge: [star, fork, follow] 
---

## Introduction

[Mr. Stitson](http://dlacko.org/blog/2016/01/01/mr-stitson-robot-car-with-lego-platform/) has an ultrasonic sonar mounted in the front. namely a HC-SR04; it is inexpensive, and ubiquitous, but unfortunately not quite reliable. I have a single reason for having a sonar installed, I want to keep away my boys playing the "bumping" game, what is, according to some scientists, has an unhealthy effect on robots.

![Mr. Stitson with sonar](/img/mr_stitson_with_sonar.jpg){: style="display:block; margin:auto"} 

To avoid hitting the wall, in the main loop there is some logic that, based on the current speed and the latest sonar readings, slows down the vehicle.

## Noisy sensor data

Not rocket science, but it works reasonably well. until the sonar returns valid data. Unfortunately, this is not the case. Standing still, these are common sonar readings:

```
30
30
30
31
5
29
31
31
5
33
4
30
31
31
31
31
31
30
30
30
30
```

I do not care about the slight fluctuation around 30 cm, but the 4 and 5 cm readings (it is called impulse noise I believe) are showstoppers, they immediately let the robot stop. Most people would cry [Kalman filter](https://en.wikipedia.org/wiki/Kalman_filter) here, but that would be like using a sledgehammer to crack a nut; a simple [median filter](https://en.wikipedia.org/wiki/Median_filter) will do the job perfectly.

## Implementing the median filter

Such erroneous readings appear quite often, but I did not observe them more than twice in a row. A median filter takes the median value from the last N readings (a.k.a the window size), N = 5 is enough to filter out two extremely bad readings (like we have here) in a row.

All we have to do is to sort the last five readings and pick up the third one. First, store the last readings in a cyclic buffer:

```c
// The cyclic buffer
int lastReadings[5];
// MAX_DISTANCE will be the the result until the filter warms up
for(int i=0; i<5; i++) lastReadings[i] = MAX_DISTANCE;
// Number of readings since the beginning
int nrReadings = 0;

// Add an element to the cyclic buffer
lastReadings[nrReadings++ % 5] = sonar->read();
```

Next, we need a function to calculate the median value based on this buffer (do not sort the buffer itself, then you loose cyclicity). That can be done very nicely and efficiently as there exist [optimal sorting network](https://%20http//cs.engr.uky.edu/~lewis/essays/algorithms/sortnets/sort-net.html) for N = 5 which uses only nine comparisons (and one can be trivially ignored as we need the third element only). There is also an algorithm for computing the median value of five elements with six comparisons, but that is anything, but nice and I want to avoid [premature optimization](http://c2.com/cgi/wiki?PrematureOptimization) anyway.

```c
#define swap(a,b) a ^= b; b ^= a; a ^= b;
#define sort(a,b) if(a>b){ swap(a,b); }

int median(int a, int b, int c, int d, int e)
{
    sort(a,b);
    sort(d,e);  
    sort(a,c);
    sort(b,c);
    sort(a,d);  
    sort(c,d);
    sort(b,e);
    sort(b,c);
    // this last one is obviously unnecessary for the median
    //sort(d,e);
  
    return c;
}
```

## Remarks

The actual code used in the robot can be found here: [https://github.com/domoszlai/robotcar/blob/master/sonarnp.cpp](https://github.com/domoszlai/robotcar/blob/master/sonarnp.cpp)

Some remarks about the code. It is based on the [NewPing](http://playground.arduino.cc/Code/NewPing) library. The class continuously reads the sonar (every 33 milliseconds only to avoid interference with the previous measurement) in an asynchronous manner. The measure() method always returns immediately with the latest (filtered) measurement. Although the results come asynchronously using a timer based mechanism, the measurements must be initialized from the main "thread" in the loop() method. This loop() method is part of the [green thread library](https://github.com/jlamothe/mthread) I use (notice the Thread base class). Read the comments in the code.

Finally, some remarks to median filter. 

Median filters introduce a delay proportional to N. For N = 5, the filter is on average two values behind the actual measurements, that is ~60 milliseconds. I have to be only faster than my kids, so it is acceptable for me.  If you think you have real-time requirements, then buy a proper sonar and use Kalman filter. If you want to smooth the sensor data fluctuations, use a Kalman filter. Median filters are very efficient, much faster than Kalman filters; if you do not have any of the previous requirements, stick with the median one. 

This [article](https://www.edn.com/design/systems-design/4417492/3/Median-Filters---An-Efficient-Way-to-Remove-Impulse-Noise) has some nice figures related to the topic.

## Conclusion and future work

I'm quite satisfied with the current implementation, it solves my main problem in a very efficient way with minimal effort. It may make sense, though, to smooth the sensor data as well. Furthermore,  I definitely want to implement prediction of future data. These can be done by Kalman filters. However, although, Kalman filter is an incredible tool for sensor fusion, when one can provide a proper physical model, I feel it too much otherwise. I would rather try something new, like [alpha-trimmed mean filters](https://www.eecis.udel.edu/~barner/courses/eleg675/papers/Restoration/Adaptive%20Alpha-Trimmed%20Mean%20Filters.pdf). Or just a simple linear prediction on top of the median filtered data.

As for the sensor itself, I think ultrasonic sensors are not the best tool to avoid obstacles, they just do not provide enough information for a proper plan. I will rather try using two cameras for creating [stereoscopic 3D images](http://docs.opencv.org/master/dd/d53/tutorial_py_depthmap.html#gsc.tab=0). Of course, than I need a proper processor, but this is [my plan](http://dlacko.blogspot.com/2015/12/how-to-install-ghc-on-intel-edison-poky.html) anyway. [Lidars](http://oceanservice.noaa.gov/facts/lidar.html) seem also very great, but they are a way too expensive just for having fun.

Do you have similar experience with HC-SR04?






