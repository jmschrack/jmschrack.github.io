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

[Here's Unity's current documentation on interacting with Javascript](https://docs.unity3d.com/Manual/webgl-interactingwithbrowserscripting.html)
This covers basic function calling and marshalling strings, but what about callbacks? If you've ever used Javascript, you know how much it loves callbacks. Unfortunately at the time of writing, there isn't any good documentation on how to work with javascript callbacks inside of Unity's WebGL builds with C#. So let's fix that.

>If you're coming from Unity and C# with little Javascript experience, you might have heard callbacks referred to as "delegates" or "Actions."  Still not ringing a bell?  If you've used a UI button in Unity, the "OnClick" field is a callback!

# Topics
 * [Pass a C# callback to Javascript](#pass-a-c%23-callback-to-javascript)
 * [Pass a Javascript callback to C#](#pass-a-javascript-callback-to-c%23)
 * [Call external Javascript on the web page.](#call-external-javascript-on-the-web-page)
 * [Creating global hooks for external Javascript to use](#creating-global-hooks-for-external-javascript-to-use)
 * [Common Pitfalls](#wrapping-up)
 

Note: Web browsers only run Javascript or WebASM.  However, in Unity we code in C# which then gets compiled and converted to WebASM, and in order to avoid confusion, I'll often refer to things as C# instead of WebASM.

# Pass a C# callback to Javascript

## On the C# side
This is pretty straight forward for C# as the runtime will automatically marshal data for us.  "Marshalling" is the term for converting the data to another format and is necessary as we pass data from C# (technically WebASM) to Javascript and back.  Effectively, these two languages are isolated from each other in memory, but Interops allows these two transfer data and execution to and from each other.

Let's look at a C# sample:
(JSAPI.cs)
``` csharp
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

``` js
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
(Why do string parameters use int? Because we copy the string to C# memory and then pass the pointer in. C# will auto-marshal this as a string for us!)

__functionPtr__: the C# function we want to call

__arguments__: (array) the parameters to pass to the C# function

In practice, it looks like this:
(example.jslib)
``` js
var myLib={
    $dependencies:{},
    JSExample: function(functionPtr){
        Runtime.dynCall("v",functionPtr,[]);
    }
};
autoAddDeps(myLib,'$dependencies');
mergInto(LibraryManager.library,myLib);

```

> Sanity Check:  Your Javascript function name must match what you called your static extern function in C#, in this case "JSExample"

Seems simple so far? 

# Pass a Javascript callback to C#
This could be a bit hacky. Unity uses "Emscripten" as part of it's toolchain to convert your C# to WASM. As of Jan2019, a helper function was added to Emscripten that wraps a Javascript function as invokable method pointers. However, depending on your version of Unity, you may or may not have that update. We'll cover that first, but if you are stuck on an older version, we'll cover how to DIY your own workaround.

## On the Javascript side (Emscripten v1.38.26 or higher)
This will look similar to what we did above.
``` js
Runtime.addFunction(jsFunction,signature):IntPtr
```
__jsFunction__: (function) The Javascript function you want to wrap as a C# method.
__signature__: (string) return type followed by each argument type. This follows the same rules as dynCall mentioned above.
__return:IntPtr__: (int) the invokable pointer we can pass to C#. 

In practice, looks like this:
(example.jslib)
``` js
//adding this after JSExample above

    JSCallbackExample: function(){
        var callback=function(){
            alert("Received a callback!");
        };
        var ptr=Runtime.addFunction(callback,'v');
        return ptr;
    }
```

## On the C# side

It's also pretty straightforward on the C# side if you let it auto-marshal the pointer as a delegate.
``` cs
[DllImport("__Internal")]
private static extern Action JSCallbackExample();

void ExampleUsage(){
    Action a = JSCallbackExample();
    a.Invoke();
}
```

You can also manually marshal it:
 
``` cs
[DllImport("__Internal")]
private static extern IntPtr JSCallbackExample();

public static Action GetJSCallback(){
    IntPtr ptr= JSCallbackExample();
    Action a= Marshal.GetDelegateForFunctionPointer<Action>(ptr);
    return a;
}

```

## On The Unity side
If you immediately tried to Build and run using your brand new JS Callbacks, you'll get a javascript error. Something along the lines of "Could not find function" or "ran out of table space"  in regards to the "addFunction" command. Emscripten keeps an internal table that maps C# delegates to Javascript functions, and by default this is created at build time and is unchangable. So we have to supply a custom argument to the Emscripten tool to enable runtime usage of 'addFunction.'  

Create an Editor script and add these lines to it.
``` cs
[MenuItem("Tools/Set WebGL Args")]
static void SetWebGLArgs(){
    PlayerSettings.WebGL.emscriptenArgs="-s ALLOW_TABLE_GROWTH";
}
[MenuItem("Tools/Unset WebGL Args")]
static void SetWebGLArgs(){
    PlayerSettings.WebGL.emscriptenArgs="";
}

```
This will add a "Tools" submenu to Unity and allow you to change the WebGL arguments sent to Emscripten.
Just run "Tools/Set WebGL Args" once and that's it. Now you should be able to build and use the AddFunction command

> If you fail to build with a "FILE NOT FOUND 'ALLOW_TABLE_GROWTH'" error, that means you are on an older version of Emscripten that does not have this ability. Don't panic.

# Pass a Javascript callback to C# (Older versions of Emscripten)
As mentioned before, Emscripten keeps an internal table of what C# delegates map to which Javascript functions. Unfortunately if you are on an older version of Emscripten, you can't add to this table dynamically. There is an easy, if less elegant, solution:  write our own lookup table in Javascript.

We simply create an array in Javascript, store our would-be Javascript functions, pass their index to C#, and then call a Javascript function with that index which fires the JS callback for us.

## On the C# side
This is going to be almost identical to earlier, with the exception of an additional extern function we use for Invoking.
``` cs
[DllImport("__Internal")]
private static extern int JSCallbackExample();
[DllImport("__Internal")]
private static extern void InvokeCallback(int cb);

void ExampleUsage(){
    int cb=JSCallbackExample();
    //do something.....
    InvokeCallback(cb);
}
```

## On the Javascript side
We're going to make use of Emscripten's "__postset" command, which will emit a string directly into the resulting Javascript file.  In this case, we emit a scope level array to hold our JS callback references. In this simple example, we just pass the index of the callback to C#, and then use that index to invoke the callback later. 

``` js
JSCallbackExample__postset:'var cbIDs=[];',
JSCallbackExample: function(){
        var callback=function(){
            alert("Received a callback!");
        };
        var id=cbIDs.push(callback)-1;
        return id;
    },
InvokeCallback: function(cb){
    var callback=cbIDs[cb];
    callback();
}
```

> Heads Up:  This example does not remove callbacks after invoking them. If you need to use Javascript callbacks sparingly or if you reuse callbacks, this will probably be okay. However, if you need to invoke non-reusable JS callbacks frequently, this array will bloat up and waste memory. In the latter case, consider using a JS Object and generating unique keys instead.

# Call external Javascript on the web page
This is probably the easiest thing here. Your JSLibs are still JavaScript running in a web browser which means they follow normal conventions. As long as the external JS library you want to use is declared in a ```<script>``` tag before the script tag that loads your UnityInstance, it will be accessible.

(index.html)
``` html
<script>
var TestExternalJS = function(){
    alert("I'm your external function!");
}
</script>
```

(example.jslib)
``` js
var myLib={
    $dependencies:{},
    CallExternal: function(){
        TestExternalJS();
    }
};
autoAddDeps(myLib,'$dependencies');
mergInto(LibraryManager.library,myLib);
```

# Creating global hooks for external Javascript to use
This last part is a combination of everything we've learned so far.  There are many ways to handle this; this is just my preferred style

We create a new script tag to hold a global object. We give the tag an id for easy access later and a "isLoaded" variable for checking.
``` html
<script id="UnityHooks">
var UnityHooks={
    isLoaded:false
}
</script>
```

In our JSLib, we add functions to our global object.  It's a good idea to marshal and cache any data you need.   We finish by setting isLoaded to true, and firing a "loaded" event on the script tag itself.  This will allow any external APIs that depend on our Unity hooks to use the standard EventListener system
``` js
SetUpHooks:function(callback){
        UnityHooks.cb=callback;
        UnityHooks.TestCallback=function(text){
            var bufferSize = lengthBytesUTF8(text) + 1;
            var buffer = _malloc(bufferSize);
            stringToUTF8(text, buffer, bufferSize);
            Runtime.dynCall('vi',UnityHooks.cb,[buffer]);
        }
        UnityHooks.isLoaded=true;
        document.getElementById('UnityHooks').dispatchEvent(new Event('loaded'));
    }
```

# Wrapping Up
I believe this covers the missing edge cases in the official Unity documentation. Here's a few common pitfalls I've run across that are worth mentioning.

## Scope and Closures
A pointer to a string passed in as a parameter will get included as the local scope of a closure.  However, the string this pointer refers to may not!  You should marshal any data you need before creating a closure in Javascript.

## What about passing objects?
While it is possible, it is a lot of extra work on the Javascript side as there is no automatic Marshalling in Javascript. In C#, you have to use a "struct" with Explicit layout instead of a "class" when you want to pass data to and from Javascript. In Javascript, you'll have to read/write bytes to a data buffer manually to recreate the data object.
**OR**
You could just use JSON to easily convert your object(s) to a string and then pass it across. Nice and easy as both Javascript and UnityC# have built in JSON parsers, but this comes at the cost of some extra memory.

## Using non-static callbacks
So remember earlier when we passed a static C# callback to Javascript?  That's a bit inconvenient. What if we just used a non-static callback?
You'll get the this error in your Javscript console:
``` text
NotSupportedException: IL2CPP does not support marshaling delegates that point to instance methods to native code.
```
So you must use static delegates.  However, you can create a look up table to cache these local delegates and just pass an ID value and a static delegate to the Javascript.  This is the exact same logic we used in [Pass a Javascript callback to C# (Older versions of Emscripten)](#pass-a-javascript-callback-to-c%23-(older-versions-of-emscripten)) except now you handle it on the C# side instead of the Javascript side.

I wrote a simple [ActionLookUpTable gist on Github](https://gist.github.com/jmschrack/4d1451a0914f210cbe481bcf176891ea) that will work for general Interops. It was built with multi-threading in mind, but it won't compile on WebGL unless you remove the System.Threading imports. However, multi-threading isn't an issue on WebGL so it's an easy remove.


Good luck!