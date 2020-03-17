---
title: Sanity Razor
description: Unreleased VR title for the Oculus DK2
date: 2016-02-24
tags:
  - portfolio
  - games
  - Unity3D
  - XR
layout: layouts/post.njk
card: /img/SanityRazor_GreenLightLogo.png
---

This was the first project I worked on at Digital Precept. Development was on a Oculus DK2 and Unity5. The game is best described as "Steel Batallion vs Call of Cthulhu." 
It was first demoed at PAX:South 2016 with great praise from the players who tried it.
However, this was still the halcyon days of VR development and our basic training demo was pushing the system to its limits to hit 90FPS, and so the studio made the decision to stop working on it for now.

The project is not considered cancelled as the studio would like to revisit the idea in the future; the project's status is "on ice."
>"That is not dead which can eternal lie. And with strange aeons even death may die." - _Call of Cthulhu_

## The Greenlight Trailer
<iframe width="560" height="315" src="https://www.youtube.com/embed/UsRlwK7xg_s" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Novel features
_At time of writing, it's 2020. Many of these features are trivial or easy now that the XR industry has had 4 years to mature._

We had a monoscopic HUD that only rendered on your left eye. This allowed use to render targeting pips for ease of aiming and enemy awareness.
Dual camera setups in Unity3D were prohibitively expensive, and we didn't want to deal with "impossible geometry" by rendering the HUD as floating 3D elements.

The simple solution:  render it like a real world HUD. The Entirety of the HUD is rendered roughly 1.5 inches to the left of the camera's center point. Or roughly 1 inch in front of your (virtual) eyeball.
The SinglePass Stereo translation math would naturally occlude this from the right eye.
Furthermore, there is only one world space canvas with all elements including the cockpit displays attached to it. This keeps all the UI drawcalls to a single batch.

