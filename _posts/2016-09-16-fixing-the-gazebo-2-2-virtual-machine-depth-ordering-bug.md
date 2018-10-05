---
layout: post
title: Fixing the Gazebo 2.2 Virtual Machine Depth Ordering Bug
---

So, there's [this bug](https://bitbucket.org/osrf/gazebo/issues/1837/vmware-rendering-z-ordering-appears-random) in version 2+ of Gazebo that pops up only in Virtual Machines (very likely a bug with OGRE rather than Gazebo, actually). It causes camera sensors in Gazebo to output an image where some objects that are supposed to be behind others to get rendered in a way that they appear to be in the front. While it makes some rather interesting modern art pieces (see below), it's bad when one is trying to do simulation accelerated robotics R&D. ![images/4290374476-screen_shot_2016-01-19_at_12.53.38_pm_720.png](/images/4290374476-screen_shot_2016-01-19_at_12-53-38_pm_720.png).

There is a solution: _make sure the [FSAA](https://en.wikipedia.org/wiki/Spatial_anti-aliasing#Full-scene_anti-aliasing) value for the OGRE Render Target is set to 0 in [line 1221 of this source file](https://bitbucket.org/osrf/gazebo/src/651b6b1fc78b05ae2825d3771371f6824f83e579/gazebo/rendering/Camera.cc?at=gazebo_2.2&fileviewer=file-view-default#Camera.cc-1221) and recompile._ There is [an easier solution](https://github.com/safijari/gazebo_depth_bugfix) that should work at least on a clean Ubuntu 14.04 install with a full ROS Indigo desktop install. This presents two problems: 

    1. Compiling things from source can be hard.
    2. I don't want to break binary compatibility, especially because I'm using ROS and may need to install Gazebo related packages.
This is a quick post to describe how to go about compiling the right version of Gazebo from source, and then patching the existing Gazebo2 installation so that binary compatibility stays intact! These instructions assume that you are running Ubuntu 14.04 with ROS Indigo installed (full desktop). Also you'll need `mercurial` for this so do a `sudo apt-get install mercurial` as well. Now follow these steps: 

  1. First, install the following dependencies `sudo apt-get install mercurial protobuf-compiler libprotoc-dev libtar-dev`
  2. Make some folder for holding the source code, I like to use `~/Source` mkdir `~/Source` 
  3. Clone the gazebo source code into this directory (will take a while depending on your internet) and then `cd` into the newly created directory. ` cd ~/Source hg clone https://bitbucket.org/osrf/gazebo cd gazebo ` 
  4. The cloned source code represents the latest version of Gazebo, if you're using ROS Indigo then we need to switch to the Gazebo 2.2 branch of the code ` hg pull && hg update gazebo_2.2 ` 
  5. Now, open the file `./gazebo/rendering/Camera.cc` in a text editor and modify line 1221 so that the `4` is changed to `0` and save it. You can also get the modified file [here](https://gist.github.com/safijari/937e7ee4baa9ac96de637ce4bc9134f7).
  6. Okay, now we need to actually compile, do the following (this is standard procedure for compiling with cmake). **Warning:** this will take a _while_. ` mkdir build && cd build cmake .. make ` Note that we are not going to do a `sudo make install`. Also, veterans may ask why I didn't use the `-j` flag, VMware gives me trouble whenever I use that so I omitted it. Your mileage may vary.
  7. Once the compilation finished, we just need to copy the relevant `.so` files to the right location. ` sudo cp ~/Source/gazebo/build/gazebo/rendering/libgazebo_rendering.so* /usr/lib/x86_64-linux-gnu/ ` 
Once this is done, the rendering bug should be fixed and you should still be able to install any robot simulators that depend on gazebo2 from the repos (e.g. `ros-indigo-turtlebot-gazebo`). Below is one example of the turtlebot simulator running on stock Gazebo 2.2. Notice how clearly the objects are being rendered incorrectly. ![Gazebo rendering depth bug](/images/gaz1.png?w=300) After the fix, we see a proper rendering happen. ![Gazebo depth rendering bug fixed](/images/gaz2.png?w=300) Hope this works for you :). If you try to do this and run into an error, please leave a comment.

## Comments

**[Colin](#2 "2017-04-08 17:27:09"):** This worked perfectly, wasn’t expecting to find such a good write-up on this niche issue. Thanks for your help.

**[Luke](#3 "2017-04-21 11:35:04"):** Hi, I believe I have a related problem, when using gazebo on a VM I get error messages or a crash when I use the camera. I am using ROS Kinetic so I followed your tutorial but adjusting for the latest version of Gazebo. I can run gazebo on it's own, however, when I try to run my .world file I get the following error: error while loading shared libraries: libsdformat.so.5: cannot open shared object file: No such file or directory. Do you have any idea how I could fix this? Thanks.

**[jarisafi](#4 "2017-04-21 23:16:31"):** I really couldn't say. You can try searching for that particular shared object file on your computer. If you find something like `libsdformat.so` but not `libsdformat.so.5` you can try creating a link yourself.

