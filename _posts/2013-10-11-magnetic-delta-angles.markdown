---
layout: post
title:  "Magnetic delta and print angles"
categories: printer delta math
description: "Calculating range of motion of a 3D printer magnetic joint"
---

I'm in the middle of designing my delta style 3D printer. Having reviewed a few designs, the magnetic joints looks to be the easiest way to go. I've noticed that many people angle the ball mount 45 degrees by either [bending the plate][berrybot-hotend] or [inserting an angle spacer][xnaron-rostock]. I wanted to avoid this if possible, so I calculated the minimum inclination possible for a given design.

<!--excerpt-->

The math was pretty simple. I use the [cosine law][cosine-law] to calculate the angle occupied by a given chord. There are two chords of importance; the diameter of the mount for the ball, and the diameter of the joint. In Xnaron's project above; we have a 3/8" ball, the mount is a 6mm screw, and the joint is 3/8". This gives the joint occupying 180&deg; and the mount 78&deg;, when we are at max range, these two angles are adjacent.

If we start with the mount, it is vertical giving -90&deg; inclination. We add half of the angle to get -51&deg; inclination, then half of the joint angle to get 39&deg;. This means the minimum inclination possible before the joint runs out of range is 39&deg;. If he were using a 1/2" ball it would extend to -13&deg;, well below horizontal.

I'll probably end up using 1/2" balls in my design, 1/4" dimple press for the mount, and then experiment with the joint design to give just enough hold.

[berrybot-hotend]: http://www.youtube.com/watch?v=WriyoSTJ51s
[xnaron-rostock]:  http://forum.seemecnc.com/viewtopic.php?f=54&t=1704&sid=512048173d7806425ee0b8a411e75957
[cosine-law]:      http://en.wikipedia.org/wiki/Law_of_cosines



