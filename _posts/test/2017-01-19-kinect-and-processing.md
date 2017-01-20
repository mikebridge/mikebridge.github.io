---
layout: article
title: Kinect and Processing
tags: [kinect, processing, windows, graphics]
categories: articles
comments: true
excerpt: Set up the graphical programming environment Processing to import motion data from a Microsoft Kinect.
image: 
  teaser: kinect/first_try_thumb_400x250.png
---

I fell into a discussion with another soccer-dad last weekend as we watched our own children 
play indoor soccer about teaching kids to code.  I talked about my experiences 
with [Raspberry Pi](https://www.raspberrypi.org/), [Scratch](https://scratch.mit.edu/), 
[Sonic Pi](http://sonic-pi.net/) at the coding club we set up at my son's school, 
and I mentioned that after reading *[Generative Art](http://abandonedart.org/)* several 
years ago I pair-programmed some happy faces with my then-seven-year-old son in 
[Processing](https://github.com/mikebridge/sketchbook):
 
<figure>
 	<img src="/images/kinect/fuzz_boy.png">
 	<figcaption>Lucas & Dad's first foray into computer graphics.</figcaption>
</figure>
 
Heh, memories. :)
 
He replied that his
daughter was involved in dance, and he'd seen that some people use 
Processing for stage productions and import them into live performances via 
[Isadora](http://troikatronix.com/isadora/about/).  A little post-discussion
[Googling](https://www.youtube.com/results?search_query=dance+interactive+projection) 
showed that some dance companies were doing some interesting 
things with motion-detection and computer-generated art.  

I don't have much experience in graphics but I wanted to try this out with Microsoft's Kinect 
and Processing—maybe we can incorporate it into an interesting project for kids later on.  I 
took several wrong turns before I got this working, so here's a description for anyone else 
wants to give it a try.  I'll post later on how to manipulate some Kinect data with this setup.

### Requirements 
- Windows 10
- [Processing 3.2.3 64-bit](https://processing.org/download)
- [Kinect for XBox One V2](http://www.xbox.com/en-US/xbox-one/accessories/kinect) (I have model 1520)
- [Kinect Adapter](http://support.xbox.com/en-CA/xbox-one/accessories/kinect-adapter)
- [Kinect SDK 2.0](https://developer.microsoft.com/en-us/windows/kinect)

There are several versions of the Kinect, and I've only tried the second generation Kinect (aka "V2").  
There was once a *Kinect V2 for Windows* as well as a **Kinect V2 for XBox One**, but the 
Windows version was discontinued in favour of using the XBox One version with an adapter—they 
realized there was no point in supporting two products.  Both will work, but I'm using
the standalone XBox One Kinect.
  
The adapter's description says that it's for the *XBox One S*, but it works fine 
to connect it to a Windows machine via USB-3.  I've read that the adapter works with a 
Mac, but I haven't verified that.

### Installation

- Download the [Kinect SDK 2.0](https://developer.microsoft.com/en-us/windows/kinect) and 
install it.  This has some programming libraries as well as a program called Kinect Studio 
that you can use to try out the Kinect.
- Set up the Kinect and attach it to the computer, then otest that it's working with Kinect 
Studio.
- Download and install the [64-bit version of Processing](https://processing.org/download).  
All you need to do is download the zip file and uncompress it somewhere, e.g. `C:\Program Files\Processing`
- From the `Sketch -> Import Library... -> Add Library` menu item, type "kinect" in the search box and choose "Kinect V2 for Processing".
Click "Install"---I have version 0.7.8.

<figure>
 	<img src="/images/kinect/kinect-libs.png">
</figure>

I had first tried the **Open Kinect for Processing** library, but it seems to have [several problems](https://github.com/shiffman/OpenKinect-for-Processing/issues),
and I couldn't get it working on either the 32 or 64 bit version.  This is too bad because the [video introduction](http://shiffman.net/p5/kinect/) is pretty good.
I didn't try [Kinect4WinSDK](https://github.com/chungbwc/Kinect4WinSDK), because it hasn't been modified in several years.

### Code

Open up Processing and, copy-and-paste the following code in the window, and click the "Run" icon:

<script src="https://gist.github.com/mikebridge/385e085895e5ef490ee53178995dba6d.js"></script>

If everything went well, you should see a depth map image and a point cloud image side-by-side like the following:

<figure>
 	<img src="/images/kinect/first_try.png">
 	<figcaption>Hello World!</figcaption>
</figure>

I'll describe what I did in Processing in a future blog post.  In the meantime, have a look at the 
[KinectPV2 documentation on github](https://github.com/ThomasLengeling/KinectPV2).

Cheers!

-Mike
