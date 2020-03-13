---
title: Unity3D AMF-Importer
description: Import "Adjutant Model Files" (.AMF) into Unity3D
date: 2019-02-04
tags:
  - blog
  - portfolio
  - Unity3D
  - github
layout: layouts/post.njk
---

![alt text]( {{ '/img/AMFImporter.png' | url }} "Adjutant to Unity")


## The impetus
I wanted to test the efficiency of rendering a FPS multiplayer map on some low end hardware. So I started to make a map when I realized, why waste time on a theoretical map, when I could use an existing map from an FPS? Then I recalled that I used to play a lot of Halo:CE in college.  Not "Combat Evolved" but "_Custom Edition_". 

Custom Edition had tools that allowed the community to create custom levels, maps, guns, etc. But I didn't want to use someone else' map; I wanted to use one of the official maps as a benchmark for my performance test. So I did some research and came across a fan made tool called "Adjutant" that lets you extract geometry and textures from the Halo map files. Even better it supported Halo 3 assets! So I open up "Isolation" multiplayer map from Halo 3. Extract the textures as PNGs and extract the model as... AMF?

## Down the rabbit hole...
So the Adjutant mod team had created their own model file format to extract as.  They then provided an import script plugin for 3DS...2005.  I don't own 3DS Max, much less the older precursor. So I started investigating if anyone had made a Blender plugin that could import AMF.  No one had. Most of the documentation and mod posts I was finding for Halo:CE were circa 2003 at best. However, by chance I ran across a forum post dated late 2018 of someone talking about making a Blender plugin. 

Huge shoutout to Shelley and the rest of the folks over on the Reclaimers discord. Shelley had a decompiled version of the 3DS importer plugin. With this, I was able to write my own C# importer to allow Unity to import AMF files natively as if they were 3D models! ...almost.

![alt text]( {{ '/img/IsolationFubar.png' | url }} "Isolation messed up")

## Geometry

Halo/3DS uses a right handed, z-up coordinate system, while Unity uses a left-handed Y-up system.  Converting between the two ended up being slightly trickier than it should have been. As the coordinates would be extracted from the raw AMF file, but then certain modifiers and transforms would be applied based on the geometry name. 
![alt text]( {{ '/img/IsolationFubar2.png' | url }} "Isolation partially correct")

Thankfully, geometry falls into two catergories of terrain or instances, so in the end it all worked out.
![alt text]( {{ '/img/IsolationFixedGeo.png' | url }} "Isolation slightly less messed up")
...But what's up with the crushed blacks?

## Materials
The AMF file format has data in it for setting up textures and materials into the shaders. Every material has an array of textures, colors, and normal maps. Unfortunately the 3DS importer script does not support anything other than a basic diffuse set up. However, looking at the textures referenced, the AMF file would preserve indexes and leave in null fields. So a pattern can be observed with what texture indexes are albedo, normals, color change, and lighting. 
That get's us about 75% of the way there, but we still have these weird, crushed black looking materials.
![alt text]( {{ '/img/crushedBlackRock.png' | url }} "Rocks with Crushed Black")
That ends up being an easy fix.  The textures have been pre-converted to linear colorspace, but by default Unity (and everything else) imports the texture as Gamma and then converts to linear. So changing the texture's sRGB setting to false fixed the colors and gets shadows to play nice as well! 
![alt text]( {{ '/img/RocksInShade.png' | url }} "Fixed rocks")


## Rigged Importing
It even works with rigged meshes, but that was whole other process of figuring out the right transforms to apply at certain steps. I saved a few screenshots of the more interesting steps in the process.

The correct skeleton is on the left. That wadded up ball of lines? That's the imported skeleton.
![alt text]( {{ '/img/MCRig1.png' | url }} "Wadded up ball of skeleton")

Skeleton is looking right, except it took a long walk off a short cliff.
![alt text]( {{ '/img/MCRig2.png' | url }} "Skeleton sleeping on the job")

Skeleton is right, but the 3D mesh is not?
![alt text]( {{ '/img/MCRig3.png' | url }} "Almost there")

Skeleton and mesh imports properly, except a basic idle animation results in MasterChief about to drop the hottest rap album of 2020.
![alt text]( {{ '/img/MCRigAnim1.gif' | url }} "MC is bit too gangsta")

Easy enough, adjusting MC to be in a T-Pose corrects the avatar's limb rotations.
![alt text]( {{ '/img/MCRigAnimFinal.gif' | url }} "perfect")

# The End Result
Is a pretty servicable importer for AMF files into Unity! It's not perfect, but neither is AMF. 

The full source code is on GitHub:
https://github.com/jmschrack/Unity3D-AMFImporter 

# Wait what about that Render test?
4 player splitscreen hit max frame rate 30FPS on an android tablet. (Screen is locked at VSync. Judging by the CPU stats, it could feasible hit 50+FPS without VSync)
<iframe width="560" height="315" src="https://www.youtube.com/embed/7W2QDMUuvdY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>