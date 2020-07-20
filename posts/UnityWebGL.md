---
title: Unleash the power of WebGL and Unity
description: How to do callbacks in Unity WebGL and other tricks
date: 2020-07-20
tags:
  - tutorial
  - blog
  - Unity3D
  - github
layout: layouts/post.njk
---

If you've ever used Javascript, you know how much it loves callbacks. Unfortunately at the time of writing, there isn't any good documentation on how to work with javascript callbacks inside of Unity's WebGL builds with C#.  

Note: Web browsers only run Javascript or WebASM.  However, in Unity we code in C# which then gets compiled and converted to WebASM, and in order to avoid confusion, I'll often refer to things as C# instead of WebASM.

>If you're coming from Unity and C# with little Javascript experience, you might have heard callbacks referred to as "delegates" or "Actions."  Still not ringing a bell?  If you've used a UI button in Unity, the "OnClick" field is a callback!

# How to use C# Callbacks from Javascript

## On the C# side
This is pretty straight forward for C# as the runtime will automatically marshal data for us.  "Marshalling" is the term for converting the data to another format and is necessary as we pass data from C# (technically WebASM) to Javascript and back.  Effectively, these two languages are isolated from each other in memory, but Interops allows these two transfer data and execution to and from each other.

Let's look at a C# sample:
(JSAPI.cs)
```
using System;
using System.Runtime.InteropServices;
using AOT;
public class JSAPI{
    
    [DllImport("__Internal")]
    private static extern void JSExample(Action callback);

    [MonoPInvokeCallback(typeof(Action))]
    public static void DefaultCallback(){
        //This fires from javascript
    }

    public void YourMethod(){
        JSExample(DefaultCallback);
    }
}
```

Let's break this down:
We declare our Javascript function called "JSExample"  as taking an "Action" reference. This is our function pointer that will get passed to Javascript.

We then declare a **static** function and mark it with ```[MonoPInvokeCallback(typeof(Action))]``` which as you can probably guess, will tell the compiler to generate a function pointer that matches type "Action".  _(Why do we use a static callback function? We'll get to that later.)_

Lastly, we can then call the Javascript function and pass in our static callback.


## On the Javascript side
This is where things start to get tricky. Fortunately, it's a lot of boilerplate code.

First off, to call a C# function from a Javascript function you want to use:

```
Runtime.dynCall(signature,functionPtr,arguments);
```

__signature__: (string) return type followed by each argument type.

v - void
i - int
f - float
d - double

example:  
*public static void SomeCallback(int id, string data)*
has the signature string:
*"vii"*
(Why do string parameters use int? We'll get to that...)

__functionPtr__: the C# function we want to call

__arguments__: (array) the parameters to pass to the C# function

In practice, it looks like this:
(example.jslib)
```
mergeInto(LibraryManager.library, {
    JSExample: function(functionPtr){
        Runtime.dynCall("v",functionPtr,[]);
    }
});
```

> Sanity Check:  Your Javascript function name must match what you called your static extern function in C#, in this case "JSExample"

Seems simple so far? 

# Strings and things

So what if we want to pass some strings back and forth? For example, we want to use the [Javascript "prompt()" command](https://www.w3schools.com/jsref/met_win_prompt.asp) to get some text input from the user.  (Fun fact:  due to iOS security restrictions, this is currently the only way to get text input from iPads in Unity WebGL.)

Prompt() takes a string as the message text, and optionally a second string as the default input text, it then returns the user's input (or null if they click cancel).

Pretty simple in C#
```
[DllImport("__Internal")]
private static extern string PromptText(string message, string text);
```

And in JS...
```
PromptText: function(titlePtr,textPtr){
        var title=Pointer_stringify(titlePtr);
        var text = Pointer_stringify(textPtr);
        var val=prompt(title,text);
        var buffer=0;
        if(val){
            var size=lengthBytesUTF8(val)+1;
            buffer=_malloc(size);
            stringToUTF8(val,buffer,size);
        }
        return buffer;
    }
```
Yikes, that's a lot of code. Let's dive in!

You can only pass primitive data types back and forth between C# and JS, and strings are not a primitive. Instead, the string is written to memory, and a pointer to that section of memory is passed to Javascript.  So we call "Pointer_stringify"  to fetch the string.  

When trying to pass the string from Javascript to C#, we do the same thing. We don't pass the string itself, but instead we copy the string to memory and then pass a pointer to that string back to C#.   "_malloc"  reserves a chunk of memory (aka "buffer")  for us to copy our string into. We always use length+1 when reserving memory, as the last byte of a string is always 0x00 to signify the end of the data. "stringToUTF8"  does the actually copying for us.

Keep in mind, that "prompt()" can return null. We handle this by checking "if(val)"  before copying the string. If they did click cancel, we just return 0, which C# will auto marshal as a null string.

## Scope and Closures
A pointer to a string passed in as a parameter will get included as the local scope of a closure.  However, the string this pointer refers to may not!  You should marshal any data you need before creating a closure.


## What about other objects?
While it is possible, it is a lot of extra work. In C# you have to use a "struct" with Explicit layout instead of a "class."   In Javascript, you'll have to manually write data to the buffer to ensure it matches the C# struct *exactly*   as it would be laid out in memory.  Keep in mind, that javascript data types don't always match up to their C# counterparts. 

**OR**

You could just use ```JSON.stringify()``` to easily convert your object to a string and then pass it across just like we did in the "prompt()" example above. Nice and easy.


# Using non-static callbacks
So remember earlier when we passed a static C# callback to Javascript?  That's a bit inconvenient. What if we just used a non-static callback?
You'll get the follow error in your Javscript console:
```
NotSupportedException: IL2CPP does not support marshaling delegates that point to instance methods to native code.
```

So you must use static delegates.  However, we know how to pass data back and forth, and now you can get around this limitation by building your own lookup table

blah blah blah example code
