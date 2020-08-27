---
layout: "post"
title: "LG IPS234V LCD Monitor Fix"
date: "2020-08-26 23:15"
---
# LG IPS234V LCD Monitor Fix

## Problem
I've used an LG IPS234V LCD monitor since 6/2013.  A few weeks ago, I started using
it at 100% brightness, which I should have done a long time ago.

About a week after that, the monitor would fail to turn on and stay on.  I would
turn on my PC and monitor.  The monitor would turn on for a second, then turn off,
then back on.  Flickering on and off every second.  Eventually it would stay on.

I found that if I set the brightness back down before shutting everything down,
the monitor would be able to turn on the next day.

Sometimes, I would forget and would have spend time to get the monitor to stay on.

## Repair attempt
I opened the back of the monitor in search of bulging capacitors or burned components.
I didn't find any.

## Search online
I searched online and others had the same blinking on/off, but didn't have a solution.
One had fixed theirs by replacing a capacitor and voltage regular in the back of the monitor.
One or two others had hissing noises, fixed by replacing the power adapter.

## The power adapter
The power adapter for this monitor is an external AC-DC adapter and connects to a DC jack
on the back of the monitor.

The monitor's back label specifies 19V 1.3A, but it comes with a 19V 1.1-1.2A (I don't remember
how much exactly) power adapter.

Maybe the increased brightness and age caused the power adapter to fail.

Opening the power adapter, I found a capacitor that was bulging.  The capacitor was one made by
Su'scon, with markings SG 105 degree 1305(M), which is a 680 uF 25V capacitor.
I was going to replace it, but found that it's a low impedance or low ESR capacitor.  I didn't
have any and didn't want to replace it with a normal capacitor, so I decided to cut off the
power plug and put on another adapter.

## The fix
I found a 19V 2A laptop power adapter, cut and spliced the old plug onto it.  The new power adapter
worked.  I have been able to keep the monitor at 100% brightness, and it turns on without
issue.  This has been working for about a week and half, so I'm confident it's fixed.

Now, I can keep using and don't need need to rush to buy a new monitor.


