---
title: BTLE 4 Unity
description: Bluetooth Low Energy 4 Unity!
date: 2019-02-28
tags:
  - portfolio
  - Unity3D
layout: layouts/post.njk
---


![alt text]( {{ '/img/BTLE4Unity.jpg' | url }} "BTLE4Unity Title Card")


I designed and developed a Bluetooth Low Energy integration for Unity! 
Nicknamed BTLE4Unity aka "Beetle"

The asset is broken into multiple parts for cross platform utilization.

 - Core API: this is the high level interface that all platforms must implement
 - Win API: this wraps the low level Windows SDK Bluetooth calls with C# headers and utilities.
 - WebGL API: this wraps the [Javascript Bluetooth implementation](https://developer.mozilla.org/en-US/docs/Web/API/Web_Bluetooth_API) spec via JSLIB bindings.
