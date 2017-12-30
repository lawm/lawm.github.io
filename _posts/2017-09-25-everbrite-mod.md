EverBrite is a solar powered motion sensing LED light:

![everbrite-orig](https://user-images.githubusercontent.com/3174101/30843331-2927a3e8-a23d-11e7-864f-c42c91d53160.png)

Amazon and eBay also have lots of similarly looking lights by different brands.

### Original Behavior
There are various versions, but at least on the EverBrite the behavior is:
* When the sun is out, charging the solar panel, the light is off
* At night, if motion is detected, the light turns on to a bright level
* After 30 seconds, the light turns down to a dim level, staying on all night.

### Problem
I don't want the light to stay on at the dim brightness level all night.  I want the light to be off at night, and only turn on when motion is detected.

### Solution
1. Open the back cover.  There are 4 small Phillips head screws holding it on.
2. If your light does not match this picture, do not proceed.
3. Remove the 270 ohm resistor, labeled 271 on the right side of the PCB

![everbrite-pcb](https://user-images.githubusercontent.com/3174101/30843333-2ce261b2-a23d-11e7-886f-e8cd4fd18258.png)

4. Now, the light will only turn on when motion is detected and then turn off, instead of going dim like before.
