---
title: Unity3D AMF-Importer
description: Import "Adjutant Model Files" (.AMF) into Unity3D
date: 2019-02-04
tags:
  - portfolio
  - Unity3D
  - github
layout: layouts/post.njk
---

# WIP

## Down the rabbit hole...
I wanted to test the efficiency of rendering a FPS multiplayer map on some low end hardware. So I started to make a map when I realized, why waste time on a theoretical map, when I could use an existing map from an FPS? Then I recalled that I used to play a lot of Halo:CE in college.  Not "Combat Evolved" but "_Custom Edition_". 

Custom Edition had tools that allowed the community to create custom levels, maps, guns, etc. But I didn't want to use someone else' map; I wanted to use one of the official maps as a benchmark for my performance test. So I did some research and came across a fan made tool called "Adjutant" that lets you extract geometry and textures from the Halo map files. Even better it supported Halo 3 assets! So I open up "Isolation" multiplayer map from Halo 3. Extract the textures as PNGs and extract the model as... AMF?

So the Adjutant mod team had created their own model file format to extract as.  They then provided an import script plugin for 3DS...2005.  I don't own 3DS Max, much less the older precursor. So I started investigating if anyone had made a Blender plugin that could import AMF.  No one had. Most of the documentation and mod posts I was finding for Halo:CE were circa 2003 at best. However, by chance I ran across a forum post dated late 2018 of someone talking about making a Blender plugin. 

Huge shoutout to Shelley and the rest of the folks over on the Reclaimers discord. Shelley had a decompiled version of the 3DS importer plugin. With this, I was able to write my own C# importer to allow Unity to import AMF files natively as if they were 3D models! ...almost.

(photo of jacked up isolation)

## Geometry

Halo/3DS uses a right handed, z-up coordinate system, while Unity uses a left-handed Y-up system.  Converting between the two ended up being slightly trickier than it should have been. As the coordinates would be extracted from the raw AMF file, but then certain modifiers and transforms would be applied based on the geometry name. 

## Materials
The AMF file format has data in it for setting up textures and materials into the shaders. Every material has an array of textures, colors, and normal maps. Unfortunately the 3DS importer script does not support anything other than a basic diffuse set up. However, looking at the textures referenced, the AMF file would preserve indexes and leave in null fields. So a pattern can be observed with what texture indexes are albedo, normals, color change, and lighting. 