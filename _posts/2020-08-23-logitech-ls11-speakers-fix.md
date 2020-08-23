---
layout: "post"
title: "Logitech LS11 Speakers Fix"
date: "2020-08-23 16:02"
---
# Logitech LS11 Speakers Fix

## Problem
I purchased 2 sets of Logitech LS11 Speakers many years ago.  After about 1 year of use,
both sets started to exhibit problems.  The sound started to go out while in-use and the
LED would dim.  They would fail to produce sound when the knob was turned higher, etc.

I contacted Logitech support with the issue, and they sent me a new pair.  After less than
a year, the replacement speakers also developed the same problem.

## Search for solution
I searched online and found it was a common problem.

One person on Logitech's site wrote that they fixed it by adding capacitors to pins 2 and
15 of the amplifier IC.  
http://reviews.logitech.com/7061/4252/logitech-ls11-speakers-reviews/reviews.htm (page now no longer exists)

Amazon reviewer Darin wrote that the switch in the potentiometer was broken and he fixed it
by shorting the 2 switch pins to make it permanently on.  
http://www.amazon.com/review/R10MLR2NPP1VS9/ref=cm_srch_res_rtr_alt_4

## The Fix
The broken speakers sat in the garage for many years until I finally took them out for repair.

It turns out my potentiometer's switch was also broken, same as written in Darin's Amazon review.

The potentiometer is a 2 gang audio taper potentiometer with an on/off switch.  Total 8 pins.

The resistance between the 2 switch pins were always 2 10K ohms, no matter on or off.

Darin said to solder a jumper across the 2 pins, which would work, but I opted to wire an
external switch to it.  After this, the speakers work fine now.  One less item in the landfill.


Here is a photo of the PCB with the potentiometer's switch pins labeled.  Connect these 2 pins
with a wire or an external switch (around maximum 10v DC 400mA will pass through it):

![logitech-ls11-bottom-pcb-fix](/images/2020/08/logitech-ls11-bottom-pcb-fix.png)


