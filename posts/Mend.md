---
title: Mend
description: 
date: 2020-02-10
tags:
  - portfolio
  - games
  - Unity3D
  - XR
  - Oculus
layout: layouts/post.njk
titlecard: img/Mend_400x400.jpg
---

![alt text]( {{ '/img/Mend_400x400.jpg' | url }} "Mend Title Card")

I am the main XR and Software Developer for Mend!   
Mend is a 2 player, asymetrical co-op game for VR.  It is designed by Justin Palmer and was originally an Oculus LaunchPad title.
I developed the core game systems built to Justin's specifications, including:
  - an XR Interaction Rig we use that abstracts object usage logic away from the core VR camera rig. 
     - "Grab & Pull" locomotion keyed to the hand controller update sequence for as smooth as possible movement
	 - Physics based object manipulation. (aka you can pick up things)
  - Extensive modifications to FMOD to allow dual speaker outputs so the VR player has their own localized audio that is distinct from the pancake player
  - An input remapper built on top of Oculus' OVRInput system to allow easy control customization
  - Spatial dependent collision platforms!

Check out the first gameplay trailer here! https://www.youtube.com/watch?v=CKeRf5Wvzjg 


I'm really proud of the spatial dependent collision platforms, and I want to explain more in depth.

## Inspiration
It's heavily inspired from Super Mario Galaxy's invisible platforms; these platforms are revealed within a radius of a torch. However, they platforms are always there regardless if you can see them or not.

## The Twist

> I really wish we could make the collisions operate the same way as the visuals. - _Justin Palmer_, _Mend's Game Designer_

 
> "Give me an hour. I think can make it work."  - _Jonathan_

The core aspect of this game is cooperative game play. The VR player can move boxes and reveal platforms for the non-VR player to use. However, you don't need the VR player if you know where the invisible platforms are.
So instead of making them invisible, we make them cease to exist outside of the orb's range.  I then took it one step further, and devised a method of controlling collisions for partials views as well.
Now the game really requires team work to complete!


# The result
(animated gif of it working)

Find out more at https://twitter.com/mendthegame 