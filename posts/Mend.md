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
card: img/Mend_400x400.jpg
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

Check out the first gameplay trailer here!  

<iframe width="560" height="315" src="https://www.youtube.com/embed/CKeRf5Wvzjg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>





# Feature Highlight: Spatial Dependent Collision
I'm really proud of this, and I want to explain more in depth. 
___Dictionary of Terms___
 - __VR Player__ : the person wearing the VR headset. This is the "Owl sister" seen in screenshots and gameplay.
 - __Flat Player__ : the person using a controller and computer monitor and _no_ VR headset. This is the "Fox sister" seen in screenshots and gameplayer.

## Inspiration
Having invisible platforms is not a novel idea in video games. The logical progression is a tool or item that reveals the platforms.
For an example from Super Mario Odyssey:
![alt text]( {{ '/img/smg_invisplat.gif' | url }} "SMG Invis Platform example")

The obvious use case for a 2 player game, is that Player1 controls the tool to reveal the platforms that Player2 has to use. However, it didn't mesh at first with our use case.


## The Twist

> I really wish we could make the collisions operate the same way as the visuals. - _Justin Palmer_, _Mend's Game Designer_

The core aspect that Justin wanted to have is that the players __need__ each other, and cannot complete the level without each other.
We already accomplished this with platforms as The VR player is the only player that can move the platforms for the non-VR player to use. 
However, once the VR Player revealed the invisible platforms to the Flat Player, the Flat Player could just remember where to run and jump, removing agency from the VR Player as they just had nothing else to do.

We already had a method of partially rendering platforms that were in range. I then took it one step further, and devised a method of controlling collisions for partials views as well. The Flat Player (and any other physics objects we want!) will now fall through any invisible sections in the geometry.

# The result
(animated gif showing partial collision dynamically with a bunch of boxes)

Now the game really requires team work to complete!
(animated gif showing Owl and Fox working in tandem to platform)

# More about Mend
<a class="twitter-timeline" href="https://twitter.com/MendTheGame?ref_src=twsrc%5Etfw">Tweets by MendTheGame</a> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
